# Coding Patterns & Techniques

> **The meta-skill:** most interview problems are one of ~15 reusable patterns wearing a costume. Your job in **Step 1–2 (Clarify → Plan)** is to read the _trigger signals_ in the prompt and pick the matching tool. This file is your reflex table: **when** each pattern applies, **how/why** it works, a **brief example**, and **which Question Bank problems** drill it.
>
> Read the "Trigger signals" lines until they're automatic. In the interview, say the trigger out loud: _"The array is sorted and I need a pair — that's a two-pointer signal."_ That narration is graded (problem-solving).

---

## Quick reflex table (skim this first)

| If you see…                                                                 | Reach for…                                   |
| --------------------------------------------------------------------------- | -------------------------------------------- |
| Sorted array + find a pair/triplet / closest sum                            | **Two pointers (opposite ends)**             |
| Linked list cycle / find middle / nth from end                              | **Two pointers (fast & slow)**               |
| Contiguous subarray/substring + "longest/shortest/at most K"                | **Sliding window**                           |
| Many range-sum queries / "subarray sums to K"                               | **Prefix sums**                              |
| "Have I seen this before?" / count / dedupe / O(1) lookup                   | **Hash map / set**                           |
| Sorted (or rotated sorted) + find/threshold / "minimize the max"            | **Binary search (incl. on answer)**          |
| Shortest path in unweighted graph / level-by-level                          | **BFS**                                      |
| All paths / connectivity / explore fully / generate combinations            | **DFS / Backtracking**                       |
| Dependencies / ordering / prerequisites                                     | **Topological sort (Kahn)**                  |
| Grouping / "are these connected" / count components                         | **Union-Find (DSU)**                         |
| "Top K / Kth / median / merge K sorted"                                     | **Heap (priority queue)**                    |
| "Next greater/smaller element" / spans / histogram                          | **Monotonic stack**                          |
| Sliding-window **max/min**                                                  | **Monotonic deque**                          |
| "# of ways / min cost / can I reach / best score" + overlapping subproblems | **Dynamic programming**                      |
| Intervals / scheduling / overlaps                                           | **Sort + sweep / greedy**                    |
| Prefix/word lookups, autocomplete                                           | **Trie**                                     |
| Build tree from traversals / sort / "halve the problem"                     | **Divide & conquer / recursion + index map** |
| Numbers in range `[1..n]`, find missing/duplicate, O(1) space               | **Cyclic sort / index-as-hash**              |
| "Appears once/twice", sets without extra space                              | **Bit manipulation**                         |

---

## 1. Two Pointers — opposite ends

**Trigger:** array is **sorted** and you need a **pair/triplet** with a target sum (or min/max width).
**How/why:** put `left=0`, `right=n-1`. The sum changes **predictably** because the array is sorted — too small → `left += 1` (only way to increase), too big → `right -= 1` (only way to decrease). Each pointer moves inward at most n times → **O(n)** instead of O(n²) for the pair subproblem. This is _exactly_ why 3Sum sorts first, then runs two pointers inside the outer loop → O(n²) overall.
**Brief example — two-sum on a sorted array:**

```python
def two_sum_sorted(nums, target):           # nums is sorted
    l, r = 0, len(nums) - 1
    while l < r:
        s = nums[l] + nums[r]
        if s == target: return [l, r]
        if s < target: l += 1               # need bigger → move left up
        else:          r -= 1               # need smaller → move right down
    return []
```

**Also powers:** 3Sum, 3Sum closest, Container With Most Water, Trapping Rain Water, Valid Palindrome, Sort Colors (Dutch flag), Move Zeroes.
**Watch:** skip duplicates after a hit (as 3Sum does) to keep triplets unique.

## 2. Two Pointers — fast & slow (Floyd's)

**Trigger:** **linked list** cycle detection, find the middle, or nth-from-end; also "find the duplicate number" framed as a cycle.
**How/why:** slow moves 1 step, fast moves 2. If there's a cycle they meet; if not, fast hits the end. For the **middle**, when fast reaches the end slow is at the midpoint. For **nth-from-end**, advance one pointer n steps first, then move both until it hits the end.

```python
def has_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow, fast = slow.next, fast.next.next
        if slow is fast: return True
    return False
```

**Also powers:** find cycle start, middle of list, reorder list, palindrome linked list.

## 3. Sliding Window

**Trigger:** **contiguous** subarray/substring + "longest / shortest / at most K / exactly K / no repeats." Keyword: _contiguous_.
**How/why:** expand `right` to grow the window; when it violates the constraint, shrink from `left`. Each index enters and leaves the window once → **O(n)**. Use a hash map/counter for character/element frequencies inside the window.

```python
def longest_unique(s):                       # longest substring w/o repeating chars
    seen = {}                                # char -> last index
    l = best = 0
    for r, ch in enumerate(s):
        if ch in seen and seen[ch] >= l:
            l = seen[ch] + 1                 # jump left past the duplicate
        seen[ch] = r
        best = max(best, r - l + 1)
    return best
```

**Also powers:** min window substring, longest repeating char replacement, max sum subarray of size k, fruit into baskets.
**Fixed vs variable:** fixed-size window = slide by one each step; variable = grow/shrink on a condition.

## 4. Prefix Sums (and prefix products)

**Trigger:** many **range-sum** queries, "subarray sums to K," or "product of array except self."
**How/why:** precompute cumulative sums so any range sum is `pre[j] - pre[i]` in O(1). Combine with a **hash map of prefix counts** to find subarrays summing to K in one pass.

```python
def subarray_sum_equals_k(nums, k):
    count = 0; pre = 0
    seen = {0: 1}                            # prefix sum -> how many times seen
    for x in nums:
        pre += x
        count += seen.get(pre - k, 0)        # a prior prefix makes a window == k
        seen[pre] = seen.get(pre, 0) + 1
    return count
```

**Also powers:** Product of Array Except Self (prefix×suffix), range sum query, pivot index.

## 5. Hash Map / Set — O(1) lookup

**Trigger:** "have I seen this?", counting frequencies, dedupe, complement lookup, grouping.
**How/why:** trade space for time — O(1) average insert/lookup turns an O(n²) scan into O(n). The canonical **Two Sum** stores each value's complement.

```python
def two_sum(nums, target):
    seen = {}                                # value -> index
    for i, x in enumerate(nums):
        if target - x in seen: return [seen[target - x], i]
        seen[x] = i
```

**Also powers:** Group Anagrams (key by sorted/char-count), Top K Frequent, Valid Anagram, Longest Consecutive Sequence, LRU/LFU (map + DLL).

## 6. Binary Search (including "search on the answer")

**Trigger:** **sorted** data + find/threshold; OR a **monotonic** "is X feasible?" question → "minimize the max / maximize the min." Also rotated sorted arrays.
**How/why:** halve the search space each step → **O(log n)**. Even when the array isn't literally sorted, if `feasible(x)` is monotonic (false…false,true…true) you can binary-search the _answer value_.

```python
def search_rotated(nums, target):
    l, r = 0, len(nums) - 1
    while l <= r:
        m = (l + r) // 2
        if nums[m] == target: return m
        if nums[l] <= nums[m]:               # left half sorted
            if nums[l] <= target < nums[m]: r = m - 1
            else: l = m + 1
        else:                                # right half sorted
            if nums[m] < target <= nums[r]: l = m + 1
            else: r = m - 1
    return -1
```

**Also powers:** first/last position, search 2D matrix, Koko eating bananas, split array largest sum, median of two sorted arrays.
**Watch:** pick `l<=r` vs `l<r` and `m±1` carefully to avoid infinite loops.

## 7. BFS (Breadth-First Search)

**Trigger:** **shortest path / fewest steps** in an **unweighted** graph or grid; level-by-level processing.
**How/why:** a queue explores in waves; the first time you reach a node is via the shortest path (unweighted). **Multi-source BFS** seeds the queue with all sources at once (e.g., rotting oranges).

```python
from collections import deque
def num_islands_bfs(grid):
    if not grid: return 0
    R, C, count = len(grid), len(grid[0]), 0
    for r in range(R):
        for c in range(C):
            if grid[r][c] == '1':
                count += 1
                q = deque([(r, c)]); grid[r][c] = '0'
                while q:
                    i, j = q.popleft()
                    for di, dj in ((1,0),(-1,0),(0,1),(0,-1)):
                        ni, nj = i+di, j+dj
                        if 0<=ni<R and 0<=nj<C and grid[ni][nj]=='1':
                            grid[ni][nj] = '0'; q.append((ni, nj))
    return count
```

**Also powers:** word ladder, rotting oranges, level-order traversal, shortest path in a maze.
**BFS vs DFS:** BFS for **shortest/level**; DFS for **paths/connectivity/cycles**.

## 8. DFS / Backtracking

**Trigger:** explore _all_ possibilities — permutations, combinations, subsets, partitions, word search, all paths; or fully explore a region/tree.
**How/why:** recurse, **choose → explore → un-choose** (backtrack). Prune branches that can't lead to a solution. Exponential in the worst case but pruning + structure make it tractable.

```python
def subsets(nums):
    res = []
    def dfs(start, path):
        res.append(path[:])                  # record every node, not just leaves
        for i in range(start, len(nums)):
            path.append(nums[i])             # choose
            dfs(i + 1, path)                 # explore
            path.pop()                       # un-choose (backtrack)
    dfs(0, [])
    return res
```

**Also powers:** permutations, combination sum, N-Queens, Word Search (grid backtracking), generate parentheses, palindrome partitioning, tree path sums.

## 9. Topological Sort (Kahn's / BFS on a DAG)

**Trigger:** **ordering with dependencies / prerequisites** ("finish course X before Y"); detect cycles in a directed graph.
**How/why:** compute in-degrees; repeatedly remove a node with in-degree 0, decrement its neighbors. If you can't remove all nodes, there's a **cycle**.

```python
from collections import deque, defaultdict
def course_order(n, prereqs):
    g = defaultdict(list); indeg = [0]*n
    for a, b in prereqs:                      # b -> a (take b before a)
        g[b].append(a); indeg[a] += 1
    q = deque(i for i in range(n) if indeg[i] == 0)
    order = []
    while q:
        u = q.popleft(); order.append(u)
        for v in g[u]:
            indeg[v] -= 1
            if indeg[v] == 0: q.append(v)
    return order if len(order) == n else []   # empty => cycle
```

**Also powers:** Course Schedule I/II, alien dictionary, build order.

## 10. Union-Find (Disjoint Set Union)

**Trigger:** "are these two connected?", count connected components, detect a cycle in an **undirected** graph, dynamic grouping.
**How/why:** each element points to a parent; `find` returns the root (with **path compression**); `union` links roots (by rank). Near-**O(1)** amortized per op (α(n)).

```python
def count_components(n, edges):
    parent = list(range(n))
    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]    # path compression
            x = parent[x]
        return x
    comp = n
    for a, b in edges:
        ra, rb = find(a), find(b)
        if ra != rb: parent[ra] = rb; comp -= 1
    return comp
```

**Also powers:** Number of Islands (alt), Redundant Connection, Accounts Merge, Kruskal's MST.

## 11. Heap / Priority Queue

**Trigger:** "**top K / Kth largest/smallest**, merge K sorted, running median, schedule by priority." Keyword: _K_ or _streaming_.
**How/why:** a binary heap gives O(log n) push/pop and O(1) peek. For **Kth largest**, keep a **min-heap of size k** (root = the answer). For **median of a stream**, use two heaps (max-heap for low half, min-heap for high half).

```python
import heapq
def k_largest(nums, k):
    h = []
    for x in nums:
        heapq.heappush(h, x)
        if len(h) > k: heapq.heappop(h)      # keep only k largest; root is kth largest
    return h[0]
```

**Also powers:** Merge K Sorted Lists, Top K Frequent, Task Scheduler, Meeting Rooms II, Find Median from Data Stream, Dijkstra.

## 12. Monotonic Stack

**Trigger:** "**next greater / next smaller** element," stock spans, daily temperatures, largest rectangle in histogram, trapping rain water.
**How/why:** keep a stack whose values stay sorted (increasing or decreasing). When the new element breaks the order, pop — each pop resolves an answer. Each element pushed/popped once → **O(n)**.

```python
def daily_temperatures(temps):
    res = [0]*len(temps)
    stack = []                               # indices, temps decreasing
    for i, t in enumerate(temps):
        while stack and temps[stack[-1]] < t:
            j = stack.pop(); res[j] = i - j   # days until warmer
        stack.append(i)
    return res
```

**Also powers:** Next Greater Element, Largest Rectangle in Histogram, Trapping Rain Water (stack variant), remove K digits.

## 13. Monotonic Deque

**Trigger:** **sliding-window maximum/minimum**.
**How/why:** a double-ended queue holds candidate indices in decreasing value order; the front is always the window max. Drop indices that fall out of the window or are dominated → amortized **O(n)**.

```python
from collections import deque
def max_sliding_window(nums, k):
    dq, res = deque(), []                     # dq holds indices, values decreasing
    for i, x in enumerate(nums):
        while dq and nums[dq[-1]] <= x: dq.pop()
        dq.append(i)
        if dq[0] == i - k: dq.popleft()       # drop index out of window
        if i >= k - 1: res.append(nums[dq[0]])
    return res
```

## 14. Dynamic Programming

**Trigger:** "**number of ways / min cost / max value / can I reach / longest…**" AND the problem has **overlapping subproblems** + **optimal substructure**. If brute-force recursion recomputes the same state, it's DP.
**How/why:** define a **state**, a **transition** (recurrence), and **base cases**. Memoize (top-down) or build a table (bottom-up). Say the recurrence out loud — that's the graded insight.

```python
def coin_change(coins, amount):              # min coins to make amount
    INF = amount + 1
    dp = [0] + [INF]*amount                   # dp[a] = min coins for amount a
    for a in range(1, amount + 1):
        for c in coins:
            if c <= a: dp[a] = min(dp[a], dp[a-c] + 1)
    return dp[amount] if dp[amount] != INF else -1
```

**Families:** 1-D (house robber, climbing stairs, coin change), 2-D grid (unique paths, min path sum, edit distance), subsequence (LIS, LCS), knapsack (subset sum, partition), interval (burst balloons), DP on trees.
**Tip:** start with the recursive brute force + memo, then convert to a table if asked for O(1) space tricks.

## 15. Greedy + Sort / Sweep Line

**Trigger:** **intervals, scheduling, overlaps**, "minimum number of X to cover Y," activity selection.
**How/why:** sort by a key (start or end time), then make the locally optimal choice and prove it's globally optimal. Sweep-line processes endpoints in order.

```python
def merge_intervals(intervals):
    intervals.sort(key=lambda x: x[0])        # sort by start
    merged = []
    for s, e in intervals:
        if merged and s <= merged[-1][1]:
            merged[-1][1] = max(merged[-1][1], e)  # overlap → extend
        else:
            merged.append([s, e])
    return merged
```

**Also powers:** Meeting Rooms I/II, Non-overlapping Intervals, Insert Interval, Jump Game, Gas Station.
**Caution:** greedy needs justification — if you can't argue it's optimal, it may need DP instead.

## 16. Trie (Prefix Tree)

**Trigger:** **prefix** queries, autocomplete, dictionary/word lookups, "search words in a grid," word-break dictionaries.
**How/why:** a tree of characters; each path is a prefix. Insert/search in **O(L)** (L = word length), independent of how many words are stored. Pair with DFS/backtracking for grid word search.

```python
class Trie:
    def __init__(self): self.root = {}
    def insert(self, word):
        node = self.root
        for ch in word: node = node.setdefault(ch, {})
        node['$'] = True                       # end-of-word marker
    def search(self, word):
        node = self.root
        for ch in word:
            if ch not in node: return False
            node = node[ch]
        return '$' in node
```

**Also powers:** Word Search II (Trie + backtracking — reported at Amazon), Add & Search Word, Replace Words, autocomplete.

## 17. Divide & Conquer / Recursion + Index Map

**Trigger:** "build tree from preorder+inorder," sort, "halve the problem," merge results.
**How/why:** split into independent subproblems, solve recursively, combine. For tree construction, a **hash map from value→inorder index** makes the split O(1), giving O(n) overall.

```python
def build_tree(preorder, inorder):
    idx = {v: i for i, v in enumerate(inorder)}   # value -> inorder index
    self_pre = iter(preorder)
    def build(lo, hi):
        if lo > hi: return None
        root_val = next(self_pre)
        root = TreeNode(root_val)
        mid = idx[root_val]
        root.left = build(lo, mid - 1)
        root.right = build(mid + 1, hi)
        return root
    return build(0, len(inorder) - 1)
```

**Also powers:** Merge Sort, Quick Sort, Construct BST from preorder, Kth largest via quickselect, merge K lists (D&C variant).

## 18. Cyclic Sort / Index-as-Hash

**Trigger:** numbers in a known range `[1..n]` or `[0..n]`; find the **missing / duplicate** number in **O(1) extra space**.
**How/why:** the value tells you where it belongs — place each number at index `value-1`, or use the sign at `index value` as a "seen" flag. Linear time, no extra structure.

```python
def find_disappeared(nums):                   # numbers 1..n, find missing
    for x in nums:
        i = abs(x) - 1
        nums[i] = -abs(nums[i])               # mark index 'seen' by negating
    return [i + 1 for i, v in enumerate(nums) if v > 0]
```

**Also powers:** Find All Duplicates, Find the Missing Number, First Missing Positive, Set Mismatch.

## 19. Bit Manipulation

**Trigger:** "appears once while others twice," subsets via bitmask, toggling flags, no-extra-space uniqueness.
**How/why:** XOR cancels pairs (`a^a=0`), so XOR-ing everything leaves the unique value. Bitmasks enumerate subsets compactly.

```python
def single_number(nums):
    x = 0
    for n in nums: x ^= n                      # pairs cancel, lone number remains
    return x
```

**Also powers:** Single Number I/II/III, Counting Bits, Subsets via bitmask, Missing Number (XOR variant).

## 20. Quickselect (bonus — average O(n) Kth element)

**Trigger:** "Kth largest/smallest" when you want **average O(n)** instead of heap's O(n log k) and don't need the order.
**How/why:** partition like quicksort, but recurse into only the side containing the Kth position. Average O(n), worst O(n²) (mitigate with random pivot).
**Also powers:** Kth Largest Element, Top K Frequent (partition variant), median.

---

## How to use this in the interview (the workflow)

1. **Read the prompt for trigger words** — _sorted, contiguous, shortest, dependencies, top-K, next-greater, # of ways…_
2. **Name the pattern out loud** and why it fits. ("Contiguous + longest-with-constraint → sliding window.")
3. **State complexity the pattern buys you** before coding ("this takes the pair search from O(n²) to O(n)").
4. If two patterns fit, mention both and pick with justification — that's the senior signal.

> Cross-reference: the **reflex table** in `Coding_Template.md` (pattern → tool) and the per-problem labels in `Question_Bank.md` (each problem lists its pattern). Drill by pattern: do 3 problems of one technique back-to-back until the trigger is automatic.
