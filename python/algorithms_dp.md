# Динамическое программирование — Interview Prep

DP-паттерны, анти-оверинжениринг, 5 шаблонов с задачами для собеседования ML/DS/AI Engineer Intern.

---

## Q1. "Как ты подходишь к DP-задачам?"

> "Когда вижу DP-задачу, первым делом формулирую dp[i] **одним предложением**. Пишу рекуррентность на бумаге. Если больше 2-3 членов или нужно больше одного индекса — останавливаюсь и ищу более простую формулировку. Начинаю с bottom-up, потом оптимизирую по памяти если dp[i] зависит только от предыдущих 1-2 значений."

### Фреймворк: 4 шага

| Шаг | Что делаю | Зачем |
|-----|-----------|-------|
| 1. `dp[i]` одним предложением | "dp[i] = ответ для первых i элементов" | Если нужно 2+ индексов — стоп, подумай |
| 2. Рекуррентность на бумаге | `dp[i] = f(dp[i-1], dp[i-2], ...)` | >2-3 слагаемых = переусложняешь |
| 3. Нужен ли весь массив? | dp[i] зависит от dp[i-1], dp[i-2]? → 2 переменных | Экономия O(n) → O(1) по памяти |
| 4. Bottom-up первым | Итеративный цикл | Чище, нет рекурсии, интервьюеры предпочитают |

### Красные флаги оверинжениринга

| Красный флаг | Что делать вместо |
|---|---|
| `dp[i][j][k]` — 3 измерения | Нужны ли все 3? Часто одно можно убрать |
| Рекурсия с 5+ аргументами | Переформулируй subproblem проще |
| Bitmask в state | Только если N $\leq$ 20 и задача явно про подмножества |
| Строишь граф для задачи на массив | DP $\neq$ граф. Массив → оставайся на массиве |
| >3 мин на формулировку dp | Вернись к brute force, выпиши что перебираешь |

---

## Q2. "5 базовых паттернов DP"

### Паттерн 1: Linear DP — "оптимум на массиве"

**Шаблон:** `dp[i] = f(dp[i-1], dp[i-2], ..., nums[i])`

> "dp[i] = максимальная сумма для первых i домов. Два варианта: не граблю дом i → dp[i-1], граблю → dp[i-2] + nums[i]. Зависит от двух предыдущих → две переменные."

**House Robber (LeetCode 198):**

```python
def rob(nums):
    if len(nums) <= 2:
        return max(nums)
    prev2, prev1 = nums[0], max(nums[0], nums[1])
    for i in range(2, len(nums)):
        prev2, prev1 = prev1, max(prev1, prev2 + nums[i])
    return prev1
# Time: O(n), Space: O(1)
```

### Паттерн 2: Unbounded Knapsack — "минимум/максимум из набора"

**Шаблон:** `dp[i] = min/max over choices(dp[i - choice] + cost)`

> "dp[n] = минимум квадратов для суммы n. Для каждого i пробую все квадраты $j^2 \leq i$. Структура как coin change."

**Perfect Squares (LeetCode 279):**

```python
def num_squares(n):
    dp = [float('inf')] * (n + 1)
    dp[0] = 0
    for i in range(1, n + 1):
        j = 1
        while j * j <= i:
            dp[i] = min(dp[i], dp[i - j * j] + 1)
            j += 1
    return dp[n]
# Time: O(n * sqrt(n)), Space: O(n)
```

### Паттерн 3: String DP (2D) — "две строки/подпоследовательности"

**Шаблон:** `dp[i][j]` — ответ для prefix1[:i] и prefix2[:j]

> "dp[i][j] = длина LCS для text1[:i] и text2[:j]. Если символы совпали — dp[i-1][j-1]+1, иначе — max(dp[i-1][j], dp[i][j-1]). Это ЛЕГИТИМНЫЙ 2D — оба индекса нужны. Но по памяти оптимизирую до одной строки."

**Longest Common Subsequence (LeetCode 1143):**

```python
def longest_common_subsequence(text1, text2):
    m, n = len(text1), len(text2)
    prev = [0] * (n + 1)
    for i in range(1, m + 1):
        curr = [0] * (n + 1)
        for j in range(1, n + 1):
            if text1[i - 1] == text2[j - 1]:
                curr[j] = prev[j - 1] + 1
            else:
                curr[j] = max(prev[j], curr[j - 1])
        prev = curr
    return prev[n]
# Time: O(m*n), Space: O(min(m, n))
```

### Паттерн 4: Decision DP — "выбор из нескольких вариантов"

**Шаблон:** `dp[i] = min over options(dp[i - duration] + cost)`

> "dp ОДНОМЕРНЫЙ — по дням. Если день не в списке путешествий — dp[i] = dp[i-1]. Иначе — минимум из трёх вариантов билетов."

**Minimum Cost For Tickets (LeetCode 983):**

```python
def min_cost_tickets(days, costs):
    last_day = days[-1]
    dp = [0] * (last_day + 1)
    travel_days = set(days)
    for i in range(1, last_day + 1):
        if i not in travel_days:
            dp[i] = dp[i - 1]
        else:
            dp[i] = min(
                dp[i - 1] + costs[0],
                dp[max(0, i - 7)] + costs[1],
                dp[max(0, i - 30)] + costs[2],
            )
    return dp[last_day]
# Time: O(last_day), Space: O(last_day)
```

### Паттерн 5: Boolean/Counting DP — "сколько способов"

**Шаблон:** `dp[i] = dp[i-1] (если A) + dp[i-2] (если B)`

> "Структура как climb_stairs, но с условиями. Если s[i] валидна — dp[i-1]. Если s[i-1:i+1] валидна — dp[i-2]. Зависит от двух предыдущих → две переменные."

**Decode Ways (LeetCode 91):**

```python
def num_decodings(s):
    if s[0] == '0':
        return 0
    prev2, prev1 = 1, 1
    for i in range(1, len(s)):
        curr = 0
        if s[i] != '0':
            curr += prev1
        if 10 <= int(s[i-1:i+1]) <= 26:
            curr += prev2
        prev2, prev1 = prev1, curr
    return prev1
# Time: O(n), Space: O(1)
```

---

## Q3. "Когда DP — это оверинжениринг?"

> "Если задача монотонна — greedy. Jump Game: можно только расширять max_reach → greedy за O(n). DP здесь даёт O(n²) без выигрыша. Другие сигналы: задача на сортировку, задача на два указателя, задача на sliding window."

**Jump Game (LeetCode 55) — greedy лучше DP:**

```python
# DP: O(n²) — оверинжениринг
def can_jump_dp(nums):
    dp = [False] * len(nums)
    dp[0] = True
    for i in range(1, len(nums)):
        for j in range(i):
            if dp[j] and j + nums[j] >= i:
                dp[i] = True
                break
    return dp[-1]

# Greedy: O(n) — правильный подход
def can_jump(nums):
    max_reach = 0
    for i, jump in enumerate(nums):
        if i > max_reach:
            return False
        max_reach = max(max_reach, i + jump)
    return True
```

| Сигнал | Подход |
|--------|--------|
| Оптимальная подструктура + перекрывающиеся подзадачи | DP |
| Монотонность (только расширяем/сужаем) | Greedy |
| Отсортированный массив + два условия | Two Pointers |
| Фиксированный размер "окна" | Sliding Window |
| Граф/дерево + кратчайший путь | BFS/DFS |

---

## Приоритет для intern-level

| Тема | Приоритет | Почему |
|------|-----------|--------|
| Linear DP (Q2, паттерн 1) | **Высокий** | Самый частый паттерн на собеседованиях |
| Unbounded Knapsack (Q2, паттерн 2) | **Высокий** | Coin Change, Perfect Squares — классика |
| Фреймворк анти-оверинжениринг (Q1) | **Высокий** | Прямой фидбек с X5 Tech |
| String DP (Q2, паттерн 3) | Средний | LCS, Edit Distance |
| Decision / Counting DP (Q2, паттерны 4-5) | Средний | Decode Ways, Tickets |
| Когда НЕ DP (Q3) | Средний | Показывает зрелость |

Источник: `interviews/companies/x5tech/X5TECH_TECHINTERVIEW_FEEDBACKRESULTS_RELEARNING.md`
