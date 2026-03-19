# Продвинутые паттерны — Interview Prep

Фреймворки, multi-agent, agentic RAG, voice AI для собеседования ML/DS/AI Engineer Intern.

---

## Q5. "Какие фреймворки для агентов знаешь?"

| Фреймворк | Особенности | Когда использовать |
|-----------|-------------|-------------------|
| **LangChain** | Самый популярный, chains + agents + tools | Быстрый прототип, большая экосистема |
| **LlamaIndex** | Фокус на RAG + data connectors | Работа с документами, knowledge base |
| **Pydantic-AI** | Type-safe, structured output, dependency injection | Production-grade, Python-first |
| **CrewAI** | Multi-agent: роли, задачи, delegation | Несколько агентов работают вместе |
| **AutoGen** (Microsoft) | Multi-agent conversations | Исследовательские задачи |
| **OpenAI Assistants API** | Hosted решение, code interpreter + retrieval | Если не хочешь инфраструктуру |

> "Я использовал Pydantic-AI в hr-breaker — agent для оптимизации резюме. Pydantic-AI хорош тем, что результат агента — это Pydantic model, строго типизированный. Плюс поддержка любого LLM-провайдера через LiteLLM."

---

## Q6. "Что такое multi-agent system?"

> "Несколько агентов с разными ролями работают вместе. Каждый специализируется на своей задаче.
>
> Паттерны:
> 1. **Hierarchical** — один 'менеджер' делегирует задачи sub-agents
> 2. **Sequential** — output одного агента → input следующего (pipeline)
> 3. **Debate** — два агента спорят, третий выбирает лучший ответ
>
> Пример для X5:
> - **Router agent**: определяет тип запроса клиента
> - **Order agent**: работает с заказами (вызывает order API)
> - **Store agent**: ищет магазины, часы работы
> - **Escalation agent**: переводит на человека, если не справился
>
> В hr-breaker я фактически сделал multi-agent pipeline: один agent генерирует резюме, другой прогоняет через фильтры, при отклонении — итерация."

---

## Q9. "Расскажи про Agentic RAG"

> "Обычный RAG: запрос → retrieval → generation (один шаг). Agentic RAG — агент сам решает, **когда и как** искать:
>
> 1. **Routing** — агент выбирает источник (vector DB, SQL database, web search, API)
> 2. **Multi-step retrieval** — если первый поиск недостаточен, уточняет запрос и ищет снова
> 3. **Self-reflection** — проверяет, достаточно ли информации для ответа
> 4. **Query decomposition** — сложный вопрос → несколько простых подзапросов
>
> Пример: 'Сравни условия доставки в Москве и Питере' → агент разбивает на 2 запроса, ищет отдельно, агрегирует результат.
>
> Это то, что делает LlamaIndex хорошо — Sub-Question Query Engine."

---

## Q10. "Voice AI agent pipeline — как работает?"

> "Голосовой агент X5 (или любой) — это pipeline:
>
> 1. **ASR** (Automatic Speech Recognition): голос → текст. Whisper (OpenAI), Google Speech-to-Text, Yandex SpeechKit
> 2. **NLU** (Natural Language Understanding): понимание интента + извлечение entities. Может быть LLM-based
> 3. **Dialog Manager**: решает, что делать: ответить, вызвать tool, уточнить
> 4. **LLM**: генерирует текстовый ответ
> 5. **TTS** (Text-to-Speech): текст → голос. Edge-TTS, ElevenLabs, Yandex TTS
>
> Latency критична: человек ждёт ответа <1 секунду. Решения: streaming ASR, streaming TTS, кэширование частых ответов, маленькие модели для routing.
>
> Мой опыт: в Text2Brainrot использовал Edge-TTS для генерации озвучки — знаю TTS-часть pipeline. ASR — следующий шаг, который готов освоить."
