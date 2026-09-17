# Day 16 — Missing Number

**LeetCode 268** · Easy
**Topics:** XOR · math (Gauss sum) · cyclic sort · bit manipulation

---

## Problem

Given an array `nums` containing `n` distinct numbers in the range `[0, n]`, return the one number missing from the range.

```
[3,0,1]        ->  2   (n=3, range 0..3)
[0,1]          ->  2   (n=2, range 0..2)
[9,6,4,2,3,5,7,0,1]  ->  8
```

**Constraints**
- `n == nums.length`, `1 <= n <= 10^4`
- `0 <= nums[i] <= n`
- All numbers are distinct

**Follow-up:** `O(1)` extra space and `O(n)` time?

---

## Questions to Ask First

- Is the range `0..n` or `1..n`? (Here `0..n` — changes the sum formula.)
- Exactly one missing, or could several be? (Exactly one.)
- Are duplicates possible? (No, all distinct.)
- Can I mutate the input? (Allows the cyclic sort approach.)
- How large can `n` get? (Matters for sum overflow.)

---

## Solutions

### 1 — XOR ← recommended

```java
package algodaily;

public class MissingNumber {

    public int missingNumber(int[] nums) {
        int result = nums.length;
        for (int i = 0; i < nums.length; i++) {
            result ^= i ^ nums[i];
        }
        return result;
    }
}
```

**Time `O(n)` · Space `O(1)`**

XOR every index against every value. Each number that *is* present cancels with its matching index; the missing one has nothing to cancel against and survives.

### 2 — Gauss sum

```java
public int missingNumber(int[] nums) {
    int n = nums.length;
    int expected = n * (n + 1) / 2;
    int actual = 0;
    for (int num : nums) actual += num;
    return expected - actual;
}
```

**Time `O(n)` · Space `O(1)`**

Sum of `0..n` is `n(n+1)/2`. Subtract what's actually there and the difference is what's missing.

### 3 — Cyclic sort

```java
public int missingNumber(int[] nums) {
    int i = 0;
    while (i < nums.length) {
        int target = nums[i];
        if (target < nums.length && target != i) {
            int temp = nums[i];
            nums[i] = nums[target];
            nums[target] = temp;      // swap into its correct slot
        } else {
            i++;
        }
    }

    for (int j = 0; j < nums.length; j++) {
        if (nums[j] != j) return j;
    }
    return nums.length;
}
```

**Time `O(n)` · Space `O(1)`** — mutates the input

Put every value at the index equal to itself. The first index whose value doesn't match is the missing number. If all `0..n-1` are in place, the answer is `n`.

### 4 — Boolean array

```java
public int missingNumber(int[] nums) {
    boolean[] seen = new boolean[nums.length + 1];
    for (int num : nums) seen[num] = true;

    for (int i = 0; i <= nums.length; i++) {
        if (!seen[i]) return i;
    }
    return -1;
}
```

**Time `O(n)` · Space `O(n)`** — fails the follow-up

Direct addressing, same as Days 1 and 4. Simple but doesn't meet the space requirement.

---

## Why XOR Works

Day 8's three properties still apply:

```
a ^ a = 0
a ^ 0 = a
order doesn't matter
```

**The difference from Single Number.** There, the duplicates were already in the array and cancelled each other. Here you *supply* the pairs yourself — one side is the index, the other is the value.

Conceptually you're XOR-ing these two lists together:

```
indices:  0 1 2 3        (plus n, added as the seed)
values:   3 0 1
```

Every number appearing in both lists cancels. `2` appears only among the indices, so it survives.

**Why `result` starts at `nums.length`.** Indices only run `0..n-1`, but the range is `0..n`. Seeding with `n` completes the index list.

**Trace of `[3,0,1]`** (n = 3):

```
result = 3
i=0:  result ^= 0 ^ 3  ->  3 ^ 0 ^ 3 = 0
i=1:  result ^= 1 ^ 0  ->  0 ^ 1 ^ 0 = 1
i=2:  result ^= 2 ^ 1  ->  1 ^ 2 ^ 1 = 2
-> 2 ✓
```

Reordered, it's clearer why:

```
(3 ^ 3) ^ (0 ^ 0) ^ (1 ^ 1) ^ 2  =  0 ^ 0 ^ 0 ^ 2  =  2
```

---

## Edge Cases

| Input | Result | Why |
|---|---|---|
| `[0]` | `1` ✅ | Missing number is `n` itself |
| `[1]` | `0` ✅ | Missing number is 0 |
| `[0,1]` | `2` ✅ | Missing at the top of the range |
| `[1,2]` | `0` ✅ | Missing at the bottom |
| `[3,0,1]` | `2` ✅ | Missing in the middle |
| `[0,1,2,3]` where n=4 | `4` ✅ | The seed value is the answer |
| n = 10⁴ | ✅ | Sum is ~5×10⁷, safe in `int` |

**`[0]` and `[1]` are the cases to check.** They test both ends of the range — the answer being `n`, and the answer being 0.

---

## Complexity Comparison

| Approach | Time | Space | Mutates input | Overflow risk |
|---|---|---|---|---|
| **XOR** | `O(n)` | `O(1)` | no | none |
| Gauss sum | `O(n)` | `O(1)` | no | yes, if n large |
| Cyclic sort | `O(n)` | `O(1)` | yes | none |
| Boolean array | `O(n)` | `O(n)` | no | none |
| Sorting | `O(n log n)` | `O(1)` | yes | none |

---

## Notes

**XOR beats the sum formula on overflow.** At n = 10⁴ the sum is about 5×10⁷ — fine. But `n(n+1)/2` overflows `int` around n ≈ 65,000. XOR never accumulates a large value, so it has no such ceiling. Use `long` if you go with the sum.

**The formula changes for range `1..n`.** Then the expected sum is `n(n+1)/2` where `n` is the *largest value*, not the array length — and the array has length `n-1`. Always confirm which range the problem means.

**Cyclic sort's `while` doesn't advance on a swap.** After swapping, the new `nums[i]` may also be misplaced, so you check the same index again. Each swap puts at least one value in its final home, so total swaps are bounded by `n` — still `O(n)` despite the nested look.

**`target < nums.length` guards the swap.** The value `n` has no valid index (indices stop at `n-1`), so it's left alone and detected by the final scan.

**Two ways to think about XOR here.** Either "pair each index with each value and see what's unpaired," or "XOR of `0..n` XOR-ed with XOR of the array." Same computation, and the second generalizes better to variants.

**Pattern to reuse.** Self-inverse cancellation and index-value pairing also solve:

- **LC 136** Single Number — duplicates cancel themselves (Day 8)
- **LC 448** Find All Numbers Disappeared in an Array — cyclic sort, multiple missing
- **LC 442** Find All Duplicates in an Array — same cyclic sort shape
- **LC 41** First Missing Positive — cyclic sort, hard variant
- **LC 645** Set Mismatch — one duplicate *and* one missing
