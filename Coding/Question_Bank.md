# Coding Question Bank: Amazon MLE

Curated to (a) match Amazon's most-asked list and (b) cover every pattern an interviewer can draw from.
**★ = highest-yield / frequently reported at Amazon.** Practice on the NeetCode Blind 75: https://neetcode.io/practice

> Prioritize by frequency: **Trees/Graphs (46%) → Arrays/Strings (38%) → Linked lists (10%) → the rest.**
> Run the **5-step method** (see `Coding_Template.md`) out loud on every single one. Re-do any you can't finish in ~25 min.

---

## 1. Trees & Graphs: HIGHEST PRIORITY (46%)

| Problem | Pattern | |
|---|---|---|
| Construct binary tree from preorder + inorder | Recursion + index map | ★ |
| Construct BST from preorder traversal | Bounds recursion / stack | ★ |
| Binary tree maximum path sum | Post-order DFS, global max | ★ |
| Serialize & deserialize binary tree | BFS/DFS encoding | ★ |
| Diameter of binary tree | DFS height, track max | |
| Left view / right view of tree | BFS level order, first/last per level | |
| Number of islands | Grid DFS/BFS / union-find | ★ |
| Count distinct islands | Grid DFS + shape normalization | |
| Course Schedule II (return ordering) | Topological sort (Kahn) | ★ |
| Reconstruct itinerary (JFK) | Eulerian path / Hierholzer | |
| Word Search II (words in a grid) | Trie + backtracking (reported verbatim at Amazon) | ★ |
| Clone graph | BFS/DFS + hash map | |
| Rotting oranges | Multi-source BFS | ★ |
| Word ladder | BFS shortest transformation | |
| Lowest common ancestor (BST) | Walk down using value bounds | |
| Lowest common ancestor (binary tree) | Post-order DFS | ★ |
| Validate BST | In-order / min-max bounds | |
| Level order / zigzag traversal | BFS with queue | |
| Binary tree right side / vertical order | BFS + column map | |

**Drill the primitives:** DFS (recursive + iterative w/ stack), BFS (queue), in/pre/post-order, topological sort (Kahn + DFS), union-find with path compression. **Know when BFS (shortest/level, unweighted) vs DFS (paths/connectivity/cycles).**

---

## 2. Arrays & Strings (38%)

| Problem | Pattern | |
|---|---|---|
| Two Sum | Hash map | ★ |
| 3Sum | Sort + two pointers | ★ |
| Best time to buy & sell stock (I) | One-pass min tracking | ★ |
| Longest substring without repeating chars | Sliding window + set/map | ★ |
| Product of array except self | Prefix/suffix products | ★ |
| Longest palindromic substring | Expand-around-center / DP | |
| Maximum subarray (Kadane) | Running sum / DP | ★ |
| Maximum product of two integers | Track two largest / two smallest | |
| Group anagrams | Hash by sorted key / char count | ★ |
| Valid anagram | Char-count compare | |
| Top K frequent elements | Hash + heap / bucket sort | ★ |
| Integer to English words | Simulation, chunk by thousands | |
| Valid Sudoku | Set checks per row/col/box | |
| All permutations of a string | Backtracking | |
| Trapping rain water | Two pointers / monotonic stack | ★ |
| Container with most water | Two pointers | |
| Merge intervals | Sort + sweep | ★ |
| Move zeroes / Sort colors | Two pointers / Dutch flag | |
| String to integer (atoi) | Careful parsing + overflow | |

---

## 3. Linked Lists (10%)

| Problem | Pattern | |
|---|---|---|
| Merge two sorted lists | Pointer merge | ★ |
| Merge K sorted lists | Min-heap / divide & conquer | ★ |
| Reverse linked list (I) | Iterative pointer flip | |
| Reverse nodes in K-group | Pointer surgery | |
| Copy list with random pointer | Hash map / interleave + split | |
| Detect cycle / find cycle start | Floyd's two pointers | |
| Remove Nth node from end | Two pointers (gap of n) | |
| Add two numbers (linked digits) | Simulation + carry | |
| Reorder list | Find mid + reverse + merge | |

---

## 4. Stacks / Queues / Heaps / Search-Sort

| Problem | Pattern | |
|---|---|---|
| Min stack (push/pop/top/min in O(1)) | Auxiliary stack | ★ |
| Valid parentheses | Stack matching | ★ |
| Daily temperatures / next greater element | Monotonic stack | |
| Largest rectangle in histogram | Monotonic stack | |
| Kth largest element in array | Heap / quickselect | ★ |
| Kth largest in a stream | Min-heap of size k | |
| Find median from data stream | Two heaps | ★ |
| Search in rotated sorted array | Modified binary search | ★ |
| Search a 2D matrix | Binary search / staircase | ★ |
| Find first/last position in sorted array | Binary search bounds | |
| Sliding window maximum | Monotonic deque | |
| Meeting rooms II (min rooms) | Heap / sweep line | ★ |
| Task scheduler | Greedy + heap / math | |

---

## 5. Backtracking / DP (lower frequency, still appears)

| Problem | Pattern | |
|---|---|---|
| Subsets / Combinations / Permutations | Backtracking templates | |
| Combination sum | Backtracking w/ pruning | |
| Word break | DP / memoized DFS | |
| Coin change (min coins) | 1-D DP | |
| Longest increasing subsequence | DP / patience sort | |
| Unique paths / Min path sum (grid) | 2-D DP | |
| Edit distance | 2-D DP | |
| House robber (I) | 1-D DP | |
| Decode ways | 1-D DP | |

---

## 6. OOP / Design-y Coding (LOW-LEVEL DESIGN, practice writing these as classes)

> These test **clean class design**, encapsulation, and the right data structures together. Narrate the API first (method signatures + complexity targets), then implement. Great place to show "Logical & Maintainable."

| Problem | Core data structures | Target complexity | |
|---|---|---|---|
| **LRU Cache** | Hash map + doubly linked list | O(1) get & put | ★ |
| **LFU Cache** | Hash maps + freq buckets (DLL per freq) | O(1) get & put | |
| **Min Stack** | Two stacks (or value+min pairs) | O(1) all ops | ★ |
| **Implement Trie (prefix tree)** | Nested dict / children array | O(L) insert/search | ★ |
| **Add & Search Word** (`.` wildcard) | Trie + DFS | O(L) avg | |
| **Design HashMap / HashSet** | Buckets + chaining | O(1) amortized | |
| **Design Tic-Tac-Toe** | Row/col/diag counters | O(1) per move | |
| **Design Hit Counter** | Queue / circular buffer | O(1) amortized | |
| **Design a Rate Limiter** | Token bucket / sliding window deque | O(1) | ★ |
| **Snapshot Array** | Per-index list of (snap_id, val) + binary search | O(log n) get | |
| **Insert/Delete/GetRandom O(1)** | Hash map + array (swap-to-end) | O(1) all | ★ |
| **Time-based Key-Value Store** | Hash map → sorted list + binary search | O(log n) get | |
| **Design Twitter (news feed)** | Hash maps + merge K heaps | merge K | |
| **Iterator classes** (BST iterator, flatten nested list) | Stack | O(1) amortized next | |
| **Tokenizer / Logger / Parking Lot / Elevator** | Enums + classes + state | discuss design | |

> The existing notebooks `Coding/oop.ipynb`, `Coding/dsa.ipynb`, `Coding/problems.ipynb` already have implementations; use them as your warm-up reference, but rewrite from scratch under time.

---

## 7. ML-Flavored Coding Twists (write clean numpy / pure Python, know these cold)

> **For THIS role (Generative AI for Amazon Customer Service), §7a below is the most likely flavor** — the role is text/embedding/LLM-centric, not vision. Keep §7b for a general ML round, but prioritize 7a.

### 7a. GenAI / LLM / text-flavored (most relevant to Amazon Customer Service)
| Task | What to implement | Notes |
|---|---|---|
| **Cosine similarity** | `dot(a,b) / (‖a‖·‖b‖)` between two embedding vectors | guard zero-norm; basis of semantic search |
| **Top-k nearest neighbors by cosine** | score a query vs N stored embeddings, return top-k | min-heap of size k → O(N log k); this *is* vector search / RAG retrieval |
| **Softmax + temperature** | `exp(x/T - max) / sum(...)` | T scales randomness; subtract max for numerical stability |
| **Top-k / top-p (nucleus) sampling** | filter logits to top-k or cumulative-p mass, renormalize, sample | the LLM decoding step — be ready to explain it |
| **BPE / simple tokenizer** | split text → tokens; (BPE) merge the most-frequent adjacent pair iteratively | or a word/char tokenizer + vocab map |
| **Levenshtein / edit distance** | 2-D DP | fuzzy matching, dedupe near-identical customer queries |
| **Token-bucket rate limiter** | refill tokens over elapsed time; allow/deny | API/LLM throttling (also OOP §6) |
| **LRU cache for responses/embeddings** | hash map + doubly linked list, O(1) | caching layer for an LLM service (also OOP §6) |
| **Streaming top-k frequent** | hash count + min-heap of size k | trending intents/topics from contact streams; O(n log k) |
| **Parse / merge structured (JSON-ish) output** | parse and validate LLM output fields, merge | post-processing an LLM response |
| **Precision / Recall / F1** | from TP, FP, FN | classifier/eval metric; guard divide-by-zero |

### 7b. Classic CV / ML-flavored (still fair game in a general ML round)
| Task | What to implement | Notes |
|---|---|---|
| **IoU** | intersection / union of two boxes | clamp negative overlap to 0 |
| **NMS** | sort by score, greedily keep, drop boxes with IoU > thresh | O(n²) naive; mention sorting |
| **K-means assignment step** | assign each point to nearest centroid (argmin dist) | then recompute centroids |
| **Sharding scheme** | hash(key) % num_workers; discuss balance/skew | consistent hashing for resize |

---

## Practice Schedule (from the 2-week plan)
- **Days 3–7:** 3–4 problems/day. Order: **Trees → Graphs → Arrays/Strings**. Strictly 5-step method, out loud, timed to 25 min. Cover the MLE twists.
- **Days 12–13:** full mock loops (1 coding + 1 behavioral each). Record yourself; cut filler, add complexity statements you forgot.

**Self-grade each problem:** Did I clarify? State brute force first? Justify the data structure? Test edge cases unprompted? State final complexity? If any "no," redo it.
