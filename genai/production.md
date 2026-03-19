# Production-ready агенты — Interview Prep

Guardrails, evaluation, error handling для собеседования ML/DS/AI Engineer Intern.

---

## Q8. "Guardrails и safety для агентов"

> "Агент может выполнять действия в реальном мире → нужны ограничения:
>
> 1. **Input guardrails** — фильтрация входящих запросов (prompt injection detection, PII detection, topic blocking)
> 2. **Output guardrails** — проверка ответа перед отправкой (toxicity check, factual consistency, format validation)
> 3. **Action guardrails** — ограничения на tools (rate limiting, whitelist разрешённых действий, human-in-the-loop для критичных операций)
>
> Prompt injection — главная угроза: пользователь пытается 'перепрограммировать' агента через хитрый input. Защита: отделять system prompt от user input, instruction hierarchy, input sanitization.
>
> В контексте X5: голосовой агент не должен раскрывать внутренние инструкции, отменять заказы без подтверждения, или обещать скидки, которых нет."

---

## Q12. "Evaluation агентов — как понять, что агент работает хорошо?"

> "Оценивать агентов сложнее, чем обычные модели:
>
> 1. **Task completion rate** — % задач, решённых полностью (главная метрика)
> 2. **Tool accuracy** — правильно ли вызывает tools (precision/recall по tool calls)
> 3. **Trajectory analysis** — оптимален ли путь? (лишние шаги = плохо)
> 4. **Cost per task** — сколько токенов/API-вызовов на одну задачу
> 5. **Latency** — время от запроса до ответа
> 6. **User satisfaction** — NPS, CSAT (для продакшн-агентов)
>
> Frameworks: RAGAS (для RAG-части), LangSmith (трейсинг), Pydantic Logfire (observability).
>
> В hr-breaker я использую внутренние фильтры как evaluation: ATS simulation, keyword matching, hallucination detection — агент считается успешным, только если пройдёт все фильтры."

---

## Q15. "Расскажи про error handling в агентах"

> "Агент работает в цикле → ошибки неизбежны. Стратегии:
>
> 1. **Retry with different approach** — tool вернул ошибку → агент пробует другой tool или переформулирует запрос
> 2. **Graceful degradation** — не могу получить точные данные → отвечаю на основе того, что есть + предупреждаю
> 3. **Max iterations limit** — не более N шагов, иначе бесконечный цикл (и расходы)
> 4. **Human escalation** — если не справился за N попыток → передать человеку
> 5. **Fallback responses** — заготовленные ответы для типичных ошибок
>
> В hr-breaker: если LLM генерирует резюме с hallucination → фильтр ловит → агент получает feedback → повторяет генерацию (до 3 итераций). Если всё равно не проходит → возвращает ошибку пользователю."
