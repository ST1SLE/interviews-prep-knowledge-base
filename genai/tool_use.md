# Инструменты и интеграция — Interview Prep

Function calling, логика вызова tools, structured output, MCP для собеседования ML/DS/AI Engineer Intern.

---

## Q3. "Что такое Function Calling / Tool Use?"

> "Function Calling — механизм, позволяющий LLM вызывать внешние функции. Ты описываешь доступные функции (имя, параметры, описание) в системном промпте или через API. LLM решает, когда вызвать функцию, и возвращает structured JSON с именем функции и аргументами. Твой код выполняет функцию и возвращает результат обратно в LLM.
>
> Поддерживается: OpenAI (function calling), Anthropic (tool use), Google (function declarations), GigaChat (functions).
>
> Пример: агент для X5 может иметь tools: `check_order_status(order_id)`, `find_nearest_store(address)`, `calculate_delivery_time(from, to)`. Клиент спрашивает 'где мой заказ?' → агент вызывает `check_order_status`."

```python
# OpenAI-style function definition
tools = [{
    "type": "function",
    "function": {
        "name": "check_order_status",
        "description": "Проверить статус заказа по номеру",
        "parameters": {
            "type": "object",
            "properties": {
                "order_id": {"type": "string", "description": "Номер заказа"}
            },
            "required": ["order_id"]
        }
    }
}]
```

---

## Q7. "Как агент решает, когда вызвать tool?"

> "LLM решает на основе:
> 1. **System prompt** — описание когда и какие tools использовать
> 2. **Tool descriptions** — чем лучше описание, тем точнее вызов
> 3. **Few-shot examples** — примеры правильных вызовов в prompt
>
> Проблемы:
> - **Hallucinated tools** — агент 'выдумывает' инструмент, которого нет
> - **Wrong parameters** — правильный tool, но неправильные аргументы
> - **Tool overuse** — вызывает tool когда мог ответить сам
> - **Tool underuse** — отвечает из головы когда нужно было проверить
>
> Решения: strict JSON schema, enum для параметров, validation перед выполнением, fallback на 'извините, не могу выполнить'."

---

## Q11. "Structured Output — зачем и как?"

> "LLM по умолчанию возвращает свободный текст. Для агента нужен structured output — JSON, чтобы программно обработать результат.
>
> Подходы:
> 1. **Prompt engineering** — 'верни строго JSON формат: {...}' — ненадёжно
> 2. **JSON mode** — у OpenAI/Anthropic есть response_format='json' — гарантирует валидный JSON, но не schema
> 3. **Structured outputs** — описываешь Pydantic model, LLM гарантированно возвращает объект этой схемы (OpenAI Structured Outputs, Pydantic-AI)
> 4. **Constrained decoding** — на уровне inference ограничиваем токены (Outlines, LMQL)
>
> В hr-breaker я использую Pydantic-AI: агент возвращает `ResumeData` — строго типизированную модель с полями name, experience, skills и т.д."

---

## Q13. "Что такое MCP (Model Context Protocol)?"

> "MCP — открытый протокол (Anthropic, 2024) для подключения инструментов к LLM. Архитектура: MCP Server предоставляет tools, resources и prompts. MCP Client — агент или IDE — подключается к серверам по JSON-RPC. Ключевое отличие от function calling: единый стандарт, динамическое обнаружение capabilities, переиспользование серверов. Аналогия — USB для AI-агентов.
>
> В hr-breaker я использую FastMCP для интеграции инструментов оптимизации резюме."

### Архитектура

```
┌─────────────┐     JSON-RPC      ┌─────────────┐
│  MCP Client │ ◄───────────────► │  MCP Server  │
│  (Агент/IDE)│                   │  (Провайдер) │
└─────────────┘                   └─────────────┘
     Claude Code                    Файловая система
     Cursor                         База данных
     Свой агент                     Jira / Slack API
```

**Транспорт:** JSON-RPC 2.0 поверх stdio (локально) или SSE/HTTP (удалённо).

### 3 типа capabilities

| Capability | Кто решает использовать | Пример |
|---|---|---|
| **Tools** | LLM (модель решает когда вызвать) | `read_file(path)`, `query_db(sql)` |
| **Resources** | Клиент/пользователь (контекст) | Файлы, записи БД, документы |
| **Prompts** | Пользователь (шаблоны) | Шаблон code review, анализа |

### MCP vs Function Calling

| | **Function Calling** | **MCP** |
|---|---|---|
| **Стандарт** | Каждый провайдер свой | Единый протокол |
| **Discovery** | Tools зашиты в prompt | Server объявляет capabilities |
| **Переиспользование** | Переписываешь под каждый фреймворк | Один server → любой client |
| **Композиция** | Один список tools | Несколько серверов |

### Пример: FastMCP Server

```python
from fastmcp import FastMCP

mcp = FastMCP("order-service")

@mcp.tool()
def check_order_status(order_id: str) -> str:
    """Проверить статус заказа по номеру"""
    return f"Заказ {order_id}: доставляется, ETA 2 часа"

@mcp.resource("orders://{order_id}")
def get_order(order_id: str) -> str:
    """Получить полную информацию о заказе"""
    return f"Order {order_id}: items=[...], total=1500₽"

mcp.run()
```

Источник: `interviews/companies/x5tech/X5TECH_TECHINTERVIEW_FEEDBACKRESULTS_RELEARNING.md`
