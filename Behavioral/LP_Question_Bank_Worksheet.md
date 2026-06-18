# Amazon LP Question Bank & Worksheet (START-R)

> **Method — START-R:** **S**ituation · **T**ask · **A**ction (with *justification* — discuss trade-offs/alternatives and why yours was most optimal) · **R**esult (quantified; **model metrics + business metrics**) · **R**eflection (what you learned).
> Lead with a one-line situation, then go deep when asked. **"I"** for your actions, **"we"** for team. **Don't play all your cards — leave room for follow-ups.**
> Timing target: Situation/Task ~1–2 min · Action ~5–6 min · Result ~1–2 min · extra stats 2–3 min if asked.

---

# PART 1 — Your Canonical Stories (full START-R)

## STORY A · QC Automation Tool (DarkVision)
**LPs:** Customer Obsession · Ownership · Invent & Simplify · Dive Deep · Bias for Action

- **S:** At DarkVision, analysts manually cross-checked engineering deliverables (interpretation checklists, 2D logs, acoustic-imaging analyses, burst-pressure and LAS reports) for consistency. It was a recurring, error-prone review that ate **~7 hours every week** and varied by reviewer.
- **T:** This wasn't assigned to me. I noticed the drag on the analysts and decided to fix the root cause rather than let it persist.
- **A:** I sat down with the analysts directly to understand their *actual* workflow and pain points, not my assumptions. Then I designed and built an **on-prem multi-agent document-QC system** (LangGraph + LangChain orchestrating local Ollama LLMs), with every extraction **schema-validated by Pydantic** so output is deterministic and auditable. I kept it **fully on-prem** to respect strict industrial compliance. *Justification:* a cloud API would have been faster to build but violated compliance and added per-call cost; deterministic schema validation was non-negotiable for an auditable QC tool.
- **R:** The recurring review dropped from **~7 hours/week to under 5 minutes**, with **consistent, auditable results** instead of reviewer-to-reviewer variation.
- **Reflection:** Talking to the people who feel the pain *before* writing a line of code is what made it land. The "not my job" reflex would have left real waste in place.

## STORY B · Biological-Age Model (Care1)
**LPs:** Invent & Simplify · Are Right A Lot · Frugality · Think Big · Learn & Be Curious

- **S:** A UK competitor built a "biological age" product by partnering with a clinic that handed them a **proprietary 10-year cardiovascular risk score**; they clustered patients by similar risk and averaged the top-k to derive bio-age. At Care1 we had **no such labeled risk score** — but we had an abundance of **fundus photographs with patient demographics including age**.
- **T:** Build a credible biological-age capability **without buying or licensing** the labeled risk score the competitor depended on.
- **A (modeling):** I reframed the problem around the data we already owned. I **LoRA-fine-tuned a DINOv2 foundation model** on bilateral fundus images so the image-embedding distribution aligns with the age distribution, using a **composite contrastive + regression objective**. Then I estimated biological age via **K-nearest-neighbors in that learned embedding space** (top-k average) — mirroring the competitor's clustering idea but grounded in imaging rather than a purchased label. *Justification:* licensing a risk score was expensive and created a vendor dependency; LoRA froze the backbone (few trainable params → less overfit on limited data) and reused an asset we already owned.
- **A (deployment at scale — the end-to-end pipeline):** TrueAge is a **visualization feature, not a real-time prediction**, so I built it as a **nightly automated batch pipeline** rather than an online endpoint:
  - **Distributed batch inference** computes embeddings for **every fundus image in the database**, sharded across workers, then writes each user's TrueAge result and **upserts the embedding into our AWS vector DB** that backs the KNN lookup.
  - I deployed model updates with **blue-green, not canary**. *Justification:* TrueAge is a KNN average over the **full** embedding population — every user's score depends on the whole index. A **canary subset would produce inconsistent/unfair scores** across users, and critically, **you can't mix embeddings from two model versions in one KNN space** (the distance geometry changes). So when the embedding model changes, I **recompute the entire index in a green environment, validate it end-to-end, then atomically switch** — guaranteeing every user is scored against one consistent embedding space.
  - Because it's not latency-sensitive, I **batch every X hours**: accumulate new/updated images into a single batch, run distributed inference, persist per-user results, and refresh the vector index — cheap, idempotent, and re-runnable.
- **R:** A working bio-age pipeline built **entirely on data we already had, with zero external label-licensing cost**, *and* a **production-grade, fully automated nightly pipeline** that re-embeds the full image database, serves consistent scores to every user via the vector DB, and updates safely with zero-downtime blue-green swaps.
- **Reflection:** The constraint (no risk-score label) forced a better, cheaper design. I learned to ask "**what signal do we already own?**" before assuming we need someone else's data — and that **deployment strategy must follow the math**: because KNN depends on a single consistent embedding space, blue-green (full recompute) was correct where canary would have broken correctness.

## STORY C · Thickness Segmentation / SAM2 (DarkVision)
**LPs:** Have Backbone; Disagree & Commit (the *disagree* side) · Insist on the Highest Standards · Invent & Simplify · Deliver Results · Earn Trust · Dive Deep

- **S:** A thickness-measurement workflow on volumetric/ultrasound data suffered from high turnaround time (TAT), algorithmic failures on poor-quality signal, and report-to-report variability between analysts on the same project.
- **T:** When I joined, the team wanted me to apply a **quick hotfix** to the existing method and avoid a new model — to dodge the burden of rigorous re-testing.
- **A:** I respectfully pushed back: the foundation itself was flawed, so patching it wouldn't fix the root causes. I **committed to delivering the hotfix they wanted (disagree *and* commit)** while, in parallel, privately developing a new segmentation approach around one principle: **if there is no signal, no mask is generated** (interpretable, no spurious masks). To prove it without risking production, I **generated synthetic data and fine-tuned SAM2** for both automatic and interactive segmentation, then demoed it. *Justification:* a working demo de-risks a bold change far better than an argument; honoring the near-term ask kept trust intact while I proved the better path.
- **R:** The demo convinced the team and **we adopted the new model**. Wins: it's **interpretable**, **eliminates analyst-to-analyst variation**, and as analysts correct masks on hard/low-signal cases through interactive mode, **the model and TAT keep improving over time** — a self-reinforcing human-in-the-loop flywheel.
- **Reflection:** I learned to honor the team's near-term ask while quietly de-risking the better idea with evidence. **A working demo beats an argument.**

## STORY D · Deployment Debate: Docker vs ONNX (DarkVision)
**LPs:** Have Backbone; Disagree & Commit (the *commit* side) · Earn Trust · Customer Obsession · Are Right A Lot
> *Counterweight to Story C. C shows you hold a position; D shows you concede to a better one. Together they prove judgment, not stubbornness. Lead with D for "disagreed and lost / changed your mind."*

- **S:** We had an internal desktop tool analysts use to process **billions of ultrasound frames**, doing heavy GPU image processing in C++/CUDA. I was deploying my interactive segmentation model into it, and the head of engineering and I disagreed on how to ship the inference.
- **T:** Decide the deployment architecture. My preference: **ONNX runtime embedded directly in the desktop app** for real-time, in-process inference. The VP of Engineering favored **containerizing with Docker and serving** (on a GPU, updated centrally via Kubernetes).
- **A:** I made my case on technical merits: auto-segmentation needs real-time, low-latency inference, and a service hop adds latency + failure modes (network errors, load balancing, infra to manage), so embedded ONNX kept it local and fast. His case was operational: Docker spares every analyst from CUDA setup and version-compatibility hell, and a single image lets us push one unified update to everyone. I **dug into the real constraints rather than defending my position**, and two facts changed my mind: **CUDA-compatibility issues across heterogeneous analyst machines were a recurring support burden**, and **some analysts had no GPU at all** — which my embedded approach simply couldn't serve. So I **committed to the Docker approach**. I contributed where it helped — keeping inference local within the container (GPU passthrough where available, **CPU fallback** for GPU-less machines) so we still addressed the latency concern — but the architecture and the call were his. On the model itself I **converted the PyTorch SAM2 model to ONNX and quantized it from FP32 → FP16**: I tested **INT8** but the **accuracy drop was too large**, so **FP16 was the deliberate balance between model size/latency and segmentation accuracy** — which kept inference fast even inside the container and on the CPU-fallback path.
- **R:** A **single maintainable deployment every analyst could run regardless of hardware**, centralized updates, no per-machine CUDA debugging. Adoption was smoother than my embedded design would have allowed.
- **Reflection:** My latency instinct wasn't wrong, but it optimized for the wrong thing. Once I weighed compatibility and the GPU-less analysts, the right call was clearly his — and **committing fully (not grudgingly)** was what mattered. **Good judgment includes working to disconfirm your own position.**

## STORY E · Developing a Junior Engineer to His Strengths (DarkVision)
**LPs:** Hire and Develop the Best · Strive to be Earth's Best Employer · Learn & Be Curious · Invent & Simplify

- **S:** I managed a junior engineer who struggled with conventional ML model development. He didn't work the textbook way — instead of reading documentation he'd stand up an MCP server to query it, and instead of running experiments by hand he built multi-LLM agents to run them for him.
- **T:** Get him productive and growing. My first instinct was that this style was bad for ML engineering and I should correct it.
- **A:** When I looked closer, I saw the pattern wasn't a weakness — it was a **different strength**: he reliably gets things done by **building tooling and automation**. So instead of forcing him onto model architecture where he stalled, I **redirected him to where he's exceptional**: building the internal tools and APIs that speed up our whole ML lifecycle. I also had him run a **lunch-and-learn**, which seeded the team's early adoption of AI assistants.
- **R:** He turned into a **force multiplier** — his tooling accelerated the team's experimentation and ML lifecycle, and he **drove company-wide adoption of AI assistants** in the early days, well before it was common practice.
- **Reflection:** Developing someone isn't molding them into the "ideal" engineer in your head; it's finding where their wiring is an asset and aiming it at the team's bottleneck. **Managing to strengths got far more out of him than correcting him would have.**

## STORY F · Shipping a Medical Model at Scale (Retina AI) — *Success & Scale → Broad Responsibility*
**LPs:** Success and Scale Bring Broad Responsibility · Insist on the Highest Standards · Customer Obsession · Deliver Results
> *Drafted from your real history to fill the matrix gap. Refine the numbers to your exact recollection.*

- **S:** At Retina AI I worked on a diabetic-retinopathy screening model deployed across a network serving **100K+ patients**, under **FDA 510(k)** clearance and **HIPAA** constraints — a setting where a model error can mean a **missed diagnosis for a real patient**.
- **T:** Ship a model that is not just accurate but **safe, auditable, and traceable at scale** — because the responsibility of a medical model grows with the number of patients it touches.
- **A:** I treated breadth of impact as breadth of responsibility. I led the **verification & validation (V&V)** work: locked datasets with documented provenance, **tuned thresholds toward recall** (missing disease is far costlier than a false alarm) and reported **PR curves** rather than headline accuracy, built an **active-learning loop** that surfaced low-confidence cases for clinician review, and kept full **traceability/audit trails** for the regulatory submission. *Justification:* in screening, recall-weighted thresholds and human-in-the-loop review were the right trade-off — a few more false positives are acceptable; a missed disease is not.
- **A (serving at scale):** Unlike the Care1 TrueAge batch pipeline, screening is **latency-sensitive at the point of care**, so I served the model for **real-time inference with NVIDIA Triton Inference Server** — fronted by a **load balancer** distributing requests across **multiple GPU replicas**, with **dynamic batching** to maximize throughput and **model versioning** for safe rollouts. That gave us the **speed and horizontal scale** to handle clinic traffic across the network without queueing patients. *(Contrast I can draw in the interview: TrueAge = batch + blue-green because it's visualization over a full KNN index; DR screening = Triton real-time + replicas because a clinician is waiting. Deployment strategy follows the use case.)*
- **R:** Cleared the regulatory **V&V bar** and deployed to **100K+ patients** with auditable performance — **~70% reduction in screening time** and **~15% accuracy improvement** over the prior workflow, while meeting HIPAA and 510(k) requirements.
- **Reflection:** At scale, "is the model accurate?" isn't enough — you own the downstream consequences. The traceability and recall-bias work wasn't overhead; it *was* the job, precisely because errors affect real patients.

## STORY G · The Distribution-Mismatch Mistake (Retina AI) — *your genuine "mistake / feedback" story*
**LPs:** Earn Trust (vocally self-critical) · Dive Deep (root cause) · Are Right A Lot (disconfirmed my own confidence) · Insist on the Highest Standards (fix it so it stays fixed) · Learn & Be Curious
> *Use this as your primary "tell me about a mistake / a bad decision / a time your model failed" answer — it's a real, self-critical, well-resolved mistake, which is exactly what the Bar Raiser wants.*

- **S:** Early in my career at Retina AI, I built a model that looked great offline — we **trained, validated, and tested** it and the test metrics were strong — so we shipped it. In **production it performed extremely poorly**.
- **T:** Figure out why a model with strong test metrics was failing on real patients, fix it, and make sure it couldn't happen again.
- **A:** I **dove deep into the failure** rather than assuming the model architecture was wrong. The root cause was a **distribution mismatch (dataset shift / train–serving skew)**: our **test set wasn't representative of the real production distribution** — different cameras/clinics, image quality, and patient populations — so the offline metrics were **optimistically biased**. The model wasn't broken; our **evaluation was**. I corrected it by **rebuilding the eval set to mirror the production distribution** (sampling across the real device/clinic/population mix), and then I **institutionalized observability so this class of bug is caught automatically**: I added **data-drift detection** to monitoring — **Kolmogorov–Smirnov (KS) tests** on input feature and score distributions (and population-stability checks) comparing live traffic against the training distribution — wired to **alerts and automated retrain triggers** when drift crosses a threshold. *Justification:* a representative test set fixes the immediate miss; **drift monitoring with triggers** fixes the *process* so silent distribution shift surfaces before patients are affected.
- **R:** Once the eval set was representative, production performance matched offline expectations again, and the **drift-monitoring + trigger system became a standard part of our ML observability** — turning a one-time failure into a permanent guardrail for every model after it.
- **Reflection:** **Offline metrics are only as trustworthy as the representativeness of your eval data.** I stopped treating a good test score as proof and started asking "does my test set look like production?" — and I learned that **owning the mistake openly** (vocally self-critical, not defensive) is what built trust with the team. Production ML isn't done at deployment; **observability and drift detection are part of the deliverable.**

---

# PART 2 — Per-LP Worksheet (questions + recommended example)

> For each LP: definition, common Amazon questions, and which story to lead with. **Blank fields are for you to fill** with your own additional examples.

---

## 1. Customer Obsession ★
**Definition:** Leaders start with the customer and work backwards; they earn and keep customer trust and obsess over customers, not competitors.
**Sample questions:**
1. Describe a difficult interaction with a customer. How did you deal with it? Outcome? What would you do differently?
2. Tell me about a time you went above and beyond for a customer. Why? How did they respond?
3. A time you anticipated a customer need with a solution they didn't know they wanted yet. How did you know? How did they respond?

**Lead with:** **Story A** (talked to analysts before building — internal customer). **Backup:** Story D (CPU fallback for GPU-less analysts), Story F (recall-tuned for patients).
**Your own example (optional):** _______________________________________________

---

## 2. Ownership ★
**Definition:** Leaders are owners; think long-term, don't sacrifice long-term value for short-term results, act on behalf of the whole company, never say "that's not my job."
**Sample questions:**
1. A time you took on something significant outside your area of responsibility. Why important? Outcome?
2. A time you didn't think you'd meet a commitment. How did you identify and communicate the risk? Do differently?
3. An initiative you undertook because it benefited the company/customers but wasn't anyone's individual responsibility.

**Lead with:** **Story A** ("this wasn't assigned to me… I decided to fix the root cause"). **Backup:** Story C (took on a flawed foundation no one wanted to rebuild).
**Your own example (optional):** _______________________________________________

---

## 3. Invent and Simplify
**Definition:** Leaders expect innovation and always find ways to simplify; externally aware, not limited by "not invented here."
**Sample questions:**
1. A complex problem you solved with a simple solution. What made it complex? How do you know it worked?
2. The most innovative thing you've done and why. (They may ask for 1–2 more to check for a *pattern*.)
3. A time you made something simpler for customers. What drove it? Impact?

**Lead with:** **Story B** (reframed around owned data — novel, cheaper). **Backup:** Story A (multi-agent QC), Story C ("no signal → no mask").
**Your own example (optional):** _______________________________________________

---

## 4. Are Right, A Lot
**Definition:** Strong judgment and good instincts; seek diverse perspectives and work to **disconfirm** your beliefs.
**Sample questions:**
1. A time you didn't have enough data to make the right decision. What did you do? Was it correct?
2. A strategic decision without clear data/benchmarks. Alternatives? Trade-offs? How did you mitigate risk?
3. A time you made a bad decision. / A time you discovered your idea wasn't the best course of action.

**Lead with:** **Story D** (actively disconfirmed my own ONNX position; changed my mind on evidence). **Backup:** Story G (trusted a good test score that was wrong → learned to disconfirm offline metrics), Story B (judgment with no labels).
**Your own example (optional):** _______________________________________________

---

## 5. Learn and Be Curious
**Definition:** Leaders are never done learning and always seek to improve; curious about new possibilities and act to explore them.
**Sample questions:**
1. The coolest thing you learned on your own that helped you perform your job.
2. A time you realized you needed deeper subject-matter expertise to do your job well.
3. A time you took on work outside your comfort area and found it rewarding.

**Lead with:** **Story E** (drove early AI-assistant adoption) or **Story B** (learned foundation-model fine-tuning / contrastive learning to crack bio-age).
**Your own example (optional):** _______________________________________________

---

## 6. Hire and Develop the Best
**Definition:** Raise the bar with every hire/promotion; recognize talent, develop leaders, take coaching others seriously.
**Sample questions:**
1. A time you provided feedback to develop someone's strengths. Did you positively impact their performance?
2. A time you coached someone through a difficult situation / saw potential others missed.

**Lead with:** **Story E** (redirected the junior engineer to his tooling strength → force multiplier).
**Your own example (optional):** _______________________________________________

---

## 7. Insist on the Highest Standards
**Definition:** Relentlessly high standards (many think unreasonably high); raise the bar; defects don't get sent down the line; fix problems so they stay fixed.
**Sample questions:**
1. A time you wouldn't compromise on a great outcome when others felt it was "good enough."
2. A time you were unsatisfied with the status quo and changed it. Impact?
3. A time you improved the quality of something already getting good feedback. Why? How did customers react?
4. A goal where you wish you'd done better. How could you improve?

**Lead with:** **Story C** (refused to patch a flawed foundation; built the rigorous replacement). **Backup:** Story F (FDA 510(k) V&V), Story G (built drift monitoring + retrain triggers so the defect couldn't recur — fix it so it stays fixed).
**Your own example (optional):** _______________________________________________

---

## 8. Think Big
**Definition:** Create and communicate a bold direction that inspires results; think differently and look around corners for customers.
**Sample questions:**
1. A time you saw an opportunity to do something much bigger/better than the initial focus. Did you take it? Outcome?

**Lead with:** **Story B** (turned an internal dataset into a whole new bio-age *capability/product* rivaling a funded competitor).
**Your own example (optional):** _______________________________________________

---

## 9. Bias for Action ★
**Definition:** Speed matters; many decisions are reversible and don't need extensive study; value calculated risk-taking.
**Sample questions:**
1. A calculated risk where speed was critical. How did you mitigate it? Outcome? Do differently?
2. A time you worked against tight deadlines without time to consider all options. Approach? Lesson?
3. An important business decision made without consulting your manager. Outcome?

**Lead with:** **Story C** (built a synthetic-data demo to move the decision forward instead of debating). **Backup:** Story A (built the QC tool proactively).
**Your own example (optional):** _______________________________________________

---

## 10. Frugality
**Definition:** Accomplish more with less; constraints breed resourcefulness and invention; no extra points for headcount, budget, or fixed expense.
**Sample questions:**
1. A time you had to do more with less. How? Outcome?
2. A time a constraint (budget/resource/time) drove a more creative solution.

**Lead with:** **Story B** (avoided licensing an expensive risk score — built on owned data, zero label cost). **Backup:** Story A (on-prem local Ollama LLMs vs paid API).
**Your own example (optional):** _______________________________________________

---

## 11. Earn Trust
**Definition:** Listen attentively, speak candidly, treat others respectfully; be **vocally self-critical** even when awkward; benchmark against the best.
**Sample questions:**
1. A time you communicated a change people would have concerns with. How did you understand & mitigate them?
2. A tough/critical piece of feedback you received. What did you do about it?
3. A time you needed to influence a peer with a differing opinion on a shared goal.
4. Direct feedback you recently gave a colleague. How did they respond? How do you like to receive feedback?

**Lead with:** **Story G** for *vocally self-critical / mistake / feedback* questions (owned the production failure openly). **Story D** for *influence / concede / changed-my-mind* questions (conceded gracefully, committed fully). **Backup:** Story C (earned buy-in via demo).
**Your own example (optional):** _______________________________________________

---

## 12. Dive Deep
**Definition:** Operate at all levels, stay connected to details, audit frequently, skeptical when metrics and anecdote differ; no task beneath you.
**Sample questions:**
1. A complex problem where you had to dig into the details. Where did you look? How did it help solve it?
2. A situation that required digging to the root cause. How did you know you were on the right things? Outcome?
3. A problem requiring in-depth thought/analysis. Outcome? Do differently?

**Lead with:** **Story D** (dug into real CUDA/GPU constraints rather than defending a position). **Backup:** Story G (root-caused a production failure to distribution mismatch), Story A (learned analysts' actual workflow), Story B (embedding-space analysis).
**Your own example (optional):** _______________________________________________

---

## 13. Have Backbone; Disagree and Commit ★
**Definition:** Respectfully challenge decisions you disagree with, even when uncomfortable; once decided, commit wholly.
**Sample questions:**
1. A time you strongly disagreed with a manager/peer on something important. How did you handle it? Do differently?
2. A time you took an unpopular stance with peers and your leader. Why? What did you do? Outcome?

**Lead with — the pair:** **Story C** = the *disagree* side (held the line on a flawed foundation). **Story D** = the *commit* side (conceded to Docker and committed fully). Use C for "stood your ground," D for "disagreed and lost / changed your mind."
**Your own example (optional):** _______________________________________________

---

## 14. Deliver Results
**Definition:** Focus on key inputs, deliver with quality and on time; despite setbacks, rise to the occasion and never settle.
**Sample questions:**
1. An important project delivered under a tight deadline. Sacrifices? Final outcome?
2. Significant unanticipated obstacles to a key goal. Successful? Do differently?
3. A time you not only met but considerably exceeded expectations. How?

**Lead with:** **Story C** (shipped the adopted model + the flywheel). **Backup:** Story F / Care1 screening (**~70% screening-time reduction, ~15% accuracy gain**).
**Your own example (optional):** _______________________________________________

---

## 15. Strive to be Earth's Best Employer
**Definition:** Create a safer, more productive, more inclusive, empathetic work environment; lead with empathy; help people grow.
**Sample questions:**
1. A time you helped a teammate grow or supported someone's wellbeing/development.
2. A time you made the team environment better / more inclusive.

**Lead with:** **Story E** (empowered and grew the junior engineer to his strengths). **Backup:** Springboard mentoring.
**Your own example (optional):** _______________________________________________

---

## 16. Success and Scale Bring Broad Responsibility
**Definition:** Size has impact and responsibility; be humble, thoughtful, do the right thing — decisions must be better for customers, employees, partners, and the world.
**Sample questions:**
1. A time you considered the broader impact/consequences of your work beyond the immediate goal.
2. A time you did the right thing even though it was harder, because of who it affected.

**Lead with:** **Story F** (shipping a medical model to 100K+ patients → V&V, traceability, recall-bias *because* errors affect real patients).
**Your own example (optional):** _______________________________________________

---

# PART 3 — Two More to Have Ready (commonly asked, light prep)

### "Why Amazon?"
Amazon's scale of ML + customer obsession means models are judged by **real customer impact**, not benchmarks — which is exactly how I like to work. The Traffic Engineering problem space (forecasting, ranking, optimization at scale) lets me apply production ML where a fraction of a percent compounds into large business value, and the ownership culture matches how I already operate (Story A — I fix root causes whether or not it's "my job").

### "Tell me about a mistake / feedback you received" (Earn Trust)
**Lead with Story G** — the production failure from distribution mismatch: I trusted a strong test score, shipped, and it failed on real patients because my eval set wasn't representative. I owned it openly, root-caused it, fixed the eval set, and institutionalized **KS-test drift monitoring + retrain triggers** so it couldn't recur. *(A real, self-critical, well-resolved mistake is exactly what the Bar Raiser wants.)*
**Alternate (changed-my-mind flavor): Story D** — my latency instinct optimized for the wrong thing; I missed the GPU-less analysts and the CUDA support burden, so I worked to disconfirm my own position and committed fully once the better call was clear.

---

# PART 4 — Deployment & Scale Talking Points (weave into Actions)

Your stories now carry **end-to-end, production-at-scale** depth. Pull the matching one when an interviewer probes "how did you deploy / scale / monitor it?":

| Pattern | Story | The crisp talking point |
|---|---|---|
| **On-device / edge optimization** | C / D | PyTorch → **ONNX**, **quantized FP32 → FP16** (tested INT8, accuracy drop too large → FP16 balances size/latency vs accuracy); GPU passthrough + **CPU fallback** in a Docker container. |
| **Distributed batch inference + blue-green** | B (TrueAge) | Nightly distributed batch re-embed of the **full** image DB → AWS vector DB; **blue-green (not canary)** because KNN needs **one consistent embedding space** across all users — recompute whole index in green, validate, atomic switch. |
| **Batch vs real-time decision** | B vs F | TrueAge = **batch every X hours** (visualization, not latency-sensitive); DR screening = **real-time** (clinician waiting). *Deployment strategy follows the use case.* |
| **Real-time serving at scale** | F (Retina AI) | **NVIDIA Triton Inference Server** + **load balancer** across **GPU replicas** + **dynamic batching** + model versioning → speed and horizontal scale. |
| **Observability / drift** | G | **KS-test** drift detection on input + score distributions vs training data, **alerts + automated retrain triggers**; representative test set = production distribution. |
| **Deployment strategies vocabulary** | — | **Canary** (small % first), **Blue-green** (two full envs, atomic switch), **Shadow** (run new model silently alongside old). Know when each applies. |

> **Why this matters for an L5 / Senior MLE bar:** the loop wants to see you can **build models AND own the infra to train, deploy, monitor, and scale them.** Each story above can flex from a behavioral answer into a mini system-design discussion if the interviewer pulls the thread — let them.

---

## Reminders before you walk in
- Lead with a **one-line situation**, go deep when asked. **Quantify** every result.
- **"I"** for actions, **"we"** for context. Discuss **trade-offs & why your choice was optimal** (Action with justification).
- For ML stories, pair **model metrics (offline + online)** with **business metrics ($ / time / efficiency)**.
- Be **vocally self-critical** on mistake/feedback questions; never blame others.
- **One story → many LPs.** Pivot the framing to the question asked.
