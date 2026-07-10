# ARRAYS & STRINGS KNOWLEDGE BASE — FILE 6
## Parts 23–24: LeetCode Pattern Grouping, Complete Java Template Library

---

# PART 23 — LEETCODE PATTERN GROUPING

| Pattern | Recognition Clues | Difficulty Range | Algorithms | Must-Know Problems |
|---|---|---|---|---|
| Two Pointers | sorted array, pair/triplet sum, in-place partition | Easy–Medium | Opposite/same-direction pointers | Two Sum II, 3Sum, Container With Most Water, Trapping Rain Water, Remove Duplicates from Sorted Array |
| Sliding Window | contiguous subarray/substring, size or property constraint | Medium | Fixed/variable window | Longest Substring Without Repeating Characters, Minimum Window Substring, Longest Repeating Character Replacement, Sliding Window Maximum |
| Hashing | fast lookup, frequency, grouping | Easy–Medium | HashMap/HashSet | Two Sum, Group Anagrams, Longest Consecutive Sequence, Subarray Sum Equals K |
| Prefix Sum | range sum queries, subarray sum conditions | Easy–Medium | 1D/2D prefix sum | Range Sum Query, Subarray Sum Equals K, Product of Array Except Self, Continuous Subarray Sum |
| Binary Search | sorted array, search space reduction, "minimize max/maximize min" | Easy–Hard | Classic BS, BS on answer | Binary Search, Search in Rotated Sorted Array, Koko Eating Bananas, Split Array Largest Sum, Find Peak Element |
| Sorting-based | order doesn't matter, enables greedy/two-pointer | Easy–Medium | Sort + sweep/greedy/two-pointer | Merge Intervals, Non-overlapping Intervals, Largest Number |
| Monotonic Stack | next/previous greater/smaller | Medium–Hard | Increasing/decreasing stack | Daily Temperatures, Next Greater Element, Largest Rectangle in Histogram, Trapping Rain Water (stack variant) |
| Heap | top-K, merge-K, running median | Medium–Hard | Min/max heap, two-heap | Kth Largest Element, Top K Frequent Elements, Merge K Sorted Lists, Find Median from Data Stream |
| Intervals | overlapping ranges, scheduling | Medium | Sort + sweep | Merge Intervals, Insert Interval, Meeting Rooms I/II, Non-overlapping Intervals |
| Matrix | 2D grid traversal/transformation | Medium | Direct simulation, transpose tricks | Rotate Image, Spiral Matrix, Set Matrix Zeroes, Search a 2D Matrix |
| String DP | longest common/edit distance/segmentation | Medium–Hard | 2D/1D DP over string indices | Longest Common Subsequence, Edit Distance, Word Break, Longest Palindromic Substring |
| String Matching | find pattern occurrences | Medium–Hard | KMP, Z, Rabin-Karp | Implement strStr(), Repeated Substring Pattern, Shortest Palindrome |
| Greedy | local optimal choice, provable global optimum | Medium | Sort + greedy sweep | Jump Game, Gas Station, Candy, Partition Labels |
| Bit Manipulation | XOR tricks, subsets, single number | Easy–Medium | Bitmasking | Single Number, Counting Bits, Subsets via bitmask |
| Trie | prefix lookups, word search | Medium–Hard | Trie structure | Implement Trie, Word Search II, Design Add and Search Words |

## Curated List Cross-Reference
- **Blind 75 (Arrays/Strings)**: Two Sum, Best Time to Buy/Sell Stock, Contains Duplicate, Product of Array Except Self, Maximum Subarray, Maximum Product Subarray, Find Minimum in Rotated Sorted Array, Search in Rotated Sorted Array, 3Sum, Container With Most Water, Longest Substring Without Repeating Characters, Longest Repeating Character Replacement, Minimum Window Substring, Valid Anagram, Group Anagrams, Valid Palindrome, Longest Palindromic Substring.
- **NeetCode 150 additions**: Two Sum II, 3Sum Closest, Merge Intervals, Insert Interval, Non-overlapping Intervals, Meeting Rooms I/II, Gas Station, Word Break, Longest Increasing Subsequence, Trapping Rain Water, Sliding Window Maximum.
- **Top Interview 150**: emphasizes Rotate Array, Jump Game, H-Index, Insert Delete GetRandom O(1), Product of Array Except Self, Gas Station, Candy, Trapping Rain Water.
- **FAANG-frequent**: Two Sum, Longest Substring Without Repeating Characters, 3Sum, Merge Intervals, Product of Array Except Self, Group Anagrams, Trapping Rain Water, Minimum Window Substring — these repeat constantly across Google/Meta/Amazon/Microsoft loops.

---

# PART 24 — COMPLETE JAVA TEMPLATE LIBRARY

```java
import java.util.*;

public class ArrayStringTemplates {

    // ===================== Two Pointers (opposite direction) =====================
    static int[] twoPointerOpposite(int[] sortedArr, int target) {
        int l = 0, r = sortedArr.length - 1;
        while (l < r) {
            int sum = sortedArr[l] + sortedArr[r];
            if (sum == target) return new int[]{l, r};
            else if (sum < target) l++;
            else r--;
        }
        return new int[]{-1, -1};
    }

    // ===================== Two Pointers (same direction, in-place) =====================
    static int twoPointerSameDirection(int[] nums, int val) {
        int slow = 0;
        for (int fast = 0; fast < nums.length; fast++) {
            if (nums[fast] != val) nums[slow++] = nums[fast];
        }
        return slow;
    }

    // ===================== Sliding Window (Maximum / longest valid) =====================
    static int maxSlidingWindowTemplate(String s) {
        Set<Character> window = new HashSet<>();
        int left = 0, best = 0;
        for (int right = 0; right < s.length(); right++) {
            char c = s.charAt(right);
            while (window.contains(c)) { window.remove(s.charAt(left)); left++; }
            window.add(c);
            best = Math.max(best, right - left + 1);
        }
        return best;
    }

    // ===================== Sliding Window (Minimum / shortest valid) =====================
    static int minSlidingWindowTemplate(int[] nums, int target) {
        int left = 0, sum = 0, best = Integer.MAX_VALUE;
        for (int right = 0; right < nums.length; right++) {
            sum += nums[right];
            while (sum >= target) {
                best = Math.min(best, right - left + 1);
                sum -= nums[left++];
            }
        }
        return best == Integer.MAX_VALUE ? 0 : best;
    }

    // ===================== Binary Search (classic) =====================
    static int binarySearch(int[] arr, int target) {
        int lo = 0, hi = arr.length - 1;
        while (lo <= hi) {
            int mid = lo + (hi - lo) / 2;
            if (arr[mid] == target) return mid;
            else if (arr[mid] < target) lo = mid + 1;
            else hi = mid - 1;
        }
        return -1;
    }

    // ===================== Binary Search (lower bound) =====================
    static int lowerBound(int[] arr, int target) {
        int lo = 0, hi = arr.length;
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (arr[mid] < target) lo = mid + 1; else hi = mid;
        }
        return lo;
    }

    // ===================== Binary Search on Answer =====================
    static int binarySearchOnAnswer(int lo, int hi, java.util.function.IntPredicate canAchieve) {
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (canAchieve.test(mid)) hi = mid; else lo = mid + 1;
        }
        return lo;
    }

    // ===================== Prefix Sum =====================
    static int[] buildPrefixSum(int[] arr) {
        int[] prefix = new int[arr.length + 1];
        for (int i = 0; i < arr.length; i++) prefix[i + 1] = prefix[i] + arr[i];
        return prefix;
    }
    static int rangeSum(int[] prefix, int l, int r) { return prefix[r + 1] - prefix[l]; }

    // ===================== Difference Array =====================
    static void rangeUpdate(int[] diff, int l, int r, int val) {
        diff[l] += val;
        if (r + 1 < diff.length) diff[r + 1] -= val;
    }
    static int[] reconstructFromDiff(int[] diff) {
        int[] result = new int[diff.length];
        result[0] = diff[0];
        for (int i = 1; i < diff.length; i++) result[i] = result[i - 1] + diff[i];
        return result;
    }

    // ===================== Frequency Map (bounded alphabet) =====================
    static int[] buildFrequency(String s) {
        int[] freq = new int[26];
        for (char c : s.toCharArray()) freq[c - 'a']++;
        return freq;
    }

    // ===================== Frequency Map (general, HashMap) =====================
    static <T> Map<T, Integer> buildFrequencyMap(T[] arr) {
        Map<T, Integer> freq = new HashMap<>();
        for (T x : arr) freq.merge(x, 1, Integer::sum);
        return freq;
    }

    // ===================== Sorting with Custom Comparator =====================
    static void sortByCustomKey(Integer[] arr) {
        Arrays.sort(arr, (a, b) -> a - b); // ascending; swap for descending
    }
    static void sortMultiKey(int[][] intervals) {
        Arrays.sort(intervals, (a, b) -> a[0] != b[0] ? a[0] - b[0] : a[1] - b[1]);
    }

    // ===================== Monotonic Stack (Next Greater Element) =====================
    static int[] nextGreaterElement(int[] nums) {
        int n = nums.length;
        int[] result = new int[n];
        Arrays.fill(result, -1);
        Deque<Integer> stack = new ArrayDeque<>();
        for (int i = 0; i < n; i++) {
            while (!stack.isEmpty() && nums[stack.peek()] < nums[i]) result[stack.pop()] = nums[i];
            stack.push(i);
        }
        return result;
    }

    // ===================== Monotonic Deque (Sliding Window Max) =====================
    static int[] slidingWindowMax(int[] nums, int k) {
        Deque<Integer> deque = new ArrayDeque<>();
        int[] result = new int[nums.length - k + 1];
        for (int i = 0; i < nums.length; i++) {
            while (!deque.isEmpty() && deque.peekFirst() <= i - k) deque.pollFirst();
            while (!deque.isEmpty() && nums[deque.peekLast()] < nums[i]) deque.pollLast();
            deque.offerLast(i);
            if (i >= k - 1) result[i - k + 1] = nums[deque.peekFirst()];
        }
        return result;
    }

    // ===================== Priority Queue (Top-K, min-heap capped at size k) =====================
    static int[] topKElements(int[] nums, int k) {
        PriorityQueue<Integer> heap = new PriorityQueue<>(); // min-heap
        for (int x : nums) {
            heap.offer(x);
            if (heap.size() > k) heap.poll();
        }
        int[] result = new int[k];
        for (int i = k - 1; i >= 0; i--) result[i] = heap.poll();
        return result;
    }

    // ===================== Priority Queue (Max-Heap) =====================
    static PriorityQueue<Integer> maxHeap() {
        return new PriorityQueue<>(Collections.reverseOrder());
    }

    // ===================== Deque (generic double-ended usage) =====================
    static void dequeExample() {
        Deque<Integer> deque = new ArrayDeque<>();
        deque.offerFirst(1);
        deque.offerLast(2);
        deque.pollFirst();
        deque.pollLast();
    }

    // ===================== KMP Failure Function + Search =====================
    static int[] buildFailureFunction(String pattern) {
        int m = pattern.length();
        int[] fail = new int[m];
        int len = 0;
        for (int i = 1; i < m; i++) {
            while (len > 0 && pattern.charAt(i) != pattern.charAt(len)) len = fail[len - 1];
            if (pattern.charAt(i) == pattern.charAt(len)) len++;
            fail[i] = len;
        }
        return fail;
    }
    static List<Integer> kmpSearch(String text, String pattern) {
        List<Integer> result = new ArrayList<>();
        if (pattern.isEmpty()) return result;
        int[] fail = buildFailureFunction(pattern);
        int j = 0;
        for (int i = 0; i < text.length(); i++) {
            while (j > 0 && text.charAt(i) != pattern.charAt(j)) j = fail[j - 1];
            if (text.charAt(i) == pattern.charAt(j)) j++;
            if (j == pattern.length()) { result.add(i - j + 1); j = fail[j - 1]; }
        }
        return result;
    }

    // ===================== Z-function =====================
    static int[] zFunction(String s) {
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

    // ===================== DSU (frequently paired with array problems too) =====================
    static class DSU {
        int[] parent, rank_;
        DSU(int n) {
            parent = new int[n];
            rank_ = new int[n];
            for (int i = 0; i < n; i++) parent[i] = i;
        }
        int find(int x) { return parent[x] == x ? x : (parent[x] = find(parent[x])); }
        void union(int x, int y) {
            int rx = find(x), ry = find(y);
            if (rx == ry) return;
            if (rank_[rx] < rank_[ry]) { int t = rx; rx = ry; ry = t; }
            parent[ry] = rx;
            if (rank_[rx] == rank_[ry]) rank_[rx]++;
        }
    }

    // ===================== Trie =====================
    static class TrieNode {
        TrieNode[] children = new TrieNode[26];
        boolean isEnd = false;
    }
    static class Trie {
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
}
```

## Java-Specific Performance Notes (apply across all templates above)
- Prefer `int[]`/`char[]` frequency arrays over `HashMap<Character,Integer>` whenever the alphabet is bounded (26 letters, ASCII) — faster and avoids boxing.
- Prefer `StringBuilder` over `String` concatenation inside any loop — this is the single highest-value Java string optimization (File 1, Part 2).
- `ArrayDeque` outperforms both `Stack` (legacy, synchronized) and `LinkedList` for stack/queue/deque use in almost all interview and competitive scenarios.
- Use `PriorityQueue<int[]>` (or packed `long[]`) instead of custom wrapper objects to reduce allocation and GC overhead in heap-heavy problems (Top-K, Merge-K, Median).
- `Arrays.sort(int[])` is unstable (dual-pivot quicksort); box to `Integer[]` and use `Arrays.sort(Integer[], Comparator)` (TimSort, stable) whenever custom ordering or stability is required.
- Avoid repeated `substring()` calls inside tight loops when only comparing characters — index directly with `charAt()` instead, since `substring()` allocates a new String each call (post-Java 7).
