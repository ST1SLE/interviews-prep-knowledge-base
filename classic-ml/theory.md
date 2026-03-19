# Classic ML — Theory Interview Prep

Теоретические вопросы по классическому ML для собеседования ML/DS/AI Engineer Intern. Градиентный бустинг, метрики, регуляризация, feature engineering.

---

## Q1. "Как работает градиентный бустинг?"

> "Градиентный бустинг — ансамбль решающих деревьев, обучаемых **последовательно**. Каждое следующее дерево обучается на **антиградиенте** (остатках) loss-функции предыдущей модели.
>
> Формально: $F_m(x) = F_{m-1}(x) + \eta \cdot h_m(x)$, где $h_m$ — новое дерево, $\eta$ — learning rate.
>
> Алгоритм:
> 1. Инициализация: $F_0(x) = \arg\min_c \sum L(y_i, c)$ (для MSE — среднее $y$)
> 2. Для каждой итерации $m = 1, ..., M$:
>    - Вычисляем псевдо-остатки: $r_i = -\frac{\partial L(y_i, F_{m-1}(x_i))}{\partial F_{m-1}(x_i)}$
>    - Обучаем дерево $h_m$ на $(x_i, r_i)$
>    - Обновляем: $F_m(x) = F_{m-1}(x) + \eta \cdot h_m(x)$
>
> Learning rate $\eta$ — регуляризация: маленький $\eta$ → больше деревьев, но лучше обобщение (shrinkage). На практике $\eta \in [0.01, 0.1]$ + early stopping."

$$F_m(x) = F_{m-1}(x) + \eta \cdot h_m(x)$$

где $h_m(x) = \arg\min_h \sum_{i=1}^{n} \left[ r_i - h(x_i) \right]^2$, $r_i$ — антиградиент loss.

---

## Q2. "CatBoost vs LightGBM vs XGBoost — отличия"

> "Все три — реализации градиентного бустинга, но с разными оптимизациями:
>
> **XGBoost**: histogram-based splits (дискретизация фичей → быстрее), regularization в loss-функции ($L1 + L2$ на веса листьев), поддержка GPU.
>
> **LightGBM**: leaf-wise рост дерева (а не level-wise как XGBoost) → быстрее сходится, но склонен к переобучению на малых данных. GOSS (Gradient-based One-Side Sampling) — оставляет сэмплы с большими градиентами. Exclusive Feature Bundling (EFB) — группирует разреженные фичи.
>
> **CatBoost**: oblivious trees (все листья на одном уровне делают split по одной фиче → быстрый inference). Ordered boosting — борьба с target leakage при вычислении target statistics. Встроенная обработка категориальных фичей без One-Hot.
>
> В Альфа-Банке использовал CatBoost для кредитного скоринга: много категориальных признаков (тип занятости, регион, банковский продукт) → CatBoost обрабатывает их нативно без ручного кодирования."

| Критерий | XGBoost | LightGBM | CatBoost |
|----------|---------|----------|----------|
| Рост дерева | Level-wise | Leaf-wise | Symmetric (oblivious) |
| Категориальные фичи | Ручной encoding | Ручной encoding | Встроенный (ordered TS) |
| Скорость обучения | Средняя | **Быстрая** | Средняя |
| Скорость inference | Средняя | Средняя | **Быстрая** (oblivious) |
| Переобучение | Средний риск | Выше (leaf-wise) | **Ниже** (ordered boosting) |
| GPU | Да | Да | Да |
| Когда выбирать | Baseline, конкурсы | Большие данные, скорость | Категориальные фичи, табличные данные |

---

## Q3. "Bias-Variance Tradeoff"

> "Ожидаемая ошибка модели раскладывается на три компоненты:
>
> $$\text{Error} = \text{Bias}^2 + \text{Variance} + \text{Irreducible Noise}$$
>
> **Bias** (смещение) — систематическая ошибка, модель слишком простая (underfitting). Пример: линейная регрессия на нелинейных данных.
>
> **Variance** (дисперсия) — разброс предсказаний при разных обучающих выборках. Модель слишком сложная (overfitting). Пример: дерево глубины 100 — чуть другие данные → совсем другие предсказания.
>
> **Bagging** снижает variance: Random Forest усредняет много глубоких деревьев → каждое переобучено, но среднее стабильно.
>
> **Boosting** снижает bias: каждое дерево исправляет ошибки предыдущих → простые деревья постепенно захватывают сложные паттерны.
>
> Как диагностировать: learning curves. Если train error ≈ val error и оба высокие → high bias. Если train error ≪ val error → high variance."

| Проблема | Диагностика | Решение |
|----------|-------------|---------|
| High Bias (underfitting) | Train ≈ Val, оба высокие | Сложнее модель, feature engineering, убрать регуляризацию |
| High Variance (overfitting) | Train ≪ Val | Регуляризация, больше данных, ensemble (bagging), dropout |
| Оптимум | Train ≈ Val, оба низкие | Текущие настройки хороши |

---

## Q4. "Предпосылки линейной регрессии"

> "Линейная регрессия работает корректно при выполнении предпосылок Гаусса-Маркова:
>
> 1. **Линейность**: $y = X\beta + \varepsilon$. Если связь нелинейная — полиномиальные фичи или другая модель.
> 2. **Гомоскедастичность**: дисперсия остатков постоянна ($\text{Var}(\varepsilon_i) = \sigma^2$). Если нет (гетероскедастичность) — WLS (Weighted Least Squares) или логарифм target.
> 3. **Независимость остатков**: $\text{Cov}(\varepsilon_i, \varepsilon_j) = 0$. Нарушается во временных рядах → используем автокорреляционные модели.
> 4. **Отсутствие мультиколлинеарности**: фичи не должны быть линейно зависимы. Диагностика: VIF (Variance Inflation Factor) > 10 → проблема. Решение: удалить фичу, Ridge (L2) регуляризация.
> 5. **Нормальность остатков** (для inference): $\varepsilon \sim \mathcal{N}(0, \sigma^2)$. Нужна для корректных CI и p-value коэффициентов.
>
> Диагностика: residual plot (остатки vs предсказания). Если паттерн виден — предпосылки нарушены."

| Предпосылка | Как проверить | Что делать при нарушении |
|-------------|---------------|-------------------------|
| Линейность | Residual plot (паттерн = нелинейность) | Полиномиальные фичи, другая модель |
| Гомоскедастичность | Residual plot (веер = гетероскедастичность) | WLS, log-трансформация target |
| Независимость остатков | Durbin-Watson test | Автокорреляционные модели |
| Нет мультиколлинеарности | VIF > 10 → проблема | Удалить фичу, Ridge (L2) |
| Нормальность остатков | Q-Q plot, Shapiro-Wilk test | Bootstrap для CI вместо формул |

---

## Q5. "Интерпретация коэффициентов логистической регрессии"

> "Логистическая регрессия моделирует вероятность: $P(y=1|x) = \sigma(w^T x + b) = \frac{1}{1 + e^{-(w^T x + b)}}$.
>
> Коэффициент $\beta_j$ — это изменение **log-odds** при увеличении фичи $x_j$ на 1 (при фиксированных остальных):
>
> $$\log\frac{P(y=1)}{P(y=0)} = \beta_0 + \beta_1 x_1 + ... + \beta_p x_p$$
>
> $e^{\beta_j}$ — **odds ratio**: во сколько раз изменяются шансы при увеличении $x_j$ на 1.
>
> Пример (кредитный скоринг в Альфа-Банке): если $\beta_{\text{income}} = 0.5$, то $e^{0.5} = 1.65$ — при увеличении дохода на 1 единицу шансы одобрения кредита увеличиваются в 1.65 раз.
>
> Важно: для сравнения важности фичей нужна **нормализация** (StandardScaler), иначе коэффициенты зависят от масштаба. После нормализации $|\beta_j|$ показывает относительную важность."

$$\text{Odds Ratio}_j = e^{\beta_j}$$

```python
from sklearn.linear_model import LogisticRegression
from sklearn.preprocessing import StandardScaler
import numpy as np

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

model = LogisticRegression()
model.fit(X_scaled, y)

for name, coef in zip(feature_names, model.coef_[0]):
    odds_ratio = np.exp(coef)
    print(f"{name}: beta={coef:.3f}, OR={odds_ratio:.3f}")
# income: beta=0.50, OR=1.649  -> доход повышает шансы одобрения
# debt:   beta=-0.30, OR=0.741 -> долг снижает шансы одобрения
```

---

## Q6. "L1 (Lasso) vs L2 (Ridge) — геометрическая интуиция"

> "Оба метода добавляют штраф к loss-функции для борьбы с переобучением:
>
> **L2 (Ridge)**: $L = \text{MSE} + \lambda \sum \beta_j^2$. Геометрия: ограничение — **круг**. Оптимум — пересечение контуров loss с кругом → все веса маленькие, но **ненулевые**.
>
> **L1 (Lasso)**: $L = \text{MSE} + \lambda \sum |\beta_j|$. Геометрия: ограничение — **ромб** (diamond). Оптимум часто попадает на **вершину ромба** (ось координат) → часть весов **точно 0** → feature selection.
>
> **ElasticNet** = L1 + L2: $L = \text{MSE} + \lambda_1 \sum |\beta_j| + \lambda_2 \sum \beta_j^2$. Комбинирует преимущества обоих.
>
> Когда что:
> - **Ridge**: при мультиколлинеарности (коррелированные фичи). Не убирает фичи, но стабилизирует коэффициенты.
> - **Lasso**: когда подозреваем, что большинство фичей неважны → автоматический feature selection.
> - **ElasticNet**: когда есть группы коррелированных фичей (Lasso случайно выберет одну из группы, ElasticNet сохранит все).
>
> Связь с Bayesian: L2 = Gaussian prior на веса, L1 = Laplace prior."

| Регуляризация | Штраф | Геометрия | Эффект на веса | Когда использовать |
|---------------|-------|-----------|----------------|-------------------|
| **L2 (Ridge)** | $\lambda \sum \beta_j^2$ | Круг | Малые, ненулевые | Мультиколлинеарность |
| **L1 (Lasso)** | $\lambda \sum |\beta_j|$ | Ромб | Разреженные (нули) | Feature selection |
| **ElasticNet** | $\lambda_1 \sum |\beta_j| + \lambda_2 \sum \beta_j^2$ | Скруглённый ромб | Компромисс | Группы коррелированных фичей |

---

## Q7. "Метрики для несбалансированных выборок"

> "При дисбалансе 99:1 (как в задаче кредитного скоринга в Альфа-Банке) Accuracy обманчива: модель 'всегда 0' даёт Accuracy = 99%, но бесполезна.
>
> **Precision** = TP / (TP + FP) — 'из тех, кого назвали дефолтом, сколько реально дефолтов'. Важна, когда FP дорогой (отказ хорошему клиенту).
>
> **Recall** = TP / (TP + FN) — 'из реальных дефолтов, сколько нашли'. Важна, когда FN дорогой (пропустили мошенника).
>
> **F1** = $2 \cdot \frac{P \cdot R}{P + R}$ — гармоническое среднее Precision и Recall.
>
> **ROC-AUC** — площадь под кривой TPR vs FPR. Threshold-independent. Хорош для сравнения моделей.
>
> **PR-AUC** — площадь под кривой Precision vs Recall. Лучше ROC-AUC при сильном дисбалансе: ROC-AUC может быть высоким (0.95) даже когда Precision низкий, т.к. TNs доминируют.
>
> **Threshold tuning**: по умолчанию threshold = 0.5, но для бизнеса оптимальный threshold зависит от стоимости ошибок. В скоринге: снизить threshold → больше Recall (найдём больше дефолтов), но меньше Precision (больше отказов хорошим клиентам)."

| Метрика | Формула | Когда важна | Связь с бизнесом |
|---------|---------|-------------|------------------|
| **Precision** | $\frac{TP}{TP + FP}$ | FP дорогой | Не отказать хорошему клиенту |
| **Recall** | $\frac{TP}{TP + FN}$ | FN дорогой | Не пропустить мошенника |
| **F1** | $2 \cdot \frac{P \cdot R}{P + R}$ | Баланс P/R | Общая оценка |
| **ROC-AUC** | Area under TPR/FPR | Общее сравнение моделей | Threshold-independent |
| **PR-AUC** | Area under P/R | **Сильный дисбаланс** | Лучше ROC-AUC при 99:1 |

```python
from sklearn.metrics import (
    precision_recall_curve, roc_auc_score, average_precision_score,
    f1_score, precision_score, recall_score
)

roc_auc = roc_auc_score(y_true, y_proba)
pr_auc = average_precision_score(y_true, y_proba)
print(f"ROC-AUC: {roc_auc:.3f}, PR-AUC: {pr_auc:.3f}")

# Threshold tuning: выбираем порог по F1
precisions, recalls, thresholds = precision_recall_curve(y_true, y_proba)
f1_scores = 2 * precisions * recalls / (precisions + recalls + 1e-8)
best_idx = f1_scores.argmax()
best_threshold = thresholds[best_idx]
print(f"Best threshold: {best_threshold:.3f}, F1: {f1_scores[best_idx]:.3f}")
```

---

## Q8. "Кодирование категориальных признаков"

> "Категориальные фичи нельзя подать в модель 'как есть' (строки). Основные методы:
>
> **One-Hot Encoding**: каждое значение → отдельный бинарный столбец. Хорошо при малом числе уникальных значений (< 10-20). Проблема: при 10K+ уникальных значений (город, товар) → проклятие размерности, разреженная матрица.
>
> **Target Encoding**: заменяем значение категории средним target. Опасность: data leakage! Решение: K-Fold Target Encoding — для каждого фолда считаем статистику только по остальным фолдам. Сглаживание: $\frac{n \cdot \text{mean}_{\text{cat}} + m \cdot \text{mean}_{\text{global}}}{n + m}$, где $m$ — параметр сглаживания.
>
> **Frequency Encoding**: заменяем категорию её частотой в train. Просто, без leakage, но теряет порядок.
>
> **Эмбеддинги**: для нейросетей — каждая категория → обучаемый вектор фиксированной размерности. Как Word2Vec, но для табличных данных.
>
> CatBoost решает эту проблему нативно: ordered target statistics — считает target encoding по предыдущим по времени наблюдениям (ordered boosting), что устраняет leakage."

| Метод | Размерность | Leakage | Когда |
|-------|------------|---------|-------|
| **One-Hot** | $k$ столбцов | Нет | Мало уникальных (< 20) |
| **Target Encoding** | 1 столбец | **Да** (без K-Fold) | Много уникальных + tree models |
| **Frequency Encoding** | 1 столбец | Нет | Быстрый baseline |
| **Эмбеддинги** | $d$ (настраиваемый) | Нет | Нейросети, очень много уникальных |
| **CatBoost native** | Автоматически | Нет (ordered TS) | В CatBoost моделях |

```python
import pandas as pd
from sklearn.model_selection import KFold
import numpy as np

def target_encode_kfold(df, col, target, n_splits=5, smoothing=10):
    """K-Fold Target Encoding — без data leakage."""
    global_mean = df[target].mean()
    encoded = pd.Series(index=df.index, dtype=float)

    kf = KFold(n_splits=n_splits, shuffle=True, random_state=42)
    for train_idx, val_idx in kf.split(df):
        stats = df.iloc[train_idx].groupby(col)[target].agg(['mean', 'count'])
        smoothed = (stats['count'] * stats['mean'] + smoothing * global_mean) / (stats['count'] + smoothing)
        encoded.iloc[val_idx] = df.iloc[val_idx][col].map(smoothed).fillna(global_mean)

    return encoded
```

---

## Q9. "Bagging vs Boosting"

> "Оба — ensemble методы, но с разной философией:
>
> **Bagging** (Bootstrap Aggregating): обучаем $N$ **независимых** моделей на bootstrap-подвыборках, усредняем предсказания. Каждая модель — глубокое дерево (low bias, high variance). Усреднение снижает variance. Пример: **Random Forest** = bagging + случайный подбор фичей для каждого split.
>
> **Boosting**: обучаем $N$ моделей **последовательно**, каждая исправляет ошибки предыдущей. Каждая модель — слабый классификатор (high bias, low variance). Итеративное обучение снижает bias. Пример: **GBM, XGBoost, CatBoost, LightGBM**.
>
> Random Forest vs GBM: RF проще (нет гиперпараметра learning rate, параллельное обучение), но GBM обычно точнее при правильном тюнинге."

| Аспект | Bagging (Random Forest) | Boosting (GBM) |
|--------|------------------------|----------------|
| Обучение | **Параллельно** | Последовательно |
| Базовая модель | Глубокие деревья | Мелкие деревья (stumps) |
| Что снижает | **Variance** | **Bias** |
| Подвыборки | Bootstrap (с возвратом) | Weighted / residuals |
| Переобучение | Устойчив | Склонен (нужен early stopping) |
| Тюнинг | Мало гиперпараметров | Много (LR, depth, iterations) |
| Скорость | Быстрее (параллелизм) | Медленнее (последовательно) |

---

## Q10. "Open-ended: как решить задачу предсказания оттока (Churn)?"

> "Типичная бизнес-задача. Пошаговый подход:
>
> **1. Метрика**: Recall важнее Precision — лучше послать оффер лишнему клиенту (FP дёшево), чем пропустить уходящего (FN дорого). PR-AUC для общей оценки модели.
>
> **2. Feature Engineering**:
> - RFM-анализ: Recency (дней с последней активности), Frequency (число визитов за 30/60/90 дней), Monetary (сумма покупок).
> - Temporal features: тренд активности (снижение за последние N недель), сезонность.
> - Агрегаты: среднее, медиана, std количества транзакций по окнам (7/14/30/60/90 дней).
> - Behavioral: изменение паттерна (смена категорий покупок, уменьшение среднего чека).
>
> **3. Модель**: градиентный бустинг (CatBoost / LightGBM) — лучший выбор для табличных данных. Baseline: логистическая регрессия (для интерпретируемости).
>
> **4. Интерпретация**: SHAP values для объяснения причин оттока каждого клиента. Бизнесу нужно знать НЕ ТОЛЬКО 'клиент уйдёт', но и ПОЧЕМУ (чтобы предпринять правильное действие).
>
> Связь с Альфа-Банком: аналогичная задача — предсказание дефолта. Те же принципы: RFM-фичи по транзакциям, CatBoost, SHAP для объяснения."

```python
import shap

explainer = shap.TreeExplainer(model)
shap_values = explainer.shap_values(X_test)

# Top факторы оттока для конкретного клиента
shap.waterfall_plot(shap.Explanation(
    values=shap_values[0],
    base_values=explainer.expected_value,
    data=X_test.iloc[0],
    feature_names=X_test.columns
))

# Глобальная важность фичей
shap.summary_plot(shap_values, X_test)
```

| Шаг | Действие | Инструменты |
|-----|----------|-------------|
| Метрика | PR-AUC, Recall@K | sklearn.metrics |
| Feature Engineering | RFM, temporal aggregates | pandas, window functions |
| Baseline | Логистическая регрессия | sklearn |
| Модель | CatBoost / LightGBM | catboost, lightgbm |
| Интерпретация | SHAP values | shap |
| Мониторинг | Drift detection (PSI) | evidently |

---

## Приоритет для intern-level

| Тема | Приоритет | Почему |
|------|-----------|--------|
| Градиентный бустинг (Q1, Q2) | **Высокий** | TOP-1 вопрос на DS-собесах |
| Bias-Variance Tradeoff (Q3) | **Высокий** | Фундаментальная концепция |
| Метрики при дисбалансе (Q7) | **Высокий** | Спрашивают в контексте бизнес-задач |
| L1/L2 регуляризация (Q6) | **Высокий** | Связь с feature selection |
| Bagging vs Boosting (Q9) | Средний | Для понимания RF vs GBM |
| Логистическая регрессия (Q5) | Средний | Интерпретация коэффициентов |
| Предпосылки линейной регрессии (Q4) | Средний | Для DS/статистических позиций |
| Кодирование категорий (Q8) | Средний | Практический навык |
| Open-ended: Churn (Q10) | Средний | Показывает системное мышление |
