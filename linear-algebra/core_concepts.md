# Линейная алгебра — Interview Prep

Ключевые концепции линалга для ML-собеседований.

---

## Q1. "Векторы и операции над ними — скалярное произведение, нормы, cosine similarity"

> "Вектор — упорядоченный набор чисел, описывающий точку или направление в пространстве. В ML каждый объект — вектор фичей. **Скалярное произведение** (dot product) $\vec{a} \cdot \vec{b} = \sum_{i=1}^{n} a_i b_i$ — мера 'совпадения направлений'. Геометрически: $\vec{a} \cdot \vec{b} = \|\vec{a}\| \cdot \|\vec{b}\| \cdot \cos\theta$.
>
> **Нормы** измеряют длину вектора:
> - $L_1$: $\|\vec{x}\|_1 = \sum |x_i|$ — Manhattan distance, даёт разреженность (Lasso)
> - $L_2$: $\|\vec{x}\|_2 = \sqrt{\sum x_i^2}$ — евклидово расстояние, по умолчанию в ML
> - $L_\infty$: $\|\vec{x}\|_\infty = \max |x_i|$ — максимальное отклонение
>
> **Cosine similarity** — $\cos(\vec{a}, \vec{b}) = \frac{\vec{a} \cdot \vec{b}}{\|\vec{a}\| \cdot \|\vec{b}\|}$ — значение от $-1$ до $1$, не зависит от длины векторов, только от угла. В NLP это ключевая метрика: в Random Coffee я использовал Sentence-BERT, который кодирует интересы пользователей в эмбеддинги, и косинусное сходство определяет, кого матчить на кофе.
>
> **Проекция** вектора $\vec{a}$ на $\vec{b}$: $\text{proj}_{\vec{b}} \vec{a} = \frac{\vec{a} \cdot \vec{b}}{\|\vec{b}\|^2} \vec{b}$. Скалярная проекция — $\frac{\vec{a} \cdot \vec{b}}{\|\vec{b}\|}$. Проекции — основа PCA: проецируем данные на собственные векторы."

| Операция | Формула | Применение в ML |
|----------|---------|-----------------|
| Dot product | $\sum a_i b_i$ | Линейные модели: $\hat{y} = \vec{w} \cdot \vec{x} + b$ |
| $L_1$ norm | $\sum \|x_i\|$ | Lasso regularization, Manhattan distance |
| $L_2$ norm | $\sqrt{\sum x_i^2}$ | Ridge regularization, Euclidean distance |
| Cosine similarity | $\frac{\vec{a} \cdot \vec{b}}{\|\vec{a}\| \|\vec{b}\|}$ | Sentence-BERT matching, TF-IDF similarity |
| Projection | $\frac{\vec{a} \cdot \vec{b}}{\|\vec{b}\|^2} \vec{b}$ | PCA, Gram-Schmidt |

```python
import numpy as np

# Косинусное сходство — основа семантического матчинга в Random Coffee
def cosine_similarity(a, b):
    """Косинусное сходство двух векторов"""
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))

# Sentence-BERT эмбеддинги пользователей
user_a = np.array([0.2, 0.8, 0.1, 0.9])  # "ML, нейросети"
user_b = np.array([0.3, 0.7, 0.2, 0.85]) # "DL, трансформеры"
user_c = np.array([0.9, 0.1, 0.8, 0.05]) # "веб-разработка, фронтенд"

print(f"sim(a, b) = {cosine_similarity(user_a, user_b):.4f}")  # ~0.99 — похожие интересы
print(f"sim(a, c) = {cosine_similarity(user_a, user_c):.4f}")  # ~0.37 — разные интересы

# Нормы
x = np.array([3, -4, 0, 5])
print(f"L1 = {np.linalg.norm(x, 1)}")    # 12
print(f"L2 = {np.linalg.norm(x, 2):.2f}")  # 7.07
print(f"Linf = {np.linalg.norm(x, np.inf)}")  # 5
```

---

## Q2. "Матрицы — умножение, обратная, определитель, ранг"

> "Матрица — прямоугольная таблица чисел, представляющая линейное преобразование. В ML: датасет — матрица $X$ размером $(n \times m)$, где $n$ — объекты, $m$ — фичи. Веса нейросети — тоже матрицы.
>
> **Умножение**: $A_{n \times k} \cdot B_{k \times m} = C_{n \times m}$. Число столбцов $A$ должно совпадать с числом строк $B$. Это основа forward pass в нейросетях: $Z = XW + b$.
>
> **Транспонирование**: $A^T$ — строки становятся столбцами. $(AB)^T = B^T A^T$. Нужно для вычисления градиентов: $\frac{\partial L}{\partial W} = X^T \cdot \delta$.
>
> **Обратная матрица**: $A^{-1}$ существует только для квадратных матриц с $\det(A) \neq 0$. $A A^{-1} = I$. Аналитическое решение линейной регрессии: $\hat{w} = (X^T X)^{-1} X^T y$ — но на практике используют градиентный спуск, т.к. обращение матрицы $O(n^3)$.
>
> **Определитель**: $\det(A)$ — 'коэффициент масштабирования' площади/объёма при линейном преобразовании. Если $\det = 0$ — матрица сингулярна, пространство 'схлопывается' в меньшую размерность, обратной нет.
>
> **Ранг**: число линейно независимых строк (или столбцов). $\text{rank}(A) \leq \min(n, m)$. Если $\text{rank}(A) < m$ — есть мультиколлинеарность: некоторые фичи линейно зависимы. В Alfa-Bank credit scoring мультиколлинеарность — частая проблема: например, доход и сумма кредита коррелируют."

| Свойство | Формула / факт | Зачем в ML |
|----------|----------------|------------|
| Умножение | $A_{n \times k} \cdot B_{k \times m} = C_{n \times m}$ | Forward pass: $Z = XW$ |
| Транспонирование | $(AB)^T = B^T A^T$ | Gradient: $\nabla_W = X^T \delta$ |
| Обратная | $A^{-1}: AA^{-1} = I$ | Normal equation: $(X^TX)^{-1}X^Ty$ |
| Определитель | $\det(A) = 0 \Rightarrow$ сингулярная | Проверка мультиколлинеарности |
| Ранг | $\text{rank}(A) \leq \min(n,m)$ | Число независимых фичей |

**Связь с системами уравнений:**

$$Ax = b$$

- $\text{rank}(A) = \text{rank}(A|b) = m$ — единственное решение
- $\text{rank}(A) = \text{rank}(A|b) < m$ — бесконечно много решений
- $\text{rank}(A) < \text{rank}(A|b)$ — нет решений

```python
import numpy as np

# Датасет как матрица
X = np.array([[1, 2, 3],
              [4, 5, 6],
              [7, 8, 9]])

print(f"Shape: {X.shape}")          # (3, 3)
print(f"Rank: {np.linalg.matrix_rank(X)}")  # 2 — строки линейно зависимы!
print(f"Det: {np.linalg.det(X):.4f}")       # ~0 — сингулярная

# Normal equation для линейной регрессии
X_feat = np.column_stack([np.ones(100), np.random.randn(100, 2)])
y = X_feat @ np.array([1, 2, 3]) + np.random.randn(100) * 0.1
w_analytical = np.linalg.inv(X_feat.T @ X_feat) @ X_feat.T @ y
print(f"Weights (analytical): {w_analytical.round(2)}")  # ~[1, 2, 3]
```

---

## Q3. "Собственные значения и собственные векторы (Eigenvalues / Eigenvectors)"

> "$Av = \lambda v$ — собственный вектор $v$ при умножении на матрицу $A$ не меняет направление, только масштабируется в $\lambda$ раз. Геометрическая интуиция: если $A$ — трансформация пространства (вращение, растяжение), то eigenvectors — направления, которые остаются на месте (только растягиваются или сжимаются).
>
> **Как считать:** решаем характеристическое уравнение $\det(A - \lambda I) = 0$. Получаем характеристический полином степени $n$ — его корни дают eigenvalues. Затем для каждого $\lambda$ решаем $(A - \lambda I)v = 0$, чтобы найти eigenvector.
>
> **Связь с PCA:** ковариационная матрица $C = \frac{1}{n} X^T X$ — симметричная, положительно полуопределённая. Её eigenvectors — это главные компоненты (направления максимальной дисперсии данных). Eigenvalues — дисперсия вдоль каждого направления. Сортируем по убыванию eigenvalues, берём top-$k$ — получаем PCA.
>
> **Spectral decomposition** (для симметричных): $A = Q \Lambda Q^T$, где $Q$ — матрица eigenvectors (ортогональная), $\Lambda$ — диагональная с eigenvalues.
>
> В Alfa-Bank credit scoring: при большом числе коррелированных финансовых фичей (доход, расходы, платежи) PCA через eigendecomposition помогает выделить 'главные паттерны' без потери информации."

$$\det(A - \lambda I) = 0 \quad \Rightarrow \quad \lambda_1, \lambda_2, \ldots, \lambda_n$$

$$A v_i = \lambda_i v_i \quad \text{для каждого } i$$

$$A = Q \Lambda Q^T \quad \text{(spectral decomposition, } A = A^T\text{)}$$

```python
import numpy as np

# Ковариационная матрица и eigendecomposition — основа PCA
np.random.seed(42)
X = np.random.randn(100, 3) @ np.array([[2, 1, 0],
                                          [1, 3, 0.5],
                                          [0, 0.5, 0.1]])

# Ковариационная матрица
C = np.cov(X, rowvar=False)

# Eigendecomposition
eigenvalues, eigenvectors = np.linalg.eigh(C)

# Сортировка по убыванию
idx = np.argsort(eigenvalues)[::-1]
eigenvalues = eigenvalues[idx]
eigenvectors = eigenvectors[:, idx]

print("Eigenvalues:", eigenvalues.round(3))
# Первый eigenvalue >> остальных → первая главная компонента объясняет большую часть дисперсии

explained = eigenvalues / eigenvalues.sum()
print("Explained variance ratio:", explained.round(3))
```

---

## Q4. "SVD — Singular Value Decomposition"

> "SVD разлагает любую матрицу $A_{m \times n}$ на три: $A = U \Sigma V^T$.
> - $U_{m \times m}$ — ортогональная, 'левые сингулярные векторы' (вращение в пространстве строк)
> - $\Sigma_{m \times n}$ — диагональная, сингулярные значения $\sigma_1 \geq \sigma_2 \geq \ldots \geq 0$ (масштабирование)
> - $V^T_{n \times n}$ — ортогональная, 'правые сингулярные векторы' (вращение в пространстве столбцов)
>
> **Интуиция: rotate $\to$ scale $\to$ rotate.** Любое линейное преобразование — это: повернуть входное пространство ($V^T$), масштабировать вдоль осей ($\Sigma$), повернуть выходное пространство ($U$).
>
> **Truncated SVD (low-rank approximation):** оставляем только $k$ наибольших сингулярных значений: $A \approx U_k \Sigma_k V_k^T$. По теореме Эккарта-Янга это наилучшее приближение ранга $k$ по Frobenius norm.
>
> **Применения:**
> - **Рекомендательные системы:** matrix factorization — матрица 'user $\times$ item' раскладывается через SVD, заполняя пропуски (Netflix Prize)
> - **Сжатие изображений:** храним $k$ компонент вместо полной матрицы пикселей
> - **LSA (Latent Semantic Analysis):** SVD матрицы 'term $\times$ document' выделяет скрытые темы в текстах — предшественник topic modeling
>
> **Связь с PCA:** если $X$ центрирован (среднее = 0), то PCA через SVD: $X = U \Sigma V^T$, главные компоненты — столбцы $V$, eigenvalues ковариационной матрицы $= \frac{\sigma_i^2}{n-1}$."

| Компонент | Размерность | Смысл |
|-----------|-------------|-------|
| $U$ | $m \times m$ | Вращение в пространстве строк (объектов) |
| $\Sigma$ | $m \times n$ | Масштабирование (сингулярные значения) |
| $V^T$ | $n \times n$ | Вращение в пространстве столбцов (фичей) |
| Truncated ($k$) | $U_k, \Sigma_k, V_k^T$ | Наилучшее приближение ранга $k$ |

```python
import numpy as np

# SVD для сжатия "изображения" (матрицы)
np.random.seed(42)
A = np.random.randn(100, 80)

U, sigma, Vt = np.linalg.svd(A, full_matrices=False)

# Truncated SVD — оставляем k компонент
for k in [5, 10, 20, 50]:
    A_approx = U[:, :k] @ np.diag(sigma[:k]) @ Vt[:k, :]
    error = np.linalg.norm(A - A_approx, 'fro') / np.linalg.norm(A, 'fro')
    compression = (100*k + k + 80*k) / (100*80)
    print(f"k={k:2d}: relative error={error:.4f}, compression ratio={compression:.2%}")

# LSA пример — SVD для текстов
from sklearn.decomposition import TruncatedSVD
from sklearn.feature_extraction.text import TfidfVectorizer

docs = ["machine learning models", "deep neural networks",
        "credit scoring banking", "financial risk assessment"]
tfidf = TfidfVectorizer().fit_transform(docs)
lsa = TruncatedSVD(n_components=2).fit_transform(tfidf)
print(f"\nLSA shape: {lsa.shape}")  # (4, 2) — 4 документа в 2D пространстве тем
```

---

## Q5. "PCA через призму линалга — ковариационная матрица, explained variance"

> "PCA (Principal Component Analysis) — метод снижения размерности. Идея: найти ортогональные направления максимальной дисперсии данных и проецировать на них.
>
> **Алгоритм:**
> 1. Центрируем данные: $X_c = X - \bar{X}$
> 2. Считаем ковариационную матрицу: $C = \frac{1}{n-1} X_c^T X_c$
> 3. Eigendecomposition: $C = Q \Lambda Q^T$
> 4. Сортируем eigenvectors по убыванию eigenvalues
> 5. Берём top-$k$ eigenvectors — это главные компоненты
> 6. Проекция: $X_{\text{new}} = X_c \cdot Q_k$
>
> **Explained variance ratio**: $\frac{\lambda_i}{\sum_j \lambda_j}$ — доля дисперсии, объяснённая $i$-й компонентой. Обычно берём компоненты, покрывающие 90-95% дисперсии.
>
> **Когда применять:**
> - **Проклятие размерности** (curse of dimensionality): с ростом числа фичей нужно экспоненциально больше данных; PCA сжимает пространство
> - **Визуализация**: проецируем данные в 2D/3D для разведочного анализа
> - **Удаление мультиколлинеарности**: PCA компоненты ортогональны по определению
> - **Ускорение обучения**: меньше фичей — быстрее модель
>
> **Ограничения:** PCA ловит только линейные зависимости. Для нелинейных — Kernel PCA, t-SNE, UMAP.
>
> В Alfa-Bank credit scoring: сотни финансовых фичей часто коррелируют. PCA позволяет уменьшить их число, ускорив обучение CatBoost без значительной потери качества. Также в hr-breaker визуализация эмбеддингов резюме через PCA помогает понять кластерную структуру."

$$C = \frac{1}{n-1} X_c^T X_c \quad \xrightarrow{\text{eigendecomposition}} \quad \lambda_1 \geq \lambda_2 \geq \ldots \geq \lambda_m$$

$$\text{Explained variance ratio}_i = \frac{\lambda_i}{\sum_{j=1}^{m} \lambda_j}$$

```python
import numpy as np
from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import load_iris

# Классический пример — Iris dataset
X, y = load_iris(return_X_y=True)
X_scaled = StandardScaler().fit_transform(X)

# PCA
pca = PCA(n_components=4)
X_pca = pca.fit_transform(X_scaled)

print("Explained variance ratio:", pca.explained_variance_ratio_.round(3))
# [0.729, 0.229, 0.037, 0.005] — первые 2 компоненты = 95.8% дисперсии

cumulative = np.cumsum(pca.explained_variance_ratio_)
print("Cumulative:", cumulative.round(3))

# На практике: выбираем k по порогу 95%
n_components_95 = np.argmax(cumulative >= 0.95) + 1
print(f"Components for 95% variance: {n_components_95}")  # 2

# Применение: PCA + классификатор
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import cross_val_score

pca_2d = PCA(n_components=2)
X_2d = pca_2d.fit_transform(X_scaled)

score_full = cross_val_score(LogisticRegression(), X_scaled, y, cv=5).mean()
score_pca = cross_val_score(LogisticRegression(), X_2d, y, cv=5).mean()
print(f"Full features (4D): {score_full:.3f}")
print(f"PCA (2D):           {score_pca:.3f}")
```

---

## Q6. "Нормы векторов и матриц в ML — регуляризация и нормализация"

> "Нормы в ML выполняют две ключевые роли: **регуляризация** (штраф на сложность модели) и **нормализация** (приведение фичей к одному масштабу).
>
> **$L_1$ regularization (Lasso):** штраф $\lambda \sum |w_i|$. Даёт **разреженность** — часть весов становятся ровно нулём. По сути, Lasso делает feature selection: ненужные фичи 'выключаются'. Геометрически: область допустимых весов — ромб (diamond) с углами на осях, и оптимум часто попадает в угол (нулевая координата).
>
> **$L_2$ regularization (Ridge):** штраф $\lambda \sum w_i^2$. Уменьшает все веса, но не обнуляет. Помогает при мультиколлинеарности — стабилизирует решение $(X^TX + \lambda I)^{-1}X^Ty$. Геометрически: область — круг (sphere), оптимум не попадает на оси.
>
> **ElasticNet:** $\alpha \cdot L_1 + (1 - \alpha) \cdot L_2$ — комбинация. Когда фичей много и часть коррелируют, Lasso выберет одну из группы, ElasticNet оставит несколько. В Alfa-Bank credit scoring ElasticNet — хороший baseline до перехода на CatBoost.
>
> **Frobenius norm** для матриц: $\|A\|_F = \sqrt{\sum_{i,j} a_{ij}^2}$ — по сути $L_2$ norm, если 'развернуть' матрицу в вектор. Используется в low-rank approximation (SVD) и weight decay нейросетей.
>
> **Нормализация фичей:** StandardScaler ($z = \frac{x - \mu}{\sigma}$) приводит фичи к среднему 0 и дисперсии 1. Без нормализации gradient descent будет 'вытянутым' (elongated contours) — медленная сходимость. С нормализацией — круглые контуры, быстрая сходимость. Для CatBoost нормализация не нужна (деревья инвариантны к масштабу), но для нейросетей и SVM — обязательна."

| Норма | Формула | Свойство | Метод |
|-------|---------|----------|-------|
| $L_1$ | $\sum \|w_i\|$ | Разреженность (feature selection) | Lasso |
| $L_2$ | $\sum w_i^2$ | Малые веса (устойчивость) | Ridge |
| $L_1 + L_2$ | $\alpha L_1 + (1-\alpha) L_2$ | Компромисс | ElasticNet |
| Frobenius | $\sqrt{\sum a_{ij}^2}$ | Weight decay для матриц | Neural nets |

$$\text{Lasso: } \min_w \|Xw - y\|_2^2 + \lambda \|w\|_1$$

$$\text{Ridge: } \min_w \|Xw - y\|_2^2 + \lambda \|w\|_2^2$$

$$\text{ElasticNet: } \min_w \|Xw - y\|_2^2 + \lambda_1 \|w\|_1 + \lambda_2 \|w\|_2^2$$

```python
import numpy as np
from sklearn.linear_model import Lasso, Ridge, ElasticNet
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import make_regression

# Разреженность: Lasso vs Ridge
X, y, true_coef = make_regression(n_samples=200, n_features=20,
                                   n_informative=5, coef=True, random_state=42)
X = StandardScaler().fit_transform(X)

lasso = Lasso(alpha=0.5).fit(X, y)
ridge = Ridge(alpha=0.5).fit(X, y)
enet = ElasticNet(alpha=0.5, l1_ratio=0.5).fit(X, y)

print(f"True non-zero features: {np.sum(true_coef != 0)}")
print(f"Lasso non-zero weights: {np.sum(np.abs(lasso.coef_) > 0.01)}")  # ~5 — разреженность!
print(f"Ridge non-zero weights: {np.sum(np.abs(ridge.coef_) > 0.01)}")  # ~20 — все ненулевые
print(f"ElasticNet non-zero:    {np.sum(np.abs(enet.coef_) > 0.01)}")

# Зачем StandardScaler — влияние на gradient descent
from sklearn.linear_model import SGDRegressor
from sklearn.model_selection import cross_val_score

X_raw, y_raw = make_regression(n_samples=500, n_features=10, random_state=42)
X_raw[:, 0] *= 1000  # фича в тысячах
X_raw[:, 1] *= 0.001  # фича в тысячных

score_raw = cross_val_score(SGDRegressor(max_iter=100), X_raw, y_raw, cv=5).mean()
score_scaled = cross_val_score(SGDRegressor(max_iter=100),
                                StandardScaler().fit_transform(X_raw), y_raw, cv=5).mean()
print(f"\nSGD without scaling: R2 = {score_raw:.3f}")
print(f"SGD with scaling:    R2 = {score_scaled:.3f}")  # значительно лучше
```

---

## Приоритет для intern-level

| Тема | Приоритет | Почему |
|------|-----------|--------|
| Скалярное произведение, нормы | **Высокий** | Основа similarity, regularization |
| PCA | **Высокий** | Спрашивают на каждом втором собесе |
| Нормы в ML (L1/L2/ElasticNet) | **Высокий** | Вопрос про регуляризацию — почти гарантирован |
| SVD | Средний | Понимание на уровне интуиции |
| Eigenvalues | Средний | Связь с PCA достаточно |
| Матрицы (ранг, определитель) | Средний | Базовая грамотность, редко спрашивают глубоко |
