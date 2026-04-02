# Learning Knowledge Base — Index

Навигация по всем материалам для подготовки к ML/DS/AI Engineer Intern собеседованиям.

**Всего: ~125 вопросов + 10 exercises | 27 файлов | 9 доменов**

---

## Math & Statistics

| Файл | Содержание | Вопросов |
|------|-----------|----------|
| [math-and-stats/probability.md](math-and-stats/probability.md) | Случайные величины, распределения, Байес, LLN | 6 |
| [math-and-stats/statistics.md](math-and-stats/statistics.md) | CLT, hypothesis testing, CI, bootstrap, Type I/II errors, Bonferroni | 7 |
| [math-and-stats/applied_stats.md](math-and-stats/applied_stats.md) | Correlation/causation, A/B tests, MLE/MAP, bias-variance, network effects | 5 |

## Classic ML — Theory

| Файл | Содержание | Вопросов |
|------|-----------|----------|
| [classic-ml/theory.md](classic-ml/theory.md) | Boosting, bias-variance, L1/L2, metrics, encoding, bagging, churn | 10 |

## Classic ML — Practice

| Файл | Содержание | Exercises |
|------|-----------|-----------|
| [classic-ml/practice_from_scratch.md](classic-ml/practice_from_scratch.md) | LogReg, K-Means, Decision Tree from scratch | Ex 1-3 |
| [classic-ml/practice_evaluation.md](classic-ml/practice_evaluation.md) | Cross-validation, model debugging | Ex 4, 8 |
| [classic-ml/practice_features.md](classic-ml/practice_features.md) | Feature engineering pipeline, permutation importance | Ex 5, 9 |
| [classic-ml/practice_training.md](classic-ml/practice_training.md) | Imbalanced data, hyperparameter tuning | Ex 6, 7 |
| [classic-ml/practice_pipeline.md](classic-ml/practice_pipeline.md) | Full end-to-end interview simulation | Ex 10 |

## Deep Learning

| Файл | Содержание | Вопросов |
|------|-----------|----------|
| [deep-learning/fundamentals.md](deep-learning/fundamentals.md) | Нейросети, backprop, активации, loss functions | 4 |
| [deep-learning/training.md](deep-learning/training.md) | Optimizers, gradients, Dropout (inverted), BatchNorm, LayerNorm, LR | 7 |
| [deep-learning/architectures.md](deep-learning/architectures.md) | CNN, RNN/LSTM, transfer learning, выбор архитектуры | 4 |
| [deep-learning/pytorch.md](deep-learning/pytorch.md) | Training loop, nn.Module, model.eval() vs no_grad() | 3 |
| [deep-learning/transformers.md](deep-learning/transformers.md) | Self-Attention, positional encoding, LoRA, ViT | 5 |

## Generative AI & Agents

| Файл | Содержание | Вопросов |
|------|-----------|----------|
| [genai/agent_fundamentals.md](genai/agent_fundamentals.md) | Agent vs LLM, ReAct, память, CoT vs Tool Use | 4 |
| [genai/tool_use.md](genai/tool_use.md) | Function calling, tool logic, structured output, MCP (architecture, capabilities) | 4 |
| [genai/advanced_patterns.md](genai/advanced_patterns.md) | Фреймворки, multi-agent, agentic RAG, voice AI | 4 |
| [genai/production.md](genai/production.md) | Guardrails, evaluation, error handling | 3 |
| [genai/rag_systems.md](genai/rag_systems.md) | RAG architecture, chunking, vector search, evaluation | 5 |

## Linear Algebra

| Файл | Содержание | Вопросов |
|------|-----------|----------|
| [linear-algebra/core_concepts.md](linear-algebra/core_concepts.md) | Vectors, matrices, eigenvalues, SVD, PCA, norms | 6 |

## Python

| Файл | Содержание | Вопросов |
|------|-----------|----------|
| [python/coding_interview.md](python/coding_interview.md) | Data structures, algorithms, OOP, decorators, generators, GIL + concurrency comparison, memory, NumPy, typing | 12 |
| [python/algorithms_dp.md](python/algorithms_dp.md) | DP anti-overengineering, 5 паттернов (Linear, Knapsack, String, Decision, Counting) | 3 |

## SQL

| Файл | Содержание | Вопросов |
|------|-----------|----------|
| [sql/fundamentals.md](sql/fundamentals.md) | JOINs, GROUP BY, window functions, CTEs, optimization, gaps & islands, ClickHouse | 10 |

## System Design

| Файл | Содержание | Вопросов |
|------|-----------|----------|
| [system-design/ml_systems.md](system-design/ml_systems.md) | Pipelines, serving, monitoring, A/B, scaling, RecSys, fraud, MLOps | 10 |
| [system-design/docker.md](system-design/docker.md) | Dockerfile, Compose, multi-stage build, кэширование, типичные ошибки | 3 |

## Soft Skills

| Файл | Содержание | Вопросов |
|------|-----------|----------|
| [soft-skills/behavioral.md](soft-skills/behavioral.md) | STAR framework, technical communication, trade-offs, "why ML" | 6 |

---

## Practice Schedule

Рекомендованный порядок выполнения exercises из `classic-ml/`:

| # | Exercise | Файл | Что тренирует | Время |
|---|----------|------|--------------|-------|
| 1 | LogReg from scratch | practice_from_scratch.md | Градиенты, sigmoid, threshold | 30 min |
| 2 | K-Means from scratch | practice_from_scratch.md | Кластеризация, K-Means++ | 30 min |
| 3 | Decision Tree from scratch | practice_from_scratch.md | Gini, splits, рекурсия | 45 min |
| 4 | CV from scratch | practice_evaluation.md | Stratified split, paired t-test | 20 min |
| 5 | Feature Engineering | practice_features.md | Pipeline, imputer, encoder | 30 min |
| 6 | Imbalanced Data | practice_training.md | class_weight, SMOTE, threshold | 20 min |
| 7 | Hyperparameter Tuning | practice_training.md | Grid vs Random vs Optuna | 20 min |
| 8 | Model Debugging | practice_evaluation.md | Типичные ошибки, learning curve | 20 min |
| 9 | Feature Selection | practice_features.md | Permutation importance | 15 min |
| 10 | Full Simulation | practice_pipeline.md | End-to-end pipeline | 40 min |

**Total: ~4.5 hours**
