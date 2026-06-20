# ML System Design: Question Bank & Worked Examples

> Run the **8-step framework** (`Design_Template.md`) on each. Write a **one-page outline per prompt** (Days 8–9 of the plan).
> **Role = MLE, Amazon Customer Service.** Closest to the actual team: **contact-volume forecasting**, **intent classification & contact routing**, and **agent-assist / RAG support assistants**. Prep those hardest. Recommendation/ranking still appears as a transferable pattern.

---

## ★ Worked Example 1: "Design a product recommendation system" (most-asked)

1. **Clarify:** Goal = increase conversion/GMV on the product page. ~100M users, ~10M items, **p99 < 100 ms**, recs refreshed per session. Metric: online conversion + CTR; guardrails on latency & diversity.
2. **Frame:** Two-stage: **retrieval** (narrow 10M → ~500 candidates) then **ranking** (score 500, return top-k).
3. **Data:** Implicit feedback (clicks, add-to-cart, purchases), user history, item metadata, context (time, device, query). **Negative sampling** for training.
4. **Features:** User embedding (history), item embedding (co-purchase/content), context features. Precompute item embeddings in **batch**; user features **near-real-time** via a **feature store**.
5. **Model:** Retrieval = **two-tower** (user tower / item tower) trained with **in-batch negatives**, served via **ANN (HNSW)**. Ranking = **GBDT or DLRM** on rich cross-features optimizing predicted conversion. **Baseline first:** co-visitation / popularity.
6. **Train:** Daily batch retrain of towers + ranker; **temporal split**; monitor feedback-loop bias.
7. **Eval:** Offline **recall@k + NDCG**; online **A/B on conversion** with latency & diversity guardrails.
8. **Serve/monitor:** Precomputed item index, cached user embeddings, ranker behind a low-latency service (FastAPI/Ray Serve), autoscale, monitor **CTR drift, position bias**, cold-start fallback to popularity.

---

## ★ Worked Example 2: "Design demand / traffic forecasting for capacity planning" (likely closest to the team)

1. **Clarify:** Goal = forecast request/demand volume per service/region to plan capacity; horizon (next hour? day? week?); granularity (per-region, per-SKU). Metric: forecast accuracy → translated to **over/under-provisioning cost**. Need **prediction intervals** (quantiles) for capacity safety margins.
2. **Frame:** Time-series **regression / probabilistic forecasting**, multi-horizon, per-series (possibly millions of series).
3. **Data:** Historical traffic, calendar (holidays, weekends), promotions/events, region metadata. Watch for outages/anomalies in history.
4. **Features:** Lag features, rolling stats, calendar/holiday flags, Fourier seasonality terms, promo indicators. **No temporal leakage.** Handle **cold-start** (new series → hierarchical/global model).
5. **Model:** **Baseline:** seasonal-naive / exponential smoothing. **Strong baseline:** **GBDT on lag + calendar features** (per-series or global). **Deep:** **DeepAR / Temporal Fusion Transformer** for many related series + quantile outputs. Justify: global models share strength across series & handle cold-start; GBDT cheap and robust.
6. **Train:** **Rolling-origin (backtest) cross-validation**; retrain daily/weekly; distributed over series.
7. **Eval:** **MAPE / sMAPE / quantile (pinball) loss**; **coverage** of prediction intervals; tie to **capacity cost** (cost of under-provision = outages, over-provision = wasted compute $).
8. **Serve/monitor:** Batch-generate forecasts on schedule → store for planners; monitor **forecast drift / regime change**, alert when actuals fall outside predicted intervals, retrain trigger on sustained error.

---

## ★ Worked Example 3: "Design a ranking system by product sales, factoring time-decay; do part in O(log n)"
*(Reported at Amazon: the O(log n) hint means a heap / balanced-BST / sorted structure question hiding inside a design.)*
- **Score = Σ sales × decay(age)**; apply **exponential time-decay** so recent sales weigh more.
- Maintain a **balanced BST / sorted structure / heap** keyed by decayed score for **O(log n)** insert/update and top-k reads.
- Trick: instead of re-decaying every item each tick, **scale incoming weights up by `e^{+λt}`** (reference-time normalization) so existing scores stay comparable, relative order preserved without touching every node.
- Discuss: batch recompute vs incremental; ties; how top-k is served (heap of size k); persistence.

---

## ★ Worked Example 4: "Design an intent classification + contact-routing system" (very on-domain for Customer Service)

1. **Clarify:** Goal = route each incoming customer contact (chat/email/call transcript) to the right queue/agent/automated flow, fast. Scale: millions of contacts/day, many intents, multilingual; p99 < ~100–300 ms for routing. Metric: routing accuracy → **first-contact resolution, handle time, CSAT, deflection rate**.
2. **Frame:** Multi-class (often multi-label / hierarchical) **text classification**; possibly a confidence threshold → fall back to human/triage.
3. **Data:** Historical contacts + resolution labels, agent disposition codes; weak/implicit labels (transfers = mis-routes). Watch label noise & class imbalance (long-tail intents).
4. **Features:** Text embeddings (transformer encoder), customer/account context, channel, recent order/contact history; PII handling.
5. **Model:** **Baseline:** TF-IDF + linear / GBDT. **Strong:** fine-tuned transformer encoder (or an **LLM classifier** for zero/few-shot on rare intents). Justify: encoder is cheap & low-latency at scale; reserve the LLM for the long tail or as a fallback.
6. **Train:** Periodic retrain; **temporal split**; handle new intents (open-set / cold-start).
7. **Eval:** Offline macro-F1 / per-intent precision-recall (long-tail matters); online A/B on **mis-route rate, transfers, handle time, CSAT**.
8. **Serve/monitor:** Low-latency encoder behind an autoscaled service, cache, **confidence-thresholded fallback to human triage**, monitor **intent drift** (new product issues spike new intents) with retrain triggers.

## ★ Worked Example 5: "Design an agent-assist / customer self-service RAG assistant" (GenAI on-domain)

1. **Clarify:** Goal = answer customer/agent questions grounded in **help articles, policies, order data**. Who: self-serve customer or assisting agent. Need **grounding + citations** (wrong policy answers are costly), low latency, guardrails.
2. **Frame:** **LLM + RAG** (retrieval-grounded generation) + tool calls for live account/order data.
3. **Data:** Knowledge base (help center, policies), resolved-ticket corpus; per-customer order/account via tools (not in the index, fetch live).
4. **Features/Index:** Chunk + embed KB, hybrid dense+BM25 retrieval, cross-encoder rerank; **MCP/tool layer** for live order lookups.
5. **Model:** Managed LLM for generation; retrieval for grounding. Baseline: retrieval-only (top article). Justify RAG vs fine-tune vs plain LLM (see `../Behavioral/GenAI_Behavioral.md`).
6. **Train/maintain:** Re-index on KB updates; eval set of question→answer pairs.
7. **Eval:** Retrieval recall@k/MRR; generation **faithfulness/groundedness**, citation accuracy, hallucination rate; online **deflection rate, CSAT, escalation rate**, cost/query.
8. **Serve/monitor:** Cache frequent Qs, guardrails (refuse/escalate on low retrieval confidence), monitor faithfulness drift, log citations for audit, fallback to human.

> **Contact-volume forecasting** (for staffing/capacity) = Worked Example 2 re-skinned: forecast contacts per queue/skill/interval with quantiles for safe staffing; cost of under-forecast = long wait/abandonment, over-forecast = idle agents.

## Other Likely Prompts (rehearse the framework on each)

| Prompt | Your angle / key components |
|---|---|
| **Image / text search engine** | Your strength: **two-tower + ANN + cross-encoder reranking**; hybrid dense + BM25. |
| **Autocomplete / spell-check** | **Trie + n-gram LM + edit distance**; prefix ranking by popularity. |
| **Click-through-rate (CTR) prediction** | Logistic/DLRM, feature crosses, calibration, online learning, position bias. |
| **Feed / news ranking** | Candidate gen + ranker, recency + engagement, diversity, freshness. |
| **Fraud / anomaly detection** | Imbalanced data, PR-AUC, threshold tuning, streaming features, human-in-loop review. |
| **A/B testing / experimentation platform** | Assignment, guardrails, sequential testing, sample size, interleaving. |
| **Model serving / inference platform** | Batching, autoscaling, ONNX/quantization, canary, multi-model, GPU sharing. |
| **Rate limiter / URL shortener** | Classic **non-ML** system design *can* appear: token bucket / hashing + KV store. Be ready. |

---

## Forecasting / Ranking Cheat-Sheet (role-specific breadth, say these crisply)
- **Time-series:** classical (ARIMA, exp-smoothing, Prophet) vs ML (**GBDT on lag/calendar features**) vs deep (**DeepAR, TFT**). Watch seasonality, holidays, cold-start, **no temporal leakage**; backtest with **rolling-origin CV**; metrics **MAPE/sMAPE/quantile loss** for intervals.
- **Ranking/recsys:** **candidate generation → ranking**; **two-tower retrieval**, **GBDT/DLRM** rankers; offline **NDCG/MAP/recall@k** vs online **CTR/conversion via A/B**. Watch cold-start, **position bias**, feedback loops.
- **Why GBDT for tabular:** mixed features, robust, strong baseline; signals you won't over-engineer with deep nets where trees win.

---

## Self-Grade per design
- [ ] Did I clarify scale, latency, and the *business* metric first?
- [ ] Did I state assumptions explicitly?
- [ ] Did I give a baseline before the fancy model?
- [ ] Did I propose ≥2 options with pros/cons and justify the pick?
- [ ] Did I address scaling, failure/fallback, drift monitoring, cost?
- [ ] Did I avoid temporal leakage / training-serving skew?
- [ ] Did I connect offline metrics → online A/B → business impact ($)?
