# Day 19 — Find Minimum in Rotated Sorted Array

**LeetCode 153** · Medium
**Topics:** binary search · modified search space · rotation

---

## Problem

A sorted ascending array was rotated some number of times. Find its minimum element in `O(log n)` time.

```
[3,4,5,1,2]      ->  1
[4,5,6,7,0,1,2]  ->  0
[11,13,15,17]    ->  11   (rotated back to sorted)
```

**Constraints**
- `1 <= n <= 5000`
- All elements unique
- `-5000 <= nums[i] <= 5000`

---

## Questions to Ask First

- Are duplicates possible? (No — duplicates break the `O(log n)` guarantee; that's LC 154.)
- Can the array be un-rotated? (Yes — rotating `n` times returns it to sorted.)
- Return the value or its index?
- Is the array guaranteed non-empty? (Yes, `n >= 1`.)

---

## Solution

```java
package algodaily;

public class FindMinimumRotatedSortedArray {

    public int findMin(int[] nums) {
        int left = 0, right = nums.length - 1;

        while (left < right) {
            int mid = left + (right - left) / 2;

            if (nums[mid] > nums[right]) {
                left = mid + 1;      // min must be to the right of mid
            } else {
                right = mid;         // min is at mid, or to its left
            }
        }
        return nums[left];
    }
}
```

**Time `O(log n)` · Space `O(1)`**

---

## The Idea

A rotated sorted array is two sorted runs stuck together, with a single "drop" between them. The minimum is the first element of the second run.

```
[4,5,6,7,0,1,2]
 \______/ \___/
  run 1    run 2
         ^ the drop — this is the minimum
```

**Compare `nums[mid]` against `nums[right]`** to decide which half holds the drop.

- **`nums[mid] > nums[right]`** — mid is in the *left* run, which is higher than the right run. The drop is after mid, so search `mid + 1 .. right`.
- **`nums[mid] <= nums[right]`** — mid to right is already sorted, so mid could be the minimum. Search `left .. mid`, keeping mid.

**Trace `[4,5,6,7,0,1,2]`:**

```
left=0 right=6 mid=3   nums[3]=7 > nums[6]=2   ->  left=4
left=4 right=6 mid=5   nums[5]=1 <= nums[6]=2  ->  right=5
left=4 right=5 mid=4   nums[4]=0 <= nums[5]=1  ->  right=4
left == right  ->  stop
-> nums[4] = 0 ✓
```

---

## Edge Cases

| Input | Result | Why |
|---|---|---|
| `[1]` | `1` ✅ | `left == right` immediately, loop never runs |
| `[1,2]` | `1` ✅ | Not rotated |
| `[2,1]` | `1` ✅ | Rotated once |
| `[11,13,15,17]` | `11` ✅ | Fully rotated — already sorted |
| `[3,4,5,1,2]` | `1` ✅ | Drop in the middle |
| `[2,3,4,5,1]` | `1` ✅ | Drop at the very end |
| All negatives | ✅ | Only comparisons, no arithmetic on values |

**Test `[11,13,15,17]`.** The un-rotated case is where a solution that assumes a drop exists will fail.

---

## Notes

**`while (left < right)`, not `<=`.** With `<=` and `right = mid` (no `-1`), `left` and `right` can both land on `mid` forever — infinite loop. The `<` form exits exactly when the search space narrows to one element, which is the answer.

**`right = mid`, not `mid - 1`.** When `nums[mid] <= nums[right]`, mid itself might *be* the minimum. Excluding it loses the answer.

**Compare against `nums[right]`, not `nums[left]`.** Comparing to `left` fails on an un-rotated array: `nums[mid] > nums[left]` is true for `[1,2,3]`, sending the search right when the answer is at index 0. Comparing to `right` handles both cases with no special-casing.

**`left + (right - left) / 2`, not `(left + right) / 2`.** The second can overflow when both are near `Integer.MAX_VALUE`. Not a risk at n = 5000, but it's the habit to build.

**No separate `mid == left` check needed.** Integer division rounds down, so `mid` never equals `right` while `left < right`. That guarantees `right = mid` always shrinks the range.

**With duplicates it breaks (LC 154).** `[1,1,1,0,1]` — `nums[mid] == nums[right]` tells you nothing about which side the drop is on. The fix is `right--`, which degrades to `O(n)` worst case.

**Pattern to reuse.** "Binary search where the array isn't fully sorted, but each half can be identified as sorted or not" also solves: LC 33 (Search in Rotated Sorted Array), LC 81 (same with duplicates), LC 162 (Find Peak Element), LC 852 (Peak Index in a Mountain Array).
