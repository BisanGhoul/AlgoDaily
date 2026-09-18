# Day 17 — Sum All Primes

**AlgoDaily** · Medium
**Topics:** sieve of Eratosthenes · number theory · boolean array · overflow

---

## Problem

Given a number `n`, find all primes ≤ `n` and return their sum.

```
15  ->  41   (2 + 3 + 5 + 7 + 11 + 13)
```

**Constraints**
- `n` is a positive integer ≤ 100000
- Expected time `O(n log n)`
- Expected space `O(n)`

---

## Questions to Ask First

- Is the bound inclusive or exclusive? (Title says ≤ `n`; the example uses 15, which isn't prime, so both readings agree there.)
- Is 1 prime? (No — primes have exactly two distinct divisors.)
- Can the sum overflow? (Primes below 100000 sum to ~4.5×10⁸ — fits `int`, but `long` is safer.)
- Called once, or repeatedly? (Repeated calls would justify caching the sieve.)

---

## Solution

```java
package algodaily;

public class SumAllPrimes {

    public long sumOfAllPrimes(int n) {
        if (n < 2) return 0;

        boolean[] composite = new boolean[n + 1];   // index 0..n inclusive
        long sum = 0;

        for (int i = 2; i <= n; i++) {
            if (!composite[i]) {
                sum += i;                            // i survived — it's prime
                for (long j = (long) i * i; j <= n; j += i) {
                    composite[(int) j] = true;       // mark its multiples
                }
            }
        }
        return sum;
    }
}
```

**Time `O(n log log n)` · Space `O(n)`**

Sieve of Eratosthenes. Start assuming everything is prime, then walk upward: whenever you reach an unmarked number, it's prime — add it, and cross out all its multiples.

**Trace for `n = 15`:**

```
i=2  unmarked -> prime, sum=2    mark 4,6,8,10,12,14
i=3  unmarked -> prime, sum=5    mark 9,12,15
i=4  marked   -> skip
i=5  unmarked -> prime, sum=10   mark 15
i=6  marked   -> skip
i=7  unmarked -> prime, sum=17   mark 14
i=8,9,10 marked -> skip
i=11 unmarked -> prime, sum=28
i=12 marked   -> skip
i=13 unmarked -> prime, sum=41
i=14,15 marked -> skip
-> 41 ✓
```

---

## Why It Works

**A number is composite exactly when it's a multiple of some smaller prime.** So if you cross out every multiple of every prime as you go, whatever is still unmarked when you reach it has no smaller prime factor — which is the definition of prime.

The order matters: by the time the outer loop reaches `i`, every prime below `i` has already crossed out its multiples. So `composite[i]` being false is proof, not a guess.

**Why start the inner loop at `i * i`, not `2 * i`.**

Any multiple `k * i` where `k < i` has a factor smaller than `i` — so it was already crossed out when that smaller factor was processed.

For `i = 5`: 10 was marked by 2, 15 by 3, 20 by 2. The first multiple of 5 that nothing smaller has touched is 25 = 5².

This is what turns the sieve from `O(n log n)` into `O(n log log n)`.

---

## Edge Cases

| Input | Result | Why |
|---|---|---|
| `1` | `0` ✅ | No primes ≤ 1 — guard required |
| `2` | `2` ✅ | Smallest prime |
| `3` | `5` ✅ | 2 + 3 |
| `4` | `5` ✅ | 4 is composite, marked by 2 |
| `15` | `41` ✅ | |
| `100000` | `454396537` ✅ | Fits in `int`, but close enough to prefer `long` |
| `0` | `0` ✅ | Guard catches it |

**`n = 1` needs the guard.** Without it, `new boolean[2]` is allocated and the loop never runs — returning 0 anyway. But `n = 0` would allocate `boolean[1]` and also work. The guard makes the intent explicit rather than relying on loop bounds.

---

## Complexity Comparison

| Approach | Time | Space |
|---|---|---|
| **Sieve of Eratosthenes** | **`O(n log log n)`** | `O(n)` |
| Trial division per number | `O(n√n)` | `O(1)` |
| Sieve, odds only | `O(n log log n)`, half the memory | `O(n/2)` |
| Segmented sieve | `O(n log log n)` | `O(√n)` |

The sieve beats the required `O(n log n)`. Trial division is `O(1)` space but far too slow at n = 10⁵.

**Where `log log n` comes from:** the inner loop runs `n/p` times for each prime `p`, so total work is `n × (1/2 + 1/3 + 1/5 + 1/7 + …)`. The sum of reciprocals of primes up to `n` grows like `log log n` — extremely slowly. At n = 10⁵ it's about 3.

---

## Notes

**`(long) i * i` — the overflow trap.** At `i` near 10⁵, `i * i` is 10¹⁰, which overflows `int` and wraps negative. A negative `j` would either skip the loop or throw on array access. Casting one operand to `long` before multiplying fixes it.

The alternative is to bound the outer loop instead:

```java
for (int i = 2; i * i <= n; i++) { ... }   // sieve only
```

But then you need a separate pass to sum, since the outer loop stops at √n.

**`sum` must be `long`.** The sum for n = 100000 is 454,396,537 — it fits `int`, but only 4× under the ceiling. Not worth the risk.

**Array size `n + 1`, not `n`.** The bound is inclusive, so index `n` must exist.

**Indices 0 and 1 are never read.** The outer loop starts at 2, so they're wasted bytes — harmless, and the alternative (offsetting every index by 2) makes the code worse.

**Odds-only optimization.** After handling 2 separately, every other prime is odd — so you can halve the array and step the inner loop by `2 * i`. Same complexity, half the memory. Worth mentioning, rarely worth writing.

**Segmented sieve for huge `n`.** When `n` is too large for an `O(n)` array, sieve in blocks of size √n using the primes below √n. `O(√n)` space. Relevant above ~10⁹.

**Pattern to reuse.** "Assume all true, eliminate by rule" also solves: LC 204 (Count Primes — identical sieve, count instead of sum), LC 279 (Perfect Squares), LC 264 (Ugly Number II), and any problem precomputing a property for a whole range at once.
