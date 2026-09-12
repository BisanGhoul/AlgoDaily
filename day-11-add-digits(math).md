# Day 10 — Add Digits

**LeetCode 58** · Easy
**Topics:** math · digital root · digit manipulation · modulo arithmetic

---

## Problem

Repeatedly add all digits of a number until the result is a single digit.

```
38  ->  3 + 8 = 11  ->  1 + 1 = 2  ->  2
```

**Constraints**
- `0 <= num <= 2^31 - 1`

**Follow-up:** `O(1)` time, no loops or recursion?

---

## Questions to Ask First

- What's the expected range of `num`? (Up to 2³¹ − 1.)
- Is the digit sum always decimal? (Yes.)
- Can the input be 0? (Yes — return 0.)

---

## Solutions

### 1 — Iterative

```java
package algodaily;

public class AddDigits {

    public int addDigits(int num) {
        while (num >= 10) {
            int sum = 0;
            while (num > 0) {
                sum += num % 10;
                num /= 10;
            }
            num = sum;
        }
        return num;
    }
}
```

**Time `O(log num)` · Space `O(1)`**

Extract digits with `num % 10`, sum them, repeat until single digit.

### 2 — Recursive

```java
public int addDigits(int num) {
    if (num < 10) return num;
    
    int sum = 0;
    while (num > 0) {
        sum += num % 10;
        num /= 10;
    }
    return addDigits(sum);
}
```

**Time `O(log num)` · Space `O(log num)` call stack**

Same extraction, but recurse on the sum instead of looping.

### 3 — Mathematical (digital root) ← the follow-up answer

```java
public int addDigits(int num) {
    if (num == 0) return 0;
    return 1 + (num - 1) % 9;
}
```

**Time `O(1)` · Space `O(1)`**

---

## Why the Math Works

**A number is congruent to the sum of its digits modulo 9.**

This is because 10 ≡ 1 (mod 9), 100 ≡ 1 (mod 9), etc.

```
38 ≡ 3 * 10 + 8 ≡ 3 * 1 + 8 ≡ 11 ≡ 2 (mod 9)
38 % 9 = 2  ✓

38 → 11 → 2   (the same as 38 % 9)
```

The digital root (repeated digit sum) is:

```
if num == 0:  0
if num % 9 == 0:  9          (the sum is 9, not 0)
else:  num % 9
```

Compact: `num == 0 ? 0 : 1 + (num - 1) % 9`

**Why `(num - 1) % 9 + 1`?** Because when `num % 9 == 0`, we want 9, not 0. Subtracting 1 first, taking modulo, then adding 1 back converts the 0 to a 9.

**Trace:**

```
num = 38:  1 + (38 - 1) % 9 = 1 + 37 % 9 = 1 + 1 = 2 ✓
num = 9:   1 + (9 - 1) % 9 = 1 + 8 % 9 = 1 + 8 = 9 ✓
num = 27:  1 + (27 - 1) % 9 = 1 + 26 % 9 = 1 + 8 = 9 ✓
num = 0:   0 ✓
```

---

## Edge Cases

| Input | Result | Why |
|---|---|---|
| `0` | `0` ✅ | Guard required — `(0 - 1) % 9 + 1 = 8 + 1 = 0` (mod 9 wraps to negative) |
| `9` | `9` ✅ | Not 0; the sum is 9 |
| `27` | `9` ✅ | Digital root of 27 is 9 |
| `38` | `2` ✅ | |
| `1` | `1` ✅ | Single digit, return as-is |
| `2^31 - 1` | `1` ✅ | Largest 32-bit int; math scales |

**`num = 0` needs the guard.** The formula produces 0 anyway on most systems, but relying on modulo wraparound is fragile.

---

## Complexity Comparison

| Approach | Time | Space | Meets follow-up |
|---|---|---|---|
| Iterative | `O(log num)` | `O(1)` | ✗ loop |
| Recursive | `O(log num)` | `O(log num)` | ✗ recursion |
| **Mathematical** | **`O(1)`** | **`O(1)`** | ✅ |

---

## Notes

**Why no loop in the math version.** Modulo is a single operation — no digit extraction, no iteration.

**The modulo 9 pattern is everywhere.** It's why old accounting systems used a "casting out nines" check: compute the digital root on both sides of an equation. If they don't match, there's an error. (False positives are possible — errors can also be multiples of 9.)

**`(num - 1) % 9 + 1` works for all positive num.** Shifting by 1 before and after converts the range `[1..9]` to `[0..8]`, so `num % 9 == 0` becomes `(num - 1) % 9 == 8`, then `8 + 1 == 9`.

**Guard `num == 0` separately.** The formula would evaluate to `1 + (-1) % 9 + 1`, and negative modulo is language-dependent. In Java, `-1 % 9 = -1`, so `1 + (-1) = 0`. It works, but it's clearer to say "0 maps to 0" up front.

**Mention but don't memorize.** This trick is LeetCode lore — interviewers ask it to see if you know the mathematical insight. Knowing it exists and why it works is the real prize; rederiving it on the fly is equally impressive.

**Pattern to reuse.** Modulo properties solve: Single Number (XOR with mod patterns), Happy Number (cycle detection), and any problem hinting "too fast to be iterative."

**Related problems:** LC 1556 (Thousand Separator — formatting), LC 202 (Happy Number — cycle detection with digit sum).
