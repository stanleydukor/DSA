# GenAI Behavioral Interview: Prep

> **What this 5th round is:** A GenAI-focused behavioral interview. They'll ask about times you **built a GenAI application**, particularly distinguishing **a plain LLM application** from an **LLM + RAG** application. They want to see that you **understand the difference and when each is appropriate**, plus scenarios where it went **well** and where it **didn't**. As with all Amazon behavioral, demonstrate **scale, depth, and production experience**, with **measurable impact**.
>
> **Your two stories:**
> - **Tucknightly**: *successful* use of an **LLM (no RAG)** for creative generation.
> - **Care1 EyeWiki RAG**: an **LLM + RAG** retrieval system that *worked but was the wrong tool for the job* (over-engineered infra; a managed API + MCP retrieval would've been cheaper/simpler) → your *"unsuccessful / would do differently"* story.

---

## FIRST: Nail the concept: LLM vs LLM + RAG

Be able to say this crisply in ~45 seconds, because the rest of the interview leans on it.

| | **Plain LLM** | **LLM + RAG (Retrieval-Augmented Generation)** |
|---|---|---|
| **Core idea** | Model generates from its parametric knowledge + the prompt/context you give it. | Before generating, **retrieve** relevant documents from an external knowledge base and **inject them into the prompt** as grounding. |
| **Pipeline** | prompt → LLM → output | query → **embed → vector search (+ rerank) → top-k chunks** → prompt(+chunks) → LLM → grounded output **with citations** |
| **Best when** | The task is **generative / creative / transformative** and correctness isn't tied to a specific external corpus. | The task is **knowledge retrieval / factual Q&A** over a **specific, changing, or private corpus** the model doesn't reliably know, and you need **grounding + citations + freshness** without retraining. |
| **Why use it** | Fluency, style, reasoning, synthesis. | **Reduce hallucination**, cite sources, keep answers current, respect a private/proprietary knowledge base. |
| **Cost / infra** | Just model calls + prompt engineering. | Heavier: ingestion/chunking, an **embedding model**, a **vector DB**, a **retriever + reranker**, plus the LLM. More moving parts to host & monitor. |

**The one-liner judgment:** *"Use RAG when the answer must come from a specific body of knowledge the model doesn't reliably hold; it grounds the output and cites sources. Use a plain LLM when the value is in generation/creativity and there's no external corpus that must be the source of truth. Adding RAG where you don't need grounding just adds infra, latency, and cost."*

> That last sentence **is** the lesson of the Care1 story. Keep it ready.

---

## STORY 1 (SUCCESS): Tucknightly: an LLM application done right (no RAG)
**Question it answers:** *"Tell me about a time you used GenAI and it was successful."*

- **S:** I built **Tucknightly**, a consumer product that generates **personalized nightly bedtime stories** for children. The core value is **emotional**: each child has a recurring named companion and a persistent "story world," and a parent can whisper one line about the child's day ("rough drop-off at preschool") so tonight's story quietly helps them process it, in character, in their familiar world. The product is the **relationship over time**, not a one-off generator.
- **T:** Produce stories that feel **creative, age-appropriate, and continuous across hundreds of nights** (where the companion *remembers* prior nights) at low enough cost and latency to run as a nightly consumer ritual.
- **A:** I deliberately chose a **plain LLM (Claude), no RAG**, because this is a **generative/creative** task: there is no external corpus that *must* be the source of truth; grounding-and-citation would actively work against creative storytelling. Instead I engineered **structured personalization in the prompt**, not retrieval:
  - A **persistent Story World + companion description** generated once per child and always injected into the system prompt (familiarity without a knowledge base).
  - **Continuity via a lightweight model call** (a cheap **Haiku** pass) after each story to summarize it and extract 0–2 open "narrative threads"; the last few summaries + open threads feed the next story's context. *(This is a compact, structured memory, not vector RAG. I matched the tool to the need: a few hundred tokens of curated state beats a vector search here.)*
  - **Reading-level adaptation** by age (word count, sentence length, vocabulary) via prompt addendums.
  - **Tiered models for cost:** the expensive model writes the story; cheap models do summarization/continuity. Strict **per-child context isolation** so siblings' worlds never leak (query-scoped + row-level security).
  - *Justification of the call:* I considered RAG for "memory," but the memory here is small, structured, and parent-editable; a vector store would add a DB, embeddings, and retrieval latency for *worse* control over what the model sees. Prompt-level structured memory was the simpler, cheaper, more auditable choice.
- **R:** A working production pipeline where stories feel **part of an ongoing world** (recurring companion, threads that resurface) rather than disposable one-offs: the **differentiator vs every "tap-for-a-story" competitor**. *[Tune to real numbers: story generation in ~X seconds, ~$Y per story via model tiering, continuity sustained across N+ nights, parent retention/engagement lift.]*
- **Reflection:** The win came from **resisting the urge to over-engineer**. The fashionable move was RAG-for-memory; the right move was a small structured context and cheap continuity calls. **Match the GenAI architecture to the actual job: here, creativity + light structured memory, not retrieval.**

---

## STORY 2 (UNSUCCESSFUL / WOULD DO DIFFERENTLY): Care1 EyeWiki RAG
**Question it answers:** *"Tell me about a time you used GenAI and it was NOT successful / didn't go as planned."*

- **S:** At Care1, eye doctors needed to look up **ophthalmology knowledge fast** during clinical work, referencing **EyeWiki** (a large, authoritative ophthalmology encyclopedia). I built an **LLM + RAG** system for **fast, grounded retrieval** so clinicians could ask natural-language questions and get **cited answers** instead of manually searching.
- **T:** Stand up a retrieval assistant that answers ophthalmology questions **grounded in EyeWiki with citations** (so clinicians trust it), self-hosted for data-control reasons.
- **A:** RAG was the **conceptually correct** pattern here: this is a **retrieval/factual** task over a specific corpus, exactly where you want grounding + citations, *not* free LLM creativity (hallucinated medical guidance is unacceptable). So I built the full self-hosted stack: **chunking + metadata extraction** of EyeWiki, **sentence-transformer embeddings**, a **Qdrant vector store**, a **retriever + cross-encoder reranker**, a **local Ollama LLM** for generation, behind a **FastAPI + Gradio** UI. It worked and returned grounded, cited answers.
  - **Where it went wrong:** I over-invested in **self-hosted infrastructure**. Maintaining the embedding model, the vector DB, the reranker, and a local LLM was **heavy to operate, harder to scale, and more expensive in engineering time** than the problem warranted, for what was essentially a **bounded retrieval task over one public corpus**. The local LLM also lagged hosted frontier models on answer quality.
  - **The better approach (my key learning):** the same retrieval could be delivered **far more cheaply and simply** with a **managed LLM API (OpenAI/Anthropic) plus an MCP server** exposing EyeWiki as a retrieval tool, letting the model call retrieval on demand. That removes the vector-DB/embedding/reranker hosting burden, gets **better generation quality**, and **costs less in total** (no GPU/infra to run and maintain), while keeping grounding and citations.
- **R:** The system **functioned**, but the **architecture was the wrong cost/complexity trade-off**: I'd shipped a heavyweight self-hosted RAG stack where a **managed API + MCP retrieval tool** would have been **cheaper to run, simpler to maintain, and higher quality**. The honest result: a working prototype that I would **re-architect** before scaling. *[Tune to real numbers: e.g., reduced a multi-minute manual EyeWiki lookup to ~seconds; but est. self-hosted infra/maintenance cost vs ~X% cheaper managed-API path.]*
- **Reflection:** **Choosing RAG was right; choosing to self-host the entire RAG stack was wrong.** GenAI architecture decisions aren't just "RAG vs no-RAG"; they're also **build-vs-buy on the infra**. I learned to weigh **total cost of ownership** (hosting, maintenance, quality) up front, and to reach for **managed APIs + MCP tools** unless there's a hard reason (compliance, data residency) to self-host. *(Contrast with DarkVision Story A, where strict industrial compliance genuinely **did** justify on-prem local LLMs; the right answer depends on the constraint.)*

---

## Why these two pair perfectly
- **Story 1 (Tucknightly):** *correctly chose a plain LLM and avoided unnecessary RAG.* → Good judgment on **when NOT to use RAG**.
- **Story 2 (Care1):** *correctly chose RAG but over-built the infra.* → Honest, self-critical judgment on **build-vs-buy / total cost of ownership**.
- Together they prove you understand **LLM vs RAG conceptually**, you can ship both in production, and you have the **engineering judgment** to pick the right architecture *and* the right infrastructure, and to admit when you'd do it differently (**Earn Trust** signal).

---

## Deepen here for the July 9 1–2 PM round (Arun Rawlani + Yogesh Chinta)
> They **built** a production LLM summarization system for customer service (69K summaries/week, 24 marketplaces, 14 languages) — this is not a hypothetical for them. Both your stories are conceptually right, but they'll probe **operability**: what happens when a model call is slow, throttled, wrong, or unsafe, at their scale. Have real answers, not textbook ones. Full context on them: `../Interviewers/Yogesh_Arun_Team_Context.md`.

### Serving-stage resilience (retries, timeouts, fallback)
- **Timeouts:** every model call gets a hard timeout budget (part of your end-to-end latency budget, e.g. TTFT target). Don't let one slow call hang the whole request.
- **Retries with backoff + jitter:** retry transient failures (throttling, 5xx) with exponential backoff and jitter, capped attempts — never retry blindly on a call with side effects without idempotency.
- **Circuit breakers:** after repeated failures/throttles against a model/provider, trip a breaker and stop hammering it; route around it instead of degrading latency for every request.
- **Fallback / model routing:** on error, throttle, or breaker-trip, **fall back to another model or provider** (this is exactly what a **model-agnostic inference layer** buys you — Yogesh's team carried this through a Nova and gpt-oss migration with minimal architecture change, and the Nova Lite move alone cut cost 60% and p99 latency 40%). If you designed anything with a swappable model client/interface, say so explicitly — it's the difference between "I called an API" and "I built an inference layer."
- **Graceful degradation:** if no model is available in budget, don't fail hard — serve a cached/last-known-good result, a simpler rule-based fallback, or escalate to a human, depending on how costly a wrong/missing answer is.
- **Streaming:** stream tokens to hide total latency behind perceived latency (TTFT) wherever the UX allows it.

### Guardrails, as a layered system (not a single filter)
Think in **layers**, the way a real production GenAI security review does:
1. **Input PII redaction** — strip/mask PII before it ever reaches the model or logs.
2. **Content/PII filtering** (e.g., Bedrock Guardrails) — block disallowed content and anonymize residual PII in both directions (input and output).
3. **Contextual grounding check** — verify the output is actually supported by the retrieved/injected context; block or flag ungrounded claims rather than let them ship.
4. **Adversarial-input protection** — defend against prompt injection / jailbreak attempts, especially if any retrieved or user-supplied text could be mistaken for instructions.
Layering matters because each layer catches a different failure mode; a single toxicity filter or a single PII regex is not a guardrail system. If your own projects only had one of these layers, say which one honestly and name which layer you'd add next — that's a better answer than implying you had all four.

### Evaluation & drift in production
- **LLM-as-judge at scale:** automated quality scoring lets you cover far more traffic than manual/human review ever could (their framework went from ~1% human feedback coverage to 20% of all contacts). Be ready to say **how you'd validate the judge** — sample and compare judge scores against human labels periodically, and watch for the judge's own drift after a model or prompt change.
- **Quality + cost + latency monitoring together:** production LLM systems are graded on all three simultaneously — a change that improves quality but blows the p99 latency or cost budget isn't a win.
- **Rollout discipline for model/prompt changes:** treat a prompt or model swap like a deploy — canary/gradual rollout, offline regression eval first, online metrics (acceptance rate, escalation rate, CSAT) watched during rollout, easy rollback.

### If asked "tell me about debugging a production GenAI issue"
Match the rigor of a real incident: **root cause via logs/metrics** (not guesswork), work across **team/system boundaries** if the failure spans them, quantify the **before/after** (an availability %, a detection-time reduction, a cost delta), and close with the **monitoring gap you fixed** so it can't silently recur. A vague "we noticed errors and fixed the bug" story will read as shallow next to what this pair has actually done.

---

## Likely follow-up questions: have answers ready
- **"When would you use RAG vs fine-tuning vs a plain LLM?"**
  - *Plain LLM / prompt:* generative tasks, no fixed corpus, behavior shapeable via prompt.
  - *RAG:* answers must be grounded in a specific/changing/private corpus; need citations + freshness without retraining.
  - *Fine-tuning:* you need a consistent *style/format/behavior* or domain adaptation that prompting can't reliably get, and you have labeled data; doesn't add fresh knowledge the way RAG does.
- **"How do you evaluate a RAG system?"** Retrieval: **recall@k / hit-rate, MRR/NDCG** on a labeled query set. Generation: **faithfulness/groundedness** (is the answer supported by retrieved context?), **answer relevance**, citation accuracy, hallucination rate; plus latency and cost. (RAGAS-style metrics.)
- **"How do you reduce hallucination?"** Ground with retrieval, instruct "answer only from context / say 'I don't know'," cite sources, rerank for precision, constrain with schema validation (cf. Story A's Pydantic-validated extraction).
- **"How did you handle chunking / retrieval quality?"** Chunk by semantic sections with metadata, embed, **hybrid dense + BM25** where helpful, **cross-encoder rerank** the top candidates for precision.
- **"What's an MCP server and why would it help here?"** Model Context Protocol lets the LLM call external **tools/data sources** in a standard way; exposing EyeWiki retrieval as an MCP tool lets a managed model fetch grounded context on demand: RAG's benefit without hosting the whole retrieval stack yourself.
- **"How would you scale / monitor it in production?"** Cache frequent queries, monitor **retrieval quality drift** and answer faithfulness, log citations for audit, add fallbacks, and watch cost-per-query.
- **"How do you make LLM calls resilient in production?"** Per-call timeouts within an overall latency budget, retries with exponential backoff + jitter (only on idempotent/safe calls), circuit breakers to stop hammering a failing provider, fallback to another model/provider, graceful degradation (cached result, simpler rule-based answer, or human escalation) when nothing is available in budget.
- **"What does a model-agnostic architecture buy you, and what does it cost?"** Buys you: fast, low-risk migration between providers/models (cost or quality driven) and a natural place to hang retries/fallback/routing. Costs you: an abstraction layer to design and maintain, and you may not get 100% of a provider's unique features. Worth it once you expect to change models more than once — which in this space is "always."
- **"How would you validate an LLM-as-judge?"** Sample its scores against human labels periodically, track agreement rate, re-validate after any prompt/model change (the judge can drift too), and don't let it fully replace a small human-audit sample for high-risk contacts.

> **Delivery tip:** open each answer by naming the **architecture choice and why** (LLM vs RAG), then the **trade-off you weighed**, then the **measurable result**, then the **honest reflection**. That sequence is exactly what this round is grading.
