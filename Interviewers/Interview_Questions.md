# Questions to Ask Each Interviewer — July 9–10, 2026 Loop

> Public-profile research on Ray Cabrera, Mael Mugerwa, and Ric Huang came back **unconfirmed** — no LinkedIn profile could be reliably matched to the specific person on your loop (common names, no Amazon-specific + role-specific match). Their questions below are built from **title + slot position only** — still tailored, just not personalized to a verified bio. Jeroen Dirks and Srikara Krishna Kanduri each had a plausible profile match (noted inline, confidence: likely, not certain). Yogesh Chinta and Arun Rawlani are fully documented — see `Yogesh_Arun_Team_Context.md` for the deep-dive; questions below are the condensed, ask-out-loud versions.

---

## Round 1 — Ray Cabrera, Software Development Manager
**July 9, 10:00–11:00 AM PDT. Likely round type: hiring-manager / team-fit / leadership-principles behavioral (possibly with a light design component).**
*No confirmed public profile found — questions below are inferred from his being the sole manager and opening slot on the loop.*

1. "What does this team look like a year from now, and what's the single biggest capability gap you're hiring for right now?"
2. "How is success measured for someone in this role at the 6-month and 12-month mark?"
3. "What's the balance between maintaining the existing production summarization system versus building new Agentic AI capability — where would you want a new hire spending their time first?"
4. "How does your team handle on-call / production ownership for a system that's directly in the customer-service critical path?"
5. "What's a recent decision where the team chose the simpler/cheaper option over the more sophisticated one — and how did that turn out?"

---

## Round 2 — Mael Mugerwa & Ric Huang, Software Dev Engineer II
**July 9, 11:00 AM–12:00 PM PDT. Likely round type: coding / DSA (paired two-interviewer panel is a common Amazon format for this).**
*No confirmed public profiles found for either — questions below are generic-but-genuine peer/IC questions, safe for a coding round's post-problem discussion.*

1. "What does the day-to-day split between new feature work and maintaining production LLM pipelines actually look like for an SDE II on this team?"
2. "What's the most annoying or surprising part of working with LLM-based systems day-to-day that you didn't expect coming from more traditional backend work?"
3. "How does code review / design review work here when a change touches the model layer versus a change that's plain application code?"
4. "What tooling or internal platform do you wish existed but doesn't yet?"
5. "What's the most interesting bug you've had to track down in this system?"

---

## Round 3 — Jeroen Dirks, Sr. Software Dev Engineer
**July 9, 12:00–1:00 PM PDT. Likely round type: coding, possibly with an architecture/design lean.**
*Confidence: likely (not certain) match — [LinkedIn profile](https://www.linkedin.com/in/jeroendirks/): Toronto-based, describes himself as someone who "loves to build elegant software," with background suggesting an architecture/quality focus (earlier testimonials reference Oracle Numetrix work on software quality and technical leadership).*

1. "You describe caring about building elegant software — where in this codebase are you proudest of the design, and where do you think it still needs to evolve?"
2. "As a senior engineer, how much of your time is hands-on-keyboard versus design review / mentoring right now, and how would that change for a new senior-leaning hire?"
3. "When the team has to trade off elegant long-term architecture against shipping fast on an LLM-related feature, how does that get resolved in practice?"
4. "What's a technical decision on this system you'd revisit if you were starting over today?"
5. "How opinionated is the team about code quality/architecture standards specifically for the parts of the system that call out to LLMs, versus the rest of the codebase?"

---

## Round 4 — Arun Rawlani (SDE II) & Yogesh Chinta (Software Dev Mgr., Amazon CS)
**July 9, 1:00–2:00 PM PDT. This is the GenAI behavioral round — see `Yogesh_Arun_Team_Context.md` for full context; these are the two most important people on the loop, and very likely your actual future team/manager.**

1. "Yogesh — you led the org's first GenAI Application Security Review; now that the 4-layer guardrail pattern exists, is it a reusable framework other teams adopt, or does each new GenAI project still have to fight that battle from scratch?"
2. "Arun — after diagnosing the EU ingestion failure, what monitoring or alerting exists today that would catch that class of problem before it got to 99% failure, or is that still a gap?"
3. "How do you decide when the LLM-as-judge framework's automated score is trustworthy enough to act on versus when it still needs a human in the loop, especially for a new language or marketplace you're just rolling out to?"
4. "Yogesh — you own the Agentic AI roadmap; what's the first thing this system does today that will look meaningfully different once it's agentic rather than single-shot summarization?"
5. "What would make you both say a new hire's first 90 days on this team were a clear success?"

---

## Round 5 — Srikara Krishna Kanduri, Software Development Engineer
**July 10, 2:00–3:00 PM PDT. Likely round type: coding, possibly with a design/architecture lean.**
*Confidence: likely (not certain) match — [LinkedIn profile](https://ca.linkedin.com/in/srikarakrishna): Vancouver-based, 6.5+ years of experience described as spanning designing and architecting, educated at NC State.*

1. "With 6+ years of experience across design and architecture work, what's the biggest shift you've seen in how this org builds software since you joined?"
2. "How much cross-team coordination does an individual contributor typically end up doing here, given how much this system touches other teams' data (CRM, order systems, contact channels)?"
3. "What does the interview loop and onboarding process tell you about what this team actually values day-to-day, versus what's on the job description?"
4. "What's a project you've architected here that you're proud of, and what would you do differently knowing what you know now?"
5. "Being on the Vancouver side, is the team distributed across multiple sites/time zones, and how does that affect how work gets divided?"

---

## General notes for all rounds
- Keep 1–2 questions **in reserve** per round in case the interviewer answers one of your planned questions organically during the interview itself.
- For the unconfirmed interviewers (Ray Cabrera, Mael Mugerwa, Ric Huang), the questions above are deliberately **role/title-safe** rather than bio-specific — don't reference LinkedIn details you don't actually have confirmed, since guessing wrong reads worse than asking a good generic question.
- Save your sharpest, most system-specific questions (Round 4) for Yogesh and Arun — that's the round where specificity will land hardest and where you have the most real signal to work with.
