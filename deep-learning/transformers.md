# Deep Learning — Transformers & Advanced Topics

Трансформеры, attention, LoRA, ViT для собеседования ML/DS/AI Engineer Intern.

---

## Q1. "Self-Attention — математика и интуиция"

> "Self-Attention позволяет каждому токену 'смотреть' на все остальные токены в последовательности и решать, на кого обращать внимание.
>
> Механизм: входная матрица $X$ проецируется в три матрицы: Query $Q = XW_Q$, Key $K = XW_K$, Value $V = XW_V$.
>
> $$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right) V$$
>
> 1. $QK^T$ — матрица 'похожести' между всеми парами токенов (dot product).
> 2. Деление на $\sqrt{d_k}$ — стабилизация softmax. Без этого при большом $d_k$ значения $QK^T$ становятся слишком большими → softmax выдаёт почти one-hot → vanishing gradients.
> 3. Softmax — нормализация в веса (сумма = 1).
> 4. Умножение на $V$ — взвешенная сумма value-векторов.
>
> Интуиция: Query = 'что я ищу', Key = 'что я предлагаю', Value = 'что я содержу'. Каждый токен формирует запрос и сопоставляет его со всеми ключами."

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right) V$$

| Компонент | Размерность | Роль |
|-----------|------------|------|
| $Q$ (Query) | $[n \times d_k]$ | "Что ищу" |
| $K$ (Key) | $[n \times d_k]$ | "Что предлагаю" |
| $V$ (Value) | $[n \times d_v]$ | "Что содержу" |
| $QK^T$ | $[n \times n]$ | Матрица внимания |
| $\sqrt{d_k}$ | скаляр | Стабилизация градиентов |

---

## Q2. "Positional Encoding и Multi-Head Attention"

> "**Positional Encoding**: Self-Attention инвариантен к порядку токенов (permutation equivariant) — 'кот ел рыбу' и 'рыбу ел кот' дают одинаковые attention scores. Нужно явно закодировать позицию.
>
> Синусоидальные PE (оригинальный Transformer):
> $$PE_{(pos, 2i)} = \sin\left(\frac{pos}{10000^{2i/d}}\right), \quad PE_{(pos, 2i+1)} = \cos\left(\frac{pos}{10000^{2i/d}}\right)$$
>
> Обучаемые PE: каждая позиция — обучаемый вектор (BERT, GPT). Плюс: модель сама решает, как кодировать позицию. Минус: фиксированная максимальная длина.
>
> RoPE (Rotary PE, LLaMA): кодирует позицию через вращение в комплексном пространстве. Плюс: лучше экстраполяция на длинные последовательности.
>
> **Multi-Head Attention**: вместо одного attention — $h$ параллельных 'голов', каждая со своими $W_Q, W_K, W_V$. Разные головы фокусируются на разных аспектах: синтаксис, семантика, позиционные паттерны. Результаты конкатенируются и проецируются:
>
> $$\text{MultiHead}(Q,K,V) = \text{Concat}(\text{head}_1, ..., \text{head}_h) W_O$$"

| Тип PE | Примеры | Плюсы | Минусы |
|--------|---------|-------|--------|
| Синусоидальные | Transformer (оригинал) | Экстраполяция, без параметров | Менее гибкие |
| Обучаемые | BERT, GPT-2 | Гибкие, адаптивные | Фикс. max length |
| RoPE | LLaMA, GPT-NeoX | Хорошая экстраполяция | Сложнее реализация |

---

## Q3. "PyTorch: autograd, DataLoader и оптимизация памяти"

> "**Dynamic Computation Graph**: PyTorch строит граф вычислений на лету при forward pass (define-by-run). Каждая операция записывается → при `loss.backward()` градиенты вычисляются автоматически через backpropagation.
>
> **`torch.no_grad()`** — отключает запись в граф. Зачем: при валидации/inference не нужны градиенты → экономим VRAM. Обязательно использовать при `model.eval()`.
>
> **`loss.item()`** vs `loss` — частая ловушка: если сохраняем `loss` в список для логирования, граф вычислений не освобождается → утечка памяти. `loss.item()` возвращает Python float, отвязанный от графа.
>
> **DataLoader** — ключевые параметры:
> - `num_workers`: параллельная загрузка данных (CPU) пока GPU обучается. Обычно 4-8.
> - `pin_memory=True`: данные копируются в pinned (page-locked) memory → быстрый transfer на GPU.
> - `prefetch_factor`: сколько батчей загружать заранее (default=2).
> - `persistent_workers=True`: не пересоздавать воркеры каждую эпоху."

```python
import torch
from torch.utils.data import DataLoader, Dataset

# === Ловушка: утечка памяти ===
losses = []
for batch in loader:
    loss = model(batch)
    loss.backward()
    # losses.append(loss)       # ПЛОХО: хранит граф вычислений!
    losses.append(loss.item())   # ХОРОШО: Python float

# === torch.no_grad() при валидации ===
model.eval()
with torch.no_grad():  # экономит VRAM, ускоряет inference
    for batch in val_loader:
        preds = model(batch)

# === DataLoader: оптимальные параметры ===
loader = DataLoader(
    dataset,
    batch_size=64,
    shuffle=True,
    num_workers=4,           # параллельная загрузка
    pin_memory=True,         # быстрый transfer на GPU
    prefetch_factor=2,       # предзагрузка батчей
    persistent_workers=True, # не пересоздавать воркеры
)
```

| Параметр | Что делает | Рекомендация |
|----------|-----------|--------------|
| `num_workers` | Параллельная загрузка данных | 4-8 (зависит от CPU) |
| `pin_memory` | Pinned memory для быстрого GPU transfer | `True` при обучении на GPU |
| `prefetch_factor` | Батчей в очереди на воркер | 2 (default) |
| `persistent_workers` | Не убивать воркеры между эпохами | `True` при больших данных |

---

## Q4. "LoRA (Low-Rank Adaptation)"

> "**LoRA** — метод parameter-efficient fine-tuning: вместо обучения всех параметров модели, замораживаем основную матрицу $W$ и добавляем низкоранговое обновление $\Delta W = AB$:
>
> $$W' = W + \Delta W = W + AB$$
>
> где $A \in \mathbb{R}^{d \times r}$, $B \in \mathbb{R}^{r \times d}$, $r \ll d$ (rank).
>
> Экономия: вместо $d^2$ параметров обучаем $2dr$ параметров. При $d = 4096$, $r = 8$: $16.8M \to 65K$ — в 256 раз меньше.
>
> Rank $r$ — ключевой гиперпараметр: маленький $r$ (4-8) — для простой адаптации (стиль), большой $r$ (32-64) — для сложных задач (новый домен).
>
> Когда использовать: дообучение LLM на ограниченных ресурсах (стажёрский GPU с 8-16 GB VRAM). Базовая модель 7B параметров → LoRA с $r=8$ добавляет ~0.1% обучаемых параметров.
>
> QLoRA: дополнительно квантизуем базовую модель до 4-bit → ещё меньше VRAM. Позволяет дообучить 7B модель на одной GPU с 16 GB."

$$\Delta W = AB, \quad A \in \mathbb{R}^{d \times r}, \; B \in \mathbb{R}^{r \times d}, \; r \ll d$$

| Метод | Обучаемые параметры | VRAM | Когда |
|-------|-------------------|------|-------|
| Full Fine-Tuning | 100% (миллиарды) | Очень много (8xGPU) | Неограниченные ресурсы |
| **LoRA** | ~0.1-1% | Умеренно (1xGPU) | Адаптация к задаче/домену |
| **QLoRA** | ~0.1% + 4-bit base | Мало (1x16GB GPU) | Ограниченные ресурсы |
| Prompt Tuning | ~0.01% | Минимально | Простая адаптация |

```python
from peft import LoraConfig, get_peft_model

lora_config = LoraConfig(
    r=8,                      # rank
    lora_alpha=16,            # scaling factor (alpha/r)
    target_modules=["q_proj", "v_proj"],  # какие слои адаптируем
    lora_dropout=0.05,
    task_type="CAUSAL_LM",
)

model = get_peft_model(base_model, lora_config)
model.print_trainable_parameters()
# trainable params: 65,536 || all params: 6,738,415,616 || trainable%: 0.001
```

---

## Q5. "Vision Transformers (ViT) и стратегии дообучения"

> "**ViT** применяет Transformer к изображениям:
>
> 1. Изображение $H \times W \times C$ разбивается на **патчи** размера $P \times P$ → $N = HW/P^2$ патчей.
> 2. Каждый патч flatten → линейная проекция в $d$-мерный вектор (patch embedding).
> 3. Добавляется `[CLS]` токен + positional embeddings.
> 4. Всё подаётся в стандартный Transformer Encoder.
> 5. `[CLS]` токен → classification head.
>
> **Feature Extraction** vs **Fine-Tuning**:
> - Feature Extraction: замораживаем все слои, обучаем только classification head. Быстро, мало данных достаточно.
> - Fine-Tuning: размораживаем часть/все слои, обучаем с маленьким LR ($10^{-5}$). Медленнее, но точнее.
>
> Когда ViT лучше CNN: большие датасеты (ViT не имеет inductive bias свёрток — locality, translation equivariance — поэтому нужно больше данных для обучения). Когда CNN лучше: маленькие датасеты (inductive bias свёрток даёт преимущество при малом количестве данных)."

| Аспект | CNN | ViT |
|--------|-----|-----|
| Inductive bias | Locality, translation equivariance | Нет (нужно больше данных) |
| Маленький датасет | **Лучше** (bias помогает) | Хуже (без pretrain) |
| Большой датасет | Хуже (bias ограничивает) | **Лучше** (больше гибкости) |
| Pretrained + fine-tune | Хорошо | **Отлично** |
| Интерпретируемость | Feature maps | Attention maps |

```python
import torch
import torchvision.models as models

# === Feature Extraction: замораживаем backbone ===
model = models.vit_b_16(weights='IMAGENET1K_V1')
for param in model.parameters():
    param.requires_grad = False  # замораживаем всё

# Заменяем classification head
model.heads.head = torch.nn.Linear(model.heads.head.in_features, num_classes)
# Обучаем только head — быстро, мало данных

# === Fine-Tuning: маленький LR для backbone ===
model = models.vit_b_16(weights='IMAGENET1K_V1')
model.heads.head = torch.nn.Linear(model.heads.head.in_features, num_classes)

optimizer = torch.optim.AdamW([
    {'params': model.encoder.parameters(), 'lr': 1e-5},  # backbone: маленький LR
    {'params': model.heads.parameters(), 'lr': 1e-3},    # head: обычный LR
])
```

| Стратегия | Что обучаем | LR | Данные | Время |
|-----------|-------------|-----|--------|-------|
| Feature Extraction | Только head | $10^{-3}$ | Мало (100+) | Быстро |
| Fine-Tuning (partial) | Head + последние слои | $10^{-5}$ / $10^{-3}$ | Средне (1K+) | Средне |
| Full Fine-Tuning | Вся модель | $10^{-5}$ | Много (10K+) | Долго |

---

## Приоритет для intern-level

| Тема | Приоритет | Почему |
|------|-----------|--------|
| Self-Attention (Q1) | **Высокий** | Основа всех LLM — спрашивают везде |
| PyTorch практика (Q3) | **Высокий** | Показывает hands-on опыт |
| Positional Encoding, Multi-Head (Q2) | Средний | Углубление после Q1 |
| LoRA (Q4) | Средний | Актуально для LLM/GenAI позиций |
| ViT (Q5) | Низкий | Для CV-специфичных позиций |
