# Training Techniques — Practice Exercises

Imbalanced data и hyperparameter tuning.

---

## Exercise 6: Imbalanced Data — Все подходы

**Задача:** Один датасет (1:10 дисбаланс). Сравни 4 подхода к дисбалансу.

```python
import numpy as np
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split, StratifiedKFold, cross_val_score
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import f1_score, classification_report
import warnings
warnings.filterwarnings('ignore')

# === ДАННЫЕ (сильный дисбаланс) ===
X, y = make_classification(n_samples=2000, n_features=10,
                           weights=[0.9, 0.1], random_state=42)
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.2, stratify=y)
print(f"Train: {np.bincount(y_tr)}  |  Test: {np.bincount(y_te)}\n")

results = {}

# --- Подход 1: Ничего не делаем (baseline) ---
lr = LogisticRegression(max_iter=1000)
lr.fit(X_tr, y_tr)
results['baseline'] = f1_score(y_te, lr.predict(X_te))

# --- Подход 2: class_weight='balanced' ---
lr_bal = LogisticRegression(max_iter=1000, class_weight='balanced')
lr_bal.fit(X_tr, y_tr)
results['class_weight'] = f1_score(y_te, lr_bal.predict(X_te))

# --- Подход 3: Oversampling (SMOTE) ---
# pip install imbalanced-learn
try:
    from imblearn.over_sampling import SMOTE
    sm = SMOTE(random_state=42)
    X_res, y_res = sm.fit_resample(X_tr, y_tr)
    lr_smote = LogisticRegression(max_iter=1000)
    lr_smote.fit(X_res, y_res)
    results['SMOTE'] = f1_score(y_te, lr_smote.predict(X_te))
except ImportError:
    print("imblearn not installed, skipping SMOTE")

# --- Подход 4: Threshold tuning ---
from sklearn.metrics import precision_recall_curve
proba = lr_bal.predict_proba(X_te)[:, 1]
prec, rec, thresh = precision_recall_curve(y_te, proba)
f1s = 2 * prec * rec / (prec + rec + 1e-8)
best_t = thresh[np.argmax(f1s)]
results['threshold_tuned'] = f1_score(y_te, (proba >= best_t).astype(int))

# === РЕЗУЛЬТАТЫ ===
print(f"{'Approach':<20} {'F1':>6}")
print("-" * 28)
for name, score in results.items():
    print(f"{name:<20} {score:.4f}")
```

---

## Exercise 7: Hyperparameter Tuning (GridSearch vs Random vs Optuna)

**Задача:** Затюнь RandomForest тремя способами. Сравни время и качество.

```python
import numpy as np
import time
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split, GridSearchCV, RandomizedSearchCV
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import roc_auc_score
from scipy.stats import randint, uniform

X, y = make_classification(n_samples=2000, n_features=15,
                           weights=[0.7, 0.3], random_state=42)
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.2, stratify=y)

param_grid = {
    'n_estimators': [50, 100, 200],
    'max_depth': [3, 5, 10, None],
    'min_samples_split': [2, 5, 10],
    'min_samples_leaf': [1, 2, 4]
}

# --- GridSearch ---
t0 = time.time()
grid = GridSearchCV(RandomForestClassifier(random_state=42), param_grid,
                    cv=3, scoring='roc_auc', n_jobs=-1)
grid.fit(X_tr, y_tr)
grid_time = time.time() - t0
grid_auc = roc_auc_score(y_te, grid.predict_proba(X_te)[:, 1])

# --- RandomSearch ---
param_dist = {
    'n_estimators': randint(50, 300),
    'max_depth': [3, 5, 10, 15, None],
    'min_samples_split': randint(2, 15),
    'min_samples_leaf': randint(1, 8)
}

t0 = time.time()
rand = RandomizedSearchCV(RandomForestClassifier(random_state=42), param_dist,
                          n_iter=30, cv=3, scoring='roc_auc', n_jobs=-1, random_state=42)
rand.fit(X_tr, y_tr)
rand_time = time.time() - t0
rand_auc = roc_auc_score(y_te, rand.predict_proba(X_te)[:, 1])

# --- Optuna (Bayesian optimization) ---
# pip install optuna
import optuna
optuna.logging.set_verbosity(optuna.logging.WARNING)

def objective(trial):
    params = {
        'n_estimators': trial.suggest_int('n_estimators', 50, 300),
        'max_depth': trial.suggest_int('max_depth', 3, 15),
        'min_samples_split': trial.suggest_int('min_samples_split', 2, 15),
        'min_samples_leaf': trial.suggest_int('min_samples_leaf', 1, 8),
    }
    from sklearn.model_selection import cross_val_score as cvs
    model = RandomForestClassifier(**params, random_state=42)
    scores = cvs(model, X_tr, y_tr, cv=3, scoring='roc_auc', n_jobs=-1)
    return scores.mean()

t0 = time.time()
study = optuna.create_study(direction='maximize')
study.optimize(objective, n_trials=30)
optuna_time = time.time() - t0

# Обучи лучшую модель
best_rf = RandomForestClassifier(**study.best_params, random_state=42)
best_rf.fit(X_tr, y_tr)
optuna_auc = roc_auc_score(y_te, best_rf.predict_proba(X_te)[:, 1])

# === СРАВНЕНИЕ ===
print(f"{'Method':<15} {'AUC':>8} {'Time':>8}")
print("-" * 33)
print(f"{'GridSearch':<15} {grid_auc:>8.4f} {grid_time:>7.1f}s")
print(f"{'RandomSearch':<15} {rand_auc:>8.4f} {rand_time:>7.1f}s")
print(f"{'Optuna':<15} {optuna_auc:>8.4f} {optuna_time:>7.1f}s")
print(f"\nGrid best:   {grid.best_params_}")
print(f"Rand best:   {rand.best_params_}")
print(f"Optuna best: {study.best_params}")

# Optuna использует байесовскую оптимизацию (TPE sampler):
# каждый следующий trial учитывает результаты предыдущих →
# быстрее находит хорошие параметры, чем random search
```
