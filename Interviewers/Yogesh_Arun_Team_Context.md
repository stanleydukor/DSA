# Yogesh Chinta & Arun Rawlani — Team Context (the two people who matter most)

> **Round:** July 9, 1:00–2:00 PM PDT. **Arun Rawlani** (SDE II) + **Yogesh Chinta** (Software Dev Mgr., Amazon CS).
> This is very likely the **GenAI Behavioral round** referenced in your prep (`../Behavioral/GenAI_Behavioral.md`) — Yogesh is the manager/tech-lead and Arun is the senior IC on the *exact* system that round is designed to probe. They are not abstractly evaluating "do you understand RAG" — they are checking **"would this person have been useful on the system I actually built."**

---

## Who they are, in one line each
- **Yogesh Chinta** — Engineering *manager* and tech lead who took the LLM summarization system from POC → production, mentoring two engineers. Owns vision/strategy/roadmap for the team's Agentic AI direction.
- **Arun Rawlani** — Senior IC (4+ yrs Amazon, prior Capital One) who did the hands-on engineering: the first production consumer of AWS Connect Contact Lens for voice-channel data, the EU ingestion outage diagnosis, the access-governance redesign.

They are describing **the same system** from two altitudes: Yogesh = strategy/roadmap/impact framing, Arun = "I built the pipes and debugged the outages." Expect Yogesh's questions to probe **judgment, trade-offs, and how you'd extend/lead this**, and Arun's to probe **how deep your hands-on production experience actually goes**.

---

## The system, distilled (know this cold — they will assume you read it)
**What it does:** Condenses customer-service interaction histories (chat, email, voice via Contact Lens transcripts) across **14 languages** into agent-facing summaries. **69K summaries/week, 24 marketplaces.** 87.9% agent-helpful rating. $2.76M projected annual savings / ~3,612 labor-hours saved per week.

**How it's built (the parts that matter for your answers):**
- **Model-agnostic inference layer** — abstracted cleanly enough to migrate across **Amazon Nova** and **OpenAI gpt-oss** with minimal architecture change. Nova Lite migration: **−60% inference cost, −40% p99 latency**. → *This is their headline cost/latency lever. If asked "how would you cut cost/latency," a generic "prompt caching" answer will read as thin next to what they actually shipped — lead with model tiering/routing behind an abstraction, then caching.*
- **LLM-as-Judge eval framework** — replaced human agent feedback (1% response rate) with automated quality scoring covering **20% of all contacts** (20x signal increase). Enables continuous quality monitoring with no manual review bottleneck. → *This is Yogesh's own JD talking point ("model evaluation"). Have a real opinion on how you'd validate an LLM-judge against human labels, and how you'd catch the judge drifting.*
- **4-layer security architecture** (Yogesh led the org's first GenAI Application Security Review): input PII redaction → Bedrock Guardrails content filtering + PII anonymization → contextual grounding → adversarial-input protection. Validated across 14 languages. → *"Guardrails" to them means this specific 4-layer stack, not a vague word. Be ready to reason about defense-in-depth, not a single filter.*
- **Team-based access governance** for a customer-escalation tool: **94% reduction in production data exposure** (4,484 → 275 weekly searches), 83% fewer access requests, +33% test-account adoption — delivered in **1.5 months** vs. a proposed multi-quarter alternative (Arun led this). → *A strong signal for "move fast, minimize scope, don't over-engineer" — the same lesson as your Care1 story. Draw the parallel if it comes up.*
- **Voice-channel unlock** — Arun integrated with AWS Connect Contact Lens as the **first production consumer**, +119% weekly summary volume.
- **EU ingestion outage** — Arun diagnosed a **99.55% EU data-ingestion failure** within 7 days across 4 teams, brought EU transcript availability from 0.45% → 81%. Yogesh separately led a postmortem for a different multi-system outage, presented to a Principal Engineer + Director, improved incident-detection time by **29%** across 5 teams. → *Both have real incident/on-call depth. If you get "tell me about debugging a production issue," match this level of rigor: root cause via logs/metrics (they used CloudWatch), cross-team coordination, a concrete detection-time or availability number, and follow-up monitoring gaps closed.*

---

## What they're each probably trying to figure out about you
**Yogesh (manager, roadmap owner):**
- Can you reason about **build vs. buy / total cost of ownership** on GenAI infra the way he did (POC → production, choosing Bedrock/managed services over self-hosting)? *(Your Care1 story is exactly this lesson — lean into it.)*
- Do you think about **safety and compliance as first-class**, not an afterthought? He personally stood up the security-review pattern for the org.
- Can you operate with **scale and ambiguity across markets** (14 languages, 24 marketplaces) — i.e., do your answers generalize past "it worked for my one use case"?
- Would you be **mentee-able / mentorable** and eventually able to own a slice of the roadmap? He explicitly mentors engineers.

**Arun (senior IC, the one who'll actually pair with you day-to-day):**
- Do you have genuine **production operability instincts** — retries, timeouts, fallback, monitoring, on-call — or only prototype-level experience?
- Can you go **deep on one technical thread** under follow-up questioning (he clearly can — a 7-day root-cause across 4 teams isn't shallow)?
- Do you sweat **cost and latency** as real engineering constraints, not just "it ran on my laptop"?
- Are you comfortable with **cross-team coordination during an incident** — proposing your part of a fix while others own theirs?

---

## What you'll possibly be doing on this team
Based on their roadmap language ("Agentic AI roadmap," "planet-scale platforms," mentoring, ongoing model migrations):
- Extending or hardening the **LLM summarization pipeline** — new channels/languages, or generalizing it toward **agentic** (multi-step, tool-calling) customer-service assistance, not just single-shot summarization.
- Working inside the **model-agnostic inference abstraction** — likely picking up the next model migration (they've already done Nova + gpt-oss; more will come) and optimizing cost/p99 latency the way the Nova Lite migration did.
- Contributing to or consuming the **LLM-as-Judge eval framework** — extending automated quality coverage, or building new eval dimensions (safety, groundedness) as the system scales beyond 20% contact coverage.
- Touching the **guardrails/security stack** — since this is an org where a formal GenAI security review already happened, expect PII/redaction, Bedrock Guardrails, and adversarial-input concerns to be a normal part of code review, not an afterthought.
- Possibly **on-call/operational ownership** — given both interviewers have hands-on outage/postmortem stories, production ownership (not just shipping a model) looks like a real part of the job.

---

## How to calibrate your answers for this specific pair
- **Vocabulary match:** use their terms — "guardrails," "grounding," "PII redaction," "model-agnostic," "LLM-as-judge," "p99 latency," "model tiering/routing," "canary/gradual rollout" for a model migration. It signals you've operated at their altitude, not just read about RAG.
- **Lead with judgment, not just outcome:** their strongest stories are all "I chose the cheaper/simpler/safer path and here's the number that proves it" (Nova migration cost, access-governance timeline, security-review pattern). Your Care1 "would do differently" story is the same shape — say so explicitly if it's natural.
- **Don't be afraid of the security/compliance angle:** most candidates under-index on this. Given Yogesh literally built the security-review process, showing you think about PII/adversarial input/guardrails unprompted is a differentiator.
- **Expect serving-level follow-ups**, not just "LLM vs RAG" — retries/timeouts/circuit breakers around model calls, fallback models on throttle/error, streaming, and how you'd validate an automated judge. See the enriched serving section in `../Behavioral/GenAI_Behavioral.md`.
