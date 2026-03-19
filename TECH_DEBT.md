# Knowledge Base — Tech Debt

Трекер пустых/неготовых материалов в `learning/`. Обновлено: 2026-03-19.

---

## Общая статистика

| Метрика | Значение |
|---------|----------|
| Всего файлов | 21 |
| Готово (полный контент) | 21 |
| Скелеты (только TODO) | 0 |
| Готовых вопросов/упражнений | 71 Q + 10 Ex |
| Пустых вопросов | **0** |
| Покрытие | **100%** |

---

## Долг по файлам

### 1. `sql/fundamentals.md` — 7 пустых вопросов

| Q | Тема | Приоритет | Объём | Статус |
|---|------|-----------|-------|--------|
| Q1 | SELECT + JOINs (INNER, LEFT, RIGHT, FULL, CROSS) | **HIGH** | medium | DONE |
| Q2 | GROUP BY + HAVING + агрегатные функции | **HIGH** | medium | DONE |
| Q3 | Window functions (ROW_NUMBER, RANK, LAG, LEAD) | **HIGH** | medium | DONE |
| Q4 | CTE (Common Table Expressions) | MED | short | DONE |
| Q5 | Подзапросы — correlated vs non-correlated | MED | short | DONE |
| Q6 | Query optimization (EXPLAIN, индексы) | LOW | medium | DONE |
| Q7 | Типичные задачи (retention, running total, top-N) | LOW | medium | DONE |

### 2. `python/coding_interview.md` — 7 пустых вопросов

| Q | Тема | Приоритет | Объём | Статус |
|---|------|-----------|-------|--------|
| Q1 | Структуры данных — сложность операций (list, dict, set, deque) | **HIGH** | medium | DONE |
| Q2 | Алгоритмы (binary search, two pointers, sliding window) | **HIGH** | medium | DONE |
| Q3 | ООП (классы, наследование, dataclasses, абстрактные) | MED | medium | DONE |
| Q4 | Декораторы (замыкания, functools.wraps, timing/caching) | MED | short | DONE |
| Q5 | Генераторы и итераторы (yield, lazy evaluation) | **HIGH** | short | DONE |
| Q6 | GIL (threading vs multiprocessing vs asyncio) | LOW | short | DONE |
| Q7 | Live coding задачи (two sum, parentheses, merge intervals) | LOW | medium | DONE |

### 3. `linear-algebra/core_concepts.md` — 6 пустых вопросов

| Q | Тема | Приоритет | Объём | Статус |
|---|------|-----------|-------|--------|
| Q1 | Векторы (скалярное произведение, нормы, косинусное сходство) | **HIGH** | medium | DONE |
| Q2 | Матрицы (умножение, обратная, определитель, ранг) | **HIGH** | medium | DONE |
| Q3 | Собственные значения и собственные векторы | MED | medium | DONE |
| Q4 | SVD (A = UΣVᵀ, low-rank approximation) | MED | medium | DONE |
| Q5 | PCA через линалг (ковариационная матрица → собственные векторы) | **HIGH** | medium | DONE |
| Q6 | Нормы в ML (L1/Lasso, L2/Ridge, Frobenius) | MED | short | DONE |

### 4. `system-design/ml_systems.md` — 6 пустых вопросов

| Q | Тема | Приоритет | Объём | Статус |
|---|------|-----------|-------|--------|
| Q1 | ML pipeline end-to-end (data → deploy → monitor) | **HIGH** | medium | DONE |
| Q2 | Feature Store (online vs offline, Feast) | LOW | short | DONE |
| Q3 | Model Serving (batch vs online, FastAPI, Triton) | MED | medium | DONE |
| Q4 | Monitoring и drift detection (KS-test, PSI) | MED | medium | DONE |
| Q5 | A/B тестирование моделей (shadow mode, canary) | MED | short | DONE |
| Q6 | Масштабирование ML (distributed training, data parallelism) | LOW | short | DONE |

---

## Рекомендованный порядок заполнения

Приоритет по ROI для intern-собеседований:

| # | Что заполнить | Почему | ~Время |
|---|--------------|--------|--------|
| 1 | SQL: JOINs, window functions, GROUP BY (Q1-3) | Спрашивают на каждом DS-собесе | 1.5h |
| 2 | Python: структуры данных + алгоритмы + generators (Q1,2,5) | Live coding раунды | 1.5h |
| 3 | Linear algebra: vectors/norms + PCA (Q1,2,5) | ML theory раунды, связь с feature engineering | 1.5h |
| 4 | SQL: CTE + подзапросы (Q4-5) | Углубление SQL | 30min |
| 5 | Python: ООП + декораторы (Q3-4) | Код-ревью вопросы | 1h |
| 6 | Linear algebra: eigenvalues, SVD, norms (Q3,4,6) | Глубокое понимание PCA/рекомендаций | 1.5h |
| 7 | System design: ML pipeline + serving (Q1,3) | Показать системное мышление | 1h |
| 8 | Остальное (SQL Q6-7, Python Q6-7, SysDes Q2,4-6) | Низкий приоритет для intern | 2h |

**Итого на заполнение всего долга: ~10-11 часов**
