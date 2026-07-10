# ARRAYS & STRINGS KNOWLEDGE BASE — FILE 3
## Parts 8–11: Hashing, Prefix Sum, Binary Search, Sorting Patterns

---

# PART 8 — HASHING

## HashMap / HashSet in Java
```java
Map<Integer, Integer> map = new HashMap<>(); // avg O(1) get/put, O(n) worst case (collisions)
Set<Integer> set = new HashSet<>();
```
Backed by an array of buckets; each key's `hashCode()` determines its bucket, collisions resolved via chaining (linked list, or a red-black tree if a bucket gets too large in Java 8+ — protects against adversarial hash-flooding attacks).

## Frequency Arrays vs HashMap
For a **known small alphabet** (lowercase letters, digits, ASCII), prefer a fixed-size array over a HashMap — it's faster (no hashing overhead, no boxing) and simpler:
```java
int[] freq = new int[26]; // beats HashMap<Character,Integer> for lowercase-only problems
```
Use `HashMap` when the key space is large/unbounded/non-integer (arbitrary strings, objects).

## Counting Patterns
```java
// Two Sum (unsorted, need indices) — classic hashing pattern
int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>(); // value -> index
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (seen.containsKey(complement)) return new int[]{seen.get(complement), i};
        seen.put(nums[i], i);
    }
    return new int[]{-1, -1};
}
```
**Why this beats two-pointers here**: the problem needs original indices returned, and sorting would destroy that mapping — hashing preserves O(n) without sacrificing index information.

## Character Mapping
Covered in File 1 Part 4 (Isomorphic Strings) — bidirectional HashMap to enforce a bijection.

## Collision Concepts
Java's `HashMap` uses `hashCode()` + `equals()`. **Critical for custom objects**: if you override `equals()`, you **must** override `hashCode()` too (contract: equal objects must have equal hash codes), or the object won't work correctly as a HashMap key / HashSet element — a very common Java interview gotcha. For arrays as keys (`int[]`), remember arrays don't override `equals`/`hashCode` meaningfully by default — use `Arrays.hashCode()`/`Arrays.equals()`, or convert to a `List<Integer>` / String key, or use `Arrays.toString(arr)` as a map key for grouping problems.

## Applications
| Problem type | Hashing role |
|---|---|
| Two Sum / K Sum variants (need indices) | value → index map |
| Group Anagrams | sorted-string or char-frequency signature → list of words |
| Longest Consecutive Sequence | HashSet for O(1) existence check, only start counting from sequence starts |
| Subarray Sum Equals K | prefix-sum → count map (see Part 9) |
| Contains Duplicate | HashSet, early exit on repeat |

```java
// Group Anagrams — classic hashing + signature pattern
List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> groups = new HashMap<>();
    for (String s : strs) {
        char[] chars = s.toCharArray();
        Arrays.sort(chars);
        String key = new String(chars);
        groups.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
    }
    return new ArrayList<>(groups.values());
}
```

---

# PART 9 — PREFIX SUM

## 1D Prefix Sum
```java
int[] prefix = new int[n + 1];
for (int i = 0; i < n; i++) prefix[i + 1] = prefix[i] + arr[i];
// sum of arr[l..r] inclusive = prefix[r+1] - prefix[l]
```

## 2D Prefix Sum
```java
int[][] prefix = new int[rows + 1][cols + 1];
for (int i = 0; i < rows; i++)
    for (int j = 0; j < cols; j++)
        prefix[i+1][j+1] = grid[i][j] + prefix[i][j+1] + prefix[i+1][j] - prefix[i][j];

// sum of rectangle (r1,c1) to (r2,c2) inclusive:
int rectSum(int[][] prefix, int r1, int c1, int r2, int c2) {
    return prefix[r2+1][c2+1] - prefix[r1][c2+1] - prefix[r2+1][c1] + prefix[r1][c1];
}
```
**Inclusion-exclusion intuition**: subtract both overlapping strips (top and left), but you've subtracted the top-left corner rectangle twice, so add it back once.

## Difference Array
Covered in File 1 Part 3 — the "inverse" operation of prefix sum, for O(1) range updates.

## Range Queries — When Prefix Sum Applies
Use prefix sum when: the array is **static** (not being modified between queries) and you need **many** range-sum queries — turns O(n) per query into O(1) after O(n) preprocessing.
**When it does NOT apply**: if the array is being updated frequently between queries, prefix sum requires O(n) rebuild per update — use a **Fenwick Tree (BIT)** or **Segment Tree** instead for O(log n) update + O(log n) query.

## Applications
| Problem | Technique |
|---|---|
| Range Sum Query - Immutable | 1D prefix sum |
| Range Sum Query 2D - Immutable | 2D prefix sum |
| Subarray Sum Equals K | prefix sum + hashmap of prefix-sum frequencies |
| Product of Array Except Self | prefix product × suffix product (no division needed) |
| Continuous Subarray Sum | prefix sum mod k + hashmap of remainders |

```java
// Subarray Sum Equals K — prefix sum + hashmap combo (extremely common pattern)
int subarraySum(int[] nums, int k) {
    Map<Integer, Integer> prefixCount = new HashMap<>();
    prefixCount.put(0, 1); // empty prefix
    int sum = 0, count = 0;
    for (int x : nums) {
        sum += x;
        count += prefixCount.getOrDefault(sum - k, 0);
        prefixCount.merge(sum, 1, Integer::sum);
    }
    return count;
}
```
**Why `prefixCount.put(0, 1)` at the start**: handles the case where a prefix itself (from index 0) equals k — without this seed, that valid subarray would be missed.

---

# PART 10 — BINARY SEARCH

## Classic Binary Search
```java
int binarySearch(int[] arr, int target) {
    int left = 0, right = arr.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2; // avoids overflow vs (left+right)/2
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}
```
**Overflow note**: `(left + right) / 2` can overflow `int` for very large indices — `left + (right - left) / 2` is the safe idiom (relevant in Java too, though less likely to bite at typical array sizes; still the professional habit to show in an interview).

## Lower Bound (first index where arr[i] >= target)
```java
int lowerBound(int[] arr, int target) {
    int left = 0, right = arr.length; // note: right = length, not length-1
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] < target) left = mid + 1;
        else right = mid;
    }
    return left;
}
```

## Upper Bound (first index where arr[i] > target)
```java
int upperBound(int[] arr, int target) {
    int left = 0, right = arr.length;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] <= target) left = mid + 1;
        else right = mid;
    }
    return left;
}
```

## First / Last Occurrence
```java
int firstOccurrence(int[] arr, int target) {
    int idx = lowerBound(arr, target);
    return (idx < arr.length && arr[idx] == target) ? idx : -1;
}
int lastOccurrence(int[] arr, int target) {
    int idx = upperBound(arr, target) - 1;
    return (idx >= 0 && arr[idx] == target) ? idx : -1;
}
```

## The "lo/hi/mid" Boundary Template — Memorize This Precisely
```
Closed interval [left, right], loop while (left <= right):
  used for: exact match search, find-if-exists

Half-open interval [left, right), loop while (left < right):
  used for: lower_bound/upper_bound style "find boundary" search
  ALWAYS set right = arr.length (not length-1) for this style
```
**This is the #1 source of off-by-one bugs in binary search** — mixing the closed-interval and half-open-interval conventions within the same function.

## Binary Search on Answer
Used when the **search space isn't the array itself**, but a range of possible answers, and there's a **monotonic predicate**: "can we achieve X with value V?" is true for all V ≥ some threshold (or false past a threshold).
```java
// Koko Eating Bananas — classic binary search on answer
int minEatingSpeed(int[] piles, int h) {
    int lo = 1, hi = Arrays.stream(piles).max().getAsInt();
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (canFinish(piles, h, mid)) hi = mid; // mid works, try smaller
        else lo = mid + 1; // mid too slow, need faster
    }
    return lo;
}
boolean canFinish(int[] piles, int h, int speed) {
    long hours = 0;
    for (int p : piles) hours += (p + speed - 1) / speed; // ceiling division
    return hours <= h;
}
```
**Recognition clue**: "minimize the maximum," "maximize the minimum," "find the smallest/largest value such that condition X holds" — and you can write a `boolean canAchieve(value)` function that's monotonic.

## Peak Element
```java
int findPeakElement(int[] nums) {
    int left = 0, right = nums.length - 1;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] > nums[mid + 1]) right = mid; // peak is at mid or to the left
        else left = mid + 1; // peak is to the right
    }
    return left;
}
```
**Why binary search works here despite no full sort**: comparing `nums[mid]` to its neighbor tells you which direction is "uphill," and following uphill is guaranteed to reach a peak (a local property extended into a global search via monotonic slope direction).

## Rotated Sorted Array Search
```java
int search(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) return mid;
        if (nums[left] <= nums[mid]) { // left half is sorted
            if (nums[left] <= target && target < nums[mid]) right = mid - 1;
            else left = mid + 1;
        } else { // right half is sorted
            if (nums[mid] < target && target <= nums[right]) left = mid + 1;
            else right = mid - 1;
        }
    }
    return -1;
}
```
**Core insight**: at least one half (left-of-mid or right-of-mid) is *always* properly sorted in a rotated array — determine which half is sorted first, then check if target falls in that half's range.

---

# PART 11 — SORTING PATTERNS

## Sorting + Greedy
Sort by a key that exposes the greedy-optimal order, then sweep once. Example: **Non-overlapping Intervals** — sort by end time, greedily keep intervals that don't conflict with the last kept one.
```java
int eraseOverlapIntervals(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[1] - b[1]); // sort by end time
    int count = 0, prevEnd = Integer.MIN_VALUE;
    for (int[] interval : intervals) {
        if (interval[0] >= prevEnd) prevEnd = interval[1];
        else count++; // overlap, must remove this one
    }
    return count;
}
```

## Sorting + Two Pointers
3Sum, 4Sum, Container-style problems — sort first to enable the monotonic two-pointer sweep (see Part 6).

## Sorting + Binary Search
Sort, then binary-search for each query — e.g., "for each element, find how many elements are smaller" via sorted array + `lowerBound`.

## Sorting + Sweep Line
Sort events by position/time, process left to right maintaining running state — used for **Meeting Rooms**, **skyline problems**, interval overlap counting (see Part 16 for full interval pattern coverage).

## Custom Comparators in Java
```java
// Sorting objects with a lambda comparator
Arrays.sort(people, (a, b) -> a.age - b.age); // ascending by age

// Sorting by multiple keys
Arrays.sort(people, (a, b) -> {
    if (a.age != b.age) return a.age - b.age;
    return a.name.compareTo(b.name);
});

// Using Comparator utility chaining (cleaner for multi-key sorts)
Arrays.sort(people, Comparator.comparingInt((Person p) -> p.age).thenComparing(p -> p.name));

// Classic "largest number" custom string comparator
String largestNumber(int[] nums) {
    String[] strs = new String[nums.length];
    for (int i = 0; i < nums.length; i++) strs[i] = String.valueOf(nums[i]);
    Arrays.sort(strs, (a, b) -> (b + a).compareTo(a + b)); // key trick: compare concatenations
    if (strs[0].equals("0")) return "0"; // handle all-zero edge case
    StringBuilder sb = new StringBuilder();
    for (String s : strs) sb.append(s);
    return sb.toString();
}
```
**Java-specific gotcha**: `Arrays.sort(int[])` cannot take a comparator — you must box to `Integer[]` (or use a list) first if you need custom ordering on primitives. This trips people up mid-interview constantly; know it cold.

---
**End of File 3.** Continue to File 4 for String Matching, Greedy Patterns, Monotonic Stack, Heap, Interval, and Matrix Patterns (Parts 12–17).
