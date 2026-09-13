# Day 11 — Find the Index of the First Occurrence in a String

**LeetCode 28** · Easy
**Topics:** string matching · sliding comparison · KMP · prefix function

---

## Problem

Return the index of the first occurrence of `needle` in `haystack`, or `-1` if it isn't present.

```
haystack = "sadbutsad", needle = "sad"   ->  0
haystack = "leetcode",  needle = "leeto" ->  -1
```

**Constraints**
- `1 <= haystack.length, needle.length <= 10^4`
- Lowercase English letters only

---

## Questions to Ask First

- Return the first occurrence or all of them? (First.)
- What if `needle` is empty? (Constraints say length ≥ 1, but the classic convention is to return 0.)
- Can `needle` be longer than `haystack`? (Then the answer is always −1.)
- Case sensitive? (Yes — lowercase only here.)
- Will this be called repeatedly against the same `haystack`? (If so, preprocessing the haystack pays off.)

---

## Solutions

### 1 — Brute force (sliding comparison)

```java
package algodaily;

public class FirstOccurrence {

    public int strStr(String haystack, String needle) {
        int n = haystack.length();
        int m = needle.length();

        for (int i = 0; i <= n - m; i++) {
            int j = 0;
            while (j < m && haystack.charAt(i + j) == needle.charAt(j)) {
                j++;
            }
            if (j == m) return i;
        }
        return -1;
    }
}
```

**Time `O(n * m)` · Space `O(1)`**

Try every starting position, compare character by character.

**Two details that matter:**

`i <= n - m` — once fewer than `m` characters remain, there's no room for a match. Using `i < n` reads past the end and throws.

`j == m` — the inner loop exits either on a mismatch or on running out of needle. Only the second means a full match.

**Trace of `haystack = "sadbutsad"`, `needle = "sad"`:**

```
i=0:  s==s, a==a, d==d  ->  j=3 == m  ->  return 0
```

**Worst case:**

```
haystack = "aaaaaaaaab"
needle   = "aaab"

i=0: matches aaa, fails on b
i=1: matches aaa, fails on b
i=2: matches aaa, fails on b
...
```

Nearly `m` work at nearly `n` positions.

---

### 2 — KMP (Knuth–Morris–Pratt)

```java
public int strStr(String haystack, String needle) {
    int n = haystack.length();
    int m = needle.length();

    int[] lps = buildLPS(needle);

    int i = 0;  // haystack pointer — never moves backward
    int j = 0;  // needle pointer

    while (i < n) {
        if (haystack.charAt(i) == needle.charAt(j)) {
            i++;
            j++;
            if (j == m) return i - m;
        } else if (j > 0) {
            j = lps[j - 1];       // fall back in the needle, not the haystack
        } else {
            i++;                  // no prefix to fall back on
        }
    }
    return -1;
}

private int[] buildLPS(String needle) {
    int m = needle.length();
    int[] lps = new int[m];
    int len = 0;   // length of the current longest prefix-suffix
    int i = 1;

    while (i < m) {
        if (needle.charAt(i) == needle.charAt(len)) {
            len++;
            lps[i] = len;
            i++;
        } else if (len > 0) {
            len = lps[len - 1];   // fall back and retry
        } else {
            lps[i] = 0;
            i++;
        }
    }
    return lps;
}
```

**Time `O(n + m)` · Space `O(m)`**

---

## How KMP Works

### The waste it fixes

In brute force, after failing at position 0 you already know what characters positions 1, 2, 3 contain — you just read them. But you throw that away and re-read them from scratch.

KMP never re-reads the haystack. `i` only moves forward.

### The LPS array

**LPS = Longest Proper Prefix which is also a Suffix.**

For each position in the needle, `lps[k]` = the length of the longest string that is both a prefix of `needle[0..k]` and a suffix of it. "Proper" means it can't be the whole string.

For `needle = "aabaab"`:

| index | prefix | longest prefix == suffix | lps |
|---|---|---|---|
| 0 | `a` | — (proper only) | 0 |
| 1 | `aa` | `a` | 1 |
| 2 | `aab` | none | 0 |
| 3 | `aaba` | `a` | 1 |
| 4 | `aabaa` | `aa` | 2 |
| 5 | `aabaab` | `aab` | 3 |

```
lps = [0, 1, 0, 1, 2, 3]
```

### What the LPS buys you

Suppose you matched 5 characters of `"aabaab"` and then hit a mismatch:

```
haystack: ... a a b a a X ...
needle:       a a b a a b
                        ^ mismatch at j=5
```

`lps[4] = 2` says: the last 2 characters you matched (`aa`) are also the first 2 characters of the needle.

So those 2 characters are *already matched*. Set `j = 2` and keep comparing from there — no need to recheck them, and `i` doesn't move at all.

```
haystack: ... a a b a a X ...
needle:             a a b a a b
                        ^ resume at j=2
```

Brute force would have restarted at `i+1` and re-compared everything.

### Why `j = lps[j - 1]` and not `lps[j]`

`j` is the index where the mismatch happened — that character *didn't* match. The last successfully matched character is at `j - 1`, so that's the position whose LPS value you want.

### Why it's `O(n + m)`

`i` only ever increases, so the search loop runs at most `n` times. `j` increases at most `n` times total and each fallback decreases it, so fallbacks are bounded by increments. Building the LPS is `O(m)` by the same argument.

---

## Edge Cases

| haystack | needle | Result | Why |
|---|---|---|---|
| `"sadbutsad"` | `"sad"` | `0` ✅ | Match at start |
| `"sadbutsad"` | `"but"` | `3` ✅ | Match in middle |
| `"leetcode"` | `"leeto"` | `-1` ✅ | Partial match then fail |
| `"a"` | `"a"` | `0` ✅ | Whole string |
| `"abc"` | `"abcd"` | `-1` ✅ | Needle longer — `n - m` is negative, loop never runs |
| `"mississippi"` | `"issip"` | `4` ✅ | Repeated substrings, false starts |
| `"aaa"` | `"aaa"` | `0` ✅ | Full overlap |
| `"aaaaab"` | `"aaab"` | `2` ✅ | Brute-force worst case |
| `"abc"` | `""` | `0` | Convention; excluded by constraints |

**`"mississippi"` / `"issip"` is the classic KMP test.** It has multiple partial matches that fail late — exactly where restarting costs the most.

---

## Complexity Comparison

| Approach | Time | Space | Preprocessing |
|---|---|---|---|
| Brute force | `O(n * m)` | `O(1)` | none |
| **KMP** | **`O(n + m)`** | `O(m)` | `O(m)` for LPS |
| `haystack.indexOf(needle)` | `O(n * m)` | `O(1)` | — (banned here) |

Brute force is near-linear on random text, since the inner loop usually dies on the first character. KMP wins on repetitive input.

---

## Notes

**`i <= n - m`, not `i < n`.** Once fewer than `m` characters remain there's no room for a match. This also handles `needle` longer than `haystack` for free — `n - m` is negative, so the loop never runs.

**Check `j == m`, not `j > 0`.** The inner loop exits on mismatch *or* on completing the needle. Only the second is a match.

**KMP's real invariant: `i` never moves backward.** That's the whole point. Brute force rewinds `i` to `i+1` on every failure; KMP only ever rewinds `j`.

**`lps[0]` is always 0.** A single character has no proper prefix.

**Don't memorize KMP — understand the LPS.** In an interview, write brute force first, state the `O(n*m)` worst case, then explain that KMP precomputes how far to jump on a mismatch so the haystack is never re-read. Naming it and explaining the idea is the expected answer; implementing it from memory isn't.

**Alternatives worth naming:**
- **Rabin–Karp** — rolling hash, `O(n + m)` average, `O(n*m)` worst on hash collisions. Better when searching for many needles at once.
- **Boyer–Moore** — scans the needle right-to-left and skips ahead on mismatch. Sublinear in practice; what `grep` uses.

**Pattern to reuse.** The LPS array itself solves: LC 459 (Repeated Substring Pattern — check if `m % (m - lps[m-1]) == 0`), LC 214 (Shortest Palindrome), and any problem about a string's periodicity.
