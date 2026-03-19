# Python Coding Interview — Interview Prep

Структуры данных, алгоритмы, ООП, декораторы, генераторы, GIL, live coding для собеседования ML/DS/AI Engineer Intern.

---

## Q1. "Структуры данных в Python — сложность операций"

> "В Python основные структуры — `list`, `dict`, `set`, `tuple`, `deque`. Главное на собесе — знать Big-O для типовых операций и понимать, **когда что использовать**.
>
> `list` — динамический массив. Append в конец $O(1)$ amortized, но insert/delete в начале $O(n)$ из-за сдвига элементов. Поиск по значению $O(n)$, по индексу $O(1)$.
>
> `dict` — hash table. Lookup, insert, delete — всё $O(1)$ average. Worst case $O(n)$ при коллизиях, но CPython использует open addressing с хорошей hash function, поэтому на практике $O(1)$.
>
> `set` — тоже hash table, но только ключи без значений. Те же $O(1)$ для add/remove/lookup. Основная сила — проверка `in` за $O(1)$, тогда как для `list` это $O(n)$.
>
> `deque` (из `collections`) — doubly-linked list. Append/pop с обоих концов $O(1)$. Идеален для очередей и sliding window. Но lookup по индексу $O(n)$.
>
> `tuple` — immutable list. Те же $O(1)$ по индексу, $O(n)$ по значению. Быстрее `list` по памяти и созданию, используется как ключ в `dict`."

| Операция | `list` | `dict` | `set` | `deque` | `tuple` |
|----------|--------|--------|-------|---------|---------|
| Append (конец) | $O(1)$* | — | — | $O(1)$ | — |
| Append (начало) | $O(n)$ | — | — | $O(1)$ | — |
| Lookup (индекс) | $O(1)$ | — | — | $O(n)$ | $O(1)$ |
| Lookup (значение / ключ) | $O(n)$ | $O(1)$ | $O(1)$ | $O(n)$ | $O(n)$ |
| Insert (середина) | $O(n)$ | $O(1)$ | $O(1)$ | $O(n)$ | — |
| Delete (середина) | $O(n)$ | $O(1)$ | $O(1)$ | $O(n)$ | — |
| Iterate | $O(n)$ | $O(n)$ | $O(n)$ | $O(n)$ | $O(n)$ |

\* amortized — при переполнении буфера происходит resize $O(n)$, но в среднем $O(1)$.

**`collections` — must know:**

```python
from collections import defaultdict, Counter, OrderedDict, deque

# defaultdict — dict с дефолтным значением, не кидает KeyError
word_counts = defaultdict(int)
for word in ["cat", "dog", "cat", "bird"]:
    word_counts[word] += 1
# {'cat': 2, 'dog': 1, 'bird': 1}

# Counter — подсчёт элементов, most_common
c = Counter(["cat", "dog", "cat", "bird"])
print(c.most_common(2))  # [('cat', 2), ('dog', 1)]

# OrderedDict — сохраняет порядок вставки (в Python 3.7+ обычный dict тоже, но OrderedDict
# нужен для move_to_end и popitem(last=False) — основа LRU cache)
od = OrderedDict()
od["a"] = 1
od["b"] = 2
od.move_to_end("a")  # a перемещается в конец

# deque — двусторонняя очередь с maxlen
recent = deque(maxlen=3)
for i in range(5):
    recent.append(i)
print(recent)  # deque([2, 3, 4], maxlen=3)
```

**Когда что выбирать:**

| Задача | Структура | Почему |
|--------|-----------|--------|
| Хранение в порядке, доступ по индексу | `list` | $O(1)$ random access |
| Быстрая очередь / BFS | `deque` | $O(1)$ popleft |
| Проверка "есть ли элемент" | `set` | $O(1)$ lookup vs $O(n)$ у list |
| Key-value маппинг | `dict` | $O(1)$ по ключу |
| Immutable ключ / запись | `tuple` | Hashable, меньше памяти |
| Подсчёт частот | `Counter` | Удобнее чем defaultdict(int) |

---

## Q2. "Алгоритмы сортировки и поиска"

> "На intern-собесе спрашивают 3 базовых паттерна: **binary search**, **two pointers**, **sliding window**. Они покрывают ~70% задач на массивах. Главное — распознать паттерн в задаче."

**Binary Search — поиск в отсортированном массиве:**

> "Binary search ищет элемент в отсортированном массиве за $O(\log n)$. В Python есть модуль `bisect` — не пишем руками, если не просят. `bisect_left` возвращает позицию для вставки (первое вхождение), `bisect_right` — после последнего."

```python
import bisect

# Найти позицию для вставки
arr = [1, 3, 5, 7, 9, 11]
pos = bisect.bisect_left(arr, 7)   # 3 (индекс элемента 7)
pos_r = bisect.bisect_right(arr, 7) # 4 (после элемента 7)

# Ручной binary search (если просят на собесе)
def binary_search(arr, target):
    lo, hi = 0, len(arr) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return -1

print(binary_search(arr, 7))   # 3
print(binary_search(arr, 6))   # -1
```

**Two Pointers — два указателя на отсортированном массиве:**

> "Паттерн two pointers: два указателя двигаются навстречу (или в одном направлении). Классика — найти пару с заданной суммой в отсортированном массиве за $O(n)$ вместо $O(n^2)$."

```python
def two_sum_sorted(arr, target):
    """Найти пару с суммой target в отсортированном массиве."""
    left, right = 0, len(arr) - 1
    while left < right:
        s = arr[left] + arr[right]
        if s == target:
            return [left, right]
        elif s < target:
            left += 1    # сумма мала — двигаем левый вправо
        else:
            right -= 1   # сумма велика — двигаем правый влево
    return []

arr = [1, 3, 5, 7, 9, 11]
print(two_sum_sorted(arr, 12))  # [2, 3] → 5 + 7 = 12
```

**Sliding Window — максимальная сумма подмассива фиксированной длины:**

> "Sliding window — фиксированное или переменное 'окно' по массиву. Обновляем сумму за $O(1)$: добавляем новый элемент, убираем старый. Итого $O(n)$ вместо $O(n \cdot k)$."

```python
def max_sum_subarray(arr, k):
    """Максимальная сумма подмассива длины k."""
    if len(arr) < k:
        return None
    # Начальная сумма первого окна
    window_sum = sum(arr[:k])
    max_sum = window_sum
    # Двигаем окно
    for i in range(k, len(arr)):
        window_sum += arr[i] - arr[i - k]  # O(1) обновление
        max_sum = max(max_sum, window_sum)
    return max_sum

arr = [2, 1, 5, 1, 3, 2]
print(max_sum_subarray(arr, 3))  # 9 → [5, 1, 3]
```

| Паттерн | Сложность | Когда использовать |
|---------|-----------|-------------------|
| Binary Search | $O(\log n)$ | Отсортированный массив, поиск границы |
| Two Pointers | $O(n)$ | Пара/тройка с суммой, палиндром, слияние |
| Sliding Window | $O(n)$ | Подмассив фиксированной/переменной длины |

---

## Q3. "ООП в Python"

> "Python — мультипарадигменный, но ООП используется повсюду: scikit-learn (BaseEstimator, fit/predict), PyTorch (nn.Module), FastAPI (Pydantic models). На собесе спрашивают: `__init__`, `__repr__`, `@property`, ABC, dataclasses."

**Базовый класс с dunder-методами:**

```python
class MLModel:
    """Базовый класс ML-модели."""

    def __init__(self, name: str, version: str = "1.0"):
        self.name = name
        self.version = version
        self._metrics = {}  # 'приватный' атрибут

    def __repr__(self) -> str:
        """Для разработчиков — однозначное представление."""
        return f"MLModel(name={self.name!r}, version={self.version!r})"

    def __str__(self) -> str:
        """Для пользователей — читаемое представление."""
        return f"{self.name} v{self.version}"

    @property
    def metrics(self) -> dict:
        """Геттер через @property — контролируем доступ."""
        return self._metrics.copy()  # возвращаем копию

    @metrics.setter
    def metrics(self, value: dict):
        """Сеттер — валидация при записи."""
        if not isinstance(value, dict):
            raise TypeError("metrics must be a dict")
        self._metrics = value

    @classmethod
    def from_config(cls, config: dict) -> "MLModel":
        """Альтернативный конструктор — создаём из dict."""
        return cls(name=config["name"], version=config.get("version", "1.0"))

    @staticmethod
    def validate_metric(value: float) -> bool:
        """Утилита, не зависит от экземпляра и класса."""
        return 0.0 <= value <= 1.0


model = MLModel("CatBoost_credit", "2.1")
print(repr(model))  # MLModel(name='CatBoost_credit', version='2.1')
print(str(model))   # CatBoost_credit v2.1

config = {"name": "SentenceBERT_matching", "version": "1.0"}
model2 = MLModel.from_config(config)
```

**ABC — абстрактные классы:**

```python
from abc import ABC, abstractmethod

class BasePredictor(ABC):
    """Абстрактный базовый класс — нельзя создать экземпляр."""

    @abstractmethod
    def fit(self, X, y):
        pass

    @abstractmethod
    def predict(self, X):
        pass

    def fit_predict(self, X, y):
        """Общая логика — наследуется."""
        self.fit(X, y)
        return self.predict(X)


class CatBoostScorer(BasePredictor):
    """Конкретная реализация — как в Alfa-Bank проекте."""

    def fit(self, X, y):
        print("Fitting CatBoost on credit data...")
        self.model = "trained"

    def predict(self, X):
        return [0.7, 0.3, 0.9]  # предсказания дефолтов


# BasePredictor()  # TypeError: Can't instantiate abstract class
scorer = CatBoostScorer()
scorer.fit_predict(None, None)  # Fitting CatBoost on credit data...
```

**`dataclasses` — для data-контейнеров:**

```python
from dataclasses import dataclass, field

@dataclass
class Experiment:
    """Запись эксперимента — автоматические __init__, __repr__, __eq__."""
    name: str
    model: str
    dataset: str
    metrics: dict = field(default_factory=dict)
    tags: list = field(default_factory=list)

    @property
    def f1(self) -> float:
        return self.metrics.get("f1", 0.0)


exp = Experiment(
    name="credit_scoring_v3",
    model="CatBoost",
    dataset="alfa_bank_2024",
    metrics={"f1": 0.87, "auc": 0.93},
)
print(exp)
# Experiment(name='credit_scoring_v3', model='CatBoost', ...)
print(exp.f1)  # 0.87
```

| Механизм | Когда использовать |
|----------|-------------------|
| `__repr__` / `__str__` | Любой класс — для отладки и логирования |
| `@property` | Контролируемый доступ, вычисляемые атрибуты |
| `@classmethod` | Альтернативные конструкторы (`from_config`, `from_json`) |
| `@staticmethod` | Утилиты без доступа к `self`/`cls` |
| `ABC` + `@abstractmethod` | Интерфейс/контракт — наследники обязаны реализовать |
| `@dataclass` | Data-контейнеры — заменяет boilerplate `__init__`/`__repr__` |

---

## Q4. "Декораторы — как работают?"

> "Декоратор — это функция, которая принимает функцию и возвращает новую функцию. Синтаксис `@decorator` — это просто syntactic sugar для `func = decorator(func)`. Под капотом — **замыкание** (closure): внутренняя функция 'запоминает' переменные из внешней.
>
> Важно: без `functools.wraps` декоратор затирает `__name__` и `__doc__` оригинальной функции. `@wraps` копирует метаданные."

**Шаг 1 — замыкание (closure):**

```python
def make_multiplier(factor):
    """Внешняя функция создаёт замыкание."""
    def multiply(x):
        return x * factor  # factor 'запомнен' из make_multiplier
    return multiply

double = make_multiplier(2)
print(double(5))   # 10
print(double(10))  # 20
```

**Шаг 2 — простой декоратор `@timer`:**

```python
import time
import functools

def timer(func):
    """Замеряет время выполнения функции."""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} took {elapsed:.4f}s")
        return result
    return wrapper

@timer
def train_model(epochs):
    """Тренировка модели."""
    time.sleep(0.1 * epochs)
    return "trained"

train_model(3)  # train_model took 0.3001s
print(train_model.__name__)  # train_model (благодаря @wraps)
```

**Шаг 3 — декоратор с аргументами `@retry(n=3)` (двойная обёртка):**

```python
import functools
import time

def retry(n=3, delay=1.0):
    """Декоратор с аргументами — три уровня вложенности."""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, n + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    print(f"Attempt {attempt}/{n} failed: {e}")
                    if attempt < n:
                        time.sleep(delay)
            raise RuntimeError(f"{func.__name__} failed after {n} attempts")
        return wrapper
    return decorator

@retry(n=3, delay=0.5)
def fetch_data(url):
    """Загрузка данных с API — может упасть."""
    import random
    if random.random() < 0.7:
        raise ConnectionError("Server unavailable")
    return {"data": [1, 2, 3]}

# fetch_data("http://api.example.com")
# Attempt 1/3 failed: Server unavailable
# Attempt 2/3 failed: Server unavailable
# {'data': [1, 2, 3]}  — если повезёт на 3-м
```

**Шаг 4 — `@lru_cache` (встроенный декоратор-кэш):**

```python
from functools import lru_cache

@lru_cache(maxsize=128)
def fibonacci(n):
    """Рекурсивный Fibonacci с мемоизацией — O(n) вместо O(2^n)."""
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

print(fibonacci(50))  # 12586269025 — мгновенно
print(fibonacci.cache_info())
# CacheInfo(hits=48, misses=51, maxsize=128, currsize=51)
```

| Паттерн | Уровни вложенности | Пример |
|---------|-------------------|--------|
| Простой декоратор | 2: `decorator(func) → wrapper` | `@timer` |
| С аргументами | 3: `retry(n) → decorator(func) → wrapper` | `@retry(n=3)` |
| Класс-декоратор | `__init__` + `__call__` | Stateful decorators |

---

## Q5. "Генераторы и итераторы"

> "Генератор — функция с `yield` вместо `return`. При вызове возвращает **iterator**, который лениво выдаёт значения по одному. Главное преимущество — **memory efficiency**: не создаём весь список в памяти.
>
> `yield` 'замораживает' состояние функции: локальные переменные, позиция выполнения. При вызове `next()` выполнение продолжается с того же места.
>
> В реальных проектах: Celery tasks в Text2Brainrot обрабатывают данные потоково — не грузим весь контент в память."

**`yield` vs `return`:**

```python
def get_squares_list(n):
    """Обычная функция — создаёт весь список в памяти."""
    result = []
    for i in range(n):
        result.append(i ** 2)
    return result

def get_squares_gen(n):
    """Генератор — выдаёт по одному значению."""
    for i in range(n):
        yield i ** 2

# list — весь результат сразу
squares = get_squares_list(5)  # [0, 1, 4, 9, 16]

# generator — ленивый итератор
gen = get_squares_gen(5)
print(next(gen))  # 0
print(next(gen))  # 1
print(next(gen))  # 4
# StopIteration при исчерпании
```

**Memory: `range` vs `list(range)` — классический вопрос:**

```python
import sys

# range — ленивый объект, хранит только start/stop/step
r = range(10**9)
print(sys.getsizeof(r))           # 48 байт (!)

# list — все элементы в памяти
# lst = list(range(10**9))        # ~8 GB RAM — убьёт процесс
lst_small = list(range(10**6))
print(sys.getsizeof(lst_small))   # ~8 MB
```

**Generator expression vs list comprehension:**

```python
import sys

# List comprehension — создаёт список в памяти
squares_list = [x**2 for x in range(10**6)]
print(sys.getsizeof(squares_list))  # ~8 MB

# Generator expression — ленивый, почти 0 памяти
squares_gen = (x**2 for x in range(10**6))
print(sys.getsizeof(squares_gen))   # ~200 байт

# Генератор можно пройти только один раз!
total = sum(squares_gen)  # потребляет генератор
total2 = sum(squares_gen) # 0 — генератор исчерпан
```

**`itertools` — must know:**

```python
from itertools import chain, islice, groupby

# chain — склеить несколько итераторов
batch1 = [1, 2, 3]
batch2 = [4, 5, 6]
for item in chain(batch1, batch2):
    print(item, end=" ")  # 1 2 3 4 5 6

# islice — срез для итераторов (без создания списка)
gen = (x**2 for x in range(100))
first_five = list(islice(gen, 5))  # [0, 1, 4, 9, 16]

# groupby — группировка последовательных элементов (данные должны быть отсортированы!)
data = [
    {"model": "catboost", "metric": 0.85},
    {"model": "catboost", "metric": 0.87},
    {"model": "logreg", "metric": 0.72},
    {"model": "logreg", "metric": 0.74},
]
for model, experiments in groupby(data, key=lambda x: x["model"]):
    scores = [e["metric"] for e in experiments]
    print(f"{model}: {scores}")
# catboost: [0.85, 0.87]
# logreg: [0.72, 0.74]
```

| Конструкция | Память | Повторный проход | Когда использовать |
|-------------|--------|------------------|--------------------|
| `list(range(n))` | $O(n)$ | Да | Нужен random access / многократный проход |
| `range(n)` | $O(1)$ | Да | Итерация по числам |
| `[x for x in ...]` | $O(n)$ | Да | Результат нужен как список |
| `(x for x in ...)` | $O(1)$ | Нет | Одноразовая потоковая обработка |

---

## Q6. "GIL — что это и как обходить?"

> "**GIL** (Global Interpreter Lock) — мьютекс в CPython, который разрешает только одному потоку исполнять Python-байткод в любой момент времени. Даже на 8-ядерном CPU `threading` не даёт параллелизма для CPU-bound задач — потоки по очереди захватывают GIL.
>
> Почему GIL существует: CPython использует reference counting для garbage collection. Без GIL каждое `Py_INCREF/Py_DECREF` нужно было бы защищать отдельным lock-ом — это дороже.
>
> Как обходить:
> - **I/O-bound** (сеть, диск, БД) → `threading` или `asyncio` — GIL отпускается во время I/O
> - **CPU-bound** (тяжёлые вычисления) → `multiprocessing` — каждый процесс со своим GIL
> - **NumPy/CatBoost/PyTorch** — C-расширения отпускают GIL при вычислениях"

| Тип задачи | Подход | Почему | Пример |
|------------|--------|--------|--------|
| **I/O-bound** (запросы к API) | `threading` | GIL отпускается при I/O | Скачивание данных для pipeline |
| **I/O-bound** (много соединений) | `asyncio` | Event loop, один поток | FastAPI endpoints в Text2Brainrot |
| **CPU-bound** (вычисления) | `multiprocessing` | Отдельные процессы, каждый со своим GIL | Параллельная обработка данных |
| **CPU-bound** (NumPy/ML) | C-расширения | NumPy/CatBoost отпускают GIL | CatBoost `.fit()` в Alfa-Bank |

```python
# === threading — для I/O-bound задач ===
import threading
import time

def download(url):
    """Имитация загрузки данных."""
    print(f"Downloading {url}...")
    time.sleep(1)  # I/O wait — GIL отпускается
    print(f"Done {url}")

urls = ["http://api1.com", "http://api2.com", "http://api3.com"]

# Последовательно: ~3 сек
start = time.perf_counter()
for url in urls:
    download(url)
print(f"Sequential: {time.perf_counter() - start:.1f}s")

# Параллельно: ~1 сек
start = time.perf_counter()
threads = [threading.Thread(target=download, args=(url,)) for url in urls]
for t in threads:
    t.start()
for t in threads:
    t.join()
print(f"Threaded: {time.perf_counter() - start:.1f}s")
```

```python
# === multiprocessing — для CPU-bound задач ===
from multiprocessing import Pool
import time

def heavy_computation(n):
    """CPU-bound: считаем сумму квадратов."""
    return sum(i * i for i in range(n))

numbers = [10**7] * 4

# Последовательно
start = time.perf_counter()
results = [heavy_computation(n) for n in numbers]
print(f"Sequential: {time.perf_counter() - start:.1f}s")

# Параллельно — 4 процесса
start = time.perf_counter()
with Pool(4) as pool:
    results = pool.map(heavy_computation, numbers)
print(f"Multiprocessing: {time.perf_counter() - start:.1f}s")
```

```python
# === asyncio — для network I/O (как в FastAPI) ===
import asyncio

async def fetch(url):
    """Асинхронная загрузка — не блокирует event loop."""
    print(f"Fetching {url}...")
    await asyncio.sleep(1)  # имитация сетевого запроса
    return f"Data from {url}"

async def main():
    urls = ["http://api1.com", "http://api2.com", "http://api3.com"]
    # gather — запускает все корутины конкурентно
    results = await asyncio.gather(*[fetch(url) for url in urls])
    print(results)  # ~1 сек вместо ~3

# asyncio.run(main())
```

---

## Q7. "Типичные задачи на live coding"

> "На intern-собесе обычно дают 1-2 задачи уровня LeetCode Easy/Medium. Самые частые: Two Sum, Valid Parentheses, Merge Intervals, Binary Search. Главное — объяснять ход мысли, начать с brute force, потом оптимизировать."

**Задача 1: Two Sum (dict, $O(n)$)**

> Дан массив чисел и target. Верни индексы двух чисел, дающих target.

```python
def two_sum(nums, target):
    """
    Two Sum — через hash map (dict) за O(n).
    Идея: для каждого числа проверяем, есть ли complement = target - num.
    """
    seen = {}  # значение → индекс
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:       # O(1) lookup в dict
            return [seen[complement], i]
        seen[num] = i
    return []

print(two_sum([2, 7, 11, 15], 9))   # [0, 1] → 2 + 7 = 9
print(two_sum([3, 2, 4], 6))        # [1, 2] → 2 + 4 = 6
# Brute force O(n^2) → оптимизация через dict O(n)
```

**Задача 2: Valid Parentheses (stack, $O(n)$)**

> Строка из скобок `()[]{}`. Проверь, правильно ли расставлены.

```python
def is_valid(s):
    """
    Valid Parentheses — через stack.
    Идея: открывающая → в stack, закрывающая → проверяем top.
    """
    stack = []
    pairs = {")": "(", "]": "[", "}": "{"}

    for char in s:
        if char in pairs:
            # Закрывающая скобка — проверяем stack
            if not stack or stack[-1] != pairs[char]:
                return False
            stack.pop()
        else:
            # Открывающая скобка — в stack
            stack.append(char)

    return len(stack) == 0  # stack должен быть пуст

print(is_valid("()[]{}"))   # True
print(is_valid("([)]"))     # False
print(is_valid("{[]}"))     # True
print(is_valid("("))        # False
```

**Задача 3: Merge Intervals (sort + iterate, $O(n \log n)$)**

> Дан список интервалов. Слей пересекающиеся.

```python
def merge_intervals(intervals):
    """
    Merge Intervals — сортируем по началу, потом сливаем.
    Идея: если текущий интервал пересекается с последним в результате — расширяем.
    """
    if not intervals:
        return []

    intervals.sort(key=lambda x: x[0])  # O(n log n)
    merged = [intervals[0]]

    for start, end in intervals[1:]:
        if start <= merged[-1][1]:
            # Пересечение — расширяем конец
            merged[-1][1] = max(merged[-1][1], end)
        else:
            # Нет пересечения — добавляем новый
            merged.append([start, end])

    return merged

print(merge_intervals([[1,3],[2,6],[8,10],[15,18]]))
# [[1, 6], [8, 10], [15, 18]]
print(merge_intervals([[1,4],[4,5]]))
# [[1, 5]]
```

**Задача 4: Binary Search (bisect / ручной, $O(\log n)$)**

> Найди target в отсортированном массиве. Верни индекс или -1.

```python
import bisect

def binary_search_manual(nums, target):
    """Ручной binary search — O(log n)."""
    lo, hi = 0, len(nums) - 1
    while lo <= hi:
        mid = lo + (hi - lo) // 2  # защита от overflow (в Python не нужно, но хорошая привычка)
        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return -1

def binary_search_bisect(nums, target):
    """Через bisect — Pythonic way."""
    idx = bisect.bisect_left(nums, target)
    if idx < len(nums) and nums[idx] == target:
        return idx
    return -1

nums = [1, 3, 5, 7, 9, 11, 13]
print(binary_search_manual(nums, 7))   # 3
print(binary_search_bisect(nums, 7))   # 3
print(binary_search_manual(nums, 8))   # -1
print(binary_search_bisect(nums, 8))   # -1
```

| Задача | Паттерн | Сложность | Ключевая идея |
|--------|---------|-----------|---------------|
| Two Sum | Hash map (dict) | $O(n)$ | `complement = target - num` |
| Valid Parentheses | Stack | $O(n)$ | Открывающая → push, закрывающая → check top |
| Merge Intervals | Sort + linear scan | $O(n \log n)$ | Сортировка по началу, расширяем конец |
| Binary Search | Divide & conquer | $O(\log n)$ | `lo`, `hi`, `mid` — сужаем диапазон |

---

## Q8. "Memory Management и Garbage Collection"

> "Python использует два механизма управления памятью:
>
> **Reference counting** — каждый объект имеет счётчик ссылок. Когда счётчик = 0, память освобождается сразу. `sys.getrefcount(obj)` показывает текущий счётчик (результат всегда на 1 больше реального — сама функция создаёт ссылку).
>
> **Циклический сборщик мусора** (generational GC) — решает проблему циклических ссылок (A → B → A, refcount никогда не станет 0). Три поколения: gen0 (новые объекты, часто проверяются), gen1, gen2 (старые, редко). Модуль `gc`: `gc.collect()`, `gc.get_referrers()`, `gc.disable()`.
>
> **`__del__`** (финализатор) — вызывается при уничтожении объекта. Подводные камни: порядок вызова не гарантирован, может воскресить объект (создать новую ссылку), тормозит GC при циклических ссылках. Лучше использовать context managers (`with`) или `weakref.finalize()`.
>
> Зачем знать: утечки памяти при обработке больших датасетов. Пример: замыкание в callback-е хранит ссылку на DataFrame → GC не может собрать."

```python
import sys
import gc

# Reference counting
a = [1, 2, 3]
print(sys.getrefcount(a))  # 2 (a + аргумент функции)
b = a                       # ещё одна ссылка
print(sys.getrefcount(a))  # 3
del b
print(sys.getrefcount(a))  # 2

# Циклическая ссылка — refcount не спасёт
class Node:
    def __init__(self):
        self.ref = None

a = Node()
b = Node()
a.ref = b  # a → b
b.ref = a  # b → a (цикл!)
del a, b   # refcount каждого = 1 (не 0!) → только GC соберёт

gc.collect()  # принудительный сбор мусора
print(gc.get_stats())  # статистика по поколениям
```

---

## Q9. "Mutable vs Immutable, copy vs deepcopy"

> "**Immutable**: `int`, `float`, `str`, `tuple`, `frozenset`, `bytes` — нельзя изменить после создания. При 'изменении' создаётся новый объект.
>
> **Mutable**: `list`, `dict`, `set`, `bytearray` — можно изменить in-place.
>
> Ловушка: mutable внутри immutable. `tuple` с `list` внутри — `tuple` нельзя заменить элемент, но `list` внутри можно мутировать!
>
> **`copy.copy()`** (shallow) — копирует объект, но вложенные объекты остаются ссылками. Изменение вложенного объекта в копии затронет оригинал.
>
> **`copy.deepcopy()`** — рекурсивно копирует всё. Безопасно, но дороже по памяти и времени.
>
> Ловушка при аугментации данных: мутация вложенных объектов в списке конфигов → все конфиги указывают на один и тот же dict."

```python
import copy

# Mutable внутри immutable
t = ([1, 2], [3, 4])
# t[0] = [5, 6]   # TypeError: tuple не поддерживает присваивание
t[0].append(99)    # Но list внутри можно мутировать!
print(t)           # ([1, 2, 99], [3, 4])

# shallow copy vs deepcopy
original = {"params": [1, 2, 3], "name": "model_v1"}

shallow = copy.copy(original)
shallow["params"].append(4)       # мутация вложенного list
print(original["params"])         # [1, 2, 3, 4] — оригинал тоже изменился!

deep = copy.deepcopy(original)
deep["params"].append(5)          # мутация копии
print(original["params"])         # [1, 2, 3, 4] — оригинал не тронут

# Ловушка: default mutable argument
def add_item(item, lst=[]):   # ПЛОХО: lst общий для всех вызовов
    lst.append(item)
    return lst

def add_item_safe(item, lst=None):  # ХОРОШО
    if lst is None:
        lst = []
    lst.append(item)
    return lst
```

---

## Q10. "NumPy — почему быстрее списков и что такое broadcasting"

> "NumPy массивы в 10-100x быстрее Python списков по трём причинам:
>
> 1. **Contiguous memory**: элементы хранятся последовательно в памяти (как C-массив). Python `list` хранит указатели на разбросанные объекты → cache misses.
> 2. **C-API + SIMD**: операции реализованы на C/Fortran с векторизацией (SIMD инструкции процессора). Один вызов NumPy заменяет миллион Python-операций.
> 3. **Нет boxing/unboxing**: элементы — сырые числа (int64, float64), а не Python-объекты с refcount/type.
>
> **Strides** — сколько байт пропустить для перехода к следующему элементу по каждой оси. Почему `transpose` за $O(1)$: не копирует данные, а меняет strides.
>
> **Broadcasting** — автоматическое расширение размерностей при операциях над массивами разной формы. Правила:
> 1. Размерности сравниваются справа налево.
> 2. Размерности совместимы, если равны или одна из них = 1.
> 3. Размерность 1 'растягивается' до нужного размера."

```python
import numpy as np
import time

# === Скорость: NumPy vs list ===
n = 10**6
lst = list(range(n))
arr = np.arange(n)

start = time.perf_counter()
result_list = [x * 2 for x in lst]      # Python loop
print(f"List: {time.perf_counter() - start:.4f}s")

start = time.perf_counter()
result_numpy = arr * 2                    # Vectorized
print(f"NumPy: {time.perf_counter() - start:.4f}s")  # ~50-100x быстрее

# === Strides: transpose за O(1) ===
a = np.array([[1, 2, 3], [4, 5, 6]])
print(a.strides)    # (24, 8) — 24 байт до следующей строки, 8 до следующего элемента
b = a.T
print(b.strides)    # (8, 24) — просто поменяли strides, данные не копировались

# === Broadcasting ===
matrix = np.array([[1, 2, 3], [4, 5, 6]])  # shape (2, 3)
row = np.array([10, 20, 30])                 # shape (3,) → broadcast до (2, 3)
print(matrix + row)
# [[11, 22, 33],
#  [14, 25, 36]]

col = np.array([[100], [200]])               # shape (2, 1) → broadcast до (2, 3)
print(matrix + col)
# [[101, 102, 103],
#  [204, 205, 206]]
```

| Аспект | Python list | NumPy array |
|--------|------------|-------------|
| Память | Указатели на объекты (разбросаны) | Contiguous block (последовательно) |
| Тип элементов | Любой (гетерогенный) | Один тип (homogeneous) |
| Скорость | Python loop ($O(n)$ интерпретатор) | C/SIMD vectorized |
| Операции | Поэлементный loop | Broadcasting, vectorization |

---

## Q11. "`__slots__` и дескрипторы"

> "`__slots__` — ограничивает атрибуты экземпляра фиксированным набором. Блокирует создание `__dict__` → экономия RAM при миллионах объектов (узлы графа, элементы дерева).
>
> Без `__slots__`: каждый экземпляр хранит `__dict__` (~100-200 байт). С `__slots__`: только значения атрибутов.
>
> **Протокол дескрипторов** (`__get__`, `__set__`, `__delete__`) — механизм, через который работают `@property`, `classmethod`, `staticmethod`. Дескриптор — объект, определяющий хотя бы один из этих методов. При доступе к атрибуту Python вызывает дескриптор вместо обычного lookup.
>
> Связь с PyTorch: `nn.Module` использует `__setattr__` и дескрипторы для автоматической регистрации `nn.Parameter` и `nn.Module` — поэтому `self.layer = nn.Linear(10, 5)` автоматически попадает в `model.parameters()`."

```python
import sys

# === __slots__: экономия памяти ===
class NodeWithDict:
    def __init__(self, value):
        self.value = value
        self.left = None
        self.right = None

class NodeWithSlots:
    __slots__ = ('value', 'left', 'right')
    def __init__(self, value):
        self.value = value
        self.left = None
        self.right = None

n1 = NodeWithDict(42)
n2 = NodeWithSlots(42)
print(sys.getsizeof(n1) + sys.getsizeof(n1.__dict__))  # ~200 байт
print(sys.getsizeof(n2))                                  # ~64 байт

# n2.color = "red"  # AttributeError: __slots__ не позволяет

# === Дескриптор: как работает @property ===
class Percentage:
    """Дескриптор — валидирует значение при присваивании."""
    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, obj, objtype=None):
        return getattr(obj, f'_{self.name}', 0)

    def __set__(self, obj, value):
        if not 0 <= value <= 100:
            raise ValueError(f"{self.name} must be 0-100, got {value}")
        setattr(obj, f'_{self.name}', value)

class ModelMetrics:
    accuracy = Percentage()
    recall = Percentage()

m = ModelMetrics()
m.accuracy = 95    # вызывает Percentage.__set__
# m.accuracy = 150 # ValueError: accuracy must be 0-100, got 150
```

---

## Q12. "Типизация (typing) и статический анализ"

> "Модуль `typing` добавляет аннотации типов в Python. Не влияет на runtime (Python остаётся динамическим), но:
>
> 1. **Документирование API** — читаешь сигнатуру и сразу понимаешь, что на вход и на выход.
> 2. **Автокомплит в IDE** — PyCharm/VSCode понимают типы → лучшие подсказки.
> 3. **Раннее обнаружение ошибок** — mypy/pyright находят баги до запуска.
>
> Ключевые типы: `Optional[X]` = `X | None`, `Union[X, Y]`, `Callable[[Args], Return]`, `TypeVar` (generic), `Generic[T]`.
>
> Зачем в ML-проектах: API FastAPI автоматически валидирует типы через Pydantic. CatBoost `.predict()` возвращает `np.ndarray` — зная это, IDE даёт правильный автокомплит."

```python
from typing import Optional, Union, Callable, TypeVar, Generic
from dataclasses import dataclass
import numpy as np

T = TypeVar('T')

def find_best_threshold(
    y_true: np.ndarray,
    y_proba: np.ndarray,
    metric: Callable[[np.ndarray, np.ndarray], float],
    thresholds: Optional[list[float]] = None,
) -> tuple[float, float]:
    """Найти лучший threshold по заданной метрике."""
    if thresholds is None:
        thresholds = [i / 100 for i in range(1, 100)]
    best_t, best_score = 0.5, 0.0
    for t in thresholds:
        preds = (y_proba >= t).astype(int)
        score = metric(y_true, preds)
        if score > best_score:
            best_t, best_score = t, score
    return best_t, best_score

@dataclass
class ModelResult(Generic[T]):
    """Результат модели с параметризованным типом."""
    prediction: T
    confidence: float
    model_name: str

result_clf: ModelResult[int] = ModelResult(prediction=1, confidence=0.95, model_name="catboost")
result_reg: ModelResult[float] = ModelResult(prediction=3.14, confidence=0.87, model_name="linear")
```

---

## Приоритет для intern-level

| Тема | Приоритет | Почему |
|------|-----------|--------|
| Структуры данных + сложность (Q1) | **Высокий** | Основа любого coding interview |
| List comprehension, generators (Q5) | **Высокий** | Pythonic code |
| NumPy, broadcasting (Q10) | **Высокий** | DS/ML — работа с массивами каждый день |
| Memory management (Q8) | Средний | Утечки памяти при больших данных |
| Mutable/Immutable, copy (Q9) | Средний | Частый источник багов |
| ООП, декораторы (Q3, Q4) | Средний | Зависит от компании |
| Типизация (Q12) | Средний | Современный Python, FastAPI |
| `__slots__`, дескрипторы (Q11) | Низкий | Для продвинутых позиций |
| GIL, asyncio (Q6) | Низкий | Для senior-позиций |
