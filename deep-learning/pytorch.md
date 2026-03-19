# PyTorch API — Interview Prep

Training loop и nn.Module для собеседования ML/DS/AI Engineer Intern.

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
