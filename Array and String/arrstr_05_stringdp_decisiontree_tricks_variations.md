# ARRAYS & STRINGS KNOWLEDGE BASE — FILE 5
## Parts 18–22: String DP Intro, Interview Pattern Recognition Decision Tree, Array Tricks, String Tricks, Common Variations

---

# PART 18 — STRING DP INTRO

## Longest Common Prefix
```java
String longestCommonPrefix(String[] strs) {
    if (strs.length == 0) return "";
    String prefix = strs[0];
    for (int i = 1; i < strs.length; i++) {
        while (!strs[i].startsWith(prefix)) {
            prefix = prefix.substring(0, prefix.length() - 1);
            if (prefix.isEmpty()) return "";
        }
    }
    return prefix;
}
```
Not technically DP, but grouped here as the simplest "compare across strings" warm-up before true string DP.

## Longest Common Subsequence (LCS)
```java
int longestCommonSubsequence(String text1, String text2) {
    int m = text1.length(), n = text2.length();
    int[][] dp = new int[m + 1][n + 1];
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (text1.charAt(i - 1) == text2.charAt(j - 1)) dp[i][j] = dp[i - 1][j - 1] + 1;
            else dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
        }
    }
    return dp[m][n];
}
```
**State definition**: `dp[i][j]` = LCS length of `text1[0..i-1]` and `text2[0..j-1]`. **Transition intuition**: if the current characters match, extend the diagonal's LCS by 1; otherwise, take the best of "drop last char of text1" or "drop last char of text2."

## Longest Palindromic Substring
```java
// Expand around center — O(n^2) time, O(1) space, simplest correct approach
String longestPalindrome(String s) {
    int start = 0, maxLen = 0;
    for (int center = 0; center < s.length(); center++) {
        int len1 = expandAroundCenter(s, center, center);     // odd length
        int len2 = expandAroundCenter(s, center, center + 1); // even length
        int len = Math.max(len1, len2);
        if (len > maxLen) { maxLen = len; start = center - (len - 1) / 2; }
    }
    return s.substring(start, start + maxLen);
}
int expandAroundCenter(String s, int l, int r) {
    while (l >= 0 && r < s.length() && s.charAt(l) == s.charAt(r)) { l--; r++; }
    return r - l - 1;
}
```
**Why 2n-1 centers**: a palindrome of odd length has a single center character; even length has a center *between* two characters — checking both `(center,center)` and `(center,center+1)` for every index covers all cases. For O(n) instead of O(n²), see Manacher's Algorithm in Part 21.

## Edit Distance (Levenshtein Distance)
```java
int minDistance(String word1, String word2) {
    int m = word1.length(), n = word2.length();
    int[][] dp = new int[m + 1][n + 1];
    for (int i = 0; i <= m; i++) dp[i][0] = i; // delete all of word1
    for (int j = 0; j <= n; j++) dp[0][j] = j; // insert all of word2
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (word1.charAt(i - 1) == word2.charAt(j - 1)) dp[i][j] = dp[i - 1][j - 1];
            else dp[i][j] = 1 + Math.min(dp[i - 1][j - 1], Math.min(dp[i - 1][j], dp[i][j - 1]));
            // dp[i-1][j-1]=replace, dp[i-1][j]=delete, dp[i][j-1]=insert
        }
    }
    return dp[m][n];
}
```

## Word Break
```java
boolean wordBreak(String s, List<String> wordDict) {
    Set<String> dict = new HashSet<>(wordDict);
    boolean[] dp = new boolean[s.length() + 1];
    dp[0] = true; // empty prefix is trivially "breakable"
    for (int i = 1; i <= s.length(); i++) {
        for (int j = 0; j < i; j++) {
            if (dp[j] && dict.contains(s.substring(j, i))) { dp[i] = true; break; }
        }
    }
    return dp[s.length()];
}
```
**State definition**: `dp[i]` = can `s[0..i-1]` be fully segmented into dictionary words. Check every split point `j` where the prefix up to `j` is already breakable and `s[j..i-1]` is itself a dictionary word.

## String DP Recognition
"Longest common X between two strings," "minimum operations to transform one string into another," "can this string be segmented/formed from smaller pieces" → 2D or 1D DP over string indices, almost always with a clean `dp[i][j]` or `dp[i]` state definition tied directly to prefix lengths.

---

# PART 19 — INTERVIEW PATTERN RECOGNITION (Decision Tree)

## Master Decision Tree

```
Does the problem involve a MATRIX/GRID?
├── YES → traversal/rotation/spiral? → Matrix Pattern (Part 17)
│         path/region counting? → likely Graph BFS/DFS (see Graph KB)
│
└── NO → Is it about a CONTIGUOUS subarray/substring?
    ├── YES → fixed size k? → Fixed Sliding Window
    │         "longest ... satisfying condition"? → Maximum Variable Window
    │         "shortest/minimum ... containing/satisfying condition"? → Minimum Variable Window
    │         range SUM specifically, many queries? → Prefix Sum
    │         range Sum with many UPDATES? → Difference Array / Fenwick Tree
    │
    └── NO → Is it about NON-contiguous subsequence?
        ├── YES → "longest common / longest increasing / edit distance"? → DP
        │
        └── NO → Is the array SORTED (or can be sorted without losing needed info)?
            ├── YES → "pair/triplet sum"? → Two Pointers
            │         "find a value / boundary"? → Binary Search
            │         "minimize max / maximize min"? → Binary Search on Answer
            │
            └── NO → Does it need FAST LOOKUP / existence / frequency?
                ├── YES → Hashing (HashMap/HashSet/frequency array)
                │
                └── NO → "next/previous greater/smaller element"? → Monotonic Stack
                          "k largest/smallest/merge k/running median"? → Heap
                          "overlapping ranges/scheduling"? → Interval Pattern (sort + sweep)
                          "local optimal choice, provably global optimal"? → Greedy
                          "XOR / subsets / single number"? → Bit Manipulation
                          "prefix-based word lookup"? → Trie
                          "find all occurrences of a pattern string"? → KMP/Z/Rabin-Karp
```

## Keyword → Pattern Cheat Table

| Question says... | → Use |
|---|---|
| "pair" / "two numbers that sum to" | Two Pointers (sorted) or HashMap (unsorted, need indices) |
| "longest substring / longest subarray (contiguous)" | Sliding Window (maximum variant) |
| "shortest / minimum window" | Sliding Window (minimum variant) |
| "frequency" / "count of characters" | HashMap or frequency array |
| "range sum" (static array, many queries) | Prefix Sum |
| "range update" (many updates) | Difference Array / Fenwick Tree |
| "sorted array" + search | Binary Search |
| "minimize the maximum" / "maximize the minimum" | Binary Search on Answer |
| "k largest / k smallest / kth" | Heap |
| "merge k sorted" | Heap |
| "running median" | Two-Heap technique |
| "rotation" (array) | Reversal trick / cyclic indexing |
| "anagram" | Frequency count / sorted-string signature |
| "duplicates" | HashSet |
| "next greater/smaller element" | Monotonic Stack |
| "overlapping intervals" / "meeting rooms" | Sort + sweep (Interval Pattern) |
| "subsequence" (non-contiguous) + "longest/common" | Dynamic Programming |
| "minimum edits/operations to transform string" | Edit Distance DP |
| "can be segmented into words" | Word Break DP |
| "find all occurrences of pattern" | KMP / Z-algorithm / Rabin-Karp |
| "palindrome" | Two-pointer check, expand-around-center, or Manacher's (if O(n) required) |
| "XOR gives unique element" | Bit Manipulation |
| "prefix search / autocomplete" | Trie |
| "small alphabet, fixed size" | Frequency array over HashMap |
| "n ≤ 20, subsets" | Bitmask DP |

**How to use this under interview pressure**: circle every quantitative constraint (`n ≤ ?`) and every qualitative keyword first, cross-reference both tables, and state your chosen approach + Big-O out loud *before* coding — this is exactly what separates a strong signal from a weak one in FAANG loops.

---

# PART 20 — COMPLETE ARRAY TRICKS

| Trick | What it does | Example use |
|---|---|---|
| In-place modification | Reuse the input array itself as extra storage instead of allocating O(n) new space | Remove Duplicates, Move Zeroes |
| Sentinel values | Add a dummy boundary value to eliminate special-case edge checks | Histogram (h=0 at the end), linked-list dummy heads |
| Swap trick | Exchange two elements via a temp variable (or XOR swap, rarely needed in Java) | Reversal, partitioning |
| Cyclic Sort | For arrays containing values in range `[1..n]` or `[0..n-1]`, place each value at its "correct" index in O(n) | Find Missing Number, Find All Duplicates |
| Dutch National Flag | 3-way partition around a pivot in one pass, O(n) time O(1) space | Sort Colors (0s, 1s, 2s) |
| Partition (Lomuto/Hoare) | Core building block of Quickselect and Quicksort | Kth Largest Element, Quickselect |
| Stable Partition | Partition while preserving relative order of equal-category elements | Some greedy/grouping problems requiring order preservation |
| Coordinate Compression | Map sparse/huge values to dense small range | Range problems with huge value bounds |
| Difference Array | O(1) range update, O(n) final build | Range Addition, Corporate Flight Bookings |
| Bucket Counting | Counting sort variant for bounded-range values | Sort by frequency, top-K with bounded value range |
| Frequency Compression | Combine frequency array with sorting/bucketing for O(n) solutions | Top K Frequent Elements (bucket sort variant, O(n) instead of O(n log n)) |
| Lazy Updates | Defer expensive updates until actually needed/queried | Segment tree lazy propagation (advanced, beyond pure array scope) |
| Index Mapping | Encode 2 values into 1 index (or vice versa) for O(1) space tricks | Encoding both old and new values in the same array cell using math (e.g. `arr[i] += (newVal) * range`) |
| Modulo indexing | Wrap around an index using `%` | Circular array problems |
| Circular Arrays | Treat the array as wrapping via modulo arithmetic | Circular Array Loop, Gas Station |
| Reverse Trick | Rotate array via 3 reversals (whole, then each part) | Rotate Array (Part 3) |
| Prefix Optimization | Precompute cumulative aggregates to answer range queries in O(1) | Prefix Sum / Prefix Product |
| Rolling Variables | Keep only the last k states instead of a full DP array, reduce O(n) space to O(1) or O(k) | Fibonacci-style DP, House Robber |

### Cyclic Sort Template
```java
int findMissingNumber(int[] nums) {
    int i = 0, n = nums.length;
    while (i < n) {
        int correctIdx = nums[i];
        if (correctIdx < n && nums[i] != nums[correctIdx]) {
            int tmp = nums[i]; nums[i] = nums[correctIdx]; nums[correctIdx] = tmp;
        } else {
            i++;
        }
    }
    for (i = 0; i < n; i++) if (nums[i] != i) return i;
    return n;
}
```

### Dutch National Flag Template
```java
void sortColors(int[] nums) {
    int low = 0, mid = 0, high = nums.length - 1;
    while (mid <= high) {
        if (nums[mid] == 0) { swap(nums, low++, mid++); }
        else if (nums[mid] == 1) { mid++; }
        else { swap(nums, mid, high--); } // don't increment mid: need to re-check swapped-in value
    }
}
void swap(int[] arr, int i, int j) { int t = arr[i]; arr[i] = arr[j]; arr[j] = t; }
```
**Why `mid` doesn't advance after swapping with `high`**: the value swapped in from the `high` region hasn't been examined yet — it could be a 0, 1, or 2, so `mid` must re-evaluate it on the next iteration.

---

# PART 21 — COMPLETE STRING TRICKS

## Character Frequency + ASCII Optimization
Use `int[26]` (or `int[128]` for full ASCII) instead of `HashMap<Character,Integer>` whenever the alphabet is bounded — faster, no boxing, no hashing overhead.

## Bitmasking Characters
When a problem only cares about **presence/absence** of each of 26 lowercase letters (not frequency), encode the whole set as a single `int` bitmask:
```java
int mask = 0;
for (char c : word.toCharArray()) mask |= (1 << (c - 'a'));
// two words share no letters iff (mask1 & mask2) == 0
```
Used in problems like "Maximum Product of Word Lengths" — turns an O(26) per-pair character comparison into an O(1) bitwise AND.

## Rolling Hash
Covered in depth in Part 12 (Rabin-Karp) — the core idea of updating a hash in O(1) as a window slides, by subtracting the outgoing character's contribution and adding the incoming one, scaled by the appropriate power of the base.

## Palindrome Expansion
Covered in Part 18 — expand-around-center technique, O(n²) but simple and reliable.

## Manacher's Algorithm (Overview)
Finds the longest palindromic substring in **O(n)**, improving on expand-around-center's O(n²).
**Core idea**: transform the string by inserting separators (`"abc"` → `"^#a#b#c#$"`) to unify odd/even-length palindrome handling, then use previously computed palindrome radii and a symmetry argument (mirror position within a known palindromic "window") to avoid redundant expansion — each position's radius is either directly inferable from its mirror, or needs limited further expansion, amortizing to O(n) total.
**When to reach for it**: only when the problem explicitly requires O(n) for longest-palindromic-substring-style queries (rare in typical interviews — expand-around-center's O(n²) is usually accepted, but knowing Manacher's exists and its O(n) guarantee signals CP-level depth).

## Trie Optimization
A tree where each path from root represents a prefix; each node has up to 26 children (lowercase) or a `Map<Character, TrieNode>` for a general alphabet.
```java
class TrieNode {
    TrieNode[] children = new TrieNode[26];
    boolean isEnd = false;
}
class Trie {
    TrieNode root = new TrieNode();
    void insert(String word) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) node.children[idx] = new TrieNode();
            node = node.children[idx];
        }
        node.isEnd = true;
    }
    boolean search(String word) {
        TrieNode node = find(word);
        return node != null && node.isEnd;
    }
    boolean startsWith(String prefix) {
        return find(prefix) != null;
    }
    private TrieNode find(String s) {
        TrieNode node = root;
        for (char c : s.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) return null;
            node = node.children[idx];
        }
        return node;
    }
}
```
**Time**: O(length of word) per insert/search — independent of how many words are stored, which is what makes tries superior to a HashSet of full strings for prefix-based queries (autocomplete, word search with backtracking + pruning).

## Prefix Function / Failure Function
This **is** the KMP failure function from Part 12 — worth re-emphasizing that "prefix function" and "failure function" are the same concept under different names across textbooks.

---

# PART 22 — COMMON VARIATIONS

| Variation | Defining trait | Special handling |
|---|---|---|
| Circular Arrays | Logical wraparound from last index to first | Use modulo indexing `(i+1) % n`; watch for infinite loop detection (Circular Array Loop) |
| Rotated Arrays | Sorted array shifted by an unknown offset | Modified binary search (Part 10) — determine which half is sorted first |
| Mountain Arrays | Strictly increases then strictly decreases | Binary search for the peak, then binary search each monotonic half separately |
| Nearly Sorted Arrays | Each element at most k positions from its sorted position | Use a min-heap of size k+1 instead of full sort — O(n log k) |
| Infinite Arrays / Unbounded Search | Size unknown in advance | Exponential/galloping search: probe at `1, 2, 4, 8...` until overshoot, then binary search within that range |
| Sparse Arrays | Mostly empty/default values | HashMap-based sparse representation instead of a dense array, to save space |
| Immutable Strings | Java's default `String` behavior | Use `StringBuilder` for any construction in a loop (File 1, Part 2) |
| Mutable Strings | `char[]` or `StringBuilder` used directly | Enables true in-place algorithms (e.g., in-place reversal) that `String` itself cannot do |
| Streaming Data | Can't store the whole input, one pass only | Reservoir sampling, running statistics (mean/median via heaps), fixed-size sliding structures |
| Online Queries | Queries interleaved with updates, must answer immediately | Fenwick Tree / Segment Tree instead of static prefix sum; balanced BST / TreeMap for order-statistics queries |

---
**End of File 5.** Continue to File 6 for LeetCode Pattern Grouping and the Java Template Library (Parts 23–24), and File 7 for Mistakes, Interview Process, Roadmap, and Cheat Sheets (Parts 25–30).
