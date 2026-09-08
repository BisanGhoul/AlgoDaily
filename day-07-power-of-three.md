# Day 7 — Power of Three

**LeetCode 326** · Easy
**Topics:** math · integer division · primality · loop vs recursion vs closed form

---

## Problem

Given an integer `n`, return `true` if it is a power of three — that is, if there exists an integer `x` such that `n == 3^x`.

```
27  ->  true    (3^3)
0   ->  false   (no x gives 0)
-1  ->  false   (no x gives -1)
```

**Constraints:** `-2^31 <= n <= 2^31 - 1`

**Follow up:** solve it without loops or recursion.

---

## Questions to Ask First

- Does `n = 1` count? (Yes — 3⁰ = 1.)
- Are negatives and zero always false? (Yes — a positive base can't produce them.)
- Can `x` be negative? (No — `x` is an integer exponent producing an integer, so `x >= 0`.)
- Is the input always a 32-bit `int`, or could it be a `long`? (Matters for the closed-form solution.)

---

## Solutions

### 1 — Iterative (divide it down)

```java
package algodaily;

public class PowerOfThree {

    public boolean isPowerOfThree(int n) {
        if (n <= 0) return false;
        while (n % 3 == 0) n /= 3;
        return n == 1;
    }
}
```

**Time `O(log₃ n)` · Space `O(1)`**

Reverse the definition. If `n == 3^x`, it was built by multiplying 3 repeatedly — so undo it by dividing repeatedly.

### 2 — Recursive

```java
public boolean isPowerOfThree(int n) {
    if (n <= 0) return false;
    if (n == 1) return true;
    if (n % 3 != 0) return false;
    return isPowerOfThree(n / 3);
}
```

**Time `O(log₃ n)` · Space `O(log₃ n)`** — call stack

Same three checks as the loop, expressed as base cases:

- `n <= 0` → invalid, stop
- `n == 1` → everything divided out, success
- `n % 3 != 0` → leftover isn't 1, failure
- otherwise → divide and recurse

Max depth is 19 (3¹⁹ is the largest power of 3 in an `int`), so there's no stack overflow risk. But Java has no tail-call optimization — every frame stays alive until the chain returns, so this is `O(log n)` space where the loop is `O(1)`.

### 3 — Closed form ← the follow-up answer

```java
public boolean isPowerOfThree(int n) {
    return n > 0 && 1162261467 % n == 0;
}
```

**Time `O(1)` · Space `O(1)`**

No loop, no recursion. See the reasoning below.

---

## Why `return n == 1` Works

The loop strips out every factor of 3. Whatever survives is the part of `n` that **wasn't** made of 3s.

```
n = 27:  27 → 9 → 3 → 1     stops: 1 % 3 != 0
                            leftover = 1  → true

n = 45:  45 → 15 → 5        stops: 5 % 3 != 0
                            leftover = 5  → false
```

45 is 3² × 5. The loop removes both 3s and the 5 is left sitting there. That leftover is proof 45 was never a pure power of 3.

So the question becomes: **after removing all the 3s, is anything else left?** Nothing left means the leftover is 1, because 1 is what remains when you've divided everything out.

This also handles `n = 1` for free: `1 % 3` is 1, so the loop never runs and `1 == 1` returns `true` — correct, since 3⁰ = 1.

---

## Why the Closed Form Works

**3 is prime.** So the only divisors of 3¹⁹ are exactly 3⁰, 3¹, … 3¹⁹ — nothing else divides it.

3¹⁹ = **1162261467**, the largest power of 3 that fits in a 32-bit `int`. If `n` divides it evenly, `n` must itself be a power of 3.

**This only works because the base is prime.** For powers of 4: 4⁷ = 16384 is divisible by 2, 8, and 32 — none of which are powers of 4. The trick doesn't generalize to composite bases.

The `n > 0` guard is still required: `1162261467 % -1 == 0`, so negatives would wrongly pass. And `n = 0` throws `ArithmeticException` on division by zero.

---

## Edge Cases

| Input | Result | Why |
|---|---|---|
| `1` | `true` ✅ | 3⁰ = 1; loop never runs |
| `3` | `true` ✅ | Smallest non-trivial case |
| `0` | `false` ✅ | Caught by the `n <= 0` guard — otherwise `0 % 3 == 0` loops forever |
| `-3` | `false` ✅ | Guard catches it; without it, `-3 / 3 = -1` and `-1 != 1` anyway |
| `-1` | `false` ✅ | Guard required — closed form would wrongly pass |
| `45` | `false` ✅ | 3² × 5, leftover of 5 |
| `9` | `true` ✅ | |
| `1162261467` | `true` ✅ | 3¹⁹, the largest that fits in an `int` |
| `2^31 - 1` | `false` ✅ | Not a power of 3 |

**`n = 0` is the case that hangs an unguarded loop.** `0 % 3 == 0` is true and `0 / 3 == 0`, so it never terminates.

---

## Complexity Comparison

| Approach | Time | Space |
|---|---|---|
| Iterative | `O(log₃ n)` | `O(1)` |
| Recursive | `O(log₃ n)` | `O(log₃ n)` stack |
| Closed form | `O(1)` | `O(1)` |

---

## Notes

**Guard `n <= 0` before anything else.** Zero loops forever, and negatives are never a power of a positive base. In the closed form, it's what stops `-1` from passing.

**Don't use logarithms.** `Math.log(n) / Math.log(3)` and check for a whole number is tempting, but floating point breaks it — `log(243)/log(3)` can evaluate to 4.999999999999999. You'd need an epsilon comparison, and choosing the tolerance is guesswork. Worth naming in an interview along with why you'd reject it.

**Recursion doesn't answer the follow-up.** The follow-up asks for no loops *or recursion*, so only the closed form qualifies.

**Transferable question — "is n a power of k?"**

| k | Approach |
|---|---|
| Any k | Divide down, check leftover == 1 |
| Prime k | `maxPower(k) % n == 0` |
| 2 | `n > 0 && (n & (n - 1)) == 0` — bit trick |
| 4 | Power of 2 **and** the set bit is at an even position |

**Related problems:** LC 231 (Power of Two), LC 342 (Power of Four), LC 1780 (Check if Number is a Sum of Powers of Three).
