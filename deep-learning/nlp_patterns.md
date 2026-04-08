# NLP & LLM Patterns — Interview Prep

Практические паттерны NLP: от токенизации до LLM inference для собеседования ML/DS/AI Engineer Intern.

---

## Q1. "Предобработка текста — базовый pipeline"

> "Стандартный NLP-pipeline: lowercase → удаление пунктуации → токенизация → удаление стоп-слов → стемминг/лемматизация. Для нейросетей стемминг обычно не нужен — модель сама выучит морфологию через embeddings."

```python
import re
from nltk.corpus import stopwords
import string

stop_words = set(stopwords.words("english"))
punct = set(string.punctuation)

def simple_tokenize(text: str) -> list[str]:
    text = text.lower()
    text = re.sub(r"[^\w\s]", " ", text)
    tokens = text.split()
    return [t for t in tokens if t not in stop_words and t not in punct]
```

---

## Q2. "Vocabulary — маппинг токенов в индексы"

> "Для Embedding-слоя нужен маппинг string ↔ int. Стандартная реализация: `stoi` (string-to-index) и `itos` (index-to-string). Обязательные специальные токены: `<unk>` (неизвестные слова) и `<pad>` (выравнивание длин). Словарь строим только по train, на test могут встретиться `<unk>`."

```python
from collections import Counter, OrderedDict

class Vocabulary:
    def __init__(self, word_freq: dict[str, int]):
        self.itos = []       # index → string
        self.stoi = {}       # string → index
        for word in word_freq:
            self.append_token(word)

    def append_token(self, token: str):
        if token not in self.stoi:
            self.itos.append(token)
            self.stoi[token] = len(self.itos) - 1

    def __getitem__(self, token):
        return self.stoi.get(token, self.stoi["<unk>"])

    def __len__(self):
        return len(self.itos)

    def tokens_to_indices(self, tokens: list[str]) -> list[int]:
        return [self[t] for t in tokens]

# Специальные токены
vocab.append_token("<unk>")
vocab.append_token("<pad>")
UNK_IDX = vocab["<unk>"]
PAD_IDX = vocab["<pad>"]
```

---

## Q3. "DataLoader для текстов переменной длины"

> "Тексты имеют разную длину → нужен padding до максимальной длины в батче. Реализуется через кастомный `collate_fn` в DataLoader. `pad_sequence` из PyTorch автоматически дополняет тензоры до одинаковой длины."

```python
from torch.nn.utils.rnn import pad_sequence
from torch.utils.data import Dataset, DataLoader
import torch

class TextDataset(Dataset):
    def __init__(self, texts: list[list[int]], targets: list):
        self.texts = texts
        self.targets = targets

    def __len__(self):
        return len(self.texts)

    def __getitem__(self, idx):
        return self.texts[idx], self.targets[idx]

def pad_collate(batch):
    texts, labels = zip(*batch)
    texts_tensors = [torch.LongTensor(t) for t in texts]
    labels = torch.LongTensor(labels)  # FloatTensor для регрессии
    texts_tensors = pad_sequence(texts_tensors, padding_value=PAD_IDX, batch_first=True)
    return texts_tensors, labels

loader = DataLoader(dataset, batch_size=32, shuffle=True, collate_fn=pad_collate)
```

**Gotcha:** для регрессии (предсказание числа из текста) — `torch.FloatTensor(labels)`, не `LongTensor`.

---

## Q4. "RNN для языкового моделирования — предсказание следующего токена"

> "Языковая модель предсказывает следующий токен по предыдущим. Архитектура: Embedding → RNN → Linear(hidden → vocab_size). Target сдвинут на 1 позицию вправо. Начальный loss должен быть ≈ ln(vocab_size) — это loss случайного классификатора."

```python
import torch.nn as nn

class CharRNN(nn.Module):
    def __init__(self, num_tokens, emb_size=16, hidden_size=64):
        super().__init__()
        self.emb = nn.Embedding(num_tokens, emb_size)
        self.rnn = nn.RNN(emb_size, hidden_size, batch_first=True)
        self.hid_to_logits = nn.Linear(hidden_size, num_tokens)

    def forward(self, x):
        h_seq, _ = self.rnn(self.emb(x))        # (batch, seq_len, hidden)
        return self.hid_to_logits(h_seq)          # (batch, seq_len, num_tokens)

# Обучение: target сдвинут на 1
logits = model(batch[:, :-1])                     # предсказания для позиций 0..T-2
target = batch[:, 1:]                              # истинные токены 1..T-1
loss = nn.CrossEntropyLoss()(logits.reshape(-1, num_tokens), target.reshape(-1))
```

---

## Q5. "LSTM для классификации текста (sentiment analysis)"

> "Для классификации берём последнее скрытое состояние LSTM (summary всей последовательности) и подаём в линейный слой. `padding_idx` в Embedding — чтобы PAD-токены не влияли на градиенты. Для бинарной классификации — один выход + BCEWithLogitsLoss."

```python
class LSTMClassifier(nn.Module):
    def __init__(self, num_tokens, emb_size=128, hidden_size=256, num_classes=1):
        super().__init__()
        self.emb = nn.Embedding(num_tokens, emb_size, padding_idx=PAD_IDX)
        self.rnn = nn.LSTM(emb_size, hidden_size, batch_first=True)
        self.classifier = nn.Linear(hidden_size, num_classes)

    def forward(self, x):
        emb = self.emb(x)                           # (batch, seq, emb)
        _, (h_state, _) = self.rnn(emb)              # h_state: (1, batch, hidden)
        logits = self.classifier(h_state.squeeze(0)) # (batch, num_classes)
        return logits

# Loss: BCEWithLogitsLoss для бинарной (объединяет sigmoid + BCE)
criterion = nn.BCEWithLogitsLoss()
loss = criterion(logits.squeeze(), labels.float())

# Предсказание:
preds = torch.round(torch.sigmoid(logits))
```

---

## Q6. "Генерация текста — temperature sampling"

> "Temperature контролирует 'креативность' генерации. Делим логиты на T перед softmax: T < 1 → более детерминированно (greedy-like), T = 1 → стандартный softmax, T > 1 → более разнообразно. На практике T ∈ [0.7, 1.2] для большинства задач."

```python
def generate(model, seed, max_length, temperature=1.0):
    x = torch.tensor([token_to_id[c] for c in seed]).unsqueeze(0)
    for _ in range(max_length - len(seed)):
        logits = model(x)
        p_next = torch.softmax(logits[0, -1] / temperature, dim=-1)
        next_ix = torch.multinomial(p_next, 1)
        x = torch.cat([x, next_ix.unsqueeze(0)], dim=1)
    return "".join(id_to_token[i] for i in x[0].tolist())
```

| T | Эффект |
|---|--------|
| < 1.0 | Консервативно, низкая энтропия |
| = 1.0 | Стандартный softmax |
| > 1.0 | Разнообразно, высокая энтропия |

**Оптимизация:** передавать hidden state между шагами генерации вместо пересчёта всей последовательности:

```python
class SmartRNN(nn.Module):
    def forward(self, x, h0=None):
        emb = self.emb(x)
        h_seq, h_last = self.rnn(emb, h0)
        return self.hid_to_logits(h_seq), h_last

# Генерация по 1 токену + передача hidden state
h = None
for step in range(max_length):
    logits, h = model(next_token.unsqueeze(0), h)
    # sample next_token from logits
```

---

## Q7. "HuggingFace transformers — загрузка и inference"

> "HuggingFace — стандартный способ работы с предобученными LLM. `AutoModelForCausalLM` для генеративных моделей, `AutoTokenizer` для токенизации. `model.generate()` поддерживает разные стратегии декодирования: greedy, sampling, top-k, nucleus (top-p)."

```python
from transformers import AutoModelForCausalLM, AutoTokenizer

model_name = "Qwen/Qwen2.5-3B-Instruct"
model = AutoModelForCausalLM.from_pretrained(model_name, torch_dtype="auto", device_map="auto")
tokenizer = AutoTokenizer.from_pretrained(model_name)

# Генерация
tokenized = tokenizer(text, return_tensors="pt").to(model.device)
generated_ids = model.generate(
    **tokenized,
    max_new_tokens=100,
    do_sample=True,         # False = greedy (argmax)
    temperature=1.05,
    top_k=50,               # оставить только top-k токенов
    top_p=0.95,             # nucleus sampling
    pad_token_id=tokenizer.eos_token_id,
)
result = tokenizer.decode(generated_ids[0], skip_special_tokens=True)
```

| Параметр | Эффект |
|----------|--------|
| `do_sample=False` | Greedy decoding (argmax) |
| `temperature` | Мягкость распределения (см. Q6) |
| `top_k` | Оставить только top-k токенов перед sampling |
| `top_p` | Nucleus: оставить токены с суммарной вероятностью ≤ p |

---

## Q8. "Chat templates для instruction-tuned моделей"

> "Instruct-модели (Qwen-Instruct, Llama-Instruct) ожидают определённый формат сообщений. `apply_chat_template` автоматически оборачивает messages в нужные спецтокены. Base-модели (без Instruct) плохо следуют инструкциям — они обучены только предсказывать следующий токен, а не отвечать на вопросы."

```python
messages = [
    {"role": "system", "content": "Ты помощник для анализа текстов."},
    {"role": "user", "content": "Определи тональность: ..."},
]

tokenized = tokenizer.apply_chat_template(
    messages,
    tokenize=True,
    add_generation_prompt=True,   # добавить начало ответа assistant
    return_tensors="pt",
    return_dict=True,
)

generated_ids = model.generate(**tokenized.to(model.device), max_new_tokens=50)
result = tokenizer.decode(generated_ids[0], skip_special_tokens=False)
```

---

## Q9. "vLLM — быстрый LLM inference и guided decoding"

> "vLLM — фреймворк для высокопроизводительного LLM inference. Ключевые преимущества: PagedAttention (эффективное управление KV-cache), continuous batching, guided decoding (ограничение формата выхода). Идеален для batch-inference и продакшн-деплоя."

```python
from vllm import LLM, SamplingParams
from vllm.sampling_params import GuidedDecodingParams

model = LLM(model="Qwen/Qwen2.5-3B-Instruct", dtype="float16")

# Guided decoding: ограничить выход списком вариантов
guided = GuidedDecodingParams(choice=["positive", "negative"])
params = SamplingParams(guided_decoding=guided, temperature=0.0)
outputs = model.chat(messages=[messages], sampling_params=params)
result = outputs[0].outputs[0].text  # "positive" или "negative"
```

### Structured output через JSON schema (Pydantic)

```python
from pydantic import BaseModel
from enum import Enum

class SentimentType(str, Enum):
    positive = "positive"
    negative = "negative"

class ReviewAnalysis(BaseModel):
    sentiment: SentimentType

guided = GuidedDecodingParams(json=ReviewAnalysis.model_json_schema())
params = SamplingParams(guided_decoding=guided, temperature=0.0)
outputs = model.chat(messages=[messages], sampling_params=params)
```

### Batch inference

```python
conversations = [
    [{"role": "user", "content": format_prompt(review)}]
    for review in batch_reviews
]
outputs = model.chat(conversations, sampling_params=params)
results = [o.outputs[0].text for o in outputs]
```

---

## Q10. "Zero-shot vs Few-shot prompting"

> "**Zero-shot** — только инструкция, без примеров. Работает для простых задач. **Few-shot** (k=3-5) — добавляем примеры в промпт. Повышает качество, особенно для нестандартных форматов. Важно балансировать примеры по классам."

```python
# Zero-shot
prompt = "Определи тональность текста. Ответь ОДНИМ словом: positive или negative.\n\nТекст: {text}"

# Few-shot (k=5)
examples = random.sample(train_data, k=5)  # сбалансировать классы!
shots = "\n".join(f"Текст: {ex['text']}\nОтвет: {ex['label']}" for ex in examples)
prompt = f"Определи тональность по примерам:\n{shots}\n\nТекст: {text}\nОтвет:"
```

**Gotcha:** примеры лучше балансировать (≈ поровну по классам), иначе модель будет предвзята к мажоритарному.

---

## Q11. "RNN vs LSTM vs GRU — ключевые отличия в PyTorch"

> "Все три — рекуррентные архитектуры, но с разной механикой. RNN — простейшая, страдает от vanishing gradients. LSTM — gates + cell state, запоминает длинные зависимости. GRU — упрощённый LSTM (2 gates вместо 3), быстрее при сравнимом качестве."

```python
# RNN: возвращает (output, h_n)
rnn = nn.RNN(input_size=emb_size, hidden_size=64, batch_first=True)
output, h_n = rnn(emb(x))  # output: (batch, seq, hidden), h_n: (1, batch, hidden)

# LSTM: возвращает (output, (h_n, c_n))
lstm = nn.LSTM(input_size=emb_size, hidden_size=64, batch_first=True)
output, (h_n, c_n) = lstm(emb(x))

# RNNCell: один шаг (ручной цикл по времени)
cell = nn.RNNCell(input_size=emb_size, hidden_size=64)
h = torch.zeros(batch_size, 64)
for t in range(seq_len):
    h = cell(emb_seq[:, t, :], h)
```

**Gotcha:** `batch_first=True` — размерность `(batch, seq, feature)`. По умолчанию PyTorch использует `(seq, batch, feature)`.

| Архитектура | Gates | Возвращает | Vanishing gradients | Скорость |
|-------------|-------|------------|---------------------|----------|
| RNN | 0 | `(output, h_n)` | Сильно страдает | Быстрая |
| LSTM | 3 (forget, input, output) | `(output, (h_n, c_n))` | Устойчив (cell state) | Медленная |
| GRU | 2 (reset, update) | `(output, h_n)` | Устойчив | Средняя |

---

## Приоритет для intern-level

| Тема | Приоритет | Почему |
|------|-----------|--------|
| HuggingFace inference (Q7, Q8) | **Высокий** | Базовый навык для LLM-позиций |
| Temperature / decoding strategies (Q6) | **Высокий** | Спрашивают на каждом LLM-собесе |
| LSTM vs RNN vs GRU (Q11) | **Высокий** | Классический вопрос по архитектурам |
| vLLM + structured output (Q9) | Средний | Для продвинутых GenAI-позиций |
| Vocabulary + DataLoader для текстов (Q2, Q3) | Средний | Показывает hands-on NLP опыт |
| Zero/Few-shot prompting (Q10) | Средний | Практический навык |
| Preprocessing (Q1) | Низкий | Базовый, редко спрашивают отдельно |
| RNN для языкового моделирования (Q4) | Низкий | Устарел, но полезен для понимания |
