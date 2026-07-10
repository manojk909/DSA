# ARRAYS & STRINGS KNOWLEDGE BASE — FILE 4
## Parts 12–17: String Matching, Greedy Patterns, Monotonic Stack, Heap, Interval Patterns, Matrix Patterns

---

# PART 12 — STRING MATCHING

## Naive Pattern Matching
```java
List<Integer> naiveSearch(String text, String pattern) {
    List<Integer> result = new ArrayList<>();
    int n = text.length(), m = pattern.length();
    for (int i = 0; i <= n - m; i++) {
        int j = 0;
        while (j < m && text.charAt(i + j) == pattern.charAt(j)) j++;
        if (j == m) result.add(i);
    }
    return result;
}
```
**Time**: O(n·m) worst case (e.g., text = "aaaa...a", pattern = "aaab").

## KMP (Knuth-Morris-Pratt)
Precomputes a **failure function** (a.k.a. "prefix function") so that on a mismatch, you never re-examine characters you've already matched — skip directly to the next viable alignment.
```java
int[] buildFailureFunction(String pattern) {
    int m = pattern.length();
    int[] fail = new int[m];
    int len = 0; // length of the previous longest prefix-suffix
    for (int i = 1; i < m; i++) {
        while (len > 0 && pattern.charAt(i) != pattern.charAt(len)) len = fail[len - 1];
        if (pattern.charAt(i) == pattern.charAt(len)) len++;
        fail[i] = len;
    }
    return fail;
}

List<Integer> kmpSearch(String text, String pattern) {
    List<Integer> result = new ArrayList<>();
    int[] fail = buildFailureFunction(pattern);
    int n = text.length(), m = pattern.length();
    int j = 0; // pattern index
    for (int i = 0; i < n; i++) {
        while (j > 0 && text.charAt(i) != pattern.charAt(j)) j = fail[j - 1];
        if (text.charAt(i) == pattern.charAt(j)) j++;
        if (j == m) { result.add(i - m + 1); j = fail[j - 1]; }
    }
    return result;
}
```
**Time**: O(n+m). **Why it works**: `fail[i]` tells you the longest proper prefix of `pattern[0..i]` that is also a suffix — when a mismatch happens after matching `j` characters, you know the last `fail[j-1]` characters already match the pattern's prefix, so you can resume from there instead of restarting.

## Z-Algorithm
Computes, for every position `i`, the length of the longest substring starting at `i` that matches a prefix of the string. Used for pattern matching by concatenating `pattern + '#' + text` and scanning the Z-array for positions equal to `pattern.length()`.
```java
int[] zFunction(String s) {
    int n = s.length();
    int[] z = new int[n];
    int l = 0, r = 0;
    for (int i = 1; i < n; i++) {
        if (i < r) z[i] = Math.min(r - i, z[i - l]);
        while (i + z[i] < n && s.charAt(z[i]) == s.charAt(i + z[i])) z[i]++;
        if (i + z[i] > r) { l = i; r = i + z[i]; }
    }
    return z;
}
```
**Time**: O(n). Similar power to KMP but sometimes more intuitive for problems directly about prefix-matching structure (e.g., "shortest palindrome," string periodicity).

## Rabin-Karp (Rolling Hash)
Computes a rolling hash of the pattern and every window of the text; compares hashes first (O(1) per window after O(m) setup), only does a full character comparison on hash collision.
```java
List<Integer> rabinKarp(String text, String pattern) {
    List<Integer> result = new ArrayList<>();
    int n = text.length(), m = pattern.length();
    if (m > n) return result;
    long BASE = 31, MOD = 1_000_000_007L;
    long patternHash = 0, windowHash = 0, power = 1;
    for (int i = 0; i < m; i++) {
        patternHash = (patternHash * BASE + pattern.charAt(i)) % MOD;
        windowHash = (windowHash * BASE + text.charAt(i)) % MOD;
        if (i > 0) power = (power * BASE) % MOD;
    }
    for (int i = 0; i <= n - m; i++) {
        if (windowHash == patternHash && text.substring(i, i + m).equals(pattern)) result.add(i);
        if (i < n - m) {
            windowHash = ((windowHash - text.charAt(i) * power % MOD + MOD) * BASE + text.charAt(i + m)) % MOD;
        }
    }
    return result;
}
```
**Time**: O(n+m) average, O(n·m) worst case (adversarial hash collisions). **Why the verification step matters**: hash collisions are possible (different substrings, same hash) — always verify with an actual string comparison before confirming a match, unless using double hashing to make collision probability negligible.

## Boyer-Moore (concept overview)
Scans the pattern **right to left** against the text window; on mismatch, uses two heuristics — the "bad character rule" (skip past the mismatched character's last occurrence in the pattern) and "good suffix rule" (skip based on the matched suffix) — to skip large chunks of text, often sub-linear in practice. Rarely implemented from scratch in interviews (more of a systems/library-level algorithm — e.g., what many real `string.find()` implementations use), but good to know conceptually.

## Applications Summary

| Algorithm | Time | Best For |
|---|---|---|
| Naive | O(n·m) | Tiny inputs, simplicity |
| KMP | O(n+m) | Guaranteed linear, single pattern |
| Z-algorithm | O(n+m) | Prefix-structure problems, single pattern |
| Rabin-Karp | O(n+m) avg | Multiple pattern search (batch hashing), plagiarism detection |
| Boyer-Moore | O(n/m) best case | Practical text search (used in real libraries) |

---

# PART 13 — GREEDY PATTERNS (Array/String Specific)

## Core Greedy Recognition
A greedy approach works when the problem has **optimal substructure + the greedy choice property**: making the locally best choice at each step never precludes reaching a globally optimal solution. Always be ready to justify *why* greedy works (exchange argument or cut-property-style reasoning) — interviewers probe this specifically.

## Common Array/String Greedy Patterns
| Problem type | Greedy rule |
|---|---|
| Jump Game | Track the farthest reachable index greedily while scanning left to right |
| Gas Station | If total gas ≥ total cost, a solution exists; greedily reset start whenever tank goes negative |
| Candy Distribution | Two greedy passes (left-to-right, then right-to-left), take the max of both requirements per child |
| Interval Scheduling (max non-overlapping) | Sort by end time, greedily pick earliest-ending compatible interval |
| Partition Labels | Track the last occurrence of each character, greedily extend the current partition until it can close |
| Assign Cookies | Sort both arrays, greedily match smallest sufficient cookie to each child |
| Minimum Number of Arrows to Burst Balloons | Sort by end coordinate, greedily "burst" whenever a balloon starts after the current arrow position |

```java
// Jump Game — classic greedy "farthest reachable" pattern
boolean canJump(int[] nums) {
    int farthest = 0;
    for (int i = 0; i < nums.length; i++) {
        if (i > farthest) return false; // stuck, can't reach index i
        farthest = Math.max(farthest, i + nums[i]);
    }
    return true;
}
```

## Greedy vs DP — How to Tell Them Apart
If a **local** decision (pick the best option available right now) never needs to be "undone" based on future information, it's greedy. If the best choice at step `i` genuinely **depends on** what happens later (need to consider multiple possibilities and pick the best in hindsight), it's DP. When unsure, try to find a counterexample to the greedy rule — if you can't after a few tries, it's a strong sign greedy is correct (though a real proof is better for interview credibility).

---

# PART 14 — MONOTONIC STACK PATTERNS

A monotonic stack maintains elements in strictly increasing or decreasing order, popping elements that violate the order as you scan — this gives O(n) total work (each element pushed/popped at most once) for "next greater/smaller" style queries that would otherwise be O(n²).

## Next Greater Element
```java
int[] nextGreaterElement(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];
    Arrays.fill(result, -1);
    Deque<Integer> stack = new ArrayDeque<>(); // stores indices, values decreasing bottom-to-top... actually top-to-bottom increasing
    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && nums[stack.peek()] < nums[i]) {
            result[stack.pop()] = nums[i];
        }
        stack.push(i);
    }
    return result;
}
```
**Why it works**: when you encounter `nums[i]` bigger than the stack's top, that top element has finally found its "next greater" — pop and record it. Anything left on the stack at the end has no next greater element (stays -1).

## Previous Greater / Next Smaller / Previous Smaller
Same skeleton, flip the comparison direction and/or the scan direction:
- **Next Smaller**: `while (!stack.isEmpty() && nums[stack.peek()] > nums[i])`
- **Previous Greater**: scan the array, but think of it as "next greater if scanned right-to-left" or maintain the stack while scanning left-to-right and record before popping.

## Histogram — Largest Rectangle in Histogram
```java
int largestRectangleArea(int[] heights) {
    Deque<Integer> stack = new ArrayDeque<>(); // indices, increasing height
    int maxArea = 0, n = heights.length;
    for (int i = 0; i <= n; i++) {
        int h = (i == n) ? 0 : heights[i]; // sentinel: force-flush remaining stack at the end
        while (!stack.isEmpty() && heights[stack.peek()] > h) {
            int height = heights[stack.pop()];
            int width = stack.isEmpty() ? i : i - stack.peek() - 1;
            maxArea = Math.max(maxArea, height * width);
        }
        stack.push(i);
    }
    return maxArea;
}
```
**Why the sentinel `h=0` at the end**: forces every remaining bar in the stack to be popped and evaluated, so you don't need special post-loop cleanup code.

## Daily Temperatures
```java
int[] dailyTemperatures(int[] temps) {
    int n = temps.length;
    int[] result = new int[n];
    Deque<Integer> stack = new ArrayDeque<>(); // indices, decreasing temperature
    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && temps[stack.peek()] < temps[i]) {
            int idx = stack.pop();
            result[idx] = i - idx;
        }
        stack.push(i);
    }
    return result;
}
```

## Stock Span
```java
class StockSpanner {
    Deque<int[]> stack = new ArrayDeque<>(); // {price, span}
    int next(int price) {
        int span = 1;
        while (!stack.isEmpty() && stack.peek()[0] <= price) {
            span += stack.pop()[1];
        }
        stack.push(new int[]{price, span});
        return span;
    }
}
```

## Recognition Checklist
"Next greater/smaller element," "daily temperatures," "stock span," "largest rectangle," "trapping rain water" (alternative to two-pointer solution), "remove K digits to form smallest number" — all scream **monotonic stack**.

---

# PART 15 — HEAP PATTERNS

Java's `PriorityQueue` is a **min-heap by default**; pass a comparator for max-heap behavior.

## Top K Elements
```java
int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int x : nums) freq.merge(x, 1, Integer::sum);
    // min-heap of size k, keyed by frequency — keeps only the top k seen so far
    PriorityQueue<Integer> heap = new PriorityQueue<>((a, b) -> freq.get(a) - freq.get(b));
    for (int key : freq.keySet()) {
        heap.offer(key);
        if (heap.size() > k) heap.poll(); // evict smallest-frequency element
    }
    int[] result = new int[k];
    for (int i = k - 1; i >= 0; i--) result[i] = heap.poll();
    return result;
}
```
**Why a size-k min-heap for "top K largest"**: keeping a min-heap capped at size k means the heap's minimum is always the "worst of the best k so far" — anything smaller than that gets discarded, so the heap always holds the true top K in O(n log k) instead of O(n log n) full sort.

## Merge K Sorted Arrays/Lists
```java
int[] mergeKArrays(int[][] arrays) {
    // {value, arrayIndex, elementIndex}
    PriorityQueue<int[]> heap = new PriorityQueue<>((a, b) -> a[0] - b[0]);
    int total = 0;
    for (int i = 0; i < arrays.length; i++) {
        if (arrays[i].length > 0) heap.offer(new int[]{arrays[i][0], i, 0});
        total += arrays[i].length;
    }
    int[] result = new int[total];
    int idx = 0;
    while (!heap.isEmpty()) {
        int[] top = heap.poll();
        result[idx++] = top[0];
        int arrIdx = top[1], elemIdx = top[2];
        if (elemIdx + 1 < arrays[arrIdx].length) {
            heap.offer(new int[]{arrays[arrIdx][elemIdx + 1], arrIdx, elemIdx + 1});
        }
    }
    return result;
}
```

## K Closest Points / Elements
Same size-k heap pattern as Top K, keyed by distance (max-heap of size k, evict farthest).

## Running Median (Two-Heap Technique)
```java
class MedianFinder {
    PriorityQueue<Integer> lowerHalf = new PriorityQueue<>(Collections.reverseOrder()); // max-heap
    PriorityQueue<Integer> upperHalf = new PriorityQueue<>(); // min-heap

    void addNum(int num) {
        lowerHalf.offer(num);
        upperHalf.offer(lowerHalf.poll()); // balance: move max of lower to upper
        if (upperHalf.size() > lowerHalf.size()) {
            lowerHalf.offer(upperHalf.poll());
        }
    }

    double findMedian() {
        if (lowerHalf.size() > upperHalf.size()) return lowerHalf.peek();
        return (lowerHalf.peek() + upperHalf.peek()) / 2.0;
    }
}
```
**Why two heaps**: `lowerHalf` (max-heap) holds the smaller half of numbers, `upperHalf` (min-heap) holds the larger half, kept balanced in size — the median is always derivable from the two heap tops in O(1), with O(log n) insertion.

## Priority Queue Java Gotchas
- `new PriorityQueue<>()` on `Integer` is a **min-heap**; use `Collections.reverseOrder()` or `(a,b) -> b - a` for max-heap.
- Storing custom objects requires either implementing `Comparable` or always passing an explicit `Comparator`.
- `PriorityQueue` does **not** support O(log n) arbitrary-element removal or decrease-key natively — `remove(Object)` is O(n). For Dijkstra-style "decrease key" needs, use the "lazy deletion" trick (push new entries, skip stale ones on pop) as shown in the Graph KB.

---

# PART 16 — INTERVAL PATTERNS

## Merge Intervals
```java
int[][] merge(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[0] - b[0]); // sort by start
    List<int[]> result = new ArrayList<>();
    for (int[] interval : intervals) {
        if (result.isEmpty() || result.get(result.size() - 1)[1] < interval[0]) {
            result.add(interval);
        } else {
            result.get(result.size() - 1)[1] = Math.max(result.get(result.size() - 1)[1], interval[1]);
        }
    }
    return result.toArray(new int[0][]);
}
```

## Insert Interval
Three phases: add all intervals ending before the new one starts, merge all overlapping intervals into the new one, add all intervals starting after the new one ends.

## Meeting Rooms (can attend all?)
```java
boolean canAttendMeetings(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] < intervals[i - 1][1]) return false; // overlap
    }
    return true;
}
```

## Meeting Rooms II (minimum rooms needed) — Line Sweep
```java
int minMeetingRooms(int[][] intervals) {
    int n = intervals.length;
    int[] starts = new int[n], ends = new int[n];
    for (int i = 0; i < n; i++) { starts[i] = intervals[i][0]; ends[i] = intervals[i][1]; }
    Arrays.sort(starts);
    Arrays.sort(ends);
    int rooms = 0, maxRooms = 0, s = 0, e = 0;
    while (s < n) {
        if (starts[s] < ends[e]) { rooms++; s++; }
        else { rooms--; e++; }
        maxRooms = Math.max(maxRooms, rooms);
    }
    return maxRooms;
}
```
**Why sorting starts/ends separately works**: you don't need to know *which* meeting is which, just the count of overlapping meetings at any point in time — a classic line-sweep / difference-array-style counting trick.

## Difference Array Approach (alternative to line sweep for interval counting)
```java
// My Calendar / interval overlap counting via difference array over compressed coordinates
int[] diff = new int[maxTime + 2];
for (int[] interval : intervals) { diff[interval[0]]++; diff[interval[1]]--; }
int running = 0, maxOverlap = 0;
for (int i = 0; i <= maxTime; i++) { running += diff[i]; maxOverlap = Math.max(maxOverlap, running); }
```

## Recognition Checklist
"Merge overlapping," "insert into sorted intervals," "can attend all meetings," "minimum rooms/resources needed," "employee free time" — all interval-pattern problems. Almost always: **sort by start (or end) first**, then either merge linearly or line-sweep with separate start/end arrays.

---

# PART 17 — MATRIX PATTERNS

## Traversal (row-major vs column-major)
```java
for (int i = 0; i < rows; i++)
    for (int j = 0; j < cols; j++) { /* process grid[i][j] */ }
```

## Spiral Traversal
```java
List<Integer> spiralOrder(int[][] matrix) {
    List<Integer> result = new ArrayList<>();
    int top = 0, bottom = matrix.length - 1, left = 0, right = matrix[0].length - 1;
    while (top <= bottom && left <= right) {
        for (int j = left; j <= right; j++) result.add(matrix[top][j]);
        top++;
        for (int i = top; i <= bottom; i++) result.add(matrix[i][right]);
        right--;
        if (top <= bottom) {
            for (int j = right; j >= left; j--) result.add(matrix[bottom][j]);
            bottom--;
        }
        if (left <= right) {
            for (int i = bottom; i >= top; i--) result.add(matrix[i][left]);
            left--;
        }
    }
    return result;
}
```
**Why the extra boundary checks (`if top<=bottom`, `if left<=right`) before the last two loops**: without them, a single-row or single-column remaining matrix would get double-counted (traversed twice) once the spiral shrinks to a degenerate strip.

## Diagonal Traversal
Elements on the same diagonal share `row - col` (for "\" diagonals) or `row + col` (for "/" diagonals) — group or iterate using this invariant.

## Rotate Image (90° in-place)
```java
void rotate(int[][] matrix) {
    int n = matrix.length;
    // transpose
    for (int i = 0; i < n; i++)
        for (int j = i + 1; j < n; j++) {
            int tmp = matrix[i][j]; matrix[i][j] = matrix[j][i]; matrix[j][i] = tmp;
        }
    // reverse each row
    for (int[] row : matrix) {
        for (int l = 0, r = n - 1; l < r; l++, r--) {
            int tmp = row[l]; row[l] = row[r]; row[r] = tmp;
        }
    }
}
```
**Why transpose + row-reverse = 90° clockwise rotation**: transposing swaps rows/columns (reflects across the main diagonal); reversing each row then flips left-right — the composition of these two reflections is exactly a 90° clockwise rotation, done in-place with O(1) extra space.

## Transpose
Shown above as the first step of rotation — swap `matrix[i][j]` with `matrix[j][i]`.

## Prefix Matrix (2D Prefix Sum)
Covered in File 3, Part 9.

## Simulation Patterns
Problems like "Game of Life," "Robot Bounded in Circle," "Spiral Matrix II" are direct simulation — just carefully implement the described process, paying attention to **in-place update ordering** (Game of Life needs to distinguish "old state" from "new state" while iterating — typically via encoding both in one cell using extra bit tricks, or using a separate copy).

---
**End of File 4.** Continue to File 5 for String DP Intro, Interview Pattern Recognition Decision Tree, Array/String Tricks, and Common Variations (Parts 18–22).
