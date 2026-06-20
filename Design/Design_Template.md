# ML System Design: Template & Framework

> **The brief:** You outline an **end-to-end ML solution** to an open-ended problem on a virtual whiteboard (Bluescape). You're assessed on **scalability, latency/throughput, failure handling, monitoring, and clear communication**, _and_ on bridging ML theory with practical system design (you can build models AND design the infra to train/deploy/maintain them at scale).
>
> **For this role (Machine Learning Engineer, Amazon Customer Service):** expect ML that powers customer support at scale: **contact/intent classification & routing, agent-assist + GenAI assistants/chatbots, contact-volume forecasting (capacity & staffing), answer/response ranking & recommendation, and escalation / CSAT / churn / sentiment prediction**. This domain is **NLP- and GenAI-heavy** (plays to your strength) with a forecasting/ranking backbone. Translate your CV/GenAI depth into the language of **text classification, retrieval/RAG, large-scale serving, and time-series**.
>
> **Golden rules:** State assumptions. Ask clarifying questions. Propose **multiple options** and **justify your pick** with pros/cons. Narrate the whole way. Draw a block diagram first; deep-dive only where asked.

---

## The 8-Step Framework (run on every design)

### 1. Clarify & Scope

Pin down the _business_ problem before the ML:

- **Business goal** and the **decision** the system drives (increase conversion? reduce capacity over-provisioning?).
- **Who's the user / consumer** of the prediction (a service, a planner, an end customer)?
- **Data:** What are the data sources? how large large is the dataset? is the data labeled?
- **Scale:** QPS, # users, # items, data volume & velocity.
- **Latency / cost budget:** p99 inference latency? batch vs real-time? cost ceiling?
- **Success metric:** the one online metric that defines success + guardrails.

> Say: _"Let me state my assumptions and confirm scope before I design."_

### 2. Frame as an ML Problem

- What is the system's **input** and **output**? What exactly do we **predict**? What's the **label** (implicit vs explicit)?
- **Online vs batch** inference? **Ranking vs classification vs regression vs forecasting?**
- What's the **prediction granularity** (per-request, per-item, per-time-bucket)?

### 3. Data

- **Sources:** logs, transactions, catalog, context. Volume & freshness.
- **Labels:** implicit (natural) feedback (e.g. clicks/purchases) vs explicit (hand). Delay between action and label?
- **Leakage:** especially **temporal leakage** in forecasting/ranking (no future info in features).
- **Privacy / compliance:** PII handling, retention.
- **Bias:** Are there any biases in the data? If yes, what kind are present, and how do we correct them?
- **ETL Pipeline:** **Extract** data from different sources, **Transform** into specific format to meeting operational needs (cleaning, schema validation and mapping), and **Load** the transformed data into the target destination (file, database or data warehouse).

### 4. Features

- **Operations:** handling missing values, feature scaling, discretization, and encoding.
- **Entities:** user / item / context features; cross features.
- **Real-time vs batch:** precompute heavy features in batch, compute fresh ones near-real-time.
- **Feature store:** consistent features for training & serving (avoid **training-serving skew**).
- **Embeddings** where cardinality is high (users, items, queries).

### 5. Model

- **Baseline first** (popularity, co-visitation, last-value/seasonal-naive, GBDT); signals you won't over-engineer.
- Then the real architecture; for retrieval/ranking: **candidate generation → ranking**.
- **Justify the choice** vs alternatives with offline/online trade-offs.
- **Why GBDT (XGBoost/LightGBM) for tabular:** handles mixed features, robust, strong baseline; trees often beat deep nets on tabular.

### 6. Training

- **Pipeline:** data → features → train → validate → register.
- **Dataset:** sampling strategy (stratified, importance etc), data split percentage, handling class imbalance
- **Loss function:** depending on the ML problem, classification vs regression losses.
- **Validation:** k-fold cross validation over regular validation when little data available, but trades off computational cost.
- **Distributed training** if data is large; **validation strategy** (temporal split for forecasting/ranking, rolling-origin backtest).
- **Model registry & versioning**, reproducibility.
- **Retraining cadence** (daily/weekly) + triggers (drift, performance drop).

### 7. Evaluation

- **Offline metrics:** NDCG / MAP / recall@k (ranking), AUC/PR-AUC (classification), MAPE/sMAPE/quantile loss (forecasting).
- **Online:** A/B test on the business metric (CTR, conversion, GMV) with **guardrails** (latency, diversity, error rate).
- Tie **model metrics to business metrics** (cost savings, efficiency).

### 8. Serve & Monitor

- **Prediction pipeline:** **batch** (precompute, serve by lookup, cheap but stale) vs **online/real-time** (compute on request, fresh but needs low-latency infra); or hybrid (batch candidates → online ranking). Pick per latency budget.
- **Where to deploy:** **cloud** (big models, easy to update/monitor, elastic, but network latency + cost) vs **on-device/edge** (low latency, offline, private, but constrained, hard to update). Default cloud.
- **Serving infra:** low-latency endpoint (FastAPI / Ray Serve / SageMaker), **batching**, autoscaling, **caching**. For retrieval/ranking: ANN index (HNSW/IVF) for candidate gen → ranker.
- **Model compression** (hit latency/cost/edge budgets): quantization, pruning, knowledge distillation, ONNX/TensorRT.
- **Deployment strategy:** canary / blue-green; **rollback** path.
- **Test in production:** **shadow** (log new model on live traffic, don't serve, zero risk) → **A/B** (split traffic, measure business metric + guardrails); also interleaving, bandits.
- **Monitoring, dual perspective:**
  - _Operational:_ latency, throughput, error rate, saturation.
  - _ML-specific:_ model performance, **feature/data drift**, **concept drift**, prediction distribution, position bias, model accuracy and versions.
- **Alerting** on degradation in _either_. **Feedback loop** to capture new labels. **Fallbacks** (cold-start → popularity; model down → cached/last-good).

---

## Whiteboard Block-Diagram Order (draw this)

```
[Data sources] → [Ingestion/ETL] → [Feature pipeline] → [Feature Store]
                                                            │
                              ┌──────── offline ───────────┤──────── online ────────┐
                              ▼                             ▼                         ▼
                      [Training pipeline] → [Model Registry] → [Serving: retrieval(ANN) → ranker]
                              ▲                                         │
                       [Eval / A/B] ←──── [Monitoring & drift] ◄────────┘ → response
```

> Draw the core blocks first, no deep explanation unless asked. Then **pick 2–3 critical components** to deep-dive (Step "Detailed Design").

---

## Deep-Dive Menu (pick 2–3; have pros/cons ready)

- **Feature engineering pipeline** (batch vs streaming, point-in-time correctness).
- **Training pipeline** (distributed training, retraining triggers, validation).
- **Model serving infra** (ANN retrieval, batching, quantization/ONNX, autoscaling).
- **Monitoring & logging** (drift detection, alerting thresholds).
- **Experimentation framework** (shadow/dark launch, A/B, interleaving, bandits, guardrail metrics).

> If the interviewer hints at a different component, **take the hint** and pivot. For each: present approaches, pros/cons, and **why you chose this one**.

---

## Always-Address Checklist (Bar Raiser watches for these)

- [ ] **Scalability:** how does training scale with data? How does serving handle traffic spikes?
- [ ] **Failure handling:** what if the model degrades / a dependency dies? Fallback ready.
- [ ] **Training-serving skew** prevented (shared feature definitions / feature store).
- [ ] **No temporal leakage** (critical for forecasting/ranking).
- [ ] **Concept/feature drift** monitored with retrain triggers.
- [ ] **Cold-start** strategy (new user/item/series).
- [ ] **Cost optimization** (ML at scale is expensive, show awareness: caching, right-sizing, spot, quantization).
- [ ] **Data privacy & security** (sensitive data handling).
- [ ] **Feedback loops / position bias** acknowledged (recommenders).

---

## How to Open (script)

_"Let me start by clarifying the business goal and scale so my design is grounded. I'll assume [X] unless you'd steer me otherwise. I'll frame it as an ML problem, lay out an end-to-end architecture at a high level, then deep-dive the 2–3 components I think are most critical. Feel free to redirect me. I'll call out trade-offs and alternatives as I go, and finish with how I'd evaluate, serve, and monitor it."_

See `Design_Question_Bank.md` for worked examples and likely prompts.
