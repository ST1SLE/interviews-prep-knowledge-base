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
| **Dropout** | Случайно выключаем $p$% нейронов при обучении | Default, обычно $p = 0.1 \text{–} 0.5$ |
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
