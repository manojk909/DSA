# ARRAYS & STRINGS KNOWLEDGE BASE — FILE 2
## Parts 5–7: Pattern Recognition Overview, Two Pointers, Sliding Window

---

# PART 5 — PATTERN RECOGNITION (Overview)

Before diving into each technique in depth, here's the map of **every** major Array/String pattern and its one-line trigger. (Full decision tree with keyword mapping is in Part 19.)

| Pattern | One-line trigger |
|---|---|
| Simulation | Problem literally describes a process to execute step by step |
| Brute Force | Small constraints (n ≤ ~20) or as a baseline before optimizing |
| Hashing | Need O(1) lookup/existence/frequency check |
| Sorting | Order doesn't matter for the answer, or enables greedy/two-pointer |
| Two Pointers | Sorted array + pair/triplet sum, or in-place partitioning |
| Sliding Window | Contiguous subarray/substring with a size or property constraint |
| Prefix Sum | Repeated range-sum queries |
| Suffix Sum | Need "everything after index i" aggregated |
| Difference Array | Many range **updates**, then read final array once |
| Greedy | Local optimal choice provably leads to global optimal (interval scheduling, etc.) |
| Binary Search on Answer | "Minimize the maximum" / "maximize the minimum" phrasing, monotonic predicate |
| Monotonic Stack | "Next/previous greater/smaller element" |
| Heap | "K largest/smallest," "merge K," "running median" |
| Counting | Values in a small bounded range |
| Frequency Array | Fixed alphabet (26 letters, digits) |
| Bit Manipulation | XOR tricks, subsets, "single number," bitmask DP |
| Trie | Prefix-based lookups, autocomplete, word search |
| Rolling Hash | Fast substring comparison/matching without full re-comparison |
| String Matching (KMP/Z) | Find all occurrences of a pattern in O(n+m) |

---

# PART 6 — TWO POINTERS

## Same Direction (Fast-Slow within one array, non-cyclic)
Both pointers move forward, at different rates or trigger conditions — used for in-place array modification.
```java
// Remove duplicates from sorted array in-place
int removeDuplicates(int[] nums) {
    if (nums.length == 0) return 0;
    int slow = 0;
    for (int fast = 1; fast < nums.length; fast++) {
        if (nums[fast] != nums[slow]) {
            slow++;
            nums[slow] = nums[fast];
        }
    }
    return slow + 1;
}
```

## Opposite Direction (Converging from both ends)
Used on **sorted** arrays for pair-sum problems, palindrome checks, container problems.
```java
// Two Sum on sorted array
int[] twoSumSorted(int[] nums, int target) {
    int l = 0, r = nums.length - 1;
    while (l < r) {
        int sum = nums[l] + nums[r];
        if (sum == target) return new int[]{l, r};
        else if (sum < target) l++;
        else r--;
    }
    return new int[]{-1, -1};
}
```
**Why it works**: since the array is sorted, if the current sum is too small, the *only* way to increase it is to move `l` right (increasing the smaller element); moving `r` left would only decrease the sum further. This monotonicity is what makes the O(n) sweep valid instead of needing O(n²).

## Fast-Slow Pointer (Floyd's Cycle Detection — technically linked-list/functional-graph territory, but shows up in array problems like "Find the Duplicate Number")
```java
int findDuplicate(int[] nums) {
    int slow = nums[0], fast = nums[0];
    do { slow = nums[slow]; fast = nums[nums[fast]]; } while (slow != fast);
    slow = nums[0];
    while (slow != fast) { slow = nums[slow]; fast = nums[fast]; }
    return slow;
}
```

## Partitioning (Dutch National Flag style)
Three pointers dividing the array into regions — see Part 20 for the full Dutch National Flag template.

## Three Pointers (3Sum)
```java
List<List<Integer>> threeSum(int[] nums) {
    Arrays.sort(nums);
    List<List<Integer>> result = new ArrayList<>();
    for (int i = 0; i < nums.length - 2; i++) {
        if (i > 0 && nums[i] == nums[i - 1]) continue; // skip duplicate anchors
        int l = i + 1, r = nums.length - 1;
        while (l < r) {
            int sum = nums[i] + nums[l] + nums[r];
            if (sum == 0) {
                result.add(Arrays.asList(nums[i], nums[l], nums[r]));
                while (l < r && nums[l] == nums[l + 1]) l++; // skip dup
                while (l < r && nums[r] == nums[r - 1]) r--; // skip dup
                l++; r--;
            } else if (sum < 0) l++;
            else r--;
        }
    }
    return result;
}
```

## K-Sum (generalization)
Recursively reduce K-Sum to (K-1)-Sum until you hit 2-Sum (two pointers) as the base case. Sort first, O(n^(k-1)) overall.

## Container Problems (Container With Most Water)
```java
int maxArea(int[] height) {
    int l = 0, r = height.length - 1, best = 0;
    while (l < r) {
        int area = Math.min(height[l], height[r]) * (r - l);
        best = Math.max(best, area);
        if (height[l] < height[r]) l++; else r--; // move the shorter side
    }
    return best;
}
```
**Why move the shorter side**: the container's height is capped by the shorter wall. Moving the taller wall inward can only decrease or maintain width while height stays capped by the same short wall — provably never improves the answer. Moving the shorter wall is the *only* move that could possibly find a taller wall and improve area.

## Recognition Checklist
- Sorted (or sortable) array + looking for pairs/triplets summing to a target → **opposite direction two pointers**.
- In-place removal/partition/dedup on an array → **same direction (fast-slow)**.
- Need to detect a cycle in an implicit "next pointer" structure → **fast-slow (Floyd's)**.

## When NOT to Use Two Pointers
- Array is unsorted **and** sorting would destroy needed information (e.g., original indices matter and can't be recovered) → use HashMap instead (e.g., classic unsorted Two Sum returns indices, not values).
- The relationship between elements isn't monotonic (moving one pointer doesn't predictably help/hurt the target condition).

---

# PART 7 — SLIDING WINDOW

Sliding window is fundamentally an optimization of the "check every subarray" brute force (O(n²) or O(n³)) down to O(n), by maintaining a window `[left, right]` and incrementally adjusting instead of recomputing from scratch.

## Fixed Window (size k)
```java
// Max sum of any subarray of size k
int maxSumFixedWindow(int[] nums, int k) {
    int windowSum = 0;
    for (int i = 0; i < k; i++) windowSum += nums[i];
    int best = windowSum;
    for (int right = k; right < nums.length; right++) {
        windowSum += nums[right] - nums[right - k]; // slide: add new, remove old
        best = Math.max(best, windowSum);
    }
    return best;
}
```

## Variable Window — Maximum Window (Longest subarray satisfying a condition)
```java
// Longest substring without repeating characters
int lengthOfLongestSubstring(String s) {
    Set<Character> window = new HashSet<>();
    int left = 0, best = 0;
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        while (window.contains(c)) {
            window.remove(s.charAt(left));
            left++;
        }
        window.add(c);
        best = Math.max(best, right - left + 1);
    }
    return best;
}
```
**Why the `while` shrinks correctly**: every character is added once (`right` loop) and removed at most once (`left` loop), so total work is O(n) amortized — not O(n²) despite the nested-looking loops.

## Variable Window — Minimum Window (Shortest subarray satisfying a condition)
```java
// Minimum Window Substring (contains all chars of t)
String minWindow(String s, String t) {
    Map<Character, Integer> need = new HashMap<>();
    for (char c : t.toCharArray()) need.merge(c, 1, Integer::sum);
    Map<Character, Integer> window = new HashMap<>();
    int have = 0, required = need.size();
    int left = 0, bestLen = Integer.MAX_VALUE, bestStart = 0;
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        window.merge(c, 1, Integer::sum);
        if (need.containsKey(c) && window.get(c).intValue() == need.get(c).intValue()) have++;
        while (have == required) {
            if (right - left + 1 < bestLen) { bestLen = right - left + 1; bestStart = left; }
            char lc = s.charAt(left);
            window.put(lc, window.get(lc) - 1);
            if (need.containsKey(lc) && window.get(lc) < need.get(lc)) have--;
            left++;
        }
    }
    return bestLen == Integer.MAX_VALUE ? "" : s.substring(bestStart, bestStart + bestLen);
}
```
**Pattern**: expand `right` until the condition is satisfied, then shrink `left` as much as possible while it stays satisfied, recording the best each time you shrink.

## The Two Variable-Window Templates (memorize both — nearly every sliding window problem is one of these)
```
MAXIMUM window (find longest valid window):
  expand right always
  while (window INVALID) shrink left
  update answer AFTER the while (window is now valid, or was never invalid)

MINIMUM window (find shortest valid window):
  expand right always
  while (window VALID) { update answer; shrink left }
```

## Distinct Elements / Frequency Window
"At most K distinct characters" — combine a frequency map with the maximum-window template, shrink when `map.size() > k`.

## Monotonic Window (Sliding Window Maximum)
Uses a **monotonic deque** (see Part 14) instead of recomputing max on every slide.
```java
int[] maxSlidingWindow(int[] nums, int k) {
    Deque<Integer> deque = new ArrayDeque<>(); // stores indices, values decreasing
    int[] result = new int[nums.length - k + 1];
    for (int i = 0; i < nums.length; i++) {
        while (!deque.isEmpty() && deque.peekFirst() <= i - k) deque.pollFirst(); // out of window
        while (!deque.isEmpty() && nums[deque.peekLast()] < nums[i]) deque.pollLast(); // maintain decreasing
        deque.offerLast(i);
        if (i >= k - 1) result[i - k + 1] = nums[deque.peekFirst()];
    }
    return result;
}
```
**Why monotonic deque gives O(n) instead of O(n·k)**: each index is pushed once and popped at most once across the entire run, so total deque operations are O(n) — the front of the deque is always the current window's max.

## Recognition Checklist
- "Subarray/substring" (contiguous!) + "of size k" → **fixed window**.
- "Longest subarray/substring satisfying X" → **maximum variable window**.
- "Shortest/minimum subarray/substring containing/satisfying X" → **minimum variable window**.
- "Sliding window maximum/minimum value" → **monotonic deque window**.
- If the problem says "subsequence" (not contiguous) → **sliding window does NOT apply**, think DP instead.

---
**End of File 2.** Continue to File 3 for Hashing, Prefix Sum, Binary Search, and Sorting Patterns (Parts 8–11).
