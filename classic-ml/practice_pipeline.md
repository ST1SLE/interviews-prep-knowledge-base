# Full Interview Simulation — Practice Exercise

End-to-end ML pipeline, как на собеседовании.

---

## Exercise 10: Full Interview Simulation — End-to-End

**Задача:** Тебе дали csv с данными. Построй модель, оцени, объясни выбор. Как на собесе.

```python
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split, StratifiedKFold, cross_val_score
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import (roc_auc_score, f1_score, classification_report,
                             roc_curve, precision_recall_curve)
from scipy import stats
import matplotlib.pyplot as plt

# === 1. ДАННЫЕ ===
np.random.seed(42)
n = 2000
data = pd.DataFrame({
    'feature_1': np.random.normal(0, 1, n),
    'feature_2': np.random.normal(0, 1, n),
    'feature_3': np.random.exponential(2, n),
    'feature_4': np.random.normal(0, 1, n),
    'feature_5': np.random.normal(0, 1, n),
    'noise_1': np.random.normal(0, 1, n),
    'noise_2': np.random.normal(0, 1, n),
})
# Таргет зависит от feature_1, feature_2, feature_3
logit = 0.5 * data['feature_1'] + 0.8 * data['feature_2'] - 0.3 * data['feature_3']
data['target'] = (np.random.random(n) < 1 / (1 + np.exp(-logit))).astype(int)

print(f"Shape: {data.shape}")
print(f"Target: {data['target'].value_counts().to_dict()}")
print(f"Missing: {data.isnull().sum().sum()}")

# === 2. SPLIT ===
X = data.drop('target', axis=1)
y = data['target'].values
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.2, stratify=y, random_state=42)

# === 3. BASELINE ===
scaler = StandardScaler()
X_tr_sc = scaler.fit_transform(X_tr)
X_te_sc = scaler.transform(X_te)

models = {
    'LogReg': LogisticRegression(max_iter=1000),
    'RF': RandomForestClassifier(n_estimators=200, random_state=42),
}

# === 4. CV СРАВНЕНИЕ ===
cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
cv_results = {}
for name, model in models.items():
    X_data = X_tr_sc if name == 'LogReg' else X_tr
    scores = cross_val_score(model, X_data, y_tr, cv=cv, scoring='roc_auc')
    cv_results[name] = scores
    ci = stats.t.interval(0.95, len(scores)-1,
                          loc=scores.mean(), scale=scores.std()/np.sqrt(len(scores)))
    print(f"{name}: AUC = {scores.mean():.4f} ± {scores.std():.4f}, 95% CI: [{ci[0]:.4f}, {ci[1]:.4f}]")

# === 5. СТАТИСТИЧЕСКОЕ СРАВНЕНИЕ ===
t_stat, p_val = stats.ttest_rel(cv_results['LogReg'], cv_results['RF'])
print(f"\nLogReg vs RF: t={t_stat:.3f}, p={p_val:.3f}")
print("Значимо" if p_val < 0.05 else "Не значимо")

# === 6. ФИНАЛЬНАЯ МОДЕЛЬ ===
best_name = max(cv_results, key=lambda k: cv_results[k].mean())
best_model = models[best_name]
X_fit = X_tr_sc if best_name == 'LogReg' else X_tr
X_eval = X_te_sc if best_name == 'LogReg' else X_te
best_model.fit(X_fit, y_tr)
proba = best_model.predict_proba(X_eval)[:, 1]
print(f"\nBest model: {best_name}")
print(f"Test AUC: {roc_auc_score(y_te, proba):.4f}")
print(classification_report(y_te, best_model.predict(X_eval)))

# === 7. ROC + PR CURVES ===
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 5))

fpr, tpr, _ = roc_curve(y_te, proba)
ax1.plot(fpr, tpr, label=f'{best_name} (AUC={roc_auc_score(y_te, proba):.3f})')
ax1.plot([0,1], [0,1], 'k--')
ax1.set_xlabel('FPR'); ax1.set_ylabel('TPR'); ax1.set_title('ROC'); ax1.legend()

prec, rec, _ = precision_recall_curve(y_te, proba)
ax2.plot(rec, prec, label=best_name)
ax2.set_xlabel('Recall'); ax2.set_ylabel('Precision'); ax2.set_title('PR Curve'); ax2.legend()

plt.tight_layout()
plt.show()
```

---

## Порядок выполнения

| # | Exercise | Что тренирует | Время |
|---|----------|--------------|-------|
| 1 | LogReg from scratch | Градиенты, sigmoid, threshold | 30 min |
| 2 | K-Means from scratch | Кластеризация, K-Means++ | 30 min |
| 3 | Decision Tree from scratch | Gini, splits, рекурсия | 45 min |
| 4 | CV from scratch | Stratified split, paired t-test | 20 min |
| 5 | Feature Engineering | Pipeline, imputer, encoder | 30 min |
| 6 | Imbalanced Data | class_weight, SMOTE, threshold | 20 min |
| 7 | Hyperparameter Tuning | Grid vs Random vs Optuna | 20 min |
| 8 | Model Debugging | Типичные ошибки, learning curve | 20 min |
| 9 | Feature Selection | Permutation importance | 15 min |
| 10 | Full Simulation | End-to-end pipeline | 40 min |

**Total: ~4.5 hours**
