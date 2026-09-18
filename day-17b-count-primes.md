# Day 18 — Count Primes

**LeetCode 204** · Medium
**Topics:** sieve of Eratosthenes · number theory · boolean array · memory optimization

---

## Problem

Return the number of primes **strictly less than** `n`.

```
10  ->  4   (2, 3, 5, 7)
0   ->  0
1   ->  0
```

**Constraints:** `0 <= n <= 5 * 10^6`

---

## Questions to Ask First

- Strictly less than `n`, or ≤ `n`? (Strictly less — so `n = 10` excludes 10, though 10 isn't prime anyway. Test `n = 7`: answer is 3, not 4.)
- Is 1 prime? (No.)
- How large is `n`? (5×10⁶ — a `boolean[]` is 5 MB, which matters.)
- Called once or repeatedly? (Repeated calls would justify caching.)

---

## Solution

```java
package algodaily;

public class CountPrimes {

    public int countPrimes(int n) {
        if (n < 3) return 0;                    // no primes below 2

        boolean[] composite = new boolean[n];   // indices 0..n-1
        int count = 0;

        for (int i = 2; i < n; i++) {
            if (!composite[i]) {
                count++;
                for (long j = (long) i * i; j < n; j += i) {
                    composite[(int) j] = true;
                }
            }
        }
        return count;
    }
}
```

**Time `O(n log log n)` · Space `O(n)`**

Identical to Day 17's sieve. Walk upward; an unmarked number is prime, so count it and cross out its multiples.

**Trace for `n = 10`:**

```
i=2  unmarked -> count=1   mark 4,6,8
i=3  unmarked -> count=2   mark 9
i=4  marked   -> skip
i=5  unmarked -> count=3   (25 >= 10, nothing to mark)
i=6  marked   -> skip
i=7  unmarked -> count=4
i=8,9 marked  -> skip
-> 4 ✓
```

---

## Differences from Day 17

| | Day 17 (Sum All Primes) | Day 18 (Count Primes) |
|---|---|---|
| Bound | ≤ `n` (inclusive) | < `n` (exclusive) |
| Array size | `n + 1` | `n` |
| Loop | `i <= n` | `i < n` |
| Accumulator | `sum += i` (`long`) | `count++` (`int`) |
| Max `n` | 10⁵ | 5×10⁶ |

The exclusive bound is the only real trap. `boolean[n]` gives valid indices `0..n-1`, which is exactly the range to test — so the off-by-one resolves itself if you size the array to `n`.

---

## Edge Cases

| Input | Result | Why |
|---|---|---|
| `0` | `0` ✅ | Guard — `new boolean[0]` would work but the intent is clearer |
| `1` | `0` ✅ | Guard |
| `2` | `0` ✅ | **Strictly less than 2 — 2 itself is excluded** |
| `3` | `1` ✅ | Only 2 counts |
| `7` | `3` ✅ | 2, 3, 5 — not 7. The case that catches an inclusive bound |
| `10` | `4` ✅ | |
| `5 * 10^6` | `348513` ✅ | ~5 MB array |

**`n = 2` is the one to check.** It's the boundary where "strictly less than" bites — an inclusive implementation returns 1 instead of 0. `n = 3` confirms the other side.

---

## Complexity Comparison

| Approach | Time | Space |
|---|---|---|
| **Sieve** | **`O(n log log n)`** | `O(n)` — 5 MB at max |
| Sieve, odds only | `O(n log log n)` | `O(n/2)` — 2.5 MB |
| Bitset sieve | `O(n log log n)` | `O(n/8)` bytes — 625 KB |
| Trial division | `O(n√n)` | `O(1)` — too slow |

At n = 5×10⁶, trial division is roughly 10¹⁰ operations. It will not pass.

---

## Notes

**`if (n < 3) return 0`.** Covers 0, 1, and 2 in one check. `n = 2` is the interesting one — there are no primes *strictly below* 2.

**`(long) i * i` is required here.** At `i` near 5×10⁶, `i * i` is 2.5×10¹³ — far past `int`. Without the cast it wraps negative and either skips the loop or throws on array access. This matters more than in Day 17, where `n` was only 10⁵.

**Alternative that avoids the cast:**

```java
for (int i = 2; (long) i * i < n; i++) {
    if (!composite[i]) {
        for (int j = i * i; j < n; j += i) composite[j] = true;
    }
}
for (int i = 2; i < n; i++) if (!composite[i]) count++;
```

Bound the sieve at √n, then count in a separate pass. `i * i` is safe inside because the loop condition already guaranteed it fits. Two passes, marginally cleaner arithmetic.

**Memory is the real constraint at this size.** `boolean[5_000_000]` is 5 MB — Java stores one byte per boolean, not one bit. Two ways down:

- **Odds only:** handle 2 separately, then index only odd numbers. Halves memory.
- **Bitset:** `long[] bits = new long[n / 64 + 1]`, set with `bits[j >> 6] |= 1L << (j & 63)`. 8× less memory, slightly slower per operation from the bit twiddling.

Neither is needed to pass, but both are good answers to "can you reduce memory?"

**Don't `count` inside the inner loop.** A common bug is counting marks instead of survivors — numbers get marked multiple times (12 is marked by 2 and by 3), so that overcounts wildly.

**Pattern to reuse.** Same as Day 17 — "assume all true, eliminate by rule" precomputes a property for an entire range in near-linear time. Also: LC 279 (Perfect Squares), LC 264 (Ugly Number II), LC 2588 (Count the Number of Beautiful Subarrays).
