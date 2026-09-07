# Day 6 — Majority Element

**LeetCode 169** · Easy
**Topics:** array · hash table · sorting · bit manipulation · Boyer–Moore majority vote

---

## Problem

Given an array `nums` of size `n`, return the element that appears more than `⌊n/2⌋` times.

```
[3,2,3]          ->  3
[2,2,1,1,1,2,2]  ->  2
```

**Constraints**
- `1 <= n <= 5 * 10^4`
- `-10^9 <= nums[i] <= 10^9`
- A majority element is guaranteed to exist

**Follow-up:** linear time, `O(1)` space?

---

## Questions to Ask First

- Does exactly `n/2` occurrences count? (No — strictly more than.)
- What to return if no majority exists? (Guaranteed here. Otherwise: `-1`, or a sentinel.)
- Can I mutate the input? (Determines whether sorting in place is allowed.)
- Is the value range bounded? (No — `±10^9` rules out direct addressing.)

---

## Solutions

### 1 — Brute force

```java
public int majorityElement(int[] nums) {
    int n = nums.length;
    for (int i = 0; i < n; i++) {
        int count = 0;
        for (int j = 0; j < n; j++) {
            if (nums[i] == nums[j]) count++;
        }
        if (count > n / 2) return nums[i];
    }
    return -1;
}
```

**Time `O(n²)` · Space `O(1)`**

---

### 2 — Sorting

```java
public int majorityElement(int[] nums) {
    Arrays.sort(nums);
    return nums[nums.length / 2];
}
```

**Time `O(n log n)` · Space `O(1)` for primitive `int[]` (dual-pivot quicksort in place)**

An element occupying more than half the slots must cover the middle index no matter where its run starts. One line, no counting loop needed.

---

### 3 — Hash map

```java
public int majorityElement(int[] nums) {
    Map<Integer, Integer> counts = new HashMap<>();
    for (int num : nums) {
        int c = counts.merge(num, 1, Integer::sum);
        if (c > nums.length / 2) return num;
    }
    return -1;
}
```

**Time `O(n)` · Space `O(n)`**

Checking inside the loop lets it exit early instead of counting everything first.

---

### 4 — Bit manipulation

```java
public int majorityElement(int[] nums) {
    int n = nums.length;
    int result = 0;

    for (int bit = 0; bit < 32; bit++) {
        int count = 0;
        for (int num : nums) {
            if ((num & (1 << bit)) != 0) count++;
        }
        if (count > n / 2) result |= (1 << bit);
    }
    return result;
}
```

**Time `O(32n)` = `O(n)` · Space `O(1)`**

If a value appears more than `n/2` times, then at every bit position where it has a `1`, that bit is set in more than half the array. Rebuild the number bit by bit from the majority vote at each position.

---

### 5 — Boyer–Moore majority vote ← the follow-up answer

```java
package algodaily;

public class MajorityElement {

    public int majorityElement(int[] nums) {
        int candidate = nums[0];
        int count = 0;

        for (int num : nums) {
            if (count == 0) {
                candidate = num;
                count = 1;
            } else if (num == candidate) {
                count++;
            } else {
                count--;
            }
        }
        return candidate;
    }
}
```

**Time `O(n)` · Space `O(1)`**

---

## Why Boyer–Moore Works

**The intuition: pair off and cancel.**

Think of each element as a vote. Whenever you see two *different* values, cancel them against each other — both are removed. Keep doing this until nothing else can be cancelled.

The majority element appears more than `n/2` times, which means it appears more often than **everything else combined**. So every cancellation destroys at most one majority occurrence, and the non-majority supply runs out first. Whatever survives must be the majority element.

`count` is the running tally of "uncancelled" copies of the current candidate:

- `num == candidate` → one more uncancelled copy → `count++`
- `num != candidate` → this element cancels one copy → `count--`
- `count == 0` → everything so far has cancelled out → adopt the new element as candidate

**Why resetting the candidate is safe.**

When `count` hits zero at index `i`, the prefix `nums[0..i]` contains exactly as many copies of the old candidate as of everything else combined. That prefix is internally balanced — it contributes nothing either way.

Since the true majority had a surplus over the whole array, and the discarded prefix is balanced, the surplus must still live in the remaining suffix. So the majority element of the whole array is still the majority of what's left. Discarding balanced prefixes never loses it.

**Trace `[2,2,1,1,1,2,2]`:**

```
num=2  count=0  -> candidate=2, count=1
num=2  match    -> count=2
num=1  differ   -> count=1
num=1  differ   -> count=0
num=1  count=0  -> candidate=1, count=1
num=2  differ   -> count=0
num=2  count=0  -> candidate=2, count=1
-> 2 ✓
```

Note the candidate was briefly `1` — a wrong guess. That's fine. The algorithm doesn't need every intermediate candidate to be right; it only guarantees the **final** one is, when a majority exists.

---

## If a Majority Element Isn't Guaranteed

Boyer–Moore always produces *some* candidate, but with no majority that candidate is meaningless — it's just whatever happened to survive the cancellations, not even necessarily the most frequent element.

`[1,2,3]` returns `3`, which appears once.

**Fix: add a verification pass.**

```java
public int majorityElement(int[] nums) {
    int candidate = nums[0];
    int count = 0;

    for (int num : nums) {
        if (count == 0) { candidate = num; count = 1; }
        else if (num == candidate) count++;
        else count--;
    }

    count = 0;
    for (int num : nums) {
        if (num == candidate) count++;
    }
    return count > nums.length / 2 ? candidate : -1;
}
```

Still `O(n)` time and `O(1)` space — just two passes instead of one.

**What else changes:**

| Approach | With guarantee | Without |
|---|---|---|
| Boyer–Moore | one pass | needs verification pass |
| Sorting | `return nums[n/2]` | must count the run at `n/2` |
| Bit manipulation | rebuild and return | must verify — bits can assemble a value not in the array |
| Hash map | unchanged | unchanged (already checks `> n/2`) |

The hash map is the only one that needs no change. The three `O(1)`-space approaches all *derive* an answer rather than counting it directly, so they all need confirming.

---

## Edge Cases

| Input | Result | Why |
|---|---|---|
| `[1]` | `1` ✅ | Single element is trivially majority |
| `[1,1]` | `1` ✅ | |
| `[2,2,1,1,1,2,2]` | `2` ✅ | Candidate flips mid-run and recovers |
| All identical | ✅ | `count` only increments |
| Majority at the end `[1,2,3,3,3,3,3]` | `3` ✅ | Early candidates cancel out |
| Negative values | ✅ | Boyer–Moore compares, never arithmetic |
| `[1,2,3]` (no majority) | `3` ❌ | Meaningless without verification |
| Exactly `n/2` occurrences | ❌ | Not a majority — `>`, not `>=` |

---

## Complexity Comparison

| Approach | Time | Space |
|---|---|---|
| Brute force | `O(n²)` | `O(1)` |
| Sorting | `O(n log n)` | `O(1)` primitives, `O(n)` merge sort |
| Hash map | `O(n)` | `O(n)` |
| Bit manipulation | `O(n)` | `O(1)` |
| **Boyer–Moore** | **`O(n)`** | **`O(1)`** |

---

## Notes

**`bit < 32`, not `bit <= 32`.** Java masks shift counts to 5 bits, so `1 << 32` is `1 << 0` — it would reprocess bit 0 and corrupt the result. The article's pseudocode says `0 to 32`; in Java that's a bug.

**Use `|=` not `+=` when setting bits.** Equivalent when the bit is clear, but `|=` says "set this bit" and can't double-add. Matters at bit 31, where `1 << 31` is `Integer.MIN_VALUE`.

**Sorting is `O(1)` space for `int[]`.** `Arrays.sort` uses dual-pivot quicksort in place for primitives, but a merge sort (`O(n)`) for objects, since it must be stable. So `Integer[]` would be `O(n)`.

**Direct addressing doesn't apply here.** Values span `±10^9`. Days 1 and 4 could index by value because the range was bounded; here it isn't, so hashing is the only counting option.

**Why not two candidates?** Boyer–Moore generalizes: for elements appearing more than `n/k` times, track `k-1` candidates. **LC 229** (Majority Element II, `n/3`) needs two candidates and two counters — the natural next problem.

**Pattern to reuse.** "Cancel opposing items and see what survives" also solves: Single Number (XOR cancellation), Find the Celebrity, Candy Crush-style elimination.
