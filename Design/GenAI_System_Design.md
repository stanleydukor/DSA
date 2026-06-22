# GenAI / LLM Application — System Design

> **Why this file exists:** This role is *"build generative AI solutions for customer service"* + *"production integration of LLMs."* A GenAI design question is **not** the classic train-a-model ML lifecycle — you usually **don't train a model**. You design **RAG + prompt/context management + LLM serving (streaming, cost, latency) + evaluation + guardrails + (sometimes) agents**. Use *this* framework when the prompt is "design an LLM-powered X." Use `Design_Template.md` when it's "design a model that predicts Y."
>
> **Golden rules (same as always):** clarify scope, state assumptions, propose options + justify, draw the architecture, deep-dive 2–3 components, address scale/reliability/cost/safety. At Amazon, anchor on **Bedrock** (managed LLMs) unless there's a reason to self-host.

---

## First: the architecture decision tree (say this out loud)

```
Need the model to use knowledge/behavior it doesn't have?
├─ Just need better instructions/format/tone?         → PROMPT ENGINEERING (cheapest, start here)
├─ Need facts from a specific/changing/private corpus? → RAG (ground + cite, no retrain)
├─ Need a consistent style/format/skill at scale,
│   prompting can't get it reliably, have labeled data? → FINE-TUNE (or distill a small model)
└─ Need multi-step actions / call external systems?    → AGENT + TOOLS (tool calling / MCP)
```
Most production customer-service systems are **RAG + prompt engineering + tools**, occasionally with a **fine-tuned small model** for a high-volume narrow task (cost/latency). Combine freely. (Full RAG-vs-LLM reasoning: `../Behavioral/GenAI_Behavioral.md`.)

---

## The LLM-Application Design Framework (8 steps)

### 1. Clarify & Scope
- **Use case & user:** self-serve customer bot? agent-assist (suggest to a human)? back-office automation (summarize/route tickets)?
- **Quality bar & risk:** is a wrong answer costly (policy/refunds/legal)? → grounding + guardrails + human fallback are mandatory.
- **Latency budget:** **TTFT** (time-to-first-token, what the user *feels* — streaming hides total latency) and **end-to-end**. Chat UX wants TTFT < ~1s.
- **Scale & cost:** QPS / contacts per day, peak vs average, **cost-per-interaction ceiling** (tokens × price).
- **Languages, channels** (chat/email/voice transcript), and **integration points** (order system, CRM).

### 2. Frame the GenAI Approach
- Walk the decision tree above; **name your pick and the cheaper alternatives you rejected and why** (prompt < RAG < fine-tune < agent in cost/complexity — don't over-engineer).
- Define the **contract**: input (user msg + context), output (answer + citations + structured action), and the **"I don't know / escalate"** path.

### 3. Knowledge & Data (the RAG layer)
- **Sources:** help center, policy docs, resolved-ticket corpus, product catalog. **Live data** (this customer's order/account) comes via **tools/APIs**, *not* the index.
- **Ingestion:** chunk (semantic sections, with metadata: source, date, product), **embed**, store in a **vector DB** (OpenSearch / pgvector / Bedrock KB). **Re-index on KB updates** (freshness).
- **Retrieval:** **hybrid dense + BM25**, then **cross-encoder rerank** for precision; return top-k chunks + source pointers.
- **Privacy:** PII handling/redaction, per-customer access scoping, retention.

### 4. Architecture (draw this)
```
Client/Channel → API Gateway (authn, rate limit)
      → Orchestrator (prompt build, routing, retries, guardrails)
           ├─ Retriever → Vector DB (+ rerank)        # grounding
           ├─ Tools/MCP → Order/CRM/Account APIs       # live data & actions
           ├─ Semantic + prompt cache                  # cost/latency
           └─ LLM Gateway/Router → model(s) (Bedrock)  # streaming response
      → Guardrails (in/out filter, PII, grounding check) → response (streamed, with citations)
                                   │
        Logging/Tracing → Eval & Observability (quality, cost, latency, drift) → Feedback loop
```

### 5. Prompt & Context Management
- **System prompt + templates**, **versioned** (treat prompts like code: source control, review, rollback).
- **Context-window budget:** decide how many retrieved chunks, history turns, and tool outputs fit; truncate/summarize older history (cf. Tucknightly's rolling summaries).
- **Structured output** (JSON / tool schema) when the answer drives an action — validate it.
- **Prompt A/B testing** and regression tests when you change a prompt (a prompt change *is* a deploy).

### 6. Serving & Scale
- **Streaming** responses (SSE/websockets) to minimize perceived latency (TTFT).
- **LLM gateway / model router:** route by task — cheap/small model for easy/classify, frontier model for hard generation (**model tiering** = the biggest cost lever). **Fallback** to another model/provider on error or throttle.
- **Caching:** **exact cache** (identical Q), **semantic cache** (similar Q via embedding match), **prompt caching** (cache the static system-prompt/KB prefix to cut input-token cost & latency).
- **Throughput & limits:** token-based rate limiting, request queueing, autoscale the orchestrator/retriever; manage provider quotas; multi-region for availability.
- **Timeouts, retries with backoff, circuit breakers** around model + tool calls.

### 7. Evaluation (the part candidates forget — JD says "model evaluation")
- **Offline eval set:** curated question→ideal-answer pairs; run on every prompt/model change (regression suite).
- **Automated metrics:** retrieval **recall@k / MRR**; generation **faithfulness/groundedness** (is the answer supported by retrieved context?), **answer relevance**, **citation accuracy**, **safety**. Use **LLM-as-judge** for scalable scoring (validate the judge against human labels).
- **Human eval** on a sample for high-risk flows.
- **Online metrics:** **containment/deflection rate**, **CSAT**, **escalation/transfer rate**, thumbs up/down, resolution rate, **cost-per-contact**, latency percentiles.
- **A/B test** prompts/models with guardrails (don't ship on offline numbers alone).

### 8. Guardrails, Safety & Observability
- **Input guardrails:** PII redaction, **prompt-injection / jailbreak** detection, off-topic/abuse filtering.
- **Output guardrails:** toxicity/policy filter, **grounding/faithfulness check** (block ungrounded claims), refuse-and-**escalate-to-human** on low confidence.
- **Hallucination mitigation:** ground with retrieval, "answer only from context / say you don't know," cite sources, constrain with schema, rerank for precision.
- **Observability:** trace every request (prompt, retrieved chunks, tool calls, tokens, latency, cost), monitor **quality drift, cost drift, latency**, capture feedback for the eval set. Log citations for **audit**.

---

## Key Concepts Cheat-Sheet (be ready to define crisply)
| Concept | One-liner |
|---|---|
| **RAG** | Retrieve relevant chunks → inject into prompt → grounded, cited generation. For factual answers over a specific corpus. |
| **Fine-tune vs RAG vs prompt** | Prompt = behavior; RAG = fresh/private knowledge + citations; fine-tune = consistent style/skill (doesn't add live facts). |
| **TTFT / TPOT** | Time-to-first-token (perceived latency, hidden by streaming) / time-per-output-token (throughput). |
| **Cost levers** | Model tiering/routing, prompt caching, semantic cache, cap max output tokens, shorter context, distill to a small model. |
| **Semantic cache** | Cache by embedding similarity of the query, not exact match. |
| **Guardrails** | Input (PII, injection) + output (toxicity, grounding) filters + escalate path. |
| **LLM-as-judge** | Use an LLM to score outputs against criteria at scale; validate against human labels. |
| **Agent / tool calling / MCP** | LLM invokes external tools/data in a loop; MCP = standard protocol to expose tools/data to the model. |
| **Prompt injection** | Malicious input that hijacks instructions; defend with input filtering, privilege separation, output checks, not trusting retrieved/user text as instructions. |
| **Faithfulness/groundedness** | Is every claim in the answer supported by the retrieved context? Core RAG quality metric. |

---

## Worked Example A — "Design a customer self-service GenAI assistant" (the most likely prompt)

1. **Clarify:** Self-serve chat that resolves common customer issues (where's my order, returns, refunds policy) before reaching a human. ~X M contacts/day, peak 3×. Goal: ↑ **containment/deflection** without hurting **CSAT**. Wrong policy/refund answers are costly → grounding + guardrails + easy human handoff. Streaming, TTFT < 1s.
2. **Frame:** **RAG + tools**, no fine-tune to start. Prompt-engineered persona; RAG for policy/help KB; **tools** for this customer's live order/account; escalate on low confidence.
3. **Knowledge/data:** Chunk+embed help center & policies (hybrid retrieval + rerank); live order/account via secured tool APIs (not indexed); PII redaction.
4. **Architecture:** gateway → orchestrator → {retriever+vector DB, order/CRM tools, cache} → LLM gateway (Bedrock) → guardrails → streamed answer + citations + suggested actions.
5. **Prompt/context:** versioned system prompt (tone, refusal rules, "answer only from context"), budget for top-k chunks + last N turns + tool results; structured output when triggering an action (e.g., initiate return).
6. **Serve/scale:** stream; **model tiering** (small model for intent/simple FAQ, frontier for complex); semantic + prompt caching for repeated policy questions; token rate limits; retries/fallback; autoscale.
7. **Eval:** offline QA set + faithfulness/citation/safety via LLM-as-judge + human sample; online **deflection, CSAT, escalation rate, cost/contact**; A/B prompt & model changes.
8. **Guardrails/observability:** injection + PII filters, grounding check + escalate-to-human on low retrieval confidence, full request tracing (tokens/cost/latency), quality-drift monitoring, feedback → eval set.
- **Deep-dive candidates:** the RAG retrieval quality pipeline; the eval/observability loop; cost/latency optimization (caching + tiering).

## Worked Example B — "Design real-time agent-assist" (suggest to a *human* agent during a live contact)
- **Use case:** while a human agent handles a contact, surface **suggested replies, relevant KB snippets, a live conversation summary, and next-best-action** in real time. Human stays in the loop → lower risk than full automation, but **latency is tight** (suggestions must keep up with the conversation).
- **Architecture:** stream the live transcript → on each customer turn, retrieve KB + generate a grounded suggestion (small/fast model, streamed) + maintain a rolling summary; agent accepts/edits (the **accept/edit signal is gold training/eval data**).
- **Key concerns:** sub-second suggestions (small model, caching, partial-context prompts), grounding/citations so the agent can trust them, measure **handle-time reduction, first-contact-resolution, suggestion acceptance rate, CSAT**.
- **Why agent-assist before full automation:** human-in-the-loop de-risks hallucination and builds the labeled data + trust to automate later (mirrors your DarkVision human-in-the-loop flywheel — Story C).

## Worked Example C — "Design an LLM evaluation & observability platform" (maps to JD's 'model evaluation')
- **Why:** you can't ship prompt/model changes safely without it; this is a strong, differentiated thing to propose.
- **Components:** (1) a **golden eval dataset** (curated + sampled-from-prod, versioned); (2) an **offline eval runner** (faithfulness, relevance, citation, safety via LLM-as-judge + metrics) gating every prompt/model change like CI; (3) **online quality** (thumbs, CSAT, escalation, deflection) wired back; (4) **tracing** (every prompt/retrieval/tool/token/cost/latency) and **drift/cost dashboards + alerts**; (5) **feedback loop** funneling failures into the eval set.
- **Discuss:** how to validate the LLM-judge against humans; regression-testing prompts; A/B + guardrail metrics; cost & latency SLOs.

## Other GenAI prompts to rehearse
| Prompt | Angle |
|---|---|
| **Ticket summarization & routing** | LLM summary + intent classify → route; structured output; eval vs human summaries; cheap model + batch. |
| **Knowledge-base Q&A / search** | RAG with hybrid retrieval + rerank + citations (your EyeWiki experience). |
| **Multilingual support bot** | Detect language → translate or multilingual model; eval per-language; fallback. |
| **Email auto-response drafting** | Draft for agent review (assist, not auto-send); tone/policy guardrails; acceptance rate metric. |
| **Voice-of-customer analytics** | Batch LLM extraction of themes/sentiment from contacts → dashboards; cost via batching + small model. |

---

## Common follow-ups — have an answer
- **"How do you reduce hallucination?"** Ground with RAG, "answer only from context / say I don't know," cite sources, rerank for precision, output grounding-check guardrail, escalate on low confidence.
- **"How do you cut cost?"** Model tiering/routing, prompt caching (static prefix), semantic cache, cap output tokens, shorter context, distill a small model for the high-volume narrow task, batch the offline jobs.
- **"How do you cut latency?"** Stream (TTFT), smaller/faster model where quality allows, cache, parallelize retrieval + tool calls, shorter prompts, speculative/prefetch.
- **"How do you evaluate with no ground truth?"** LLM-as-judge validated against a human-labeled sample; reference-free metrics (faithfulness vs retrieved context); online behavioral signals (deflection, escalation, thumbs).
- **"Defend against prompt injection?"** Treat retrieved/user text as data not instructions, input filters, privilege separation for tools, output validation, least-privilege tool scopes, human approval for risky actions.
- **"When would you fine-tune?"** A consistent style/format/skill prompting can't reliably hit, a high-volume narrow task where a small fine-tuned model is cheaper/faster, and you have labeled data — *not* to inject fresh/changing knowledge (that's RAG).
