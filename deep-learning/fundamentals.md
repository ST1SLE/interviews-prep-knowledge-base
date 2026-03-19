# Основы нейросетей — Interview Prep

Нейроны, backprop, активации, loss functions для собеседования ML/DS/AI Engineer Intern.

---

## Q1. "Что такое нейронная сеть? Как она обучается?"

> "Нейронная сеть — это функция-аппроксиматор, состоящая из слоёв нейронов. Каждый нейрон: $y = \text{activation}(W \cdot x + b)$. Обучение — итеративный процесс: forward pass (получаем предсказание) → loss function (считаем ошибку) → backpropagation (вычисляем градиенты через chain rule) → optimizer (обновляем веса в сторону уменьшения ошибки). Повторяем по мини-батчам до сходимости."

**Формула обновления весов:**

$$w_{\text{new}} = w_{\text{old}} - \text{lr} \cdot \frac{\partial L}{\partial w}$$

---

## Q2. "Объясни backpropagation"

> "Backprop — алгоритм вычисления градиентов loss по всем параметрам сети через chain rule. Идём от выхода к входу: градиент loss по последнему слою → умножаем на локальный градиент каждого слоя → получаем градиенты для каждого веса. Это именно то, что делает `loss.backward()` в PyTorch — autograd строит computation graph при forward pass и проходит по нему в обратном направлении."

**Chain rule пример:**

$$z = W \cdot x + b$$

$$a = \text{ReLU}(z)$$

$$L = \text{loss}(a, y)$$

$$\frac{\partial L}{\partial W} = \frac{\partial L}{\partial a} \cdot \frac{\partial a}{\partial z} \cdot \frac{\partial z}{\partial W}$$

---

## Q3. "Какие функции активации знаешь? Когда какую?"

| Активация | Формула | Когда использовать | Проблемы |
|-----------|---------|-------------------|----------|
| **ReLU** | $\max(0, x)$ | Default для скрытых слоёв | Dying ReLU (нейрон "умирает" при $x < 0$) |
| **LeakyReLU** | $\max(0.01x, x)$ | Если ReLU dying | — |
| **GELU** | $x \cdot \Phi(x)$ | Transformers (GPT, BERT) | Дороже вычислительно |
| **Sigmoid** | $\frac{1}{1 + e^{-x}}$ | Бинарная классификация (выход) | Vanishing gradients, не для скрытых |
| **Softmax** | $\frac{e^{x_i}}{\sum e^{x_j}}$ | Мультиклассовый выход | — |
| **Tanh** | $\frac{e^x - e^{-x}}{e^x + e^{-x}}$ | RNN (если используете) | Vanishing gradients |

> "В скрытых слоях — ReLU (или GELU в transformers). На выходе — зависит от задачи: sigmoid для бинарной, softmax для мультиклассовой, ничего для регрессии."

---

## Q4. "Какие loss functions знаешь?"

| Task | Loss | PyTorch |
|------|------|---------|
| Бинарная классификация | Binary Cross-Entropy | `nn.BCEWithLogitsLoss()` |
| Мультиклассовая | Cross-Entropy | `nn.CrossEntropyLoss()` |
| Регрессия | MSE | `nn.MSELoss()` |
| Регрессия (робастный) | MAE / Huber | `nn.L1Loss()` / `nn.HuberLoss()` |

**Cross-Entropy формула:**

$$L = -\sum y_i \cdot \log(\hat{y}_i)$$

> "BCE — для бинарных задач (как кредитный скоринг в Альфа-Банке). CrossEntropy — для мультиклассовых. MSE — для регрессии, но чувствителен к выбросам, тогда Huber Loss."
