# Generative AI — RAG Systems

RAG-системы: архитектура, chunking, vector search, evaluation для собеседования ML/DS/AI Engineer Intern.

---

## Q1. "Архитектура RAG-системы"

> "**RAG** (Retrieval-Augmented Generation) — паттерн, где LLM генерирует ответ, опираясь на извлечённый контекст из внешней базы знаний. Решает проблемы: галлюцинации (grounding в фактах), устаревшие знания (динамические данные), domain-specific знания.
>
> Полный пайплайн:
>
> **Indexing (offline)**:
> 1. Document Ingestion — загрузка документов (PDF, HTML, markdown)
> 2. Chunking — нарезка на фрагменты (стратегии в Q3)
> 3. Embedding — векторизация чанков (sentence-transformers)
> 4. Indexing — сохранение в vector DB (Qdrant, pgvector)
>
> **Query Pipeline (online)**:
> 1. Query Embedding — векторизация запроса пользователя
> 2. Retrieval — поиск top-K релевантных чанков
> 3. Reranking — переранжирование для повышения точности
> 4. Context Assembly — сборка промпта с контекстом
> 5. LLM Generation — генерация ответа
>
> Связь с ODS.ai проектом: RAG-based PaaS для менеджеров — загрузка документации компании, поиск по ней, генерация ответов с цитатами."

| Компонент | Инструменты | Роль |
|-----------|-------------|------|
| Document Loading | LangChain loaders, Unstructured | Парсинг PDF/HTML/docx |
| Chunking | LangChain splitters, custom | Нарезка на фрагменты |
| Embedding | sentence-transformers, OpenAI Ada | Векторизация текста |
| Vector DB | Qdrant, Milvus, pgvector, Pinecone | Хранение и поиск векторов |
| Reranker | cross-encoder, Cohere Rerank | Точное переранжирование |
| LLM | GigaChat, GPT-4, Claude | Генерация ответа |

---

## Q2. "RAG vs Fine-Tuning — когда что"

> "RAG и Fine-Tuning решают разные проблемы:
>
> **RAG** — для **фактов**: динамические данные (цены, документация, новости), grounding в конкретных документах, прозрачность (можно показать источник). Не меняет веса модели → быстро обновляется (добавил документ → сразу доступен).
>
> **Fine-Tuning** — для **навыков**: стиль/тон ответов, формат вывода, доменный жаргон, поведение модели. Меняет веса → нужно переобучение при изменениях.
>
> Можно комбинировать: Fine-Tuned модель + RAG. Например: дообучили модель на доменном жаргоне (FT) и подключили базу знаний с актуальными данными (RAG).
>
> Пример: чат-бот для банка. FT — чтобы модель отвечала в формальном стиле и знала банковские термины. RAG — чтобы отвечала на вопросы по конкретным продуктам (тарифы меняются каждый месяц)."

| Критерий | RAG | Fine-Tuning |
|----------|-----|-------------|
| Что добавляет | Факты, данные | Навыки, стиль, формат |
| Обновление | Быстро (добавить документ) | Медленно (переобучение) |
| Прозрачность | Высокая (цитаты, источники) | Низкая (чёрный ящик) |
| Стоимость | Inference дороже (retrieval + LLM) | Обучение дороже |
| Галлюцинации | Снижает (grounding) | Не решает |
| Когда | Динамические данные, Q&A по документам | Стиль, формат, доменный язык |

---

## Q3. "Chunking и стратегии нарезки"

> "Chunking — ключевой этап RAG pipeline. Плохой chunking → плохой retrieval → плохие ответы.
>
> **Fixed-size** (по символам/токенам): просто, но ломает предложения на границах. Chunk size 500-1000 токенов.
>
> **Recursive** (LangChain `RecursiveCharacterTextSplitter`): сначала делит по заголовкам `\n\n`, потом по абзацам `\n`, потом по предложениям `. `, потом по символам. Сохраняет структуру документа.
>
> **Semantic** (по смыслу): разбивает, когда эмбеддинг соседних предложений сильно отличается. Самый точный, но дорогой.
>
> **Chunk overlap** — перекрытие между чанками (50-200 токенов). Зачем: контекст на стыках не теряется. 'Градиентный бустинг обучается последовательно.' может оказаться в конце одного чанка и начале другого.
>
> Размер чанка — tradeoff:
> - Маленькие (200-500): точный поиск, но потеря контекста.
> - Большие (1000-2000): больше контекста, но перегружают context window LLM и снижают точность retrieval."

| Стратегия | Как работает | Плюсы | Минусы |
|-----------|-------------|-------|--------|
| Fixed-size | По N токенов | Просто, предсказуемо | Ломает предложения |
| Recursive | По заголовкам → абзацам → предложениям | Сохраняет структуру | Размер непредсказуем |
| Semantic | По изменению эмбеддинга | Самый точный | Дорого (нужны embeddings) |
| По документу | Одна страница = один чанк | Нет потери контекста | Может быть слишком большим |

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,        # максимальный размер чанка (символов)
    chunk_overlap=100,     # перекрытие между чанками
    separators=["\n\n", "\n", ". ", " ", ""],  # приоритет разделителей
)

chunks = splitter.split_text(document_text)
# Каждый чанк <= 500 символов с перекрытием 100
```

---

## Q4. "Vector Search: Dense vs Sparse, Hybrid Search, Reranking"

> "**Sparse retrieval** (BM25, TF-IDF): лексический поиск по точным совпадениям слов. Хорошо для имён, артикулов, специфических терминов. Не понимает синонимы ('автомобиль' != 'машина').
>
> **Dense retrieval** (BERT/sentence-transformers embeddings): семантический поиск по смыслу. Понимает синонимы и перифразы. Хуже для точных совпадений (имена, коды).
>
> **Hybrid Search** = sparse + dense: комбинируем оба подхода. Reciprocal Rank Fusion (RRF) или weighted sum. Лучший подход в большинстве случаев.
>
> **Reranking** (двухэтапный поиск):
> 1. **Bi-encoder** (быстрый): закодировал документы заранее, поиск по dot product. Миллионы → top-100.
> 2. **Cross-encoder** (точный, но медленный): берёт пару (query, document) целиком, выдаёт score. Top-100 → top-10.
>
> **ANN алгоритмы**: точный поиск по всем векторам — $O(n)$. Приближённые (ANN): HNSW (графовый, самый популярный), IVF (кластеризация), PQ (сжатие). HNSW даёт recall >95% при скорости $O(\log n)$.
>
> **Vector DBs**: Qdrant (Rust, production-ready), Milvus (distributed), pgvector (PostgreSQL extension — удобно если уже есть PG), Pinecone (managed)."

| Тип поиска | Что находит | Скорость | Пример |
|------------|-------------|----------|--------|
| **Sparse (BM25)** | Точные совпадения слов | Быстро | 'артикул SKU-12345' |
| **Dense (embeddings)** | Семантически похожее | Средне | 'как вернуть товар' ~ 'процедура возврата' |
| **Hybrid** | И точное, и семантическое | Средне | Лучший выбор для production |
| **Cross-encoder reranking** | Точное переранжирование top-K | Медленно | Финальная фильтрация |

| Vector DB | Язык | Особенность | Когда |
|-----------|------|-------------|-------|
| **Qdrant** | Rust | Filtering, payload, production-ready | Default choice |
| **pgvector** | C (PG extension) | Интеграция с PostgreSQL | Уже есть PG в стеке |
| **Milvus** | Go | Distributed, масштабирование | Миллиарды векторов |
| **Pinecone** | Managed | Serverless, zero-ops | Быстрый старт |

---

## Q5. "Evaluation RAG-систем"

> "Классические метрики (BLEU, ROUGE) плохо работают для GenAI: сравнивают n-граммы, а не смысл. Нужны специализированные метрики:
>
> **RAGAS framework** — стандарт оценки RAG:
> - **Faithfulness**: ответ основан на контексте? Нет галлюцинаций? LLM проверяет каждое утверждение.
> - **Context Precision**: релевантные чанки в top-K? Нет мусора?
> - **Context Recall**: все нужные факты извлечены? Нет пропусков?
> - **Answer Relevance**: ответ отвечает на вопрос? (а не на что-то другое)
>
> **LLM-as-a-judge**: GPT-4 / Claude оценивает ответы по шкале 1-5. Дешевле human eval, но может иметь biases (предпочитает свой стиль, длинные ответы).
>
> **Human evaluation** — gold standard. Дорого, но незаменимо для финальной валидации. Метрики: correctness, helpfulness, harmlessness.
>
> Связь с ODS.ai проектом: в RAG-based PaaS используем RAGAS для автоматической оценки качества ответов + human eval на тестовой выборке."

| Метрика | Что измеряет | Как | Уровень |
|---------|-------------|-----|---------|
| **Faithfulness** | Нет галлюцинаций | LLM проверяет claims vs context | Ответ |
| **Context Precision** | Качество retrieval (precision) | Доля релевантных чанков в top-K | Retrieval |
| **Context Recall** | Полнота retrieval | Все ли факты извлечены | Retrieval |
| **Answer Relevance** | Ответ по теме | LLM оценивает соответствие вопросу | Ответ |
| **LLM-as-a-judge** | Общее качество | GPT-4 ставит оценку 1-5 | End-to-end |

```python
from ragas import evaluate
from ragas.metrics import faithfulness, context_precision, context_recall, answer_relevancy

results = evaluate(
    dataset,  # вопросы + ground truth + контексты + ответы
    metrics=[
        faithfulness,        # нет галлюцинаций?
        context_precision,   # retrieval точный?
        context_recall,      # retrieval полный?
        answer_relevancy,    # ответ по теме?
    ],
)

print(results)
# {'faithfulness': 0.87, 'context_precision': 0.82,
#  'context_recall': 0.75, 'answer_relevancy': 0.91}
```

---

## Приоритет для intern-level

| Тема | Приоритет | Почему |
|------|-----------|--------|
| Архитектура RAG (Q1) | **Высокий** | Основа — спрашивают на каждом GenAI-собесе |
| RAG vs Fine-Tuning (Q2) | **Высокий** | Показывает понимание trade-offs |
| Chunking (Q3) | Средний | Практический навык |
| Vector Search + Reranking (Q4) | Средний | Углубление retrieval |
| Evaluation (Q5) | Средний | Важно для production RAG |
