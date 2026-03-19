# SQL Fundamentals — Interview Prep

SQL для DS/ML собеседований. Ключевые темы: JOINs, агрегаты, оконные функции, оптимизация запросов.

---

## Q1. "SELECT + JOINs — виды и когда какой"

> "JOIN объединяет строки из двух таблиц по условию. Основные виды: **INNER** — только совпадения, **LEFT** — все из левой + совпадения из правой (остальное NULL), **RIGHT** — зеркально, **FULL OUTER** — все из обеих, **CROSS** — декартово произведение. На практике 90% — это INNER и LEFT. LEFT JOIN критичен для аналитики: видим клиентов **без заказов** (NULL в полях заказа). В Alfa-Bank использовал LEFT JOIN для обогащения таблицы заявок данными из кредитных историй — не у каждого клиента была история, и NULL-значения были важным признаком для CatBoost."

| JOIN | Что возвращает | NULL-поведение | Когда использовать |
|------|---------------|----------------|-------------------|
| **INNER JOIN** | Только совпавшие строки из обеих таблиц | Нет NULL от JOIN | Нужны только клиенты с заказами |
| **LEFT JOIN** | Все из левой + совпадения из правой | NULL в правой, если нет match | Клиенты без заказов тоже нужны |
| **RIGHT JOIN** | Все из правой + совпадения из левой | NULL в левой, если нет match | Редко — проще переписать как LEFT |
| **FULL OUTER JOIN** | Все из обеих таблиц | NULL с обеих сторон, где нет match | Reconciliation двух источников |
| **CROSS JOIN** | Каждая строка $\times$ каждая строка | Нет NULL от JOIN | Генерация всех комбинаций (дата $\times$ продукт) |

```sql
-- Тестовые таблицы
CREATE TABLE customers (
    id    INT PRIMARY KEY,
    name  VARCHAR(50),
    city  VARCHAR(50)
);

CREATE TABLE orders (
    id          INT PRIMARY KEY,
    customer_id INT,
    amount      NUMERIC(10,2),
    order_date  DATE
);

INSERT INTO customers VALUES (1, 'Алиса', 'Москва'), (2, 'Борис', 'СПб'), (3, 'Вика', 'Казань');
INSERT INTO orders VALUES (101, 1, 5000, '2025-01-15'), (102, 1, 3000, '2025-02-10'),
                          (103, 2, 7000, '2025-01-20'), (104, NULL, 1000, '2025-03-01');

-- INNER JOIN: только клиенты с заказами (Вика пропадёт, заказ 104 пропадёт)
SELECT c.name, o.amount
FROM customers c
INNER JOIN orders o ON c.id = o.customer_id;
-- Алиса | 5000
-- Алиса | 3000
-- Борис | 7000

-- LEFT JOIN: все клиенты, даже без заказов (Вика будет с NULL)
SELECT c.name, o.amount, o.order_date
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id;
-- Алиса | 5000  | 2025-01-15
-- Алиса | 3000  | 2025-02-10
-- Борис | 7000  | 2025-01-20
-- Вика  | NULL  | NULL          ← клиент без заказов

-- Найти клиентов БЕЗ заказов (антиджойн)
SELECT c.name
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE o.id IS NULL;
-- Вика

-- CROSS JOIN: все комбинации (3 клиента × 4 заказа = 12 строк)
SELECT c.name, o.id AS order_id
FROM customers c
CROSS JOIN orders o;
```

---

## Q2. "GROUP BY + HAVING + агрегатные функции"

> "**GROUP BY** группирует строки по значениям столбца, потом агрегатная функция считает что-то по каждой группе. **WHERE** фильтрует строки **до** группировки, **HAVING** фильтрует **группы после** агрегации. Порядок выполнения: FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY. Ключевое: в SELECT можно только столбцы из GROUP BY или агрегаты — иначе ошибка. В Alfa-Bank: GROUP BY по сегментам клиентов + AVG(probability_of_default), HAVING отсекал сегменты с малым кол-вом наблюдений."

| Агрегат | Описание | NULL-поведение |
|---------|----------|----------------|
| `COUNT(*)` | Кол-во строк (включая NULL) | Считает **все** строки |
| `COUNT(col)` | Кол-во **не NULL** значений в столбце | **Игнорирует** NULL |
| `SUM(col)` | Сумма | Игнорирует NULL |
| `AVG(col)` | Среднее | Игнорирует NULL (делит на COUNT(col), не COUNT(*)) |
| `MIN(col)` / `MAX(col)` | Мин/макс | Игнорирует NULL |

```sql
-- Средний чек по городам, только города с > 100 заказами
SELECT
    c.city,
    COUNT(*)        AS total_orders,
    AVG(o.amount)   AS avg_check,
    SUM(o.amount)   AS total_revenue,
    MIN(o.amount)   AS min_check,
    MAX(o.amount)   AS max_check
FROM orders o
JOIN customers c ON c.id = o.customer_id
WHERE o.order_date >= '2025-01-01'   -- WHERE: фильтр строк ДО группировки
GROUP BY c.city
HAVING COUNT(*) > 100                -- HAVING: фильтр ПОСЛЕ группировки
ORDER BY avg_check DESC;

-- COUNT(*) vs COUNT(col) — ключевое отличие
SELECT
    COUNT(*)              AS total_rows,      -- 4 (все строки)
    COUNT(customer_id)    AS non_null_cust,   -- 3 (NULL в заказе 104 не считается)
    COUNT(DISTINCT customer_id) AS unique_cust -- 2 (Алиса и Борис)
FROM orders;
```

**Порядок выполнения SQL-запроса:**

$$\text{FROM} \to \text{WHERE} \to \text{GROUP BY} \to \text{HAVING} \to \text{SELECT} \to \text{ORDER BY} \to \text{LIMIT}$$

---

## Q3. "Window functions"

> "Оконные функции — вычисления по 'окну' строк, **без схлопывания** в одну строку (в отличие от GROUP BY). Синтаксис: `FUNC() OVER (PARTITION BY ... ORDER BY ...)`. PARTITION BY делит на группы (как GROUP BY, но строки остаются), ORDER BY задаёт порядок внутри окна. Три категории: ранжирующие (ROW_NUMBER, RANK, DENSE_RANK), аналитические (LAG, LEAD), агрегатные (SUM, AVG OVER). Для DS это ключевая тема — оконные функции незаменимы для feature engineering: скользящие средние, ранги, дельты."

**Ранжирующие функции — разница на одних данных:**

| Продажи | ROW_NUMBER | RANK | DENSE_RANK |
|---------|-----------|------|------------|
| 100 | 1 | 1 | 1 |
| 90 | 2 | 2 | 2 |
| 90 | 3 | 2 | 2 |
| 80 | 4 | 4 | 3 |

> ROW_NUMBER — всегда уникальный номер. RANK — одинаковые значения получают одинаковый ранг, **пропуск** после (2, 2, 4). DENSE_RANK — без пропуска (2, 2, 3).

```sql
-- Ранг товара по продажам в каждой категории
SELECT
    category,
    product_name,
    sales,
    ROW_NUMBER() OVER (PARTITION BY category ORDER BY sales DESC) AS rn,
    RANK()       OVER (PARTITION BY category ORDER BY sales DESC) AS rnk,
    DENSE_RANK() OVER (PARTITION BY category ORDER BY sales DESC) AS dense_rnk
FROM products;

-- LAG / LEAD: дельта продаж по месяцам (MoM change)
SELECT
    month,
    revenue,
    LAG(revenue, 1)  OVER (ORDER BY month) AS prev_month,
    revenue - LAG(revenue, 1) OVER (ORDER BY month) AS mom_delta,
    LEAD(revenue, 1) OVER (ORDER BY month) AS next_month
FROM monthly_sales;

-- SUM / AVG OVER: скользящее среднее и нарастающий итог
SELECT
    order_date,
    amount,
    SUM(amount) OVER (ORDER BY order_date)                          AS running_total,
    AVG(amount) OVER (ORDER BY order_date ROWS BETWEEN 2 PRECEDING
                                          AND CURRENT ROW)         AS moving_avg_3
FROM orders;

-- Доля заказа от общей суммы по городу
SELECT
    c.city,
    o.amount,
    SUM(o.amount) OVER (PARTITION BY c.city)                       AS city_total,
    ROUND(o.amount * 100.0 / SUM(o.amount) OVER (PARTITION BY c.city), 1) AS pct_of_city
FROM orders o
JOIN customers c ON c.id = o.customer_id;
```

---

## Q4. "CTE (Common Table Expressions)"

> "**CTE** — именованный временный результат запроса через `WITH ... AS (...)`. Делает сложные запросы читаемыми: разбиваем на логические шаги, каждый CTE — один шаг. Можно строить цепочки CTE. **Рекурсивный CTE** — для иерархий (оргструктура, категории товаров). CTE vs подзапрос: CTE читабельнее и можно ссылаться несколько раз; подзапрос бывает быстрее, т.к. optimizer может его инлайнить. В PostgreSQL CTE раньше были optimization fence (до v12), сейчас нет."

```sql
-- Цепочка CTE: воронка по городам
WITH order_stats AS (
    -- Шаг 1: агрегируем заказы по клиентам
    SELECT
        customer_id,
        COUNT(*)       AS order_count,
        SUM(amount)    AS total_spent,
        AVG(amount)    AS avg_check
    FROM orders
    WHERE order_date >= '2025-01-01'
    GROUP BY customer_id
),
enriched AS (
    -- Шаг 2: обогащаем данными клиента
    SELECT
        c.city,
        c.name,
        os.order_count,
        os.total_spent,
        os.avg_check
    FROM order_stats os
    JOIN customers c ON c.id = os.customer_id
),
city_summary AS (
    -- Шаг 3: агрегация по городам
    SELECT
        city,
        COUNT(*)            AS customers,
        SUM(total_spent)    AS revenue,
        AVG(avg_check)      AS avg_check
    FROM enriched
    GROUP BY city
)
SELECT * FROM city_summary
ORDER BY revenue DESC;

-- Рекурсивный CTE: иерархия сотрудников (org chart)
WITH RECURSIVE org_tree AS (
    -- Базовый случай: CEO (manager_id IS NULL)
    SELECT id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Рекурсивный шаг: подчинённые
    SELECT e.id, e.name, e.manager_id, t.level + 1
    FROM employees e
    JOIN org_tree t ON e.manager_id = t.id
)
SELECT
    REPEAT('  ', level - 1) || name AS org_chart,
    level
FROM org_tree
ORDER BY level, name;
```

| Критерий | CTE | Подзапрос |
|----------|-----|-----------|
| Читаемость | Высокая (именованные блоки) | Низкая при вложенности |
| Повторное использование | Можно ссылаться несколько раз | Копировать код |
| Рекурсия | Поддерживает `RECURSIVE` | Нет |
| Оптимизация (PostgreSQL 12+) | Optimizer может инлайнить | Всегда инлайнится |
| Когда использовать | Сложные запросы с 3+ шагами | Простой однократный подзапрос |

---

## Q5. "Подзапросы — correlated vs non-correlated"

> "**Non-correlated** подзапрос выполняется один раз, результат подставляется во внешний запрос. **Correlated** выполняется для **каждой строки** внешнего запроса — ссылается на его столбцы. Correlated медленнее (N раз вместо 1), но иногда единственный вариант. **IN** vs **EXISTS**: IN — сравнивает значение со списком (хорош, когда подзапрос возвращает мало строк). EXISTS — проверяет наличие хотя бы одной строки (хорош, когда внешняя таблица маленькая, а внутренняя большая — останавливается при первом match). Скалярный подзапрос в SELECT — удобно, но опасно для performance."

```sql
-- NON-CORRELATED: выполняется один раз, результат подставляется
-- "Клиенты из городов, где есть хотя бы 5 клиентов"
SELECT name, city
FROM customers
WHERE city IN (
    SELECT city
    FROM customers
    GROUP BY city
    HAVING COUNT(*) >= 5
);

-- CORRELATED: выполняется для каждой строки внешнего запроса
-- "Заказы, сумма которых выше среднего для их клиента"
SELECT o.id, o.customer_id, o.amount
FROM orders o
WHERE o.amount > (
    SELECT AVG(o2.amount)
    FROM orders o2
    WHERE o2.customer_id = o.customer_id  -- ссылка на внешний запрос
);

-- IN vs EXISTS
-- IN: хорош, когда подзапрос возвращает мало строк
SELECT name
FROM customers
WHERE id IN (SELECT customer_id FROM orders WHERE amount > 10000);

-- EXISTS: хорош, когда внешняя таблица мала, внутренняя велика
-- Останавливается при первом совпадении — не сканирует всё
SELECT c.name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id AND o.amount > 10000
);

-- Скалярный подзапрос в SELECT
-- Удобно, но выполняется для каждой строки — O(N)
SELECT
    c.name,
    c.city,
    (SELECT COUNT(*) FROM orders o WHERE o.customer_id = c.id) AS order_count,
    (SELECT MAX(o.amount) FROM orders o WHERE o.customer_id = c.id) AS max_order
FROM customers c;
```

| Тип | Выполнение | Производительность | Пример |
|-----|-----------|-------------------|--------|
| **Non-correlated** | Один раз | $O(1)$ для подзапроса | `WHERE city IN (SELECT ...)` |
| **Correlated** | $N$ раз (для каждой строки) | $O(N \cdot M)$ без оптимизации | `WHERE amount > (SELECT AVG(...) WHERE id = outer.id)` |
| **IN** | Вычислить список, сравнить | Быстро при малом списке | Подзапрос возвращает 10-100 значений |
| **EXISTS** | Проверить наличие, остановиться | Быстро при большой внутренней таблице | Внутренняя таблица — миллионы строк |

---

## Q6. "Query optimization"

> "Оптимизация начинается с **EXPLAIN ANALYZE** — показывает план выполнения запроса, реальное время и кол-во строк. Ключевое: **Seq Scan** (полный просмотр таблицы) vs **Index Scan** (поиск по индексу, $O(\log n)$ для B-tree). **B-tree индекс** — основной тип, работает для `=`, `<`, `>`, `BETWEEN`, `ORDER BY`. **Composite index** `(a, b)` — работает для запросов по `a` или `a + b`, но **не** по `b` отдельно (leftmost prefix rule). **Covering index** — содержит все нужные столбцы, Index Only Scan без обращения к таблице. Индекс НЕ помогает при: малой таблице, низкой селективности (bool-столбец), `LIKE '%abc'`, функции от столбца. **N+1 problem**: 1 запрос для списка + N запросов для деталей каждого элемента — решается через JOIN или batch query."

```sql
-- EXPLAIN ANALYZE: смотрим план выполнения
EXPLAIN ANALYZE
SELECT c.name, SUM(o.amount)
FROM customers c
JOIN orders o ON c.id = o.customer_id
WHERE c.city = 'Москва'
GROUP BY c.name;

-- Seq Scan (плохо на большой таблице):
--   Seq Scan on orders  (cost=0.00..15420.00 rows=500000 ...)
-- Index Scan (хорошо):
--   Index Scan using idx_orders_customer_id on orders  (cost=0.43..8.50 rows=5 ...)
```

| Тип плана | Описание | Когда используется | Сложность |
|-----------|----------|-------------------|-----------|
| **Seq Scan** | Полный просмотр таблицы | Нет индекса или малая таблица | $O(n)$ |
| **Index Scan** | Поиск по B-tree индексу | Есть индекс, высокая селективность | $O(\log n)$ |
| **Index Only Scan** | Ответ только из индекса (covering) | Все нужные столбцы в индексе | $O(\log n)$ |
| **Bitmap Index Scan** | Индекс → bitmap → таблица | Средняя селективность | $O(\log n + k)$ |
| **Hash Join** | Hash-таблица для JOIN | Equality JOIN на больших таблицах | $O(n + m)$ |
| **Nested Loop** | Перебор пар строк | Маленькая внешняя таблица | $O(n \cdot m)$ |

```sql
-- B-tree индекс: ускоряет =, <, >, BETWEEN, ORDER BY
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_orders_date ON orders(order_date);

-- Composite index: работает для (customer_id) и (customer_id, order_date)
-- НЕ работает для (order_date) отдельно — leftmost prefix rule
CREATE INDEX idx_orders_cust_date ON orders(customer_id, order_date);

-- Covering index: все нужные столбцы в индексе → Index Only Scan
CREATE INDEX idx_orders_covering ON orders(customer_id, order_date) INCLUDE (amount);

-- Когда индекс НЕ помогает:
-- 1. Функция от столбца — optimizer не может использовать индекс
SELECT * FROM orders WHERE YEAR(order_date) = 2025;      -- Seq Scan!
SELECT * FROM orders WHERE order_date >= '2025-01-01'
                       AND order_date <  '2026-01-01';    -- Index Scan

-- 2. LIKE с wildcard в начале
SELECT * FROM customers WHERE name LIKE '%иван%';         -- Seq Scan!
SELECT * FROM customers WHERE name LIKE 'Иван%';          -- Index Scan

-- 3. Низкая селективность (bool-столбец: 50/50)
SELECT * FROM orders WHERE is_paid = true;                 -- Seq Scan выгоднее

-- N+1 problem: 1 запрос + N запросов
-- ПЛОХО: (в коде) for customer in customers: SELECT * FROM orders WHERE customer_id = ?
-- ХОРОШО: один JOIN
SELECT c.name, o.amount
FROM customers c
JOIN orders o ON c.id = o.customer_id;
-- Или batch: SELECT * FROM orders WHERE customer_id IN (1, 2, 3, ...);
```

---

## Q7. "Типичные задачи на SQL-собеседовании"

> "Четыре классические задачи, которые покрывают 80% SQL-вопросов на собеседовании: retention rate (удержание пользователей), running total (нарастающий итог), top-N per group (топ в каждой категории), удаление дубликатов. Все решаются через оконные функции + CTE. В Random Coffee бот использует подобные паттерны — расчёт активности пользователей и дедупликация пар."

### Задача 1: Retention Rate

Посчитать долю пользователей, вернувшихся на следующий месяц.

```sql
WITH monthly_users AS (
    SELECT
        user_id,
        DATE_TRUNC('month', event_date) AS month
    FROM events
    GROUP BY user_id, DATE_TRUNC('month', event_date)
),
retention AS (
    SELECT
        curr.month                       AS current_month,
        COUNT(DISTINCT curr.user_id)     AS active_users,
        COUNT(DISTINCT next_m.user_id)   AS retained_users
    FROM monthly_users curr
    LEFT JOIN monthly_users next_m
        ON curr.user_id = next_m.user_id
       AND next_m.month = curr.month + INTERVAL '1 month'
    GROUP BY curr.month
)
SELECT
    current_month,
    active_users,
    retained_users,
    ROUND(100.0 * retained_users / active_users, 1) AS retention_pct
FROM retention
ORDER BY current_month;
```

### Задача 2: Running Total (нарастающий итог)

```sql
SELECT
    order_date,
    amount,
    SUM(amount) OVER (ORDER BY order_date) AS running_total
FROM orders
ORDER BY order_date;

-- По клиентам отдельно
SELECT
    customer_id,
    order_date,
    amount,
    SUM(amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date
    ) AS customer_running_total
FROM orders
ORDER BY customer_id, order_date;
```

### Задача 3: Top-N per Group

Топ-3 товара по продажам в каждой категории.

```sql
WITH ranked AS (
    SELECT
        category,
        product_name,
        sales,
        ROW_NUMBER() OVER (
            PARTITION BY category
            ORDER BY sales DESC
        ) AS rn
    FROM products
)
SELECT category, product_name, sales
FROM ranked
WHERE rn <= 3
ORDER BY category, rn;
```

### Задача 4: Удаление дубликатов

Оставить одну запись из каждой группы дубликатов (по email).

```sql
-- Найти дубликаты
SELECT email, COUNT(*) AS cnt
FROM users
GROUP BY email
HAVING COUNT(*) > 1;

-- Удалить дубликаты, оставив строку с минимальным id
WITH duplicates AS (
    SELECT
        id,
        email,
        ROW_NUMBER() OVER (
            PARTITION BY email
            ORDER BY id
        ) AS rn
    FROM users
)
DELETE FROM users
WHERE id IN (
    SELECT id FROM duplicates WHERE rn > 1
);

-- Альтернатива: оставить через временную таблицу (безопаснее)
CREATE TABLE users_clean AS
SELECT DISTINCT ON (email) *
FROM users
ORDER BY email, id;
```

---

## Q8. "Gaps and Islands — поиск непрерывных периодов активности"

> "Классическая задача на собеседовании: найти пользователей с 5+ днями подряд. Паттерн **Gaps and Islands**: разница `date - ROW_NUMBER()` одинакова для последовательных дат → группируем по этому вычисленному идентификатору.
>
> Идея: если даты идут подряд (1, 2, 3, 4) и вычитаем ROW_NUMBER (1, 2, 3, 4), получаем одинаковую разницу (0, 0, 0, 0). Если есть пропуск (1, 2, 5, 6), разница будет (0, 0, 2, 2) — разные группы.
>
> Этот паттерн работает для любых последовательных значений: даты, номера транзакций, ID."

```sql
-- Задача: найти пользователей с 5+ днями подряд
WITH daily_logins AS (
    SELECT DISTINCT user_id, login_date::date AS dt
    FROM logins
),
islands AS (
    SELECT
        user_id,
        dt,
        dt - ROW_NUMBER() OVER (
            PARTITION BY user_id ORDER BY dt
        )::int AS island_id
    FROM daily_logins
),
streaks AS (
    SELECT
        user_id,
        island_id,
        MIN(dt) AS streak_start,
        MAX(dt) AS streak_end,
        COUNT(*) AS streak_length
    FROM islands
    GROUP BY user_id, island_id
)
SELECT user_id, streak_start, streak_end, streak_length
FROM streaks
WHERE streak_length >= 5
ORDER BY streak_length DESC;
```

---

## Q9. "Conditional aggregation и Pivot (сводные таблицы)"

> "**Conditional aggregation** — построение сводных таблиц за один проход с помощью `CASE WHEN` внутри агрегатной функции. Позволяет 'развернуть' строки в столбцы без отдельного PIVOT-оператора (которого нет в PostgreSQL).
>
> **Anti-Join** паттерн: найти строки из левой таблицы, которых нет в правой. Три способа:
> - `LEFT JOIN ... WHERE right.id IS NULL` — самый читаемый
> - `NOT EXISTS (SELECT 1 FROM ...)` — часто быстрее
> - `NOT IN (SELECT ...)` — **опасен с NULL**: если подзапрос содержит NULL, `NOT IN` вернёт пустой результат (любое сравнение с NULL = UNKNOWN)."

```sql
-- Conditional aggregation: сводная таблица за один проход
SELECT
    product_id,
    SUM(CASE WHEN month = '2025-01' THEN revenue ELSE 0 END) AS jan,
    SUM(CASE WHEN month = '2025-02' THEN revenue ELSE 0 END) AS feb,
    SUM(CASE WHEN month = '2025-03' THEN revenue ELSE 0 END) AS mar,
    SUM(revenue) AS total
FROM monthly_sales
GROUP BY product_id;

-- Anti-Join: клиенты без заказов в 2025
-- Способ 1: LEFT JOIN + IS NULL (читаемый)
SELECT c.id, c.name
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id AND o.order_date >= '2025-01-01'
WHERE o.id IS NULL;

-- Способ 2: NOT EXISTS (часто быстрее)
SELECT c.id, c.name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1 FROM orders o
    WHERE o.customer_id = c.id AND o.order_date >= '2025-01-01'
);

-- Способ 3: NOT IN — ОПАСНО с NULL!
SELECT c.id FROM customers c
WHERE c.id NOT IN (SELECT customer_id FROM orders);
-- Если хотя бы один customer_id = NULL → результат ПУСТОЙ
-- Защита: WHERE customer_id IS NOT NULL
```

---

## Q10. "ClickHouse vs PostgreSQL — когда что"

> "**PostgreSQL** — строковая (row-based) СУБД, оптимизирована для OLTP: вставка/обновление/удаление отдельных строк, транзакции (ACID), сложные JOIN-ы.
>
> **ClickHouse** — колоночная (column-based) СУБД, оптимизирована для OLAP: аналитические запросы по миллиардам строк. Хранит каждый столбец отдельно → при `SELECT AVG(amount)` читает только столбец `amount`, пропуская остальные.
>
> Почему ClickHouse быстрее для аналитики: колоночное хранение → лучшее сжатие (одинаковые типы данных), меньше I/O (читаем только нужные столбцы), vectorized execution.
>
> Особенности ClickHouse: `ARRAY JOIN` (разворачивает массив в строки), higher-order functions (`arrayMap`, `arrayFilter`), partitioning по датам (MergeTree engine).
>
> Когда спрашивают: Яндекс, VK, любые компании с большими логами / аналитикой."

| Критерий | PostgreSQL | ClickHouse |
|----------|-----------|------------|
| Тип | Row-based (OLTP) | Column-based (OLAP) |
| Сильные стороны | Транзакции, ACID, JOIN-ы | Аналитика, агрегации, >1B строк |
| Вставка | По одной строке, быстро | Батчами (bulk insert) |
| UPDATE/DELETE | Быстро | Медленно / не рекомендуется |
| Типичный запрос | `SELECT * FROM users WHERE id = 42` | `SELECT city, AVG(amount) FROM orders GROUP BY city` |
| Сжатие | Умеренное | **Отличное** (LZ4, ZSTD, Delta) |
| Когда | Основная БД приложения | Логи, аналитика, метрики |

```sql
-- ClickHouse: пример аналитического запроса
SELECT
    toDate(event_time) AS dt,
    countIf(event = 'purchase') AS purchases,
    countIf(event = 'view') AS views,
    round(purchases / views * 100, 2) AS conversion_pct
FROM events
WHERE event_time >= today() - 30
GROUP BY dt
ORDER BY dt;

-- ARRAY JOIN: разворачиваем теги
SELECT
    user_id,
    tag
FROM user_profiles
ARRAY JOIN tags AS tag;  -- tags = ['ml', 'python', 'sql'] → 3 строки
```

---

## Приоритет для intern-level

| Тема | Приоритет | Почему |
|------|-----------|--------|
| JOINs (Q1) | **Высокий** | Спрашивают всегда |
| Window functions (Q3) | **Высокий** | Отличает junior от intern |
| GROUP BY + HAVING (Q2) | **Высокий** | Базовая аналитика |
| Типичные задачи (Q7) | **Высокий** | Retention, top-N, дубликаты — классика |
| Gaps & Islands (Q8) | **Высокий** | Продвинутая задача, впечатляет интервьюера |
| CTE (Q4) | Средний | Для читаемости сложных запросов |
| Подзапросы (Q5) | Средний | IN vs EXISTS — частый вопрос |
| Conditional aggregation (Q9) | Средний | Anti-Join и NOT IN ловушка |
| ClickHouse (Q10) | Средний | Для Яндекс / VK / аналитических ролей |
| Optimization (Q6) | Низкий | Больше для backend/DE |
