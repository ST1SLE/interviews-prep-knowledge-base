# Архитектуры нейросетей — Interview Prep

CNN, RNN/LSTM, transfer learning, выбор архитектуры для собеседования ML/DS/AI Engineer Intern.

---

## Q7. "Что такое CNN? Зачем свёртки?"

> "CNN (Convolutional Neural Network) — архитектура для изображений. Ключевая идея: свёрточный слой применяет маленькие фильтры (3×3, 5×5) ко всему изображению, выделяя локальные паттерны (края, текстуры, формы). Преимущества над полносвязным слоем:
> 1. **Parameter sharing** — один фильтр для всего изображения (vs отдельный вес для каждого пикселя)
> 2. **Translation invariance** — кот в углу и кот в центре → один и тот же фильтр распознает
> 3. **Иерархия признаков** — первые слои: edges → средние: textures → глубокие: objects
>
> Архитектура: Conv → ReLU → Pooling → ... → Flatten → FC → Output."

**Основные архитектуры:**
- LeNet (1998) — простая, 2 conv слоя
- ResNet (2015) — skip connections, решают vanishing gradients для глубоких сетей
- EfficientNet (2019) — compound scaling (width × depth × resolution)

### Практический паттерн: извлечение промежуточных активаций CNN

```python
# Прогон через слои nn.Sequential, сбор карт активаций до Flatten
x = img.unsqueeze(0).to(device)
for layer in model:
    if isinstance(layer, nn.Flatten):
        break
    x = layer(x)
    # x.shape → (1, C, H, W) — карты активаций после каждого слоя
```

---

## Q11. "RNN/LSTM — зачем нужны, почему устарели?"

> "**RNN** — для последовательных данных (текст, временные ряды). Обрабатывает токены по одному, передавая скрытое состояние $h_t = f(h_{t-1}, x_t)$. Проблема: vanishing gradients на длинных последовательностях — не может 'запомнить' начало текста.
>
> **LSTM** — решает это через gates: forget gate (что забыть), input gate (что запомнить), output gate (что вернуть). Отдельная cell state позволяет информации 'течь' без деградации.
>
> **Почему устарели:** Transformer (2017) обрабатывает всю последовательность параллельно через attention, лучше улавливает дальние зависимости, и масштабируется на GPU. Все современные LLM — transformers."

### Практический паттерн: Embedding + LSTM в PyTorch

```python
# Embedding с обнулением PAD-токена
emb = nn.Embedding(num_tokens, emb_size, padding_idx=PAD_IDX)

# LSTM — возвращает (output, (h_n, c_n))
lstm = nn.LSTM(input_size=emb_size, hidden_size=64, batch_first=True)
output, (h_n, c_n) = lstm(emb(x))
# output: (batch, seq, hidden) — все скрытые состояния
# h_n: (1, batch, hidden) — последнее скрытое состояние (для классификации)

# pad_sequence — паддинг текстов переменной длины
from torch.nn.utils.rnn import pad_sequence
padded = pad_sequence(tensors, padding_value=PAD_IDX, batch_first=True)
```

**Gotcha:** `batch_first=True` — размерность `(batch, seq, feature)`. По умолчанию PyTorch: `(seq, batch, feature)`.

---

## Q12. "Что такое transfer learning и fine-tuning?"

> "**Transfer learning** — использование модели, обученной на одной задаче, для другой. Нижние слои учат универсальные паттерны (edges в CV, грамматика в NLP), их можно переиспользовать.
>
> **Подходы:**
> 1. **Feature extraction** — замораживаем предобученную модель, обучаем только новый 'head' (последние слои)
> 2. **Full fine-tuning** — обучаем все слои с маленьким lr
> 3. **Parameter-efficient** (LoRA, QLoRA) — обучаем маленькие адаптерные матрицы, не трогая основные веса
>
> Пример: BERT предобучен на миллиардах слов → fine-tune на 1000 примерах для классификации отзывов. Я использовал готовый Sentence-BERT в Random Coffee без fine-tuning — feature extraction подход."

---

## Q13. "Как выбрать архитектуру нейросети?"

| Задача | Архитектура | Почему |
|--------|-------------|--------|
| Изображения | CNN (ResNet, EfficientNet) | Locality + translation invariance |
| Текст (2026) | Transformer (BERT, GPT) | Attention, параллелизм |
| Последовательности (короткие) | LSTM / GRU | Если ресурсы ограничены |
| Табличные данные | **Не нейросети!** CatBoost/XGBoost | Бустинг до сих пор лучше на таблицах |
| Генерация текста | Decoder-only Transformer (GPT) | Авторегрессия |
| Понимание текста | Encoder Transformer (BERT) | Bidirectional context |
| Документы/OCR | Vision Transformer (ViT) или CNN+Transformer | Мультимодальность |

> "Важно: для табличных данных нейросети почти всегда проигрывают gradient boosting. Я это видел в Альфа-Банке — CatBoost давал лучший ROC-AUC, чем MLP."
