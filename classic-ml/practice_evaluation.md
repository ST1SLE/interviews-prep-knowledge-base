# Evaluation & Debugging — Practice Exercises

Cross-validation from scratch и model debugging.

---

## Exercise 4: Cross-Validation from Scratch

**Задача:** Реализуй Stratified K-Fold CV без sklearn. Используй для сравнения моделей.

```python
import numpy as np
from sklearn.datasets import make_classification
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import roc_auc_score

def stratified_k_fold(X, y, k=5, seed=42):
    """Генератор (train_idx, val_idx) для каждого фолда."""
    rng = np.random.RandomState(seed)
    folds = [[] for _ in range(k)]

    for cls in np.unique(y):
        cls_idx = np.where(y == cls)[0]
        rng.shuffle(cls_idx)
        for i, idx in enumerate(cls_idx):
            folds[i % k].append(idx)

    for i in range(k):
        val_idx = np.array(folds[i])
        train_idx = np.concatenate([folds[j] for j in range(k) if j != i])
        yield train_idx, val_idx

def cross_val_score(model_class, model_params, X, y, k=5):
    scores = []
    for train_idx, val_idx in stratified_k_fold(X, y, k=k):
        model = model_class(**model_params)
        model.fit(X[train_idx], y[train_idx])
        proba = model.predict_proba(X[val_idx])[:, 1]
        scores.append(roc_auc_score(y[val_idx], proba))
    return np.array(scores)

# === СРАВНЕНИЕ МОДЕЛЕЙ ===
X, y = make_classification(n_samples=1000, n_features=15,
                           weights=[0.8, 0.2], random_state=42)

lr_scores = cross_val_score(LogisticRegression, {'max_iter': 1000}, X, y)
rf_scores = cross_val_score(RandomForestClassifier, {'n_estimators': 100, 'random_state': 42}, X, y)

print(f"LogReg AUC: {lr_scores.mean():.4f} ± {lr_scores.std():.4f}")
print(f"RF AUC:     {rf_scores.mean():.4f} ± {rf_scores.std():.4f}")

# Paired t-test
from scipy import stats
t, p = stats.ttest_rel(lr_scores, rf_scores)
print(f"\nPaired t-test: t={t:.4f}, p={p:.4f}")
print("Значимо!" if p < 0.05 else "Разница не значима")
```

---

## Exercise 8: Model Debugging — "Почему модель плохая?"

**Задача:** Дан плохой pipeline. Найди и исправь 5 ошибок.

```python
import numpy as np
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import roc_auc_score

X, y = make_classification(n_samples=1000, n_features=20,
                           n_informative=5, n_redundant=10,
                           weights=[0.95, 0.05], random_state=42)

# ========== BUGGY CODE (найди 5 ошибок) ==========
# scaler = StandardScaler()
# X_scaled = scaler.fit_transform(X)        # BUG 1: scale ПЕРЕД split → leakage
# X_tr, X_te, y_tr, y_te = train_test_split(X_scaled, y, test_size=0.2)  # BUG 2: нет stratify
# model = LogisticRegression()               # BUG 3: нет class_weight для дисбаланса
# model.fit(X_tr, y_tr)
# pred = model.predict(X_te)                 # BUG 4: predict вместо predict_proba для AUC
# print(f"AUC: {roc_auc_score(y_te, pred)}") # BUG 5: accuracy бы показала 0.95 (useless)

# ========== FIXED CODE ==========
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.2,
                                            stratify=y, random_state=42)
scaler = StandardScaler()
X_tr_scaled = scaler.fit_transform(X_tr)
X_te_scaled = scaler.transform(X_te)

model = LogisticRegression(class_weight='balanced', max_iter=1000)
model.fit(X_tr_scaled, y_tr)
proba = model.predict_proba(X_te_scaled)[:, 1]
print(f"AUC: {roc_auc_score(y_te, proba):.4f}")

# Bonus: learning curve — overfitting или underfitting?
from sklearn.model_selection import learning_curve
import matplotlib.pyplot as plt

train_sizes, train_scores, val_scores = learning_curve(
    LogisticRegression(class_weight='balanced', max_iter=1000),
    X, y, cv=5, scoring='roc_auc',
    train_sizes=np.linspace(0.1, 1.0, 10))

plt.plot(train_sizes, train_scores.mean(axis=1), label='Train')
plt.plot(train_sizes, val_scores.mean(axis=1), label='Validation')
plt.xlabel('Training Size')
plt.ylabel('AUC')
plt.legend()
plt.title('Learning Curve')
plt.show()
# Train >> Val → overfitting. Train ≈ Val (оба низкие) → underfitting.
```
