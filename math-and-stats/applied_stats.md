# Прикладная статистика в ML — Interview Prep

Корреляция, A/B тесты, оценки параметров, bias-variance для собеседования ML/DS/AI Engineer Intern.

---

## Q9. "Correlation vs Causation"

> "**Корреляция** — линейная связь между двумя переменными. Pearson $r \in [-1, 1]$. Correlation $\neq$ causation: продажи мороженого коррелируют с числом утоплений, но мороженое не убивает людей — обе переменные вызваны жарой (конфаундер).
>
> Чтобы доказать causation нужен:
> 1. **Рандомизированный эксперимент** (A/B тест) — устраняем конфаундеры
> 2. **Causal inference** (Pearl's do-operator, backdoor criterion) — когда эксперимент невозможен
>
> В ML это критично: модель находит корреляции, но для принятия решений нужна каузальность. Пример: модель скоринга может найти, что 'цвет телефона' коррелирует с дефолтом — но это не причина, а proxy."

---

## Q10. "Что такое A/B тест? Расскажи полный pipeline"

> "A/B тест — controlled experiment для проверки гипотезы.
>
> **Pipeline:**
> 1. **Гипотеза**: 'Новый промпт для голосового агента увеличит конверсию' ($H_1$)
> 2. **Метрика**: primary (конверсия), guardrail (time-to-resolution, NPS)
> 3. **Размер выборки**: power analysis — нужно $N$ пользователей для обнаружения MDE (minimum detectable effect) с power=0.8
> 4. **Рандомизация**: случайно делим пользователей 50/50 (или 90/10 для рискованных)
> 5. **Запуск**: собираем данные минимум 1-2 недели (учитываем день недели, сезонность)
> 6. **Анализ**: t-test/z-test, p-value < 0.05 → статистически значимо
> 7. **Решение**: если значимо И практически значимо (effect size достаточный) → раскатываем
>
> **Ошибки:** peeking (смотреть p-value до набора выборки), multiple testing (много метрик без коррекции Bonferroni), selection bias, novelty effect."

---

## Q11. "Что такое MLE и MAP?"

> "**MLE (Maximum Likelihood Estimation)** — находим параметры $\theta$, которые максимизируют вероятность наблюдённых данных: $\theta_{\text{MLE}} = \arg\max P(\text{data} \mid \theta)$. Не учитывает prior knowledge.
>
> **MAP (Maximum A Posteriori)** — добавляем prior: $\theta_{\text{MAP}} = \arg\max P(\theta \mid \text{data}) = \arg\max P(\text{data} \mid \theta) \cdot P(\theta)$. Эквивалентно MLE + regularization: если prior — Gaussian → L2 regularization, если Laplace → L1.
>
> Пример: оценка вероятности дефолта. MLE на 10 наблюдениях: 3 дефолта → $P = 0.3$. MAP с prior 'обычно дефолтов ~5%' → оценка 'подтянется' к 0.05, особенно при малом $n$. При большом $n$ — MLE и MAP сходятся."

---

## Q12. "Bias-Variance Tradeoff через призму статистики"

> "Ожидаемая ошибка модели $= \text{Bias}^2 + \text{Variance} + \text{Irreducible Noise}$.
>
> **Bias** — систематическая ошибка, модель слишком простая (underfitting). Высокий bias: линейная регрессия на нелинейных данных.
>
> **Variance** — разброс предсказаний при разных обучающих выборках. Высокая variance: дерево глубины 100 (overfitting) — чуть другие данные → совсем другие предсказания.
>
> **Tradeoff**: увеличиваем сложность → bias ↓ variance ↑. Оптимум — в точке минимальной суммарной ошибки.
>
> **Как бороться с high variance:** regularization, ensemble (RF усредняет деревья → variance ↓), больше данных, dropout.
> **Как бороться с high bias:** более сложная модель, feature engineering, убрать regularization."
