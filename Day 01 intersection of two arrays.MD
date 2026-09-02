# Day 1 — Intersection of Two Arrays

**LeetCode 349** · Easy
**Topics:** hash set · array traversal · deduplication · boxing vs primitives

---

## Problem

Given two integer arrays `nums1` and `nums2`, return an array of their intersection. Each element in the result must be unique, and the result may be in any order.

```
nums1 = [1,2,2,1], nums2 = [2,2]        ->  [2]
nums1 = [4,9,5],   nums2 = [9,4,9,8,4]  ->  [9,4]   ([4,9] also accepted)
```

**Constraints**

- `1 <= nums1.length, nums2.length <= 1000`
- `0 <= nums1[i], nums2[i] <= 1000`

---

## Questions to Ask First

- Does the output need to be sorted, or is any order fine? (Here: any order.)
- Can the inputs contain duplicates? (Yes — and the output must not.)
- Is the value range bounded? A small bounded range opens up a faster array-based approach instead of hashing.
- Can either array be empty or `null`? (Constraints say length ≥ 1, but ask anyway.)
- Should I optimise for time or space if they conflict?

---

## Solution

```java
package algodaily;

import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

public class IntersectionOfTwoArrays {

    public int[] intersection(int[] nums1, int[] nums2) {
        Set<Integer> set = new HashSet<>();
        for (int n : nums1) {
            set.add(n);
        }

        List<Integer> list = new ArrayList<>();
        for (int n : nums2) {
            if (set.contains(n)) {
                list.add(n);
                set.remove(n);   // prevents duplicates in the output
            }
        }

        int[] result = new int[list.size()];
        for (int i = 0; i < list.size(); i++) {
            result[i] = list.get(i);
        }
        return result;
    }
}
```

Dump `nums1` into a set (which dedups it for free). Walk `nums2`; on a hit, record the value and immediately remove it from the set so a second occurrence in `nums2` can't match again.

```
nums1 = [4,9,5]  ->  set = {4,5,9}

nums2:  9  hit  -> list=[9]    set={4,5}
        4  hit  -> list=[9,4]  set={5}
        9  miss (already removed)
        8  miss
        4  miss (already removed)
```

---

## Edge Cases

| Input                             | Result     | Why                                                   |
| --------------------------------- | ---------- | ----------------------------------------------------- |
| No overlap: `[1,2]`, `[3,4]`      | `[]` ✅    | Empty list → zero-length array, which is valid        |
| Duplicates in `nums2`             | ✅         | `set.remove(n)` makes the second hit a miss           |
| Duplicates in `nums1`             | ✅         | `HashSet` silently drops them on insert               |
| Identical arrays `[1,2]`, `[1,2]` | `[1,2]` ✅ |                                                       |
| Value `0` present                 | ✅         | No sentinel-value assumption anywhere                 |
| Single element each               | ✅         |                                                       |
| `null` input                      | ❌ NPE     | Excluded by constraints, but the for-each would throw |

---

## Complexity

Let `n = nums1.length`, `m = nums2.length`.

**Time — `O(n + m)`**

Building the set is `n` insertions. Scanning `nums2` is `m` iterations, each doing a `contains` and possibly a `remove` — both `O(1)` average for a hash set. Copying the list to an array is at most `min(n, m)`, which is dominated by the two main loops.

Worst case is technically `O(n · m)` if every hash collided, but with `Integer` keys and Java's spreading function that doesn't happen in practice.

**Space — `O(n)`**

The set holds up to `n` distinct values, the list up to `min(n, m)`. Both scale with input, so `O(n)`.

Note the set stores `Integer` objects, not `int`. Each is a heap-allocated box with an object header — meaningfully more memory than 4 bytes per value. Java caches `Integer` objects from −128 to 127, so small values reuse instances, but anything above 127 allocates fresh.

---

## Notes

**The bug: `list.toArray()` doesn't compile.**

`ArrayList<Integer>.toArray()` is declared to return `Object[]`, and Java has no implicit conversion from `Object[]` to `int[]`. Even `list.toArray(new Integer[0])` gives you `Integer[]`, which is still not `int[]` — arrays are not covariant across the primitive/wrapper divide, so no cast rescues it.

Three ways out:

```java
// 1. Manual loop — clearest, no hidden cost
int[] result = new int[list.size()];
for (int i = 0; i < list.size(); i++) result[i] = list.get(i);

// 2. Stream — concise, slight overhead
return list.stream().mapToInt(Integer::intValue).toArray();

// 3. Skip the List entirely — write straight into a right-sized array
```

**Missing imports.** LeetCode preloads `java.util.*`, so this compiles there but not in your repo. Worth adding them every time so the file actually runs locally.

**Better approach for this input range.** Constraints cap values at 1000, so a fixed boolean array beats hashing — no boxing, no hash computation, better cache behaviour:

```java
public int[] intersection(int[] nums1, int[] nums2) {
    boolean[] seen = new boolean[1001];
    for (int n : nums1) seen[n] = true;

    int[] buffer = new int[1001];
    int count = 0;
    for (int n : nums2) {
        if (seen[n]) {
            buffer[count++] = n;
            seen[n] = false;
        }
    }
    return Arrays.copyOf(buffer, count);
}
```

Same `O(n + m)` time, but space is `O(1)` — 1001 booleans regardless of input size. Whenever you see a tight bound on _values_ (not just length), consider indexing by value instead of hashing.

**Iterate the smaller array first.** Building the set from whichever array is shorter keeps the set smaller. Doesn't change the Big-O, but it's a free win worth mentioning aloud.

**Pattern to reuse.** "Set for membership, remove-on-consume for dedup" also solves: Intersection of Two Arrays II (use a frequency map instead), Happy Number (cycle detection), Contains Duplicate, Longest Consecutive Sequence.
