# Docker & Docker Compose — Interview Prep

Dockerfile, Compose, multi-stage build, кэширование слоёв для собеседования ML/DS/AI Engineer Intern.

---

## Q1. "Dockerfile — основные инструкции"

> "Dockerfile — набор инструкций для сборки образа. Ключевые: FROM (базовый образ), COPY (файлы), RUN (команды при сборке), CMD (команда при запуске). Важно: порядок слоёв влияет на кэширование — requirements.txt копирую первым, код последним. Multi-stage build для минимизации production-образа."

| Инструкция | Что делает | Пример |
|---|---|---|
| `FROM` | Базовый образ (первая строка) | `FROM python:3.11-slim` |
| `WORKDIR` | Рабочая директория внутри контейнера | `WORKDIR /app` |
| `COPY` | Копирует файлы с хоста в контейнер | `COPY requirements.txt .` |
| `RUN` | Выполняет команду при **сборке** образа | `RUN pip install -r requirements.txt` |
| `CMD` | Команда по умолчанию при **запуске** контейнера | `CMD ["uvicorn", "main:app"]` |
| `ENTRYPOINT` | Фиксированная точка входа (CMD → аргументы) | `ENTRYPOINT ["python"]` |
| `EXPOSE` | Документирует порт (**не открывает!**) | `EXPOSE 8000` |
| `ENV` | Переменная окружения (сборка + runtime) | `ENV PYTHONUNBUFFERED=1` |
| `ARG` | Переменная только для сборки | `ARG VERSION=1.0` |

### CMD vs ENTRYPOINT

```dockerfile
# CMD — команда по умолчанию. Можно переопределить при docker run
CMD ["uvicorn", "main:app", "--port", "8000"]
# docker run myimage                    → uvicorn main:app --port 8000
# docker run myimage python test.py     → python test.py (CMD перезаписан)

# ENTRYPOINT — фиксированная точка входа. CMD становится аргументами
ENTRYPOINT ["python"]
CMD ["main.py"]
# docker run myimage                    → python main.py
# docker run myimage test.py            → python test.py (ENTRYPOINT остаётся)
```

**Правило:** ENTRYPOINT для "этот контейнер всегда запускает X". CMD для "по умолчанию запусти X, но можно заменить".

### COPY vs ADD

| | **COPY** | **ADD** |
|---|---|---|
| Копирование файлов | Да | Да |
| Распаковка tar.gz | Нет | Да (автоматически) |
| URL | Нет | Да (скачивает) |
| **Рекомендация** | **Использовать всегда** | Только если нужна распаковка |

### Порядок слоёв (кэширование)

```dockerfile
# ПРАВИЛЬНО: requirements меняются редко → слой кэшируется
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["uvicorn", "main:app"]

# ПЛОХО: при любом изменении кода pip install заново
FROM python:3.11-slim
COPY . .
RUN pip install -r requirements.txt
```

**Принцип:** то что меняется редко — наверх, то что часто — вниз.

---

## Q2. "Multi-stage build и best practices"

> "Multi-stage build: первая стадия — сборка (компиляторы, dev-зависимости), вторая — production (только результат). Образ в 2-5 раз меньше. Плюс: не от root, --no-cache-dir, фиксированные теги."

```dockerfile
# Стадия 1: сборка
FROM python:3.11-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt
COPY . .

# Стадия 2: production
FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /install /usr/local
COPY --from=builder /app ./
RUN useradd -m appuser && chown -R appuser /app
USER appuser
EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Типичные ошибки в Dockerfile

| Ошибка | Почему плохо | Правильно |
|---|---|---|
| `COPY . .` перед `RUN pip install` | Кэш ломается при любом изменении кода | Сначала COPY requirements.txt |
| Запуск от `root` | Escape из контейнера = root на хосте | `useradd` + `USER appuser` |
| Без `--no-cache-dir` | pip-кэш остаётся в образе (+100-500MB) | `--no-cache-dir` |
| Много `RUN` подряд | Каждый RUN = новый слой | Объединять через `&&` |
| `ADD` вместо `COPY` | Неявное поведение (распаковка, URL) | `COPY` для файлов |
| `latest` тег | Непредсказуемый результат сборки | `python:3.11-slim` |

---

## Q3. "Docker Compose — оркестрация сервисов"

> "Docker Compose — оркестрация нескольких контейнеров в одном YAML-файле. Описываю services, ports, volumes, depends_on. В Text2Brainrot 4 сервиса: API (FastAPI) + Worker (Celery) + PostgreSQL + Redis."

### Пример: ML-сервис с БД и очередью

```yaml
services:
  api:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb
      - REDIS_URL=redis://redis:6379
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    restart: unless-stopped

  worker:
    build: .
    command: celery -A app.worker worker --loglevel=info
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis

  db:
    image: postgres:16
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=mydb
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine

volumes:
  postgres_data:
```

### Ключевые директивы

| Директива | Что делает | Пример |
|---|---|---|
| `build` | Собрать из Dockerfile | `build: .` |
| `image` | Готовый образ с Docker Hub | `image: postgres:16` |
| `ports` | Проброс портов (host:container) | `"8000:8000"` |
| `volumes` | Persistent storage / bind mount | `postgres_data:/var/lib/...` |
| `environment` | Переменные окружения | `DATABASE_URL=...` |
| `depends_on` | Порядок запуска (**не ждёт готовности!**) | `depends_on: [db]` |
| `healthcheck` | Проверка готовности сервиса | `pg_isready`, `curl` |
| `command` | Переопределить CMD | `celery -A ...` |
| `restart` | Политика перезапуска | `unless-stopped` |

### depends_on — важный нюанс

`depends_on` гарантирует **порядок запуска**, но **не ждёт готовности** сервиса. БД может быть "запущена" но ещё не принимать подключения.

Решение: `healthcheck` + `condition: service_healthy`.

### Volumes: named vs bind mount

| Тип | Синтаксис | Когда |
|-----|-----------|-------|
| Named volume | `postgres_data:/var/lib/postgresql/data` | Persistent data (БД) |
| Bind mount | `./app:/app/app` | Разработка (hot reload) |
| Read-only | `./config:/app/config:ro` | Конфиги |

---

## Приоритет для intern-level

| Тема | Приоритет | Почему |
|------|-----------|--------|
| Dockerfile базовые инструкции (Q1) | **Высокий** | Спрашивают почти везде |
| Порядок слоёв, кэширование (Q1) | **Высокий** | Частый вопрос "найди ошибку" |
| Multi-stage build (Q2) | Средний | Показывает production-опыт |
| Docker Compose (Q3) | Средний | Text2Brainrot использует |
| Типичные ошибки (Q2) | Средний | Формат "code review" на собесе |

Источник: `interviews/companies/x5tech/X5TECH_TECHINTERVIEW_FEEDBACKRESULTS_RELEARNING.md`
