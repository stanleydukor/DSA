# Coding Interview — Template & Checklist

> **Amazon MLE loop reality:** 3 of your 5 interviews are coding (LeetCode easy/medium, occasionally hard) on a shared LiveCode editor. They grade **three** things explicitly:
> 1. **DS&A knowledge** — pick the right data structure/algorithm and *justify* it.
> 2. **Problem-solving** — disambiguate, break the problem down, narrate trade-offs.
> 3. **Logical & maintainable code** — clean, correct, extensible code another engineer could read.
>
> Behavioral/LP questions are woven into *every* coding round too. One interviewer is the **Bar Raiser**. **Talk out loud the entire time — silence reads as "stuck."**

---

## The 5-Step Method (run this on EVERY problem)

### 1. Clarify (before writing anything)
Restate the problem in your own words, then ask:
- **Input ranges / scale:** how large is `n`? Fits in memory? Streaming?
- **Types & domain:** integers? negatives? floats? Unicode? signed overflow?
- **Edge inputs:** empty? single element? all duplicates? null? already sorted?
- **Duplicates allowed?** Is the array **sorted**? Are values **unique**?
- **Output shape:** indices vs values? any valid answer or all answers? in-place?
- **Constraints:** must it be in-place / O(1) space? latency budget?
- **Examples:** ask for or construct 1 normal example + 1 edge example.

> ✅ Explore edges *before* coding. "If I don't have enough info to solve it, I probably don't — so I'll ask." This is a graded signal (Problem Solving).

### 2. Plan (get a nod before coding)
- State the **brute force** first + its time/space complexity. ("Naively this is O(n²) because…")
- Propose the **better approach** and *why* it's better (what insight unlocks it — sorting? a hash map? two pointers? a heap?).
- Name the **data structure** and justify it against alternatives. ("A hash map gives O(1) lookup vs O(log n) for a BST, and I don't need ordering, so hash map.")
- **Pause for buy-in:** "Does that approach sound good before I implement?"

### 3. Implement (real code, not pseudocode)
- Descriptive names (`seen`, `left`/`right`, `freq`, `node.next`), not `x`, `tmp2`.
- **Narrate as you go** — say what each block does and why.
- Write the helper signatures first if it's complex; fill in bodies.
- Keep it modular: a clean helper beats a 40-line monolith. Think "what if requirements change?"
- Handle the edge cases you surfaced in Step 1 (guard clauses up top).

### 4. Test (do this UNPROMPTED)
- **Dry-run** a simple example line by line, tracking variable state out loud.
- Then hit **edge/corner cases**: empty, single element, duplicates, overflow, cycle, negative, max size.
- If you find a bug, fix it calmly and explain the fix. Finding your own bug is a *positive* signal.

### 5. Optimize & State Complexity
- State final **time and space complexity** explicitly (Big-O, and *why*).
- Discuss trade-offs: "I could cut space to O(1) but it'd cost readability / a second pass."
- Mention how you'd push further or what changes at scale (sharding, streaming, external sort).

---

## Maintainable-Code Checklist (the "Logical & Maintainable" rubric)
- [ ] Would someone with *minimal context* understand this solution?
- [ ] Are names self-documenting? No magic numbers.
- [ ] Are edge cases guarded explicitly, not implicitly?
- [ ] Is logic broken into small, testable units (helpers) where it helps?
- [ ] If a new requirement landed (e.g., "now also return the path"), could I extend without a rewrite?
- [ ] What downstream effects if I change this function's contract?
- [ ] No dead code, no copy-paste duplication.

---

## Complexity Cheat Sheet (know these cold)
| Structure / Op | Access | Search | Insert | Delete | Notes |
|---|---|---|---|---|---|
| Array | O(1) | O(n) | O(n) | O(n) | O(log n) search if sorted |
| Hash map / set | — | O(1)* | O(1)* | O(1)* | *amortized; worst O(n) |
| Balanced BST / TreeMap | — | O(log n) | O(log n) | O(log n) | ordered iteration |
| Binary heap | O(1) peek | O(n) | O(log n) | O(log n) | top-k, streaming |
| Stack / Queue / Deque | — | O(n) | O(1) | O(1) | LIFO/FIFO/both |
| Doubly linked list | O(n) | O(n) | O(1)** | O(1)** | **given the node (LRU) |
| Trie | — | O(L) | O(L) | O(L) | L = key length, prefix search |
| Union-Find (DSU) | — | ~O(α(n)) | ~O(α(n)) | — | connectivity, cycles in undirected |

**Pattern → tool reflexes:**
- Sorted array / "find pair / closest" → **two pointers** or **binary search**.
- Contiguous subarray / substring window → **sliding window**.
- "Top / Kth / median / merge K" → **heap**.
- Prefix relationships / autocomplete → **trie** or **prefix sums**.
- Shortest path on unweighted graph / level order → **BFS**. Connectivity / all paths / cycle → **DFS**.
- Dependencies / ordering → **topological sort (Kahn's)**.
- Overlap / scheduling → **sort + sweep**.
- "Number of ways / min cost / can we reach" → **DP** (define state + transition out loud).
- LRU / O(1) get+put → **hash map + doubly linked list**.
- O(1) min alongside stack ops → **auxiliary min stack**.

---

## Frequency by Topic (Amazon SDE/MLE) — budget your prep here
- **Graphs / Trees — 46%** (highest priority)
- **Arrays / Strings — 38%**
- **Linked lists — 10%**
- Search/Sort — 2% · Stacks/Queues — 2% · Hash tables — 2%

> Trees + Graphs + Arrays/Strings ≈ **84%** of questions. See `Question_Bank.md`.

---

## MLE-Flavored Coding Twists (be ready to write clean numpy / pure Python)
Amazon sometimes blends ML into DSA. Know these cold — they're fair game:
- **NMS (non-max suppression)** and **IoU** for bounding boxes.
- **K-means assignment step** (assign points to nearest centroid).
- **Softmax** (with the max-subtraction numerical-stability trick).
- **Precision / recall / F1** calculation from TP/FP/FN.
- **Streaming top-k frequent items** (heap + hash map).
- **Sharding scheme** to distribute a dataset across workers (hash partitioning, balance).

These let you show ML depth *inside* a coding round — narrate the ML reasoning, not just the loops.

---

## Talk-Out-Loud Script (the words to actually say)
- *"Let me restate the problem to make sure I understand…"*
- *"Before I code, a few clarifying questions: what's the range of n? Can the input be empty? Are duplicates allowed?"*
- *"The brute force is O(n²) — for each element I scan the rest. I think I can do better with a hash map for O(n). Let me confirm that direction sounds good."*
- *"I'll name this `seen` because it tracks values I've already visited."*
- *"Let me dry-run with `[2,7,11,15], target 9`… index 0, complement 7 not in map, store it… index 1, complement 2 is in map, return [0,1]. Good."*
- *"Edge cases: empty array returns []; a single element can't form a pair. Both handled."*
- *"Final complexity: O(n) time, O(n) space for the map. I could sort and two-pointer for O(1) extra space but that's O(n log n) time and loses original indices — so the hash map is the better trade-off here."*

---

## Day-Of Tactics
- Narrate everything; clarify before solving; test with edge cases unprompted; always state complexity.
- Use the shared editor cleanly — real, runnable code.
- If stuck, say your current hypothesis out loud and reason toward it — *thinking* is being graded.
- They may weave in an LP question ("tell me about a time you simplified a complex system"). Have a one-liner ready and pivot back to code.
- Ask for a moment to think if you need it — that's allowed and better than rambling.
