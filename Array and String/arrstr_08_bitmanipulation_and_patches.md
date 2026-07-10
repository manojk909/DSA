# ARRAYS & STRINGS KNOWLEDGE BASE — FILE 8
## Gap Fill: Bit Manipulation (full pattern) + Patches to Thinner Sections

This file closes one real gap (Bit Manipulation was referenced throughout Files 1-7 but never taught) and fleshes out five sections that were covered only in prose. Read this alongside File 2 (Part 6/7), File 4 (Part 16/17), and File 5 (Part 19) — it slots into those Parts.

---

# PART 5b — BIT MANIPULATION (full deep dive)

## Core Operators & Properties
| Operator | Meaning | Key property to remember |
|---|---|---|
| `&` (AND) | 1 only if both bits are 1 | `x & 0 = 0`, `x & x = x` |
| `\|` (OR) | 1 if either bit is 1 | `x \| 0 = x`, `x \| x = x` |
| `^` (XOR) | 1 if bits differ | `x ^ x = 0`, `x ^ 0 = x`, **self-cancelling and order-independent** |
| `~` (NOT) | flips all bits | `~x = -x - 1` (two's complement) |
| `<<` (left shift) | multiply by 2^k | `x << k` |
| `>>` (right shift, signed) | divide by 2^k, sign-extends | preserves sign bit |
| `>>>` (unsigned right shift, Java-specific) | divide treating as unsigned | fills with 0, no sign extension — important for bit-counting tricks |

**The single most important property**: XOR is its own inverse and commutative/associative — `a ^ b ^ a = b`. This is *the* mechanism behind nearly every "find the unique element" trick.

## Common Bit Tricks
```java
boolean isPowerOfTwo(int n) {
    return n > 0 && (n & (n - 1)) == 0; // power of 2 has exactly one set bit; n-1 flips it and everything below
}

int countSetBits(int n) { // Brian Kernighan's algorithm
    int count = 0;
    while (n != 0) {
        n &= (n - 1); // clears the lowest set bit each iteration
        count++;
    }
    return count; // O(number of set bits), not O(32)
}

int lowestSetBit(int n) {
    return n & (-n); // isolates the lowest set bit, e.g. 12 (1100) -> 4 (0100)
}

boolean getBit(int n, int i) { return (n & (1 << i)) != 0; }
int setBit(int n, int i)   { return n | (1 << i); }
int clearBit(int n, int i) { return n & ~(1 << i); }
int toggleBit(int n, int i){ return n ^ (1 << i); }
```
**Why `n & (n-1)` clears the lowest set bit**: subtracting 1 flips every bit from the lowest set bit down to bit 0 (borrow propagation); ANDing with the original cancels exactly that lowest set bit while leaving everything above it untouched.

## Single Number (XOR trick)
```java
// Every element appears twice except one — find it in O(n) time, O(1) space
int singleNumber(int[] nums) {
    int result = 0;
    for (int x : nums) result ^= x;
    return result; // all paired elements cancel to 0 via a^a=0; only the unique one survives
}
```
**Why it works**: XOR-ing the whole array cancels every value that appears an even number of times (since `a^a=0` and XOR is commutative/associative, pairs vanish regardless of order), leaving only the value that appears an odd number of times.

## Single Number II / III (recognition, not full derivation)
- **Single Number II** (every element appears **3** times except one): plain XOR breaks down (cancels in pairs, not triples) — solved by tracking bit counts mod 3 per bit position, or the classic `ones`/`twos` bitmask state-machine trick.
- **Single Number III** (**two** unique elements, everything else appears twice): XOR all elements to get `a^b` (the two uniques XORed together), find any set bit in that result to partition the array into two groups (one containing `a`, the other `b`), then XOR each group independently.
**Recognition clue**: "every element appears K times except one/two" is the signature phrasing for this whole family.

## Counting Bits (DP + bit trick combined)
```java
// For every number 0..n, count its set bits, in O(n) total (not O(n log n))
int[] countBits(int n) {
    int[] dp = new int[n + 1];
    for (int i = 1; i <= n; i++) {
        dp[i] = dp[i >> 1] + (i & 1); // dp[i] = bits(i/2) + (last bit of i)
    }
    return dp;
}
```
**Why the recurrence works**: shifting `i` right by 1 drops the last bit; `dp[i>>1]` already has the count for everything except that dropped bit, so add it back via `(i & 1)`.

## Subsets via Bitmask
```java
// Generate all 2^n subsets by treating each integer 0..2^n-1 as a "which elements are included" mask
List<List<Integer>> subsets(int[] nums) {
    int n = nums.length;
    List<List<Integer>> result = new ArrayList<>();
    for (int mask = 0; mask < (1 << n); mask++) {
        List<Integer> subset = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if ((mask & (1 << i)) != 0) subset.add(nums[i]);
        }
        result.add(subset);
    }
    return result;
}
```
**Time**: O(n · 2^n) — only viable when `n ≤ ~20`. This same bitmask-as-a-subset idea is the foundation of **bitmask DP** (`dp[mask]` = best value achievable using exactly the subset `mask`) used in Traveling Salesman / Hamiltonian Path problems — see the Graph Knowledge Base, Part 16, for that extension.

## Recognition Checklist
| Clue phrase | → Technique |
|---|---|
| "every element appears twice except one" | XOR (Single Number) |
| "every element appears three times except one" | Bit-count-mod-3 state machine |
| "two elements appear once, rest twice" | XOR + partition by a set bit |
| "count set bits for every number up to n" | DP + `i & 1` recurrence |
| "generate all subsets," n ≤ ~20 | Bitmask enumeration |
| "check power of two" | `n & (n-1) == 0` |
| "toggle/set/clear a specific bit" | Shift + AND/OR/XOR |
| state includes "visited subset" and n ≤ ~20 | Bitmask DP |

---

# PATCH — Part 6 (Two Pointers): Dry Run & Visualization

## Opposite-Direction Two Pointers — Traced Example
Array (sorted): `[2, 7, 11, 15]`, target = `18`
```
l=0(val=2)                          r=3(val=15)
[ 2,   7,  11,  15 ]
  ^                 ^
sum = 2+15 = 17 < 18  -> move l right (need a bigger sum)

l=1(val=7)                          r=3(val=15)
[ 2,   7,  11,  15 ]
       ^            ^
sum = 7+15 = 22 > 18  -> move r left (need a smaller sum)

l=1(val=7)             r=2(val=11)
[ 2,   7,  11,  15 ]
       ^        ^
sum = 7+11 = 18 == 18  -> FOUND, return indices [1,2]
```
**Why this converges correctly and never misses the answer**: at each step, exactly one direction of movement could possibly fix the current sum's over/undershoot, because the array is sorted — moving the "wrong" pointer would only push the sum further from the target, never closer. This is the monotonicity argument that justifies collapsing an O(n²) pair search into O(n).

---

# PATCH — Part 6 (K-Sum): Full 4Sum Code

```java
List<List<Integer>> fourSum(int[] nums, int target) {
    Arrays.sort(nums);
    List<List<Integer>> result = new ArrayList<>();
    int n = nums.length;
    for (int i = 0; i < n - 3; i++) {
        if (i > 0 && nums[i] == nums[i - 1]) continue; // skip duplicate 1st anchor
        for (int j = i + 1; j < n - 2; j++) {
            if (j > i + 1 && nums[j] == nums[j - 1]) continue; // skip duplicate 2nd anchor
            int l = j + 1, r = n - 1;
            while (l < r) {
                long sum = (long) nums[i] + nums[j] + nums[l] + nums[r]; // long: avoid overflow
                if (sum == target) {
                    result.add(Arrays.asList(nums[i], nums[j], nums[l], nums[r]));
                    while (l < r && nums[l] == nums[l + 1]) l++;
                    while (l < r && nums[r] == nums[r - 1]) r--;
                    l++; r--;
                } else if (sum < target) l++;
                else r--;
            }
        }
    }
    return result;
}
```
**Pattern generalization**: K-Sum = fix `(K-2)` anchors via nested loops (each skipping duplicates the same way), then run the same two-pointer 2Sum sweep on the remaining sorted subarray. This is exactly how you'd derive 5Sum, 6Sum, etc. under interview pressure — one more nested loop each time, same terminal two-pointer base case.

---

# PATCH — Part 7 (Sliding Window): Distinct Elements / At-Most-K-Distinct, Full Code

```java
// Longest Substring with At Most K Distinct Characters
int lengthOfLongestSubstringKDistinct(String s, int k) {
    Map<Character, Integer> freq = new HashMap<>();
    int left = 0, best = 0;
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        freq.merge(c, 1, Integer::sum);
        while (freq.size() > k) { // window invalid: too many distinct characters
            char lc = s.charAt(left);
            freq.put(lc, freq.get(lc) - 1);
            if (freq.get(lc) == 0) freq.remove(lc);
            left++;
        }
        best = Math.max(best, right - left + 1);
    }
    return best;
}
```
**Why this is the "maximum window" template applied directly**: expand `right` unconditionally, and the `while` loop is the exact "shrink while invalid" clause from the general maximum-window template in File 2 Part 7 — the only thing that changes between problems in this family is the definition of "invalid" (`freq.size() > k` here, vs. "contains a duplicate character" for Longest Substring Without Repeating Characters).

---

# PATCH — Part 16 (Interval Patterns): Insert Interval, Full Code

```java
int[][] insert(int[][] intervals, int[] newInterval) {
    List<int[]> result = new ArrayList<>();
    int i = 0, n = intervals.length;

    // Phase 1: add all intervals ending strictly before newInterval starts
    while (i < n && intervals[i][1] < newInterval[0]) {
        result.add(intervals[i]);
        i++;
    }

    // Phase 2: merge all intervals that overlap with newInterval
    while (i < n && intervals[i][0] <= newInterval[1]) {
        newInterval[0] = Math.min(newInterval[0], intervals[i][0]);
        newInterval[1] = Math.max(newInterval[1], intervals[i][1]);
        i++;
    }
    result.add(newInterval);

    // Phase 3: add all remaining intervals (start strictly after newInterval ends)
    while (i < n) {
        result.add(intervals[i]);
        i++;
    }

    return result.toArray(new int[0][]);
}
```
**Why three clean phases instead of a general merge-and-sort**: the input is already sorted and non-overlapping, so you never need to re-sort — a single linear pass classifying each interval as "before," "overlapping," or "after" relative to `newInterval` is sufficient, giving O(n) instead of the O(n log n) you'd pay by appending and re-running general Merge Intervals.

---

# PATCH — Part 17 (Matrix Patterns): Diagonal Traversal & Simulation, Full Code

## Diagonal Traversal
```java
// Diagonal Traverse: zigzag across "/" diagonals (LeetCode 498 style)
int[] findDiagonalOrder(int[][] mat) {
    int rows = mat.length, cols = mat[0].length;
    int[] result = new int[rows * cols];
    int idx = 0;
    for (int d = 0; d < rows + cols - 1; d++) {
        List<Integer> diagonal = new ArrayList<>();
        int r = (d < cols) ? 0 : d - cols + 1;
        int c = (d < cols) ? d : cols - 1;
        while (r < rows && c >= 0) {
            diagonal.add(mat[r][c]);
            r++; c--;
        }
        if (d % 2 == 0) Collections.reverse(diagonal); // alternate direction per diagonal
        for (int val : diagonal) result[idx++] = val;
    }
    return result;
}
```
**Why `r + c` is constant along a "/" diagonal**: moving one step right (`c+1`) and one step down (`r-1` relatively, i.e. going the other way `r+1,c-1`) both preserve `r+c` — grouping cells by that invariant is exactly how you enumerate diagonals without tracking two separate loop variables per diagonal.

## Simulation — Game of Life (in-place state encoding trick)
```java
// Encode both old and new state in each cell using extra bits, avoiding a full copy
void gameOfLife(int[][] board) {
    int rows = board.length, cols = board[0].length;
    int[] dr = {-1,-1,-1,0,0,1,1,1}, dc = {-1,0,1,-1,1,-1,0,1};

    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            int liveNeighbors = 0;
            for (int d = 0; d < 8; d++) {
                int nr = r + dr[d], nc = c + dc[d];
                if (nr >= 0 && nr < rows && nc >= 0 && nc < cols) {
                    liveNeighbors += (board[nr][nc] & 1); // check ORIGINAL state (bit 0)
                }
            }
            // Rules of Life, writing the NEW state into bit 1, leaving bit 0 (old state) untouched
            if ((board[r][c] & 1) == 1 && (liveNeighbors == 2 || liveNeighbors == 3)) {
                board[r][c] |= 2; // stays alive
            } else if ((board[r][c] & 1) == 0 && liveNeighbors == 3) {
                board[r][c] |= 2; // becomes alive
            }
        }
    }
    for (int r = 0; r < rows; r++)
        for (int c = 0; c < cols; c++)
            board[r][c] >>= 1; // shift new state (bit 1) down into bit 0
}
```
**Why this needs a trick at all**: naively updating `board[r][c]` in place would corrupt neighbor calculations for cells processed later in the same pass (they'd read *already-updated* neighbors instead of the original board). Encoding old state in bit 0 and new state in bit 1 lets every cell's neighbor-check always read the untouched original value, achieving O(1) extra space instead of allocating a full second grid.

---
**End of File 8.** This closes the Bit Manipulation gap and the five thin spots identified in the audit. Everything else from the original 30-part spec was already complete across Files 1–7.
