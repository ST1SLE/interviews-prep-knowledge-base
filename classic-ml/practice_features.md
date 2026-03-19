# Feature Engineering & Selection — Practice Exercises

Feature engineering pipeline и permutation importance.

---

## Exercise 5: Feature Engineering Pipeline

**Задача:** Построй pipeline обработки "грязных" данных, как на реальном проекте.

```python
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.impute import SimpleImputer
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report

# === ГЕНЕРАЦИЯ "ГРЯЗНЫХ" ДАННЫХ ===
np.random.seed(42)
n = 1000
df = pd.DataFrame({
    'age': np.random.normal(35, 10, n),
    'salary': np.random.lognormal(10, 1, n),
    'experience': np.random.randint(0, 20, n),
    'city': np.random.choice(['Moscow', 'SPb', 'Kazan', 'Novosibirsk'], n),
    'education': np.random.choice(['Bachelor', 'Master', 'PhD', None], n, p=[0.5, 0.3, 0.1, 0.1]),
    'target': np.random.binomial(1, 0.3, n)
})

# Добавляем пропуски
mask = np.random.random(n) < 0.1
df.loc[mask, 'age'] = np.nan
mask = np.random.random(n) < 0.15
df.loc[mask, 'salary'] = np.nan

print(f"Missing values:\n{df.isnull().sum()}\n")

# === PIPELINE ===
num_features = ['age', 'salary', 'experience']
cat_features = ['city', 'education']

num_pipeline = Pipeline([
    ('imputer', SimpleImputer(strategy='median')),
    ('scaler', StandardScaler())
])

cat_pipeline = Pipeline([
    ('imputer', SimpleImputer(strategy='constant', fill_value='Unknown')),
    ('encoder', OneHotEncoder(handle_unknown='ignore', sparse_output=False))
])

preprocessor = ColumnTransformer([
    ('num', num_pipeline, num_features),
    ('cat', cat_pipeline, cat_features)
])

# === FEATURE ENGINEERING ===
# Добавляем ручные фичи ПЕРЕД pipeline
df['salary_per_year'] = df['salary'] / (df['experience'] + 1)
df['is_senior'] = (df['experience'] >= 10).astype(int)
df['age_group'] = pd.cut(df['age'], bins=[0, 25, 35, 50, 100],
                         labels=['junior', 'mid', 'senior', 'lead'])

num_features_ext = num_features + ['salary_per_year', 'is_senior']
cat_features_ext = cat_features + ['age_group']

preprocessor_ext = ColumnTransformer([
    ('num', num_pipeline, num_features_ext),
    ('cat', cat_pipeline, cat_features_ext)
])

full_pipeline = Pipeline([
    ('preprocess', preprocessor_ext),
    ('model', RandomForestClassifier(n_estimators=100, random_state=42))
])

# === ОЦЕНКА ===
X = df.drop('target', axis=1)
y = df['target'].values

X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.2, stratify=y, random_state=42)

full_pipeline.fit(X_tr, y_tr)
y_pred = full_pipeline.predict(X_te)
print(classification_report(y_te, y_pred))

# CV
scores = cross_val_score(full_pipeline, X, y, cv=5, scoring='roc_auc')
print(f"CV AUC: {scores.mean():.4f} ± {scores.std():.4f}")
```

---

## Exercise 9: Permutation Importance + Feature Selection

**Задача:** Обучи модель на зашумленных данных. Найди важные фичи, выкинь мусор, сравни.

```python
import numpy as np
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.ensemble import RandomForestClassifier
from sklearn.inspection import permutation_importance

# 5 полезных фич + 15 шумовых
X, y = make_classification(n_samples=1000, n_features=20,
                           n_informative=5, n_redundant=0,
                           n_clusters_per_class=1, random_state=42)
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.2, random_state=42)

# Baseline: все 20 фич
rf_all = RandomForestClassifier(n_estimators=100, random_state=42)
scores_all = cross_val_score(rf_all, X_tr, y_tr, cv=5, scoring='roc_auc')

# Permutation importance
rf_all.fit(X_tr, y_tr)
perm = permutation_importance(rf_all, X_te, y_te, n_repeats=10, random_state=42)

# Отбери фичи с importance > 0
important_mask = perm.importances_mean > 0.005
print(f"Important features: {np.where(important_mask)[0]}")
print(f"Importances: {perm.importances_mean[important_mask].round(4)}")

# Модель на отобранных фичах
X_tr_sel = X_tr[:, important_mask]
X_te_sel = X_te[:, important_mask]
scores_sel = cross_val_score(
    RandomForestClassifier(n_estimators=100, random_state=42),
    X_tr_sel, y_tr, cv=5, scoring='roc_auc')

print(f"\nAll features ({X_tr.shape[1]}):     AUC = {scores_all.mean():.4f} ± {scores_all.std():.4f}")
print(f"Selected ({X_tr_sel.shape[1]}): AUC = {scores_sel.mean():.4f} ± {scores_sel.std():.4f}")
```
