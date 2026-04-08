# Knowledge Base — Tech Debt

Трекер пустых/неготовых материалов в KB. Обновлено: 2026-04-08.

---

## Общая статистика

| Метрика | Значение |
|---------|----------|
| Всего файлов | 26 |
| Готово (полный контент) | 26 |
| Скелеты (только TODO) | 0 |
| Готовых вопросов/упражнений | 129 Q + 10 Ex |
| Пустых вопросов | **0** |
| Покрытие | **100%** |

---

## Долг по файлам

### 1. `sql/fundamentals.md` — 10 вопросов

| Q | Тема | Приоритет | Объём | Статус |
|---|------|-----------|-------|--------|
| Q1 | SELECT + JOINs (INNER, LEFT, RIGHT, FULL, CROSS) | **HIGH** | medium | DONE |
| Q2 | GROUP BY + HAVING + агрегатные функции | **HIGH** | medium | DONE |
| Q3 | Window functions (ROW_NUMBER, RANK, LAG, LEAD) | **HIGH** | medium | DONE |
| Q4 | CTE (Common Table Expressions) | MED | short | DONE |
| Q5 | Подзапросы — correlated vs non-correlated | MED | short | DONE |
| Q6 | Query optimization (EXPLAIN, индексы) | LOW | medium | DONE |
| Q7 | Типичные задачи (retention, running total, top-N) | LOW | medium | DONE |
| Q8 | Gaps and Islands — поиск непрерывных периодов | **HIGH** | medium | DONE |
| Q9 | Conditional aggregation, Pivot, Anti-Join | MED | medium | DONE |
| Q10 | ClickHouse vs PostgreSQL — когда что | MED | medium | DONE |

### 2. `python/coding_interview.md` — 12 вопросов

| Q | Тема | Приоритет | Объём | Статус |
|---|------|-----------|-------|--------|
| Q1 | Структуры данных — сложность операций (list, dict, set, deque) | **HIGH** | medium | DONE |
| Q2 | Алгоритмы (binary search, two pointers, sliding window) | **HIGH** | medium | DONE |
| Q3 | ООП (классы, наследование, dataclasses, абстрактные) | MED | medium | DONE |
| Q4 | Декораторы (замыкания, functools.wraps, timing/caching) | MED | short | DONE |
| Q5 | Генераторы и итераторы (yield, lazy evaluation) | **HIGH** | short | DONE |
| Q6 | GIL (threading vs multiprocessing vs asyncio) | LOW | short | DONE |
| Q7 | Live coding задачи (two sum, parentheses, merge intervals) | LOW | medium | DONE |
| Q8 | Memory Management и Garbage Collection | MED | medium | DONE |
| Q9 | Mutable vs Immutable, copy vs deepcopy | MED | medium | DONE |
| Q10 | NumPy — broadcasting, strides, vectorization | **HIGH** | medium | DONE |
| Q11 | `__slots__` и дескрипторы | LOW | medium | DONE |
| Q12 | Типизация (typing) и статический анализ | MED | medium | DONE |

### 3. `linear-algebra/core_concepts.md` — 6 вопросов

| Q | Тема | Приоритет | Объём | Статус |
|---|------|-----------|-------|--------|
| Q1 | Векторы (скалярное произведение, нормы, косинусное сходство) | **HIGH** | medium | DONE |
| Q2 | Матрицы (умножение, обратная, определитель, ранг) | **HIGH** | medium | DONE |
| Q3 | Собственные значения и собственные векторы | MED | medium | DONE |
| Q4 | SVD (A = UΣVᵀ, low-rank approximation) | MED | medium | DONE |
| Q5 | PCA через линалг (ковариационная матрица → собственные векторы) | **HIGH** | medium | DONE |
| Q6 | Нормы в ML (L1/Lasso, L2/Ridge, Frobenius) | MED | short | DONE |

### 4. `system-design/ml_systems.md` — 10 вопросов

| Q | Тема | Приоритет | Объём | Статус |
|---|------|-----------|-------|--------|
| Q1 | ML pipeline end-to-end (data → deploy → monitor) | **HIGH** | medium | DONE |
| Q2 | Feature Store (online vs offline, Feast) | LOW | short | DONE |
| Q3 | Model Serving (batch vs online, FastAPI, Triton) | MED | medium | DONE |
| Q4 | Monitoring и drift detection (KS-test, PSI) | MED | medium | DONE |
| Q5 | A/B тестирование моделей (shadow mode, canary) | MED | short | DONE |
| Q6 | Масштабирование ML (distributed training, data parallelism) | LOW | short | DONE |
| Q7 | Рекомендательная система: Two-Stage Funnel | **HIGH** | medium | DONE |
| Q8 | Антифрод: проектирование системы | **HIGH** | medium | DONE |
| Q9 | Cold Start Problem в рекомендациях | MED | short | DONE |
| Q10 | MLOps инструменты: DVC, MLflow, Airflow | MED | medium | DONE |

### 5. `classic-ml/theory.md` — 10 вопросов (NEW)

| Q | Тема | Приоритет | Объём | Статус |
|---|------|-----------|-------|--------|
| Q1 | Градиентный бустинг — алгоритм | **HIGH** | medium | DONE |
| Q2 | CatBoost vs LightGBM vs XGBoost | **HIGH** | medium | DONE |
| Q3 | Bias-Variance Tradeoff | **HIGH** | medium | DONE |
| Q4 | Предпосылки линейной регрессии | MED | medium | DONE |
| Q5 | Интерпретация коэффициентов логрег | MED | medium | DONE |
| Q6 | L1 (Lasso) vs L2 (Ridge) | **HIGH** | medium | DONE |
| Q7 | Метрики для несбалансированных выборок | **HIGH** | medium | DONE |
| Q8 | Кодирование категориальных признаков | MED | medium | DONE |
| Q9 | Bagging vs Boosting | MED | medium | DONE |
| Q10 | Open-ended: предсказание оттока (Churn) | MED | medium | DONE |

### 6. `deep-learning/nlp_patterns.md` — 11 вопросов (NEW)

| Q | Тема | Приоритет | Объём | Статус |
|---|------|-----------|-------|--------|
| Q1 | Предобработка текста — базовый pipeline | LOW | short | DONE |
| Q2 | Vocabulary — маппинг токенов в индексы | MED | medium | DONE |
| Q3 | DataLoader для текстов переменной длины | MED | medium | DONE |
| Q4 | RNN для языкового моделирования | LOW | medium | DONE |
| Q5 | LSTM для классификации текста | MED | medium | DONE |
| Q6 | Temperature sampling | **HIGH** | medium | DONE |
| Q7 | HuggingFace transformers — загрузка и inference | **HIGH** | medium | DONE |
| Q8 | Chat templates для instruct-моделей | **HIGH** | medium | DONE |
| Q9 | vLLM — быстрый inference и guided decoding | MED | medium | DONE |
| Q10 | Zero-shot vs Few-shot prompting | MED | short | DONE |
| Q11 | RNN vs LSTM vs GRU — отличия в PyTorch | **HIGH** | medium | DONE |

### 7. `deep-learning/pytorch.md` — 8 вопросов (+4 new)

| Q | Тема | Приоритет | Объём | Статус |
|---|------|-----------|-------|--------|
| Q17 | Autograd: clone vs detach | MED | medium | DONE |
| Q18 | CNN в PyTorch: Conv2d и формула размера | **HIGH** | medium | DONE |
| Q19 | Инициализация весов — Xavier vs He | MED | short | DONE |
| Q20 | Инспекция модели — параметры и граф | LOW | short | DONE |

### 8. `math-and-stats/probability.md` — 7 вопросов (+4 new)

| Q | Тема | Приоритет | Объём | Статус |
|---|------|-----------|-------|--------|
| Q15 | Метод обратной функции распределения (Inverse CDF) | MED | medium | DONE |
| Q16 | Преобразование Бокса-Мюллера | MED | medium | DONE |
| Q17 | Тяжёлые хвосты — распознавание | MED | medium | DONE |
| Q18 | scipy.stats — краткий справочник | LOW | short | DONE |

### 9. `math-and-stats/applied_stats.md` — 7 вопросов (+2 new)

| Q | Тема | Приоритет | Объём | Статус |
|---|------|-----------|-------|--------|
| Q14 | Оценка размера выборки (Монте-Карло) | MED | medium | DONE |
| Q15 | Марковские цепи и эффективный размер выборки | MED | medium | DONE |

### 10. `deep-learning/transformers.md` — 5 вопросов

| Q | Тема | Приоритет | Объём | Статус |
|---|------|-----------|-------|--------|
| Q1 | Self-Attention — математика и интуиция | **HIGH** | medium | DONE |
| Q2 | Positional Encoding и Multi-Head Attention | MED | medium | DONE |
| Q3 | PyTorch: autograd, DataLoader, оптимизация памяти | **HIGH** | medium | DONE |
| Q4 | LoRA (Low-Rank Adaptation) | MED | medium | DONE |
| Q5 | Vision Transformers (ViT) и стратегии дообучения | LOW | medium | DONE |

### 7. `genai/rag_systems.md` — 5 вопросов (NEW)

| Q | Тема | Приоритет | Объём | Статус |
|---|------|-----------|-------|--------|
| Q1 | Архитектура RAG-системы | **HIGH** | medium | DONE |
| Q2 | RAG vs Fine-Tuning — когда что | **HIGH** | medium | DONE |
| Q3 | Chunking и стратегии нарезки | MED | medium | DONE |
| Q4 | Vector Search: Dense vs Sparse, Hybrid, Reranking | MED | medium | DONE |
| Q5 | Evaluation RAG-систем (RAGAS, LLM-as-judge) | MED | medium | DONE |

### 8. `soft-skills/behavioral.md` — 6 вопросов (NEW)

| Q | Тема | Приоритет | Объём | Статус |
|---|------|-----------|-------|--------|
| Q1 | STAR framework — расскажи про проект | **HIGH** | medium | DONE |
| Q2 | Объясни алгоритм продакт-менеджеру | MED | short | DONE |
| Q3 | Нейросеть vs интерпретируемая модель | **HIGH** | short | DONE |
| Q4 | Грязные данные / нехватка разметки | MED | short | DONE |
| Q5 | Конфликт по выбору архитектуры | MED | short | DONE |
| Q6 | Почему ML/DS? Почему наша компания? | **HIGH** | short | DONE |

### 9. `math-and-stats/statistics.md` — 7 вопросов (+2 new)

| Q | Тема | Приоритет | Объём | Статус |
|---|------|-----------|-------|--------|
| Q16 | Ошибки I и II рода, мощность теста | **HIGH** | medium | DONE |
| Q17 | Multiple Hypothesis Testing — Бонферрони, BH | MED | medium | DONE |

### 10. `math-and-stats/applied_stats.md` — 5 вопросов (+1 new)

| Q | Тема | Приоритет | Объём | Статус |
|---|------|-----------|-------|--------|
| Q13 | Сетевые эффекты (Network Effects) в A/B тестах | MED | medium | DONE |
