# Обучение нейросетей — Interview Prep

Оптимизаторы, градиенты, регуляризация, batch norm, LR scheduling для собеседования ML/DS/AI Engineer Intern.

---

## Q5. "Расскажи про оптимизаторы: SGD vs Adam"

> "**SGD** — базовый: обновляет веса пропорционально градиенту. Нужно подбирать learning rate, медленно сходится.
>
> **SGD + Momentum** — добавляет 'инерцию': $v = \beta v + \frac{\partial L}{\partial w}$, $w = w - \text{lr} \cdot v$. Быстрее проходит плоские участки.
>
> **Adam** — адаптивный: хранит moving average градиентов (momentum) И квадратов градиентов (адаптивный lr). Каждый параметр получает свой learning rate. На практике — default выбор, сходится быстрее. AdamW — вариант с decoupled weight decay, стандарт для transformers."

---

## Q8. "Что такое vanishing/exploding gradients? Как бороться?"

> "**Vanishing gradients** — при backprop через много слоёв градиенты уменьшаются экспоненциально (особенно с sigmoid/tanh), нижние слои почти не обучаются. **Exploding gradients** — наоборот, градиенты растут, веса 'взрываются'.
>
> Решения:
> - **ReLU** вместо sigmoid (градиент = 0 или 1, не затухает)
> - **Skip connections** (ResNet): градиент обходит слои напрямую
> - **Batch Normalization**: нормализует активации, стабилизирует обучение
> - **Gradient clipping**: обрезаем градиенты по норме (для exploding)
> - **Правильная инициализация**: Xavier (для sigmoid/tanh), He/Kaiming (для ReLU)"

---

## Q9. "Regularization в нейросетях"

| Метод | Как работает | Когда |
|-------|-------------|-------|
| **Dropout** | Inverted dropout: обнуляем $p$% нейронов, остальные $\times \frac{1}{1-p}$ | Default, $p = 0.1 \text{–} 0.5$ |
| **Weight Decay** (L2) | Штраф $\lambda \lVert w \rVert^2$ к loss | AdamW (decoupled) |
| **Batch Norm** | Нормализация активаций между слоями | Ускоряет обучение + регуляризирует |
| **Data Augmentation** | Рандомные трансформации данных (flip, crop, rotate) | Для изображений |
| **Early Stopping** | Остановить обучение когда val_loss растёт | Всегда используем |
| **Label Smoothing** | Мягкие метки (0.9 вместо 1.0) | Для классификации |

> "В CatBoost я использовал l2_leaf_reg и early stopping — аналогичные принципы. В нейросетях добавляется Dropout и Batch Norm."

---

## Q10. "Что такое Batch Normalization?"

> "Batch Norm нормализует выход каждого слоя: вычитает среднее и делит на std по мини-батчу, потом применяет learnable параметры $\gamma$ (scale) и $\beta$ (shift). Зачем: стабилизирует обучение, позволяет использовать бо́льший learning rate, ускоряет сходимость, имеет регуляризующий эффект. При inference использует running mean/std, накопленные за обучение."

$$\hat{x} = \frac{x - \mu_{\text{batch}}}{\sqrt{\sigma^2_{\text{batch}} + \varepsilon}}$$

$$y = \gamma \cdot \hat{x} + \beta$$

---

## Q11. "Dropout — как работает и почему?"

> "Dropout при обучении случайно обнуляет нейроны с вероятностью p и масштабирует оставшиеся на $\frac{1}{1-p}$ — это **inverted dropout**, именно его реализует PyTorch. Работает по двум причинам: каждый forward pass — случайная подсеть, полная сеть при inference аппроксимирует ансамбль. Второе — предотвращает ко-адаптацию нейронов. При inference dropout выключается через `model.eval()`."

```python
def manual_dropout(x, p=0.3, training=True):
    if not training:
        return x  # Inference: все нейроны, без масштабирования
    mask = (torch.rand_like(x) > p).float()
    return x * mask / (1 - p)  # Масштабируем чтобы E[output] = x
```

| Режим | Dropout | Масштабирование |
|---|---|---|
| `model.train()` | Нейроны обнуляются с вероятностью p | Выход $\times \frac{1}{1-p}$ |
| `model.eval()` | **Все** нейроны активны | Без масштабирования |

Почему работает:
1. **Ансамбль подсетей** — при N нейронах экспоненциальное количество подсетей ($2^N$). Полная сеть при inference — аппроксимация ансамблевого среднего.
2. **Ко-адаптация** — без Dropout нейроны "полагаются" друг на друга. Dropout заставляет каждый быть полезным самостоятельно.

---

## Q12. "LayerNorm — зачем и чем отличается от BatchNorm?"

> "LayerNorm нормализует вектор каждого токена **независимо** — считает mean и std по feature dimension. В отличие от BatchNorm не зависит от других примеров в batch'е. Это критично для Transformers: batch-independent (batch_size=1), variable-length sequences, стабильно при autoregressive generation. В современных Transformers используется Pre-LN — LayerNorm стоит **до** attention и **до** FFN."

$$\text{LayerNorm}(x) = \gamma \cdot \frac{x - \mu}{\sqrt{\sigma^2 + \varepsilon}} + \beta$$

где $\mu$, $\sigma^2$ считаются по **feature dimension** (не по batch).

### 3 причины замены BatchNorm в Transformers

| # | Причина | Подробности |
|---|---------|-------------|
| 1 | **Batch-independent** | BN при batch_size=1 — шумные статистики. LN работает без проблем |
| 2 | **Variable-length sequences** | BN по позициям бессмысленен (позиция 5 в разных предложениях — разная семантика) |
| 3 | **Autoregressive generation** | Токены по одному, batch_size=1. BN не имеет смысла |

### Pre-LN vs Post-LN

```
Post-LN (оригинальный Transformer, 2017):
  x → Attention → Add(x + ...) → LayerNorm → FFN → Add → LayerNorm
  Проблема: нестабильные градиенты, нужен warmup

Pre-LN (GPT-2+, современный стандарт):
  x → LayerNorm → Attention → Add(x + ...) → LayerNorm → FFN → Add
  Плюс: стабильнее, не нужен warmup
```

### BatchNorm vs LayerNorm

| | **BatchNorm** | **LayerNorm** |
|---|---|---|
| Нормализует по | Batch dimension | Feature dimension |
| Зависит от батча | Да | Нет |
| Running stats | Да (нужны при eval) | Нет |
| Поведение при eval() | Переключается на running stats | Не меняется |
| Где используют | CNN, MLP (CV) | Transformers (NLP, LLM) |

Источник: `interviews/companies/x5tech/X5TECH_TECHINTERVIEW_FEEDBACKRESULTS_RELEARNING.md`

---

## Q15. "Learning Rate Scheduling — зачем?"

> "Постоянный lr — не оптимально. В начале нужен большой шаг (быстро приближаемся к минимуму), в конце — маленький (не перепрыгнуть). Популярные schedulers:
> - **StepLR** — уменьшаем lr в N раз каждые K эпох
> - **CosineAnnealing** — lr следует косинусу (плавное уменьшение)
> - **OneCycleLR** — warmup → peak → decay (лучший на практике)
> - **ReduceLROnPlateau** — уменьшаем если val_loss перестал падать
>
> Warmup критичен для transformers — без него Adam может сделать слишком большие обновления на первых шагах."

```python
scheduler = torch.optim.lr_scheduler.CosineAnnealingLR(optimizer, T_max=100)
for epoch in range(100):
    train(...)
    scheduler.step()
```
