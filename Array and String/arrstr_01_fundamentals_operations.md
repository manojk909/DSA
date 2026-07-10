# ARRAYS & STRINGS KNOWLEDGE BASE — FILE 1
## Parts 1–4: Array Fundamentals, String Fundamentals, Complete Array Operations, Complete String Operations

---

# PART 1 — ARRAY FUNDAMENTALS

## What is an Array?
A contiguous block of memory holding elements of the same type, accessed via an index. The defining property is **contiguity** — element `i` sits at `base_address + i * element_size`, which is what makes O(1) random access possible.

```
Array: [10, 20, 30, 40, 50]
Index:   0   1   2   3   4
Memory: base, base+4, base+8, base+12, base+16   (assuming 4-byte ints)
```

## Static vs Dynamic Arrays
- **Static array** (`int[] arr = new int[5]` in Java): fixed size at creation, size baked into the object.
- **Dynamic array** (`ArrayList<Integer>` in Java): resizable wrapper around an internal static array. When capacity is exceeded, it allocates a new array (typically 1.5x–2x the size), copies elements over — this is why `ArrayList.add()` is **amortized O(1)**, not strictly O(1).

## Memory Layout, Contiguous Memory, Cache Locality
Because array elements sit next to each other in memory, iterating sequentially is cache-friendly — the CPU pre-fetches nearby memory into cache lines, so `for (int x : arr)` is dramatically faster in practice than the same number of operations on a linked list (scattered memory, cache misses on every hop). This is *why* arrays are preferred over lists whenever random access or fast iteration matters, even though both may be "O(n)" in Big-O terms for a full traversal.

**Java-specific nuance**: `int[]` is a true contiguous primitive array (fast, cache-friendly). `Integer[]` and `ArrayList<Integer>` store **references** to boxed `Integer` objects scattered on the heap — you lose contiguity and pay autoboxing overhead. **Always prefer primitive arrays (`int[]`, `long[]`, `char[]`) over boxed collections in performance-sensitive interview code.**

## Indexing & Random Access
O(1) access via `arr[i]` because the address is computed directly, not traversed. This is the single biggest advantage of arrays over linked structures.

## Insertion & Deletion
| Operation | Time | Why |
|---|---|---|
| Insert/delete at end | O(1) amortized (dynamic array) | No shifting needed |
| Insert/delete at index i | O(n) | Must shift all elements after i |
| Insert/delete at start | O(n) | Shift entire array |

```java
// Insert at index (manual shift, since Java arrays are fixed-size)
int[] insertAt(int[] arr, int index, int value) {
    int[] result = new int[arr.length + 1];
    System.arraycopy(arr, 0, result, 0, index);
    result[index] = value;
    System.arraycopy(arr, index, result, index + 1, arr.length - index);
    return result;
}
```

## Time & Space Complexity Summary

| Operation | Array (static) | ArrayList (dynamic) |
|---|---|---|
| Access by index | O(1) | O(1) |
| Search (unsorted) | O(n) | O(n) |
| Search (sorted, binary search) | O(log n) | O(log n) |
| Insert/delete at end | N/A (fixed size) | O(1) amortized |
| Insert/delete at arbitrary index | O(n) | O(n) |
| Space | O(n) | O(n), with some slack for growth |

## Multidimensional Arrays
`int[][] grid = new int[3][4];` — in Java this is actually an **array of arrays** (each row is a separate object on the heap), not a single contiguous 2D block like C++'s `int arr[3][4]`. This means:
- Row access `grid[i]` is O(1) (just a reference lookup).
- Rows **can have different lengths** — this leads directly into jagged arrays.
- Slightly worse cache locality than a true flat 2D array, but idiomatic and simple in Java.

## Jagged Arrays
Arrays of arrays where each row has a **different length**:
```java
int[][] jagged = new int[3][];
jagged[0] = new int[]{1, 2};
jagged[1] = new int[]{3, 4, 5, 6};
jagged[2] = new int[]{7};
```
Common in adjacency-list-style problems, or triangular DP tables (Pascal's Triangle).

---

# PART 2 — STRING FUNDAMENTALS

## String Representation
In Java, `String` is backed internally by a `byte[]` (since Java 9's "Compact Strings" — `byte[]` with a coder flag for Latin-1 vs UTF-16, instead of always `char[]`). Conceptually, still think of it as a **sequence of characters**.

## Character Encoding
- **ASCII**: 7-bit, 128 characters (English letters, digits, punctuation). `'a'` = 97, `'A'` = 65 — this numeric gap (`32`) is exploited constantly for case conversion tricks.
- **Unicode**: a universal character set covering virtually all scripts; a *code point* (like `U+0041` for 'A') is an abstract number, not a byte layout.
- **UTF-8**: a variable-length **encoding** of Unicode — 1 to 4 bytes per character, backward-compatible with ASCII for the first 128 codepoints. Java's internal `char` is UTF-16, so surrogate pairs matter for characters outside the Basic Multilingual Plane (emoji, some CJK) — `str.length()` counts UTF-16 code units, not visual characters, in those edge cases.

## Mutable vs Immutable Strings
**Java `String` is immutable** — every "modification" (`concat`, `substring`, `replace`) creates a **new** String object. This has major performance implications:
```java
String s = "";
for (int i = 0; i < n; i++) s += i; // O(n^2)! Each += creates a new String, copying everything so far
```
Use **`StringBuilder`** (mutable, backed by a resizable `char[]`) for any loop that builds a string incrementally:
```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < n; i++) sb.append(i); // O(n) amortized total
String result = sb.toString();
```
**This single fact — String immutability — is the #1 Java-specific interview trap** for string-heavy problems. Interviewers explicitly watch for whether you reach for `StringBuilder` in a loop.

## String Operations (Java specifics)
| Operation | Java syntax | Complexity |
|---|---|---|
| Length | `s.length()` | O(1) |
| Char at index | `s.charAt(i)` | O(1) |
| Substring | `s.substring(start, end)` | O(end-start) — Java 7+ copies, unlike old shared-backing-array versions |
| Concatenation | `s1 + s2` or `s1.concat(s2)` | O(n+m), creates new String |
| Comparison | `s1.equals(s2)` (content) vs `s1 == s2` (reference!) | O(n) for equals |
| Convert to char array | `s.toCharArray()` | O(n) |
| Convert char array to String | `new String(charArray)` or `String.valueOf(charArray)` | O(n) |
| StringBuilder append | `sb.append(x)` | O(1) amortized |
| StringBuilder reverse | `sb.reverse()` | O(n) |

**Critical Java gotcha**: `s1 == s2` compares **references**, not content, for String objects (except for interned string literals which may coincidentally match). **Always use `.equals()`** for content comparison. This trips up even experienced engineers coming from languages where `==` compares values.

## Substrings, Subsequences, Prefix, Suffix
- **Substring**: contiguous slice, e.g. `"abc"` is a substring of `"xabcy"`.
- **Subsequence**: not necessarily contiguous, but order-preserving, e.g. `"ace"` is a subsequence of `"abcde"`.
- **Prefix**: substring starting at index 0.
- **Suffix**: substring ending at the last index.

This distinction is a **constant source of confusion** and directly determines algorithm choice: substring problems often use sliding window; subsequence problems often use DP.

## Palindromes
A string equal to its own reverse. Key techniques: two-pointer check (O(n)), expand-around-center (O(n²) for all palindromic substrings), Manacher's algorithm (O(n) for longest palindromic substring — see Part 21).

## Lexicographical Order
Dictionary ordering — compare character by character, first difference decides; shorter string is "smaller" if it's a prefix of the longer one. `String.compareTo()` in Java implements exactly this.

## StringBuilder Concepts (deep dive)
Internally: a resizable `char[]` (like an ArrayList for characters) with a `count` field tracking used length. `append()` is amortized O(1) because of the same doubling-capacity strategy as ArrayList. **Always use `StringBuilder` instead of string concatenation in loops** — this is the single highest-value Java string optimization to internalize.

---

# PART 3 — COMPLETE ARRAY OPERATIONS

## Traversal
```java
for (int i = 0; i < arr.length; i++) { /* process arr[i] */ }
for (int x : arr) { /* process x, no index available */ }
```

## Insertion / Deletion
Since Java arrays are fixed-size, "insertion/deletion" typically means either (a) using `ArrayList` which handles resizing, or (b) manual shifting within a fixed array (common in in-place interview problems like "Remove Duplicates from Sorted Array").
```java
// In-place deletion pattern (two-pointer, used heavily in LeetCode "remove X" problems)
int removeElement(int[] nums, int val) {
    int k = 0; // write pointer
    for (int i = 0; i < nums.length; i++) {
        if (nums[i] != val) nums[k++] = nums[i];
    }
    return k; // new logical length
}
```

## Rotation
```java
// Rotate right by k using reversal trick — O(n) time, O(1) space
void rotate(int[] nums, int k) {
    int n = nums.length;
    k %= n;
    reverse(nums, 0, n - 1);
    reverse(nums, 0, k - 1);
    reverse(nums, k, n - 1);
}
void reverse(int[] arr, int left, int right) {
    while (left < right) {
        int tmp = arr[left]; arr[left] = arr[right]; arr[right] = tmp;
        left++; right--;
    }
}
```
**Why the reversal trick works**: reversing the whole array flips the order globally; reversing each of the two resulting segments locally restores the correct internal order within each segment — net effect is a clean rotation in O(1) extra space.

## Reversal
Standard two-pointer swap, shown above.

## Merge (two sorted arrays)
```java
int[] merge(int[] a, int[] b) {
    int[] result = new int[a.length + b.length];
    int i = 0, j = 0, k = 0;
    while (i < a.length && j < b.length) result[k++] = (a[i] <= b[j]) ? a[i++] : b[j++];
    while (i < a.length) result[k++] = a[i++];
    while (j < b.length) result[k++] = b[j++];
    return result;
}
```

## Split
Simple index-based slicing via `Arrays.copyOfRange(arr, start, end)`.

## Resize
`Arrays.copyOf(arr, newLength)` — creates a new array, copies old contents, pads with default values (0/null) if growing.

## Sorting
```java
Arrays.sort(arr); // primitives: dual-pivot Quicksort, O(n log n) avg, NOT stable
Arrays.sort(objArr); // objects: TimSort, O(n log n), STABLE
Arrays.sort(arr, (a, b) -> a - b); // custom comparator (objects/boxed types only!)
```
**Java gotcha**: `Arrays.sort(int[])` uses Quicksort (unstable, but faster, no comparator overhead); `Arrays.sort(Integer[])` or any `Collections.sort` uses TimSort (stable, needed when order of equal elements matters, e.g. stable-sort-dependent greedy problems). You **cannot** pass a custom comparator to sort primitive `int[]` directly — you must box to `Integer[]` first if custom ordering is needed.

## Searching
Linear O(n) for unsorted; `Arrays.binarySearch(arr, key)` for sorted, O(log n) — see Part 10 for the full binary search deep dive.

## Frequency Counting
```java
int[] freq = new int[26]; // for lowercase letters
for (char c : s.toCharArray()) freq[c - 'a']++;
// or, for general values:
Map<Integer, Integer> freqMap = new HashMap<>();
for (int x : arr) freqMap.merge(x, 1, Integer::sum);
```

## Prefix Sum
```java
int[] prefix = new int[n + 1]; // prefix[i] = sum of arr[0..i-1]
for (int i = 0; i < n; i++) prefix[i + 1] = prefix[i] + arr[i];
// range sum [l, r] inclusive = prefix[r+1] - prefix[l]
```

## Suffix Sum
Same idea, built from the right: `suffix[i] = arr[i] + suffix[i+1]`.

## Difference Array
Supports O(1) range-update, O(n) final reconstruction — the inverse of prefix sum.
```java
int[] diff = new int[n + 1];
void rangeUpdate(int[] diff, int l, int r, int val) {
    diff[l] += val;
    diff[r + 1] -= val;
}
// reconstruct actual array:
int[] result = new int[n];
result[0] = diff[0];
for (int i = 1; i < n; i++) result[i] = result[i-1] + diff[i];
```

## Coordinate Compression
Map large/sparse values to a dense `0..k-1` range while preserving relative order — used when actual values are huge (up to 10^9) but only relative ordering matters.
```java
int[] sorted = arr.clone();
Arrays.sort(sorted);
int[] unique = Arrays.stream(sorted).distinct().toArray();
Map<Integer, Integer> rank = new HashMap<>();
for (int i = 0; i < unique.length; i++) rank.put(unique[i], i);
// now rank.get(x) gives x's compressed index
```

## Bucket Counting
Distribute values into buckets by range for counting-sort-style processing, e.g. "sort ages 0-120" in O(n+k) instead of O(n log n).

---

# PART 4 — COMPLETE STRING OPERATIONS

## Reverse
```java
String reverse(String s) {
    return new StringBuilder(s).reverse().toString();
}
```

## Palindrome Check
```java
boolean isPalindrome(String s) {
    int l = 0, r = s.length() - 1;
    while (l < r) {
        if (s.charAt(l) != s.charAt(r)) return false;
        l++; r--;
    }
    return true;
}
```

## Anagram Check
```java
boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    int[] freq = new int[26];
    for (char c : s.toCharArray()) freq[c - 'a']++;
    for (char c : t.toCharArray()) if (--freq[c - 'a'] < 0) return false;
    return true;
}
```

## Frequency Count
See Part 3 — same pattern applied to characters.

## Character Mapping (Isomorphic Strings style)
```java
boolean isIsomorphic(String s, String t) {
    Map<Character, Character> mapST = new HashMap<>();
    Map<Character, Character> mapTS = new HashMap<>();
    for (int i = 0; i < s.length(); i++) {
        char a = s.charAt(i), b = t.charAt(i);
        if (mapST.containsKey(a) && mapST.get(a) != b) return false;
        if (mapTS.containsKey(b) && mapTS.get(b) != a) return false;
        mapST.put(a, b);
        mapTS.put(b, a);
    }
    return true;
}
```
**Why two maps**: a one-directional map alone allows two different source characters to map to the same target character, violating the bijection required by "isomorphic."

## Case Conversion
`Character.toUpperCase(c)`, `Character.toLowerCase(c)`, `s.toUpperCase()`, `s.toLowerCase()`. ASCII trick: `c ^ 32` toggles case for letters (since 'a'-'A' = 32 and this bit happens to be the differentiator) — rarely needed given built-ins, but occasionally shown as a "clever" optimization.

## Split / Trim / Replace
```java
String[] parts = s.split(","); // regex-based, careful with special regex chars
String trimmed = s.trim(); // or s.strip() (Java 11+, Unicode-aware)
String replaced = s.replace("a", "b"); // literal replace
String replacedRegex = s.replaceAll("[0-9]+", "#"); // regex replace
```

## Compression / Run-Length Encoding
```java
String compress(String s) {
    StringBuilder sb = new StringBuilder();
    int n = s.length(), i = 0;
    while (i < n) {
        char c = s.charAt(i);
        int j = i;
        while (j < n && s.charAt(j) == c) j++;
        sb.append(c).append(j - i);
        i = j;
    }
    return sb.length() < s.length() ? sb.toString() : s;
}
```

## Pattern Matching / Substring Search
Naive O(n·m), or KMP/Z-algorithm/Rabin-Karp for O(n+m) — full deep dive in Part 12.

## Rotation Check (is s2 a rotation of s1?)
```java
boolean isRotation(String s1, String s2) {
    if (s1.length() != s2.length()) return false;
    return (s1 + s1).contains(s2); // classic trick: double s1, check containment
}
```
**Why it works**: concatenating `s1` with itself contains every possible rotation of `s1` as a substring.

---
**End of File 1.** Continue to File 2 for Pattern Recognition, Two Pointers, and Sliding Window (Parts 5–7).
