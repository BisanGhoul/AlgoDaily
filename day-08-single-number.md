# Day 8 — Single Number

**LeetCode 136** · Easy
**Topics:** bit manipulation · XOR properties · hash set · running fold

---

## Problem

Every element in a non-empty array appears twice except one. Find that one.

```
[2,2,1]      ->  1
[4,1,2,1,2]  ->  4
[1]          ->  1
```

**Constraints**
- `1 <= nums.length <= 3 * 10^4`
- `-3 * 10^4 <= nums[i] <= 3 * 10^4`
- Exactly one element appears once; all others appear exactly twice

**Required:** linear time, constant extra space.

---

## Questions to Ask First

- Is exactly one element unpaired, or could there be several? (Exactly one.)
- Do the duplicates appear next to each other? (No — the array is unsorted.)
- Can values be negative? (Yes, down to −3×10⁴.)
- Can I mutate or sort the input?

---

## Solution

```java
package algodaily;

public class SingleNumber {

    public int singleNumber(int[] nums) {
        int result = 0;
        for (int num : nums) {
            result ^= num;
        }
        return result;
    }
}
```

**Time `O(n)` · Space `O(1)`**

---

## Why XOR Works

Three properties:

```
a ^ a = 0               anything XOR itself cancels
a ^ 0 = a               XOR with zero changes nothing
commutative & associative    order doesn't matter
```

**`a ^ a = 0`** — XOR outputs 1 only where bits *differ*. A number against itself has identical bits everywhere, so every position outputs 0.

```
  0101
^ 0101
------
  0000
```

**Order doesn't matter** — this is what makes it work on a shuffled array. Duplicates don't need to be adjacent; you can mentally reorder the expression:

```
4 ^ 1 ^ 2 ^ 1 ^ 2
= 4 ^ (1 ^ 1) ^ (2 ^ 2)
= 4 ^ 0 ^ 0
= 4
```

**Starting at 0** — zero is the identity for XOR, so the initial value contributes nothing.

**Trace of `[4,1,2,1,2]`:**

```
result = 0
0 ^ 4 = 4   (100)
4 ^ 1 = 5   (101)
5 ^ 2 = 7   (111)
7 ^ 1 = 6   (110)
6 ^ 2 = 4   (100)  ✓
```

Intermediate values like 7 are meaningless — only the final fold matters.

---

## Alternative Approaches

### Hash set — add/remove toggle

```java
public int singleNumber(int[] nums) {
    Set<Integer> seen = new HashSet<>();
    for (int num : nums) {
        if (!seen.add(num)) seen.remove(num);
    }
    return seen.iterator().next();
}
```

**Time `O(n)` · Space `O(n)`** — violates the constant-space requirement.

### Sorting — compare in pairs

```java
public int singleNumber(int[] nums) {
    Arrays.sort(nums);
    for (int i = 0; i < nums.length - 1; i += 2) {
        if (nums[i] != nums[i + 1]) return nums[i];
    }
    return nums[nums.length - 1];
}
```

**Time `O(n log n)` · Space `O(1)`** — violates the linear-time requirement.

---

## Edge Cases

| Input | Result | Why |
|---|---|---|
| `[1]` | `1` ✅ | `0 ^ 1 = 1` |
| `[2,2,1]` | `1` ✅ | Pair cancels first |
| `[1,2,2]` | `1` ✅ | Loner first — order is irrelevant |
| `[4,1,2,1,2]` | `4` ✅ | Duplicates non-adjacent |
| `[-5,-5,3]` | `3` ✅ | Two's complement: `-5 ^ -5 == 0` |
| `[0,1,1]` | `0` ✅ | Zero as the answer works — no sentinel assumption |
| `[-1,-1,-2]` | `-2` ✅ | All-negative |

**`[0,1,1]` is worth testing.** A solution that used 0 as a "not found" marker would break here.

---

## Complexity Comparison

| Approach | Time | Space | Meets requirements |
|---|---|---|---|
| **XOR** | **`O(n)`** | **`O(1)`** | ✅ |
| Hash set | `O(n)` | `O(n)` | ✗ space |
| Sorting | `O(n log n)` | `O(1)` | ✗ time |

---

## Notes

**Initialize to 0, not `nums[0]`.** Starting at `nums[0]` and looping from index 1 also works, but 0 is the XOR identity and lets the loop be a clean fold over the whole array.

**Negatives are free.** XOR operates on the two's-complement bit pattern, so `-5 ^ -5 == 0` like any other value. No special handling.

**No overflow risk.** XOR is bitwise — unlike a sum-based approach, nothing accumulates.

**Why this beats the sum trick.** `2 * sum(distinct) - sum(all)` also finds the loner, but needs a set for the distinct values (`O(n)` space) and can overflow.

**Pattern to reuse — self-inverse operations cancel pairs.** Also solves:

- **LC 268** Missing Number — XOR indices against values
- **LC 389** Find the Difference — XOR both strings together
- **LC 260** Single Number III — two loners; XOR everything, then split by a set bit
- **LC 137** Single Number II — triples instead of pairs; XOR fails, needs bit-position counting mod 3
