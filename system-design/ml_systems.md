# ML System Design — Interview Prep

Проектирование ML-систем для собеседований ML/DS/AI Engineer Intern.

---

## Q1. "ML pipeline — от данных до продакшна"

> "ML pipeline — это последовательность шагов от сырых данных до модели в продакшне. Основные этапы:
>
> 1. **Data Collection** — сбор данных из источников (БД, API, логи). Важно: версионирование данных (DVC), схема хранения.
> 2. **Data Validation** — проверка качества: пропуски, выбросы, дрифт распределений. Great Expectations / Pandera для автоматических проверок.
> 3. **Feature Engineering** — создание признаков из сырых данных. Кодирование категорий, нормализация, агрегаты. Здесь же feature selection.
> 4. **Training** — обучение модели с cross-validation. Подбор гиперпараметров (Optuna). Логирование экспериментов (MLflow / W&B).
> 5. **Evaluation** — оценка на hold-out: метрики + статистическая значимость. Сравнение с baseline и текущей продакшн-моделью.
> 6. **Deployment** — упаковка модели (Docker), деплой (Kubernetes / облако), API endpoint.
> 7. **Monitoring** — отслеживание drift, degradation метрик, alerting.
> 8. **Retrain** — автоматическое или ручное переобучение при деградации.
>
> В Альфа-Банке я строил кредитный скоринг на CatBoost: данные из PostgreSQL → feature engineering (агрегаты по транзакциям, кодирование категорий) → обучение CatBoost с Optuna → оценка AUC/Gini на hold-out → деплой модели. Это классический пример supervised ML pipeline для бинарной классификации."

| Этап | Инструменты | Артефакт |
|------|-------------|----------|
| Data Collection | PostgreSQL, S3, Kafka | Raw dataset |
| Data Validation | Great Expectations, Pandera | Validation report |
| Feature Engineering | pandas, Spark, Feature Store | Feature matrix |
| Training | CatBoost, scikit-learn, PyTorch + Optuna | Model artifact (.pkl / .pt) |
| Experiment Tracking | MLflow, W&B | Run logs, metrics |
| Evaluation | scikit-learn metrics, scipy.stats | Evaluation report |
| Deployment | Docker, FastAPI, Kubernetes | API endpoint |
| Monitoring | Prometheus, Grafana, Evidently | Dashboards, alerts |

```yaml
# Пример: MLflow tracking в pipeline
mlflow:
  experiment_name: "credit_scoring_v2"
  tracking_uri: "http://mlflow-server:5000"
  log_params: [learning_rate, depth, iterations, l2_leaf_reg]
  log_metrics: [auc_train, auc_val, gini, ks_statistic]
  log_artifacts: [model.cbm, feature_importance.png]
```

---

## Q2. "Feature Store — зачем и как"

> "Feature Store — это централизованное хранилище признаков, которое решает две главные проблемы:
>
> 1. **Consistency** — одни и те же фичи для training и serving (нет training-serving skew).
> 2. **Reusability** — фичи, посчитанные одной командой, доступны всем (не пересчитывать заново).
>
> Два компонента:
> - **Offline Store** — для batch training. Хранит исторические значения фич (Parquet, Hive, BigQuery). Поддерживает point-in-time correctness — при формировании train-набора берём фичи, которые были доступны на момент события, а не будущие значения. Без этого — data leakage.
> - **Online Store** — для low-latency serving ($< 10$ ms). Key-value хранилище (Redis, DynamoDB). При inference запрашиваем свежие фичи по entity_id.
>
> Feast — open-source Feature Store: определяешь фичи в Python, Feast берёт данные из offline store, материализует в online store, и сервит через API.
>
> Когда нужен: несколько моделей используют одни и те же фичи, есть training-serving skew, команда > 3 ML-инженеров.
> Когда overkill: один ML-инженер, одна модель, все фичи считаются на лету. В Альфа-Банке Feature Store был бы полезен — разные модели скоринга использовали похожие агрегаты по транзакциям, и пересчёт занимал время."

| Аспект | Offline Store | Online Store |
|--------|---------------|--------------|
| Latency | Минуты | $< 10$ ms |
| Объём | Терабайты | Гигабайты (последние значения) |
| Формат | Parquet, Hive, BigQuery | Redis, DynamoDB |
| Использование | Training, batch prediction | Real-time serving |
| Обновление | Batch (ежедневно / ежечасно) | Materialization из offline |

**Point-in-time correctness — почему это критично:**

$$\text{features}(t) = \text{values\_available\_at}(t), \quad \text{NOT} \quad \text{values\_known\_later}$$

```python
# Feast: определение Feature View
from feast import Entity, FeatureView, Field, FileSource
from feast.types import Float32, Int64
from datetime import timedelta

# Сущность — клиент
customer = Entity(name="customer_id", join_keys=["customer_id"])

# Источник данных (offline)
customer_stats_source = FileSource(
    path="data/customer_stats.parquet",
    timestamp_field="event_timestamp",
)

# Feature View — набор фич с point-in-time correctness
customer_stats_fv = FeatureView(
    name="customer_stats",
    entities=[customer],
    ttl=timedelta(days=1),  # фичи актуальны 1 день
    schema=[
        Field(name="avg_transaction_amount", dtype=Float32),
        Field(name="transaction_count_30d", dtype=Int64),
        Field(name="days_since_last_transaction", dtype=Int64),
    ],
    source=customer_stats_source,
)
```

---

## Q3. "Model Serving — batch vs online"

> "Два основных паттерна serving:
>
> **Batch prediction** — модель запускается по расписанию (cron / Airflow), обрабатывает все данные разом, результаты сохраняются в БД. Плюсы: просто, дёшево, можно использовать Spark для больших объёмов. Минус: нет real-time — данные устаревают между запусками. Пример: ночной пересчёт кредитных рейтингов.
>
> **Online prediction** — модель развёрнута как сервис, отвечает на запросы в реальном времени. Latency $< 100$ ms. Реализация: FastAPI (для классических ML), TorchServe / Triton Inference Server (для DL моделей). gRPC быстрее REST за счёт бинарной сериализации (Protobuf), но REST проще для дебага.
>
> В Text2Brainrot я использовал FastAPI для online serving: запрос → генерация текста через GigaChat → озвучка Edge-TTS. Тяжёлые задачи (генерация видео) уходили в Celery + Redis как async tasks, а FastAPI сразу возвращал task_id. Это паттерн: лёгкий запрос — sync, тяжёлый — async через очередь."

| Аспект | Batch Prediction | Online Prediction |
|--------|-----------------|-------------------|
| Latency | Минуты—часы | $< 100$ ms |
| Когда обновляется | По расписанию (cron) | На каждый запрос |
| Инфраструктура | Spark, Airflow | FastAPI, Kubernetes |
| Стоимость | Дешевле (ресурсы по расписанию) | Дороже (always-on) |
| Примеры | Рекомендации на главной, рассылки | Кредитный скоринг, fraud detection |

| Serving Framework | Протокол | Модели | Когда использовать |
|-------------------|----------|--------|-------------------|
| **FastAPI** | REST | scikit-learn, CatBoost, любые | Классические ML, микросервисы |
| **TorchServe** | REST / gRPC | PyTorch | DL модели, batching |
| **Triton** | REST / gRPC | PyTorch, TF, ONNX, TensorRT | Мультимодельный serving, GPU |
| **BentoML** | REST / gRPC | Любые | Удобная упаковка + деплой |

```python
# FastAPI serving — паттерн из Text2Brainrot
from fastapi import FastAPI, BackgroundTasks
from celery import Celery
import pickle
import numpy as np

app = FastAPI()
celery_app = Celery("tasks", broker="redis://localhost:6379/0")

# Лёгкий запрос — sync (скоринг через CatBoost, <50ms)
@app.post("/predict")
def predict(features: dict):
    model = pickle.load(open("model.pkl", "rb"))
    X = np.array([list(features.values())])
    proba = model.predict_proba(X)[0, 1]
    return {"score": float(proba)}

# Тяжёлый запрос — async через Celery + Redis
@celery_app.task
def generate_video(text: str):
    # GigaChat → Edge-TTS → ffmpeg (минуты)
    ...

@app.post("/generate")
def generate(text: str):
    task = generate_video.delay(text)
    return {"task_id": task.id, "status": "processing"}

@app.get("/status/{task_id}")
def get_status(task_id: str):
    result = celery_app.AsyncResult(task_id)
    return {"status": result.status, "result": result.result}
```

---

## Q4. "Monitoring и drift detection"

> "После деплоя модель деградирует — мир меняется, а модель фиксирована. Три типа drift:
>
> 1. **Data drift** (covariate shift) — изменилось распределение входных фич. Пример: после пандемии паттерн транзакций изменился → фичи модели скоринга стали выглядеть иначе.
> 2. **Concept drift** — изменилась связь между фичами и таргетом. $P(y|X)$ поменялся. Пример: раньше высокий доход = низкий риск, после кризиса — не обязательно.
> 3. **Feature drift** — конкретная фича сломалась (баг в pipeline, изменился формат данных upstream).
>
> Детекция:
> - **KS-test** (Kolmogorov-Smirnov) — сравниваем распределение фичи в training vs production. $p < 0.05$ → drift.
> - **PSI** (Population Stability Index) — $PSI = \sum (p_i - q_i) \cdot \ln(p_i / q_i)$. PSI $< 0.1$ — стабильно, $0.1$—$0.25$ — умеренный drift, $> 0.25$ — значительный.
> - **Performance monitoring** — следим за метриками модели на свежих данных (если есть ground truth с задержкой).
>
> Alerting pipeline: Evidently AI считает drift-метрики → результаты в Prometheus → Grafana dashboard + PagerDuty alerts.
>
> Переобучать когда: PSI $> 0.25$ на ключевых фичах, или бизнес-метрика (конверсия, default rate) упала больше порога. В Альфа-Банке это было актуально: макроэкономика меняется → распределение транзакций дрифтит → модель скоринга требует переобучения."

**PSI формула:**

$$PSI = \sum_{i=1}^{N} (p_i - q_i) \cdot \ln\left(\frac{p_i}{q_i}\right)$$

где $p_i$ — доля в training bin, $q_i$ — доля в production bin.

| Метрика | Что измеряет | Порог | Инструмент |
|---------|--------------|-------|------------|
| **KS-test** | Расхождение распределений фичи | $p < 0.05$ | `scipy.stats.ks_2samp` |
| **PSI** | Стабильность распределения | $> 0.25$ = alarm | Evidently, custom |
| **Accuracy / AUC drop** | Деградация модели | $> 5\%$ от baseline | MLflow + мониторинг |
| **Prediction distribution** | Смещение предсказаний | Визуально / KS-test | Grafana |

```python
# Мониторинг drift: PSI + KS-test
import numpy as np
from scipy.stats import ks_2samp

def compute_psi(train_dist: np.ndarray, prod_dist: np.ndarray, bins: int = 10) -> float:
    """PSI между train и production распределениями."""
    breakpoints = np.quantile(train_dist, np.linspace(0, 1, bins + 1))
    train_counts = np.histogram(train_dist, bins=breakpoints)[0] / len(train_dist)
    prod_counts = np.histogram(prod_dist, bins=breakpoints)[0] / len(prod_dist)

    # Защита от деления на 0
    train_counts = np.clip(train_counts, 1e-6, None)
    prod_counts = np.clip(prod_counts, 1e-6, None)

    psi = np.sum((prod_counts - train_counts) * np.log(prod_counts / train_counts))
    return psi

def check_drift(train_data: dict, prod_data: dict, psi_threshold: float = 0.25):
    """Проверка drift для каждой фичи."""
    alerts = []
    for feature in train_data:
        psi = compute_psi(train_data[feature], prod_data[feature])
        ks_stat, p_value = ks_2samp(train_data[feature], prod_data[feature])
        if psi > psi_threshold or p_value < 0.05:
            alerts.append({
                "feature": feature,
                "psi": round(psi, 4),
                "ks_p_value": round(p_value, 4),
                "action": "RETRAIN" if psi > 0.25 else "INVESTIGATE",
            })
    return alerts
```

---

## Q5. "A/B тестирование моделей в production"

> "Нельзя просто заменить модель в продакшне — нужно проверить, что новая лучше. Три стратегии:
>
> 1. **Shadow mode** (теневой режим) — новая модель получает тот же трафик, что и текущая, делает предсказания, но пользователь видит только результат старой модели. Сравниваем предсказания offline. Плюс: нулевой риск для пользователей. Минус: не видим реальное влияние на поведение.
>
> 2. **Canary deployment** — новая модель получает малую долю трафика (1—5%). Если метрики ОК — увеличиваем до 50%/100%. Плюс: видим реальное влияние. Минус: если модель плохая — 1% пользователей пострадали.
>
> 3. **Interleaving** — для ранжирования: результаты двух моделей перемешиваются в одном списке, смотрим какие элементы пользователь выбирает. Быстрее чем A/B за счёт within-subject сравнения.
>
> Важнейший нюанс: ML metrics $\neq$ business metrics. AUC может вырасти, а конверсия упасть. Пример: модель рекомендаций стала точнее по NDCG, но рекомендует только популярные товары → diversity упала → пользователи уходят. Поэтому всегда мониторим оба уровня.
>
> Статистика: нужен достаточный sample size для significance. Для бинарных метрик (CTR, конверсия) — формула:
>
> $$n = \frac{(z_{\alpha/2} + z_{\beta})^2 \cdot 2p(1-p)}{\delta^2}$$
>
> где $\delta$ — минимальный detectable effect."

| Стратегия | Риск | Скорость | Когда использовать |
|-----------|------|----------|-------------------|
| **Shadow mode** | Нулевой | Медленно (нет реального feedback) | Первая проверка новой модели |
| **Canary (1—5%)** | Минимальный | Средне | Основной подход для деплоя |
| **A/B 50/50** | Средний | Быстро (больше данных) | Когда canary прошёл, нужна значимость |
| **Interleaving** | Минимальный | Очень быстро | Ранжирование, рекомендации |

| Уровень метрик | Примеры | Кто следит |
|----------------|---------|------------|
| **ML metrics** | AUC, NDCG, F1, MSE | ML-инженер |
| **Product metrics** | CTR, конверсия, retention | Product manager |
| **Business metrics** | Revenue, LTV, default rate | Бизнес |

```python
# Пример: A/B тест с проверкой significance
from scipy import stats
import numpy as np

def ab_test_proportions(
    control_conversions: int,
    control_total: int,
    treatment_conversions: int,
    treatment_total: int,
    alpha: float = 0.05,
):
    """Z-test для сравнения конверсий двух моделей."""
    p_control = control_conversions / control_total
    p_treatment = treatment_conversions / treatment_total
    p_pool = (control_conversions + treatment_conversions) / (control_total + treatment_total)

    se = np.sqrt(p_pool * (1 - p_pool) * (1/control_total + 1/treatment_total))
    z_stat = (p_treatment - p_control) / se
    p_value = 2 * (1 - stats.norm.cdf(abs(z_stat)))

    return {
        "control_rate": round(p_control, 4),
        "treatment_rate": round(p_treatment, 4),
        "lift": round((p_treatment - p_control) / p_control * 100, 2),
        "z_stat": round(z_stat, 4),
        "p_value": round(p_value, 4),
        "significant": p_value < alpha,
    }

# Пример: новая модель скоринга в Альфа-Банке
result = ab_test_proportions(
    control_conversions=450, control_total=10000,   # старая модель
    treatment_conversions=480, treatment_total=10000, # новая модель
)
# {'control_rate': 0.045, 'treatment_rate': 0.048, 'lift': 6.67%,
#  'p_value': 0.3012, 'significant': False}
# Не значимо → нужно больше данных или эффект слишком мал
```

---

## Q6. "Масштабирование ML"

> "Масштабирование нужно на трёх уровнях: данные, обучение, serving.
>
> **Data / Feature Engineering at scale:**
> pandas не тянет $> 10$ GB → переходим на Spark (PySpark), Dask или Polars. Spark: ленивые вычисления, партиционирование по кластеру. Dask: API как у pandas, но параллельно. Polars: быстрее pandas за счёт Rust и lazy evaluation.
>
> **Distributed Training:**
> - **Data Parallelism** — копия модели на каждом GPU, данные разбиты на батчи. Каждый GPU считает градиенты на своём батче, затем AllReduce — усреднение градиентов. PyTorch: `DistributedDataParallel (DDP)`. Подходит когда модель помещается на одном GPU.
> - **Model Parallelism** — модель разбита по GPU (разные слои на разных GPU). Нужно когда модель не влезает в память одного GPU (LLM с миллиардами параметров). Pipeline parallelism — разновидность: микро-батчи проходят через GPU как по конвейеру.
>
> **Serving at scale:**
> Horizontal scaling через Kubernetes: несколько replica pods за load balancer. Автоскейлинг по CPU/memory или custom metrics (requests per second, latency p99).
>
> Когда масштабировать: данные $> 10$ GB → Spark/Dask, модель не влезает на GPU → model parallelism, latency не укладывается → больше replicas. Когда НЕ нужно: данные $< 1$ GB, одна модель, $< 100$ RPS — pandas + FastAPI на одном сервере достаточно."

| Уровень | Проблема | Решение | Инструменты |
|---------|----------|---------|-------------|
| **Данные** | pandas не тянет $> 10$ GB | Distributed dataframes | Spark, Dask, Polars |
| **Training** | Обучение слишком долгое | Data parallelism | PyTorch DDP, Horovod |
| **Training** | Модель не влезает на GPU | Model / Pipeline parallelism | DeepSpeed, FSDP, Megatron |
| **Serving** | Не хватает throughput | Horizontal scaling | Kubernetes, replicas |
| **Serving** | Latency слишком высокий | Оптимизация модели | ONNX, TensorRT, quantization |

**Data Parallelism vs Model Parallelism:**

| Аспект | Data Parallelism | Model Parallelism |
|--------|-----------------|-------------------|
| Что на каждом GPU | Полная модель | Часть модели (слои) |
| Данные | Разные батчи | Одни и те же данные |
| Коммуникация | AllReduce (градиенты) | Активации между слоями |
| Когда нужен | Модель влезает на 1 GPU | Модель НЕ влезает на 1 GPU |
| Сложность | Простой (DDP — 3 строки) | Сложный (нужно разбивать модель) |

```python
# Data Parallelism: PyTorch DDP — минимальный пример
import torch
import torch.distributed as dist
from torch.nn.parallel import DistributedDataParallel as DDP

def train_ddp(rank, world_size):
    # Инициализация процесса
    dist.init_process_group("nccl", rank=rank, world_size=world_size)
    torch.cuda.set_device(rank)

    model = MyModel().to(rank)
    model = DDP(model, device_ids=[rank])  # оборачиваем в DDP

    optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)
    # Sampler гарантирует, что каждый GPU видит разные данные
    sampler = torch.utils.data.distributed.DistributedSampler(dataset)
    loader = torch.utils.data.DataLoader(dataset, sampler=sampler)

    for epoch in range(10):
        sampler.set_epoch(epoch)  # для корректного шаффлинга
        for batch in loader:
            loss = model(batch)
            loss.backward()       # градиенты auto-synced через AllReduce
            optimizer.step()
            optimizer.zero_grad()

    dist.destroy_process_group()

# Запуск: torchrun --nproc_per_node=4 train.py
```

---

## Q7. "Рекомендательная система: Two-Stage Funnel"

> "Рекомендательные системы работают двухэтапно — полный ranking на миллионах items невозможен ($O(n)$ по тяжёлой модели):
>
> **Stage 1: Candidate Generation** (миллионы → тысячи, <10ms)
> Быстрые и грубые модели, задача — не пропустить релевантное:
> - Коллаборативная фильтрация (ALS, implicit feedback)
> - Two-Tower нейросети (user tower + item tower → dot product)
> - Popularity-based (новые/трендовые items)
> - Content-based (по похожести фичей)
>
> **Stage 2: Ranking** (тысячи → десятки, <50ms)
> Точные и тяжёлые модели:
> - GBM (CatBoost/LightGBM) с rich features (user x item x context)
> - Deep ranking models (DeepFM, Wide&Deep)
> - Метрики: NDCG, MAP, MRR
>
> Примеры: Spotify Discover Weekly, Netflix, Яндекс Музыка. Почему двухэтапная: 10M items x 100ms/item для точной модели = 11.5 дней. С candidate generation: 10M → 1000 → 100ms x 1000 = 100s → ещё нужен fast model."

| Этап | Задача | Модели | Скорость | Метрика |
|------|--------|--------|----------|---------|
| Candidate Generation | Миллионы → тысячи | ALS, Two-Tower, BM25 | $< 10$ ms | Recall@K |
| Ranking | Тысячи → десятки | CatBoost, DeepFM | $< 50$ ms | NDCG, MAP |
| Re-ranking | Бизнес-логика | Rules (diversity, freshness) | $< 5$ ms | Business KPIs |

---

## Q8. "Антифрод: проектирование системы обнаружения мошенничества"

> "Fraud detection — одна из самых частых system design задач на DS-собесе.
>
> **Особенности задачи:**
> - Экстремальный дисбаланс: < 1% фродовых транзакций
> - Low-latency: решение до списания средств (< 100ms)
> - Высокая цена FN: пропуск мошенничества → финансовые потери
> - Высокая цена FP: блокировка легитимной транзакции → ухудшение UX
>
> **Метрики**: PR-AUC > ROC-AUC (при сильном дисбалансе). Recall@FPR=1% — 'сколько фрода ловим, если блокируем 1% легитимных'.
>
> **Архитектура**: микросервис с моделью в inference pipeline. Транзакция → feature computation → model inference → decision → proceed/block. Cascade: лёгкая модель (rules + logistic regression, <5ms) → тяжёлая модель (GBM, <50ms) только для пограничных случаев. Human-in-the-loop: пограничные кейсы → ручная проверка аналитиком.
>
> Связь с Альфа-Банком: кредитный скоринг — та же архитектура: low-latency decision, дисбаланс, каскад моделей."

| Компонент | Роль | Latency |
|-----------|------|---------|
| Rule engine | Жёсткие правила (страна в blacklist, сумма > X) | < 1 ms |
| Light model | LogReg / simple GBM, первичный scoring | < 5 ms |
| Heavy model | CatBoost с rich features | < 50 ms |
| Human review | Пограничные кейсы | Минуты |

---

## Q9. "Cold Start Problem в рекомендациях"

> "Cold Start — нет истории для нового пользователя или нового item. Коллаборативная фильтрация не работает (нет взаимодействий).
>
> **Новый пользователь:**
> - Глобальная популярность (показать самое популярное)
> - Гео-тренды (популярное в городе пользователя)
> - Онбординг-опрос ('выбери 5 любимых жанров')
> - Контентная фильтрация (по демографии, если есть)
> - Explore-exploit: Thompson Sampling / UCB — баланс между показом проверенного и исследованием вкусов
>
> **Новый item:**
> - Metadata-based: жанр, теги, описание → content-based similarity
> - Холодный буст: принудительно показываем новый item части пользователей для сбора feedback
> - Explore-exploit (MAB): Multi-Armed Bandit для балансировки exploitation (проверенные items) и exploration (новые items)
>
> Постепенный переход: по мере накопления данных → от content-based к коллаборативной фильтрации."

---

## Q10. "MLOps инструменты: DVC, MLflow, Airflow"

> "Три ключевых инструмента MLOps pipeline:
>
> **DVC** (Data Version Control) — 'Git для данных'. Хранит метаданные в Git (`.dvc` файлы), сами данные — в S3/GCS/SSH. Позволяет версионировать датасеты и воспроизводить эксперименты: `git checkout v1.0 && dvc pull` → данные версии 1.0.
>
> **MLflow** — трекинг экспериментов + Model Registry:
> - Tracking: `mlflow.log_param('lr', 0.01)`, `mlflow.log_metric('auc', 0.95)` → все эксперименты в UI
> - Model Registry: версионирование моделей (Staging → Production), артефакты
> - Serving: `mlflow models serve -m 'models:/credit_scoring/Production'`
>
> **Airflow** — оркестрация ETL/ML пайплайнов. DAGs (Directed Acyclic Graphs) — граф задач с зависимостями. Scheduler запускает по расписанию. Retry, alerting, backfill.
>
> Когда нужны: команда > 2 человек, несколько моделей, регулярное переобучение.
> Когда overkill: один ML-инженер, один notebook, разовый анализ.
>
> **Offline vs Online метрики**: ROC-AUC = 0.95 в Jupyter ≠ CTR вырос в проде. Offline метрики оценивают модель, online — продукт. Между ними — A/B тест."

| Инструмент | Что делает | Аналог | Когда нужен |
|------------|-----------|--------|-------------|
| **DVC** | Версионирование данных | Git для файлов > 1 MB | Датасеты меняются со временем |
| **MLflow** | Трекинг экспериментов, Model Registry | W&B, Neptune | > 10 экспериментов, нужен Model Registry |
| **Airflow** | Оркестрация пайплайнов (DAGs) | Prefect, Dagster | Регулярное переобучение, ETL |

```python
import mlflow

with mlflow.start_run(run_name="catboost_v3"):
    mlflow.log_params({
        "learning_rate": 0.05,
        "depth": 6,
        "iterations": 1000,
    })
    mlflow.log_metrics({
        "auc_train": 0.97,
        "auc_val": 0.94,
        "gini": 0.88,
    })
    mlflow.catboost.log_model(model, "model")
```

---

## Приоритет для intern-level

| Тема | Приоритет | Почему |
|------|-----------|--------|
| ML pipeline end-to-end (Q1) | **Высокий** | Показывает системное мышление |
| RecSys funnel (Q7) | **Высокий** | Частый system design вопрос |
| Антифрод (Q8) | **Высокий** | Связь с финтехом (Альфа-Банк) |
| Model serving (Q3) | Средний | FastAPI — уже в стеке |
| Monitoring, drift (Q4) | Средний | Часто спрашивают на DS |
| A/B тестирование (Q5) | Средний | Базовое понимание обязательно |
| Cold Start (Q9) | Средний | Показывает понимание рекомендаций |
| MLOps инструменты (Q10) | Средний | DVC/MLflow — must know tools |
| Feature Store, масштабирование (Q2, Q6) | Низкий | Для senior ML Engineer |
