# ARRAYS & STRINGS KNOWLEDGE BASE — FILE 7
## Parts 25–30: Common Mistakes, Interview Thinking Process, Practice Roadmap, Cheat Sheets, Master Complexity Table, Final Goal

---

# PART 25 — COMMON MISTAKES

| Mistake | Why it happens | Fix |
|---|---|---|
| Off-by-one errors | Confusing closed `[l,r]` vs half-open `[l,r)` interval conventions in loops/binary search | Pick one convention per function and stay consistent; File 3 Part 10 shows both templates explicitly |
| Overflow | Summing large values into `int` (max ~2.1×10^9) | Use `long` for sums/products that could exceed int range; Java does **not** auto-promote silently the way you might expect — `int * int` overflows even when assigned to a `long` unless one operand is cast first |
| Window update order | Updating window state (add/remove) in the wrong order relative to checking window validity | Follow the exact template structure from Part 7 — expand first, then check/shrink, don't interleave arbitrarily |
| Duplicate handling | Forgetting to skip duplicate values in 3Sum/4Sum-style problems, producing duplicate result sets | Sort first, explicitly skip `nums[i]==nums[i-1]` at each recursion/loop level |
| Wrong binary search boundaries | Mixing `hi = arr.length` with `hi = arr.length - 1` conventions | Memorize: closed interval → `hi = length-1`, `while(lo<=hi)`; half-open → `hi = length`, `while(lo<hi)` |
| Incorrect hashing | Forgetting to override `hashCode()` when overriding `equals()` on custom objects used as HashMap keys | Always override both together, or use built-in types/records |
| Uninitialized frequency arrays | Reusing a `int[26]` frequency array across multiple test cases without resetting | `Arrays.fill(freq, 0)` between independent uses, or allocate fresh inside the relevant scope |
| String indexing | Using `substring()` inside a hot loop (each call allocates a new String) instead of `charAt()` | Index with `charAt(i)` when only comparing characters, not extracting logical substrings |
| `==` vs `.equals()` for Strings | `==` compares references, not content, for String objects | Always use `.equals()` (or `.equalsIgnoreCase()`) for content comparison |
| Mutating input during iteration | Modifying an `ArrayList` while iterating with a for-each loop throws `ConcurrentModificationException` | Use an explicit index-based loop, an `Iterator.remove()`, or collect changes and apply after the loop |
| Autoboxing overhead in hot loops | Using `Integer`/`ArrayList<Integer>` for heavy numeric loops (Dijkstra-style PQ, DP tables) | Prefer primitive arrays (`int[]`, `long[][]`) where performance matters |
| Shallow copy of 2D arrays | `int[][] copy = arr.clone()` only copies the outer array (row references), not the inner rows | Deep copy each row individually: `for(i) copy[i] = arr[i].clone();`, or use `Arrays.stream(arr).map(int[]::clone).toArray(int[][]::new)` |
| Integer division truncation | `a / b` truncates toward zero for negative operands in unexpected ways, breaking "ceiling division" logic | Use `(a + b - 1) / b` for ceiling division only when `a, b > 0`; otherwise use `Math.ceilDiv` (Java 18+) or explicit sign handling |

---

# PART 26 — INTERVIEW THINKING PROCESS

For **every** array/string problem, work through these 7 steps out loud:

## 1. What clues identify the pattern?
Scan for: is the array sorted? is a substring/subarray (contiguous) mentioned vs subsequence (non-contiguous)? are there range queries? is there a fixed/bounded alphabet? Cross-reference Part 19's decision tree.

## 2. Why not another approach?
*"I won't brute-force all subarrays here (O(n²)) because sliding window exploits the fact that the window's validity changes monotonically as we expand/shrink."* Show you considered and ruled out alternatives.

## 3. Complexity analysis
State time AND space before coding: *"With n up to 10^5, an O(n log n) sort-based approach is safe; O(n²) would likely TLE."*

## 4. Dry run
Trace a small example (5-8 elements) by hand, including edge cases like empty input or all-identical elements.

## 5. Edge cases
- Empty array / empty string.
- Single element.
- All elements identical.
- Already sorted / reverse sorted.
- Negative numbers (if the problem allows them and your solution assumed non-negative).
- Integer overflow boundary values.
- Unicode/multi-byte characters (if string problem doesn't restrict to ASCII).

## 6. Interview follow-up questions
- "What if the array is too large to fit in memory?" (→ streaming, external sort, reservoir sampling)
- "What if there are frequent updates between queries?" (→ Fenwick Tree/Segment Tree instead of static prefix sum)
- "Can you do this in O(1) extra space?" (→ in-place tricks: Part 20)
- "What if the input arrives as a stream, one element at a time?" (→ online algorithms: running heaps, two-heap median)
- "How would this change if we needed the actual elements, not just the count?" (→ often requires switching from a counting approach to a reconstruction approach)

## 7. Possible optimizations
Can a HashMap turn an O(n²) nested loop into O(n)? Can sorting enable two pointers or binary search? Can a monotonic stack/deque eliminate redundant re-computation? Can prefix sums avoid recomputing range sums from scratch?

---

# PART 27 — PRACTICE ROADMAP

## Beginner (Weeks 1–2)
- Array/String fundamentals in Java: primitive arrays vs ArrayList, String immutability, StringBuilder.
- Basic traversal, reversal, frequency counting.
- **Problems**: Two Sum, Valid Anagram, Contains Duplicate, Reverse String, Valid Palindrome.

## Intermediate (Weeks 3–5)
- Two pointers (both directions), sliding window (fixed and variable), prefix sum.
- Binary search (classic + lower/upper bound).
- **Problems**: 3Sum, Container With Most Water, Longest Substring Without Repeating Characters, Product of Array Except Self, Search in Rotated Sorted Array, Merge Intervals.

## Advanced (Weeks 6–8)
- Monotonic stack, heap patterns, greedy, interval sweep.
- String matching (KMP, Rabin-Karp), string DP (LCS, Edit Distance).
- **Problems**: Daily Temperatures, Largest Rectangle in Histogram, Top K Frequent Elements, Merge K Sorted Lists, Minimum Window Substring, Word Break, Edit Distance.

## Expert (Weeks 9–11)
- Binary search on answer, Manacher's algorithm, Trie-based problems, bitmask tricks.
- Combine multiple patterns in a single problem (e.g., sort + greedy + two pointers together).
- **Problems**: Koko Eating Bananas, Split Array Largest Sum, Word Search II (Trie + backtracking), Longest Palindromic Substring (Manacher's for O(n) bonus), Sliding Window Maximum.

## Master (Ongoing)
- Timed contests on Codeforces/AtCoder (Div 2 A-C are mostly array/string pattern recognition speed drills).
- CSES Introductory Problems + Sorting and Searching sections — dense, well-curated array/string practice.
- Re-derive every template from memory under time pressure; explain the "why" for each, not just the "how."

## Resource Map
| Resource | Use For |
|---|---|
| Blind 75 | Core interview coverage baseline |
| NeetCode 150 | Broader coverage with explicit pattern grouping |
| Top Interview 150 | FAANG-weighted practice |
| CSES Problem Set — Introductory + Sorting and Searching | Clean, well-tested array/string-only practice, ~30-40 problems |
| Codeforces (tags: `two pointers`, `binary search`, `greedy`, `strings`, `hashing`) | Timed competitive practice |
| LeetCode Explore Cards (Arrays 101, Strings, Two Pointers) | Structured guided learning with built-in problem sequencing |

---

# PART 28 — CHEAT SHEETS

## Arrays Cheat Sheet
- **Contiguous subarray** → sliding window (fixed size = fixed window; "longest satisfying X" = max window; "shortest satisfying X" = min window).
- **Sorted array** → two pointers (pair/triplet sum) or binary search (find value/boundary).
- **Range sum queries, static array** → prefix sum.
- **Range updates, then read once** → difference array.
- **Next/previous greater/smaller** → monotonic stack.
- **Top-K / merge-K / running median** → heap.
- **Overlapping intervals** → sort by start or end, then sweep.
- **In-place O(1) space needed** → cyclic sort, Dutch flag, reversal trick, two-pointer partitioning.
- **Minimize max / maximize min** → binary search on answer.

## Strings Cheat Sheet
- **Building a string in a loop** → always `StringBuilder`, never `+=`.
- **Content comparison** → always `.equals()`, never `==`.
- **Anagram / permutation check** → frequency array (26-size) or sorted-signature.
- **Palindrome** → two pointers (check) or expand-around-center (find longest); Manacher's only if O(n) is explicitly required.
- **Find all occurrences of a pattern** → KMP (guaranteed O(n+m)) or Rabin-Karp (good for multi-pattern).
- **Prefix-based lookup** → Trie.
- **Two strings, longest common / edit distance** → 2D DP.
- **Bounded alphabet (lowercase/ASCII)** → `int[26]`/`int[128]` beats HashMap.

## Pattern Recognition Guide
See Part 19's full decision tree and keyword table — the fastest lookup during practice/interviews.

## Complexity Table
See Part 29 below.

## Interview Revision Notes
1. State the pattern you recognize and why, out loud, before coding.
2. State time/space complexity before coding.
3. Write the brute force mentally (or briefly) even if you won't code it, to have a fallback and a complexity baseline to improve on.
4. Dry run on a small example after coding, not just before.
5. Explicitly call out edge cases you're handling (empty input, single element, duplicates).
6. If stuck, fall back to brute force and optimize incrementally out loud — partial credit for reasoning beats silence.

---

# PART 29 — MASTER COMPLEXITY TABLE

| Technique | Time | Space | When to Use | When NOT to Use |
|---|---|---|---|---|
| Brute Force | O(n²) or O(n³) | O(1) | Tiny n, or as a correctness baseline | n > ~10^4 typically too slow |
| Two Pointers | O(n) (after O(n log n) sort if needed) | O(1) | Sorted array, pair/triplet sum, in-place ops | Original order/indices must be preserved and can't be recovered post-sort |
| Sliding Window | O(n) | O(k) or O(alphabet size) | Contiguous subarray/substring with size/property constraint | Non-contiguous (subsequence) requirements |
| Hashing (HashMap/Set) | O(n) avg | O(n) | Fast lookup, frequency, unsorted pair-sum with indices needed | Extremely tight memory constraints, or need sorted-order iteration |
| Frequency Array | O(n) | O(alphabet size) | Bounded, small alphabet | Unbounded/large key space |
| Prefix Sum | O(n) build, O(1) query | O(n) | Many range-sum queries, static array | Frequent updates between queries |
| Difference Array | O(1) update, O(n) rebuild | O(n) | Many range updates, single final read | Need to query mid-update repeatedly (use Fenwick/Segment Tree instead) |
| Binary Search | O(log n) | O(1) | Sorted array, monotonic predicate | Unsorted data without a monotonic property |
| Binary Search on Answer | O(n log(range)) | O(1) | "Minimize max/maximize min" with checkable predicate | No clear monotonic feasibility function |
| Sorting | O(n log n) | O(n) or O(log n) | Enables greedy/two-pointer/binary search | Original order must be preserved without extra bookkeeping |
| Monotonic Stack | O(n) | O(n) | Next/previous greater/smaller, histogram-style | No "next/previous extremum" structure in the problem |
| Heap | O(n log k) | O(k) | Top-K, merge-K, running median | Need full sorted order of all n elements repeatedly (just sort instead) |
| Greedy | O(n log n) (usually sort-dominated) | O(1)–O(n) | Provable local-to-global optimality | Can't prove greedy-choice property; likely needs DP |
| KMP / Z-algorithm | O(n+m) | O(m) or O(n+m) | Single pattern search, guaranteed linear | N/A — nearly always safe when applicable |
| Rabin-Karp | O(n+m) avg, O(n·m) worst | O(1) extra | Multiple pattern search via batch hashing | Adversarial inputs without double-hashing safeguards |
| Trie | O(L) per op (L = word length) | O(total characters stored) | Prefix search, autocomplete, word search | Few strings, no prefix-sharing benefit |
| String DP (LCS/Edit Distance) | O(n·m) | O(n·m), reducible to O(min(n,m)) | Two-string comparison problems | Single-string simple checks (use two pointers instead) |
| Bitmask Tricks | O(1) per operation | O(1) | Small fixed alphabet presence/absence, subsets with n ≤ ~20-25 | Large or unbounded sets |

---

# PART 30 — FINAL GOAL / HOW TO USE THIS KNOWLEDGE BASE

You now have 7 files covering all 30 parts, all code in Java:
1. **File 1**: Array Fundamentals, String Fundamentals, Complete Array Operations, Complete String Operations (Parts 1–4)
2. **File 2**: Pattern Recognition Overview, Two Pointers, Sliding Window (Parts 5–7)
3. **File 3**: Hashing, Prefix Sum, Binary Search, Sorting Patterns (Parts 8–11)
4. **File 4**: String Matching, Greedy Patterns, Monotonic Stack, Heap, Interval Patterns, Matrix Patterns (Parts 12–17)
5. **File 5**: String DP Intro, Interview Pattern Recognition Decision Tree, Array Tricks, String Tricks, Common Variations (Parts 18–22)
6. **File 6**: LeetCode Pattern Grouping, Complete Java Template Library (Parts 23–24)
7. **File 7** (this file): Common Mistakes, Interview Thinking Process, Practice Roadmap, Cheat Sheets, Master Complexity Table (Parts 25–30)

**Suggested study loop for each pattern**: read intuition + brute force + optimal reasoning (Files 1-5) → dry run by hand → implement from the Java template library (File 6) → solve 3-5 LeetCode problems from File 6's pattern grouping → re-derive the pattern's template from memory a week later without looking, and explain out loud why it works (Part 26's 7-step process).

Given you're already building a parallel Graph Algorithms knowledge base and juggling hackathons (Flipkart Grid, BAH 2026, ET AI Hackathon) alongside this — treat the two knowledge bases as complementary: Arrays/Strings patterns (two pointers, sliding window, prefix sum, monotonic stack) show up constantly *inside* graph problems too (e.g., grid BFS uses array bounds-checking, Dijkstra uses a heap, topological sort uses array-based in-degree counting) — mastering both together compounds faster than either alone.
