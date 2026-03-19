# ML from Scratch — Practice Exercises

Реализация LogReg, K-Means, Decision Tree без sklearn.

---

## Exercise 1: LogReg from Scratch + Threshold Tuning

**Задача:** Реализуй логистическую регрессию без sklearn. Затем найди оптимальный threshold для F1.

```python
import numpy as np
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import f1_score, precision_recall_curve

# === РЕАЛИЗАЦИЯ ===
class LogReg:
    def __init__(self, lr=0.1, n_iter=1000):
        self.lr = lr
        self.n_iter = n_iter

    def sigmoid(self, z):
        z = np.clip(z, -500, 500)
        return 1 / (1 + np.exp(-z))

    def fit(self, X, y):
        n, m = X.shape
        self.w = np.zeros(m)
        self.b = 0.0
        for _ in range(self.n_iter):
            p = self.sigmoid(X @ self.w + self.b)
            dw = (1/n) * X.T @ (p - y)
            db = (1/n) * np.sum(p - y)
            self.w -= self.lr * dw
            self.b -= self.lr * db

    def predict_proba(self, X):
        return self.sigmoid(X @ self.w + self.b)

    def predict(self, X, threshold=0.5):
        return (self.predict_proba(X) >= threshold).astype(int)

# === ДАННЫЕ ===
X, y = make_classification(n_samples=1000, n_features=10,
                           weights=[0.7, 0.3], random_state=42)
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.2, stratify=y)

scaler = StandardScaler()
X_tr = scaler.fit_transform(X_tr)
X_te = scaler.transform(X_te)

model = LogReg(lr=0.1, n_iter=1000)
model.fit(X_tr, y_tr)

# === THRESHOLD TUNING ===
proba = model.predict_proba(X_te)
precisions, recalls, thresholds = precision_recall_curve(y_te, proba)

# F1 для каждого threshold
f1s = 2 * precisions * recalls / (precisions + recalls + 1e-8)
best_idx = np.argmax(f1s)
best_threshold = thresholds[best_idx]

print(f"Default F1 (0.5):    {f1_score(y_te, model.predict(X_te)):.4f}")
print(f"Best F1 ({best_threshold:.2f}): {f1s[best_idx]:.4f}")
```

---

## Exercise 2: K-Means from Scratch

**Задача:** Реализуй K-Means без sklearn. Визуализируй кластеры.

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.datasets import make_blobs

class KMeans:
    def __init__(self, k=3, max_iter=100):
        self.k = k
        self.max_iter = max_iter

    def fit(self, X):
        n = X.shape[0]
        # K-Means++ инициализация
        idx = [np.random.randint(n)]
        for _ in range(1, self.k):
            dists = np.min([np.sum((X - X[i])**2, axis=1) for i in idx], axis=0)
            probs = dists / dists.sum()
            idx.append(np.random.choice(n, p=probs))
        self.centroids = X[idx].copy()

        for _ in range(self.max_iter):
            # Assign
            dists = np.array([np.sum((X - c)**2, axis=1) for c in self.centroids])
            self.labels = np.argmin(dists, axis=0)
            # Update
            new_centroids = np.array([X[self.labels == i].mean(axis=0)
                                      for i in range(self.k)])
            if np.allclose(self.centroids, new_centroids):
                break
            self.centroids = new_centroids
        return self

    def predict(self, X):
        dists = np.array([np.sum((X - c)**2, axis=1) for c in self.centroids])
        return np.argmin(dists, axis=0)

# === ТЕСТ ===
X, y_true = make_blobs(n_samples=300, centers=4, cluster_std=0.8, random_state=42)
km = KMeans(k=4).fit(X)

plt.scatter(X[:, 0], X[:, 1], c=km.labels, cmap='viridis', s=20)
plt.scatter(km.centroids[:, 0], km.centroids[:, 1], c='red', marker='x', s=200)
plt.title('K-Means (from scratch)')
plt.show()

# Сравни с sklearn
from sklearn.cluster import KMeans as SKKMeans
from sklearn.metrics import adjusted_rand_score
sk = SKKMeans(n_clusters=4, random_state=42, n_init=10).fit(X)
print(f"ARI (my vs true):     {adjusted_rand_score(y_true, km.labels):.4f}")
print(f"ARI (sklearn vs true): {adjusted_rand_score(y_true, sk.labels_):.4f}")
```

---

## Exercise 3: Decision Tree from Scratch (Classification)

**Задача:** Реализуй дерево решений с Gini impurity. Без sklearn.

```python
import numpy as np
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

class Node:
    def __init__(self, feature=None, threshold=None, left=None, right=None, value=None):
        self.feature = feature
        self.threshold = threshold
        self.left = left
        self.right = right
        self.value = value  # Класс в листе

class DecisionTree:
    def __init__(self, max_depth=5, min_samples=2):
        self.max_depth = max_depth
        self.min_samples = min_samples

    def gini(self, y):
        if len(y) == 0:
            return 0
        probs = np.bincount(y) / len(y)
        return 1 - np.sum(probs ** 2)

    def best_split(self, X, y):
        best_gain = -1
        best_feat, best_thresh = None, None
        parent_gini = self.gini(y)
        n = len(y)

        for feat in range(X.shape[1]):
            thresholds = np.unique(X[:, feat])
            for thresh in thresholds:
                left_mask = X[:, feat] <= thresh
                right_mask = ~left_mask
                if left_mask.sum() < 1 or right_mask.sum() < 1:
                    continue
                gain = parent_gini - (
                    left_mask.sum()/n * self.gini(y[left_mask]) +
                    right_mask.sum()/n * self.gini(y[right_mask])
                )
                if gain > best_gain:
                    best_gain = gain
                    best_feat = feat
                    best_thresh = thresh
        return best_feat, best_thresh, best_gain

    def build(self, X, y, depth=0):
        # Лист: одноклассовый или достигнут лимит
        if len(np.unique(y)) == 1 or depth >= self.max_depth or len(y) < self.min_samples:
            return Node(value=np.bincount(y).argmax())

        feat, thresh, gain = self.best_split(X, y)
        if gain <= 0:
            return Node(value=np.bincount(y).argmax())

        left_mask = X[:, feat] <= thresh
        left = self.build(X[left_mask], y[left_mask], depth + 1)
        right = self.build(X[~left_mask], y[~left_mask], depth + 1)
        return Node(feature=feat, threshold=thresh, left=left, right=right)

    def fit(self, X, y):
        self.root = self.build(X, y)
        return self

    def _predict_one(self, x, node):
        if node.value is not None:
            return node.value
        if x[node.feature] <= node.threshold:
            return self._predict_one(x, node.left)
        return self._predict_one(x, node.right)

    def predict(self, X):
        return np.array([self._predict_one(x, self.root) for x in X])

# === ТЕСТ ===
X, y = make_classification(n_samples=500, n_features=5, random_state=42)
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.2)

tree = DecisionTree(max_depth=5).fit(X_tr, y_tr)
print(f"My tree accuracy:     {accuracy_score(y_te, tree.predict(X_te)):.4f}")

from sklearn.tree import DecisionTreeClassifier
sk_tree = DecisionTreeClassifier(max_depth=5).fit(X_tr, y_tr)
print(f"Sklearn tree accuracy: {accuracy_score(y_te, sk_tree.predict(X_te)):.4f}")
```
