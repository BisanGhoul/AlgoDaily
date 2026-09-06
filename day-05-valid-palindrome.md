# Day 5 — Valid Palindrome

**LeetCode 125** · Easy
**Topics:** two pointers · character filtering · case normalization · branch dispatch

---

## Problem

A phrase is a palindrome if, after lowercasing and removing all non-alphanumeric characters, it reads the same forward and backward.

```
"A man, a plan, a canal: Panama"  ->  true   ("amanaplanacanalpanama")
"race a car"                      ->  false  ("raceacar")
" "                               ->  true   (empty after filtering)
```

**Constraints**
- `1 <= s.length <= 2 * 10^5`
- Printable ASCII only

---

## Questions to Ask First

- Do digits count as alphanumeric? (Yes — `"0P"` is a real test case.)
- Is an empty result a palindrome? (Yes, `" "` → `true`.)
- Can I allocate a copy, or should space be constant?
- ASCII only, or Unicode?

---

## Solutions

Two independent choices:

1. **How to skip non-alphanumerics** — `else if` dispatch, plain `if` + `continue`, or nested `while`
2. **When to lowercase** — whole string up front, or per comparison

Any skip style pairs with any normalization style. All six are below.

---

### 1A — `else if` dispatch + per-character lowercase ← submitted

```java
package algodaily;

public class ValidPalindrome {

    public boolean isPalindrome(String s) {
        int start = 0;
        int last = s.length() - 1;

        while (start < last) {
            char currFirst = s.charAt(start);
            char currLast = s.charAt(last);

            if (!Character.isLetterOrDigit(currFirst)) {
                start++;
            } else if (!Character.isLetterOrDigit(currLast)) {
                last--;
            } else {
                if (Character.toLowerCase(currFirst) != Character.toLowerCase(currLast)) {
                    return false;
                }
                start++;
                last--;
            }
        }
        return true;
    }
}
```

**Time `O(n)`** — one pointer moves per iteration, neither reverses, combined travel ≤ n.
**Space `O(1)`** — two ints, two chars.

---

### 1B — `else if` dispatch + whole-string lowercase

```java
public boolean isPalindrome(String s) {
    s = s.toLowerCase();
    int start = 0;
    int last = s.length() - 1;

    while (start < last) {
        char currFirst = s.charAt(start);
        char currLast = s.charAt(last);

        if (!Character.isLetterOrDigit(currFirst)) {
            start++;
        } else if (!Character.isLetterOrDigit(currLast)) {
            last--;
        } else {
            if (currFirst != currLast) {
                return false;
            }
            start++;
            last--;
        }
    }
    return true;
}
```

**Time `O(n)`** — one extra linear pass for `toLowerCase()`, still linear.
**Space `O(n)`** — `toLowerCase()` allocates a new `String` with its own backing array.

---

### 2A — plain `if` + `continue` + per-character lowercase

```java
public boolean isPalindrome(String s) {
    int start = 0;
    int last = s.length() - 1;

    while (start < last) {
        if (!Character.isLetterOrDigit(s.charAt(start))) { start++; continue; }
        if (!Character.isLetterOrDigit(s.charAt(last)))  { last--;  continue; }

        if (Character.toLowerCase(s.charAt(start)) != Character.toLowerCase(s.charAt(last))) {
            return false;
        }
        start++;
        last--;
    }
    return true;
}
```

**Time `O(n)`** — identical iteration count to 1A.
**Space `O(1)`**

---

### 2B — plain `if` + `continue` + whole-string lowercase

```java
public boolean isPalindrome(String s) {
    s = s.toLowerCase();
    int start = 0;
    int last = s.length() - 1;

    while (start < last) {
        if (!Character.isLetterOrDigit(s.charAt(start))) { start++; continue; }
        if (!Character.isLetterOrDigit(s.charAt(last)))  { last--;  continue; }

        if (s.charAt(start) != s.charAt(last)) {
            return false;
        }
        start++;
        last--;
    }
    return true;
}
```

**Time `O(n)`**
**Space `O(n)`** — the `toLowerCase()` copy.

---

### 3A — nested `while` + per-character lowercase

```java
public boolean isPalindrome(String s) {
    int start = 0;
    int last = s.length() - 1;

    while (start < last) {
        while (start < last && !Character.isLetterOrDigit(s.charAt(start))) start++;
        while (start < last && !Character.isLetterOrDigit(s.charAt(last)))  last--;

        if (Character.toLowerCase(s.charAt(start)) != Character.toLowerCase(s.charAt(last))) {
            return false;
        }
        start++;
        last--;
    }
    return true;
}
```

**Time `O(n)`** — nested loops are *not* quadratic here; the inner loops don't restart, each index is visited once. Fewer outer iterations than 1A/2A since whole runs are skipped at once.
**Space `O(1)`**

---

### 3B — nested `while` + whole-string lowercase

```java
public boolean isPalindrome(String s) {
    s = s.toLowerCase();
    int start = 0;
    int last = s.length() - 1;

    while (start < last) {
        while (start < last && !Character.isLetterOrDigit(s.charAt(start))) start++;
        while (start < last && !Character.isLetterOrDigit(s.charAt(last)))  last--;

        if (s.charAt(start) != s.charAt(last)) {
            return false;
        }
        start++;
        last--;
    }
    return true;
}
```

**Time `O(n)`**
**Space `O(n)`**

---

### Comparison

| Version | Skip style | Lowercase | Time | Space | Outer iterations on `"a......b"` |
|---|---|---|---|---|---|
| 1A | `else if` | per char | `O(n)` | `O(1)` | 6 |
| 1B | `else if` | whole string | `O(n)` | `O(n)` | 6 |
| 2A | `if` + `continue` | per char | `O(n)` | `O(1)` | 6 |
| 2B | `if` + `continue` | whole string | `O(n)` | `O(n)` | 6 |
| 3A | nested `while` | per char | `O(n)` | `O(1)` | 1 |
| 3B | nested `while` | whole string | `O(n)` | `O(n)` | 1 |

**3A is optimal** — `O(1)` space, fewest iterations. **1A is the most readable** and what I'd write in an interview.

---

### Trace of `"a..bcb.a"` (filters to `"abcba"`)

```
a . . b c b . a
0 1 2 3 4 5 6 7

start=0 'a', last=7 'a'   both alnum -> equal, start=1, last=6
start=1 '.'               -> start=2
start=2 '.'               -> start=3
start=3 'b', last=6 '.'   -> last=5
start=3 'b', last=5 'b'   -> equal, start=4, last=4
start=4, last=4           -> start < last false, stop
-> true
```

---

## Edge Cases

| Input | Result | Why |
|---|---|---|
| `" "` | `true` ✅ | `start` climbs to meet `last`, comparison never fires |
| `".,!"` | `true` ✅ | Nothing alphanumeric survives |
| `"a"` | `true` ✅ | `start == last`, loop never runs |
| `"aa"` | `true` ✅ | Smallest real comparison |
| `"ab"` | `false` ✅ | |
| `"0P"` | `false` ✅ | Breaks naive case folding — see notes |
| `"a1a"` | `true` ✅ | Digits are alphanumeric |
| `"Aa"` | `true` ✅ | Case folding works |
| `"a..bcb.a"` | `true` ✅ | Consecutive symbols |
| `"race a car"` | `false` ✅ | Mismatch before pointers meet |

---

## Complexity Summary

**Time — `O(n)` for all six.** Every iteration moves one pointer by one and neither reverses, so combined travel is bounded by `n`. Whole-string lowercase adds one extra linear pass.

**Space — `O(1)` with per-character lowercase, `O(n)` with whole-string.** `String` is immutable, so `toLowerCase()` can't modify in place; it allocates.

`O(1)` is reachable here because the return type is `boolean`. Day 0 and Day 3 returned a `String`, so allocation was unavoidable no matter how the algorithm was written.

---

## Notes

**Unguarded sequential `if`s are a bug.**

```java
if (!isAlnum(s.charAt(start))) start++;
if (!isAlnum(s.charAt(last)))  last--;
if (toLower(...) != toLower(...)) return false;   // runs unconditionally
```

An `if` skips at most one char, then falls into the comparison without rechecking. On `"a..bcb.a"`, `start` moves off the first dot onto the second and compares `'.'` vs `'b'` → wrong `false`. The `else`, the `continue`, and the nested `while` all exist to make the comparison unreachable unless both chars are valid.

**`start < last`, not `<=`.** With `<=` the pointers meet and compare a char to itself. Always true, one wasted iteration.

**Skip the `isEmpty()` guard.** On `""`, `last = -1`, so `0 < -1` is false and it returns `true` anyway.

**`isLetterOrDigit`, not `isLetter`.** Digits count. `isLetter` filters `"0P"` to `"P"` → wrong `true`.

**`"0P"` breaks hand-rolled case folding.** ASCII `'0'` = 48, `'P'` = 80. Adding 32 to everything turns `'0'` into `'P'`. `Character.toLowerCase` only touches letters.

**`toLowerCase()` is eager and locale-sensitive.** A mismatch at index 0 still means 200k chars were lowercased. And the no-arg version uses `Locale.getDefault()` — Turkish maps `'I'` → `'ı'`, not `'i'`. Use `toLowerCase(Locale.ROOT)`. `Character.toLowerCase(char)` has no locale issue.

**Pattern to reuse:**

| Day | Problem | Predicate | Action |
|---|---|---|---|
| 0 | Reverse a String | — | swap |
| 3 | Reverse Only Letters | `isLetter` | swap |
| 5 | Valid Palindrome | `isLetterOrDigit` | compare |

Next: **LC 680** Valid Palindrome II (allow one deletion), **LC 167** Two Sum II.
