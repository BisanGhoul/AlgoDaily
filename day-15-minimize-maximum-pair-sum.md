# Day 15 — Minimize Maximum Pair Sum in Array

**LeetCode 1877** · Medium
**Topics:** greedy · sorting · two pointers · exchange argument

---

## Problem

Given an array `nums` of even length, pair up all elements into `n/2` pairs so that the **largest** pair sum is as small as possible. Return that minimized maximum.

```
[3,5,2,3]      ->  7   pairs (3,3) and (5,2)  ->  max(6,7) = 7
[3,5,4,2,4,6]  ->  8   pairs (3,5),(4,4),(6,2) ->  max(8,8,8) = 8
```

**Constraints**
- `2 <= n <= 10^5`, `n` is even
- `1 <= nums[i] <= 10^5`

---

## Questions to Ask First

- Must every element be used exactly once? (Yes.)
- Return the pair sum, or the pairs themselves? (Just the sum.)
- Can I mutate the input by sorting it?
- Are values bounded? (Yes, `1..10^5` — opens up counting sort.)

---

## Solution

```java
package algodaily;

import java.util.Arrays;

public class MinimizeMaximumPairSum {

    public int minPairSum(int[] nums) {
        Arrays.sort(nums);

        int max = 0;
        for (int i = 0, j = nums.length - 1; i < j; i++, j--) {
            max = Math.max(max, nums[i] + nums[j]);
        }
        return max;
    }
}
```

**Time `O(n log n)` · Space `O(1)`**

Sort, then pair smallest with largest, second-smallest with second-largest, and so on. Track the biggest sum produced.

**Trace of `[3,5,4,2,4,6]`:**

```
sorted:  2  3  4  4  5  6
         i              j

2 + 6 = 8       i=0, j=5
3 + 5 = 8       i=1, j=4
4 + 4 = 8       i=2, j=3
i >= j, stop
-> 8
```

---

## Why Pairing Largest with Smallest Is Optimal

**Intuition:** the largest element has to go somewhere, and whatever it's paired with determines a sum you can't avoid. So give it the smallest partner available — that's the cheapest home for it. Then the problem repeats on what's left.

**Exchange argument (the formal version):**

Suppose an optimal pairing does *not* pair the max with the min. Then:
- max `M` is paired with some `x`
- min `m` is paired with some `y`

Those two pairs contribute `max(M + x, m + y)`. Since `M` is the largest element, `M + x >= m + y`, so the contribution is `M + x`.

Now swap partners to get `(M, m)` and `(x, y)`:
- `M + m <= M + x` because `m <= x` (m is the minimum)
- `x + y <= M + x` because `y <= M` (M is the maximum)

So the new maximum is at most `M + x` — the swap never makes things worse. Repeat this argument on the remaining `n-2` elements and you arrive at the sorted-ends pairing.

**Why balance beats grouping.** The instinct to pair similar values together (`(2,3), (4,4), (5,6)`) gives `max = 11`. Pairing extremes balances every sum toward the average, and the maximum of balanced values is lower than the maximum of unbalanced ones.

---

## Edge Cases

| Input | Result | Why |
|---|---|---|
| `[1,1]` | `2` ✅ | Minimum size, single pair |
| `[3,5,2,3]` | `7` ✅ | |
| `[1,1,1,1]` | `2` ✅ | All identical — every pair the same |
| `[1,2,3,4]` | `5` ✅ | Perfectly balanced: 1+4 = 2+3 |
| `[1,1,100,100]` | `101` ✅ | Extreme spread |
| Already sorted | ✅ | Sort is a no-op, logic unchanged |
| Reverse sorted | ✅ | Sort fixes it |
| 10⁵ elements | ✅ | `O(n log n)` is fine |

---

## Complexity Comparison

| Approach | Time | Space |
|---|---|---|
| **Sort + two pointers** | **`O(n log n)`** | **`O(1)`** |
| Counting sort + two pointers | `O(n + k)`, k = 10⁵ | `O(k)` |
| Brute force (all pairings) | `O(n!)` | — |

Counting sort is asymptotically faster since values are bounded at 10⁵, but the 10⁵-element array costs more than the sort saves at these sizes. Worth naming, not worth writing.

---

## Notes

**`i < j`, not `i <= j`.** With `<=` and an even-length array the pointers cross without meeting, so it doesn't actually break here — but `<` states the intent: stop when every element has a partner.

**`Arrays.sort` on `int[]` is `O(1)` space.** Dual-pivot quicksort, in place. `Integer[]` would use merge sort (`O(n)`) because it has to be stable — same distinction as Day 6.

**Initializing `max = 0` is safe.** Values are `>= 1`, so any real pair sum exceeds it. If negatives were possible, use `Integer.MIN_VALUE`.

**No overflow.** Max pair sum is 2 × 10⁵, well inside `int`.

**Recognizing greedy problems.** The signal here is "minimize the maximum" (or maximize the minimum) — it usually means a sorted order plus a local rule gives the global optimum. The way to *justify* it is an exchange argument: show that any solution violating your rule can be swapped toward it without getting worse.

**Pattern to reuse.** Sort-then-pair-from-the-ends also solves: LC 881 (Boats to Save People — nearly the same problem with a capacity limit), LC 1african (Two Sum on sorted arrays), LC 2144 (Minimum Cost of Buying Candies). The broader "minimize the maximum" family often needs binary search on the answer instead — LC 410, LC 1011, LC 875.
