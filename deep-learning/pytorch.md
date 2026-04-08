# PyTorch API — Interview Prep

Training loop, nn.Module, autograd, CNN patterns для собеседования ML/DS/AI Engineer Intern.

---

## Q6. "Напиши training loop в PyTorch"

```python
import torch
import torch.nn as nn
from torch.utils.data import DataLoader

# Модель
model = nn.Sequential(
    nn.Linear(10, 64),
    nn.ReLU(),
    nn.Linear(64, 32),
    nn.ReLU(),
    nn.Linear(32, 1)
)

criterion = nn.MSELoss()
optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)

# Цикл обучения
for epoch in range(100):
    model.train()
    for X_batch, y_batch in train_loader:
        optimizer.zero_grad()          # 1. Обнулить градиенты
        pred = model(X_batch)          # 2. Forward pass
        loss = criterion(pred, y_batch) # 3. Вычислить loss
        loss.backward()                # 4. Backprop (вычислить градиенты)
        optimizer.step()               # 5. Обновить веса

    # Валидация
    model.eval()
    with torch.no_grad():
        val_pred = model(X_val)
        val_loss = criterion(val_pred, y_val)
    print(f"Epoch {epoch}: train_loss={loss:.4f}, val_loss={val_loss:.4f}")
```

> "Запомнить 5 шагов: zero_grad → forward → loss → backward → step. model.train() включает dropout и batch norm в режиме обучения, model.eval() + torch.no_grad() — для валидации."

---

## Q14. "Что такое nn.Module в PyTorch? Напиши свою модель"

```python
import torch.nn as nn

class CreditScorer(nn.Module):
    def __init__(self, n_features):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(n_features, 128),
            nn.BatchNorm1d(128),
            nn.ReLU(),
            nn.Dropout(0.3),
            nn.Linear(128, 64),
            nn.ReLU(),
            nn.Dropout(0.2),
            nn.Linear(64, 1)  # Без sigmoid — используем BCEWithLogitsLoss
        )

    def forward(self, x):
        return self.net(x)

model = CreditScorer(n_features=20)
print(sum(p.numel() for p in model.parameters()))  # Количество параметров
```

> "nn.Module — базовый класс для всех моделей в PyTorch. Наследуешь, определяешь `__init__` (слои) и `forward` (как данные проходят). PyTorch автоматически отслеживает параметры для backward."

---

## Q16. "model.eval() vs torch.no_grad() — в чём разница?"

> "model.eval() и torch.no_grad() делают **разные вещи** и **независимы**.
>
> **model.eval()** меняет поведение слоёв: Dropout перестаёт выключать нейроны, BatchNorm переключается на running statistics. Это про **корректность** предсказаний.
>
> **torch.no_grad()** отключает запись computational graph — PyTorch не хранит промежуточные активации. Это про **экономию VRAM** и скорость.
>
> Для inference нужны **оба**: eval() для правильного поведения, no_grad() для экономии памяти. Но можно eval() без no_grad() — например, при fine-tuning с выключенным Dropout."

| | **model.eval()** | **torch.no_grad()** |
|---|---|---|
| **Что делает** | Меняет **поведение** слоёв | Отключает **вычисление** градиентов |
| **Dropout** | Выключает | Не влияет |
| **BatchNorm** | Использует running mean/std | Не влияет |
| **Gradients** | **Не влияет!** | Отключает |
| **VRAM** | Не экономит | Экономит |
| **Обратное** | `model.train()` | Выход из `with`-блока |

### Матрица 2x2: все комбинации

| eval() | no_grad() | Результат |
|---|---|---|
| Нет | Нет | **Training**: Dropout ON, BN по batch, градиенты считаются |
| **Да** | Нет | Dropout OFF, BN running stats, но градиенты считаются → VRAM зря |
| Нет | **Да** | Dropout **ON** → предсказания **стохастичны**, но VRAM экономится |
| **Да** | **Да** | **Правильный inference**: корректное поведение + экономия VRAM |

```python
model = nn.Sequential(nn.Linear(10, 64), nn.Dropout(0.5), nn.Linear(64, 1))
x = torch.randn(1, 10)

# Сценарий 1: eval() БЕЗ no_grad() — корректно, но жрёт VRAM
model.eval()
output = model(x)       # Dropout OFF — предсказание правильное
output.backward()        # Можно! Используется при fine-tuning

# Сценарий 2: no_grad() БЕЗ eval() — экономим VRAM, но результат неправильный!
model.train()
with torch.no_grad():
    output = model(x)    # Dropout СЛУЧАЙНО выключает нейроны → шум

# Сценарий 3: ПРАВИЛЬНЫЙ inference
model.eval()
with torch.no_grad():
    output = model(x)    # Dropout OFF + нет графа + экономия VRAM
```

---

## Q17. "Autograd: clone vs detach — когда что?"

> "В PyTorch тензоры связаны через computation graph. При копировании важно понимать, что сохраняет связь с графом, а что разрывает. `y = x` — не копирует данные, тот же граф. `y = x.clone()` — копирует данные, но остаётся связан с графом. `y = x.detach().clone()` — полная независимая копия, отвязанная от графа."

| Способ | Данные | Граф | Независимость |
|--------|--------|------|---------------|
| `y = x` | общие | общий | нет |
| `y = x.clone()` | копия | связан | частичная |
| `y = x.detach().clone()` | копия | отвязан | полная |

### Высшие производные через autograd.grad

```python
grad = torch.autograd.grad(outputs=f, inputs=x, create_graph=True)[0]
grad2 = torch.autograd.grad(outputs=grad, inputs=x, create_graph=True)[0]
```

### retain_grad() для промежуточных тензоров

```python
y = x ** 2
y.retain_grad()  # без этого y.grad будет None после backward
loss = y.sum()
loss.backward()
y.grad  # теперь доступен
```

---

## Q18. "CNN в PyTorch: Conv2d и формула выходного размера"

> "Для CNN критично уметь вычислять размеры тензоров после свёрток и пулингов — иначе nn.Linear после Flatten получит неправильный input_size."

$$H_{\text{out}} = \left\lfloor \frac{H - K + 2P}{S} \right\rfloor + 1$$

```python
model = nn.Sequential(
    nn.Conv2d(1, 32, kernel_size=3),  # (1,28,28) → (32,26,26)
    nn.ReLU(),
    nn.MaxPool2d(2),                   # → (32,13,13)
    nn.Conv2d(32, 64, kernel_size=3),  # → (64,11,11)
    nn.ReLU(),
    nn.MaxPool2d(2),                   # → (64,5,5)
    nn.Flatten(),                      # → 1600
    nn.Linear(1600, 128),
    nn.ReLU(),
    nn.Linear(128, 10),
).to(device)
```

---

## Q19. "Инициализация весов — Xavier vs He/Kaiming"

> "Правильная инициализация предотвращает vanishing/exploding gradients на старте. **Xavier** — для sigmoid/tanh. **He/Kaiming** — для ReLU (учитывает, что ReLU обнуляет половину значений)."

```python
def init_weights(m):
    if isinstance(m, nn.Linear):
        nn.init.xavier_uniform_(m.weight)
        nn.init.zeros_(m.bias)
    elif isinstance(m, nn.Conv2d):
        nn.init.kaiming_normal_(m.weight, mode='fan_out', nonlinearity='relu')

model.apply(init_weights)
```

| Инициализация | Активация | Дисперсия |
|---------------|-----------|-----------|
| **Xavier** (Glorot) | Sigmoid, Tanh | $\frac{2}{n_{in} + n_{out}}$ |
| **He** (Kaiming) | ReLU, LeakyReLU | $\frac{2}{n_{in}}$ |

---

## Q20. "Инспекция модели — параметры и граф"

> "На собеседовании могут попросить оценить количество параметров или объяснить структуру модели."

```python
# Количество параметров
total = sum(p.numel() for p in model.parameters())
trainable = sum(p.numel() for p in model.parameters() if p.requires_grad)
print(f"Total: {total:,}, Trainable: {trainable:,}")

# Детальная сводка (pip install torchinfo)
from torchinfo import summary
summary(model, input_size=(1, 1, 28, 28))  # CNN: (batch, C, H, W)

# Визуализация графа (pip install torchviz)
import torchviz
torchviz.make_dot(output, params=dict(model.named_parameters()))
```
