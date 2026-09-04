# Day 3 — Reverse Only Letters

**LeetCode 917** · Easy
**Topics:** two pointers · conditional pointer advancement · character classification · in-place swap · `if` vs `else if` control flow

---

## Problem

Reverse all English letters in a string. Every non-letter character stays in its original position.

```
"ab-cd"                ->  "dc-ba"
"a-bC-dEf-ghIj"        ->  "j-Ih-gfE-dCba"
"Test1ng-Leet=code-Q!" ->  "Qedo1ct-eeLg=ntse-T!"
```

**Constraints**
- `1 <= s.length <= 100`
- ASCII values in range `[33, 122]`
- No `"` or `\`

---

## Questions to Ask First

- "English letters" — strictly `a-z` / `A-Z`, or should Unicode letters count? Matters because `Character.isLetter` accepts far more than English.
- Are digits and symbols both frozen? (Yes — anything not a letter.)
- Should case be preserved? (Yes — letters move, they aren't transformed.)
- Can the string be all letters? All non-letters?

---

## Solution

Three versions, all correct. They differ in how many iterations they take and — more importantly — in how much reasoning you need to trust them.

### Version 1 — sequential `if`s (my first attempt)

```java
while (l < r) {
    if (!Character.isLetter(arr[r])) r--;
    if (!Character.isLetter(arr[l])) l++;
    if (Character.isLetter(arr[l]) && Character.isLetter(arr[r])) {
        swap(arr, l, r);
        l++;
        r--;
    }
}
```

Produces correct output. Skips both sides in one iteration when both are non-letters, which is efficient.

**The problem:** the third `if` reads `l` and `r` that the first two just modified. The branches aren't independent — each one sees the previous one's edits. To trust it you have to trace: *"`r` only decremented because `arr[r]` was a non-letter, so the both-letters check can't pass on a crossed pair..."*

That reasoning is sound. But it lives in your head, not in the code.

### Version 2 — `else if`

```java
while (l < r) {
    if (!Character.isLetter(arr[l]))      l++;
    else if (!Character.isLetter(arr[r])) r--;
    else {
        swap(arr, l, r);
        l++;
        r--;
    }
}
```

Exactly one branch fires per iteration. Progress per iteration is obvious, crossing is impossible. But it's the slowest of the three — it only handles one pointer per pass.

### Version 3 — nested `while` ← use this one

```java
package algodaily;

public class ReverseOnlyLetters {

    public String reverseOnlyLetters(String s) {
        char[] arr = s.toCharArray();
        int l = 0, r = arr.length - 1;

        while (l < r) {
            while (l < r && !Character.isLetter(arr[l])) l++;
            while (l < r && !Character.isLetter(arr[r])) r--;

            if (l < r) {
                char temp = arr[l];
                arr[l] = arr[r];
                arr[r] = temp;
                l++;
                r--;
            }
        }

        return new String(arr);
    }
}
```

Skips *entire runs* of non-letters, not one at a time. The `l < r` guard inside each inner loop and before the swap makes crossing impossible by construction — nothing to trace.

**Trace of `"Test1ng-Leet=code-Q!"`:**

```
T e s t 1 n g - L e e t = c o d e - Q !
l=0 'T' letter    r=19 '!' skip -> r=18 'Q'
swap T/Q          l=1, r=17
l=1 'e' letter    r=17 '-' skip -> r=16 'e'
swap e/e          l=2, r=15
...
```

**Iterations on `"a-----b"`:**

| Version | Outer iterations | Correctness is |
|---|---|---|
| 1 — sequential `if`s | 5 | provable, but you must prove it |
| 2 — `else if` | 6 | obvious |
| 3 — nested `while` | 1 | obvious *and* fastest |

---

## Edge Cases

| Input | Result | Why |
|---|---|---|
| `"abc"` (no symbols) | `"cba"` ✅ | Degrades to plain reversal |
| `"---"` (no letters) | `"---"` ✅ | Inner loop stops at `l < r`, swap never fires |
| `"a"` | `"a"` ✅ | `l == r`, outer loop never runs |
| `"-a-"` | `"-a-"` ✅ | Single letter, nothing to pair with |
| `"ab"` | `"ba"` ✅ | Smallest real swap |
| `"a-----b"` | `"b-----a"` ✅ | Whole run skipped in one iteration |
| `"ab---"` | `"ba---"` ✅ | Run at the edge, `r` walks left three times |
| `"a1b"` | `"b1a"` ✅ | Digits are non-letters, frozen |
| Pointers meet on a non-letter | ✅ | `l < r` fails before any bad access |

---

## Complexity

**Time — `O(n)`**

`l` only ever increases and `r` only ever decreases. Their combined travel is bounded by `n`, so total work across the whole run is linear.

The nested `while` loops look quadratic but aren't — the inner loops don't restart from scratch. Each index is visited exactly once across the entire execution. This is the argument to say out loud, because "nested loops" triggers an automatic `O(n²)` guess.

Distribution doesn't matter: 99 hyphens and 1 letter costs the same as 100 letters.

**Space — `O(n)`**

The `char[]` copy plus the new `String`. Auxiliary space for the algorithm itself is `O(1)` — just `l`, `r`, `temp`. Same Java immutability constraint as Day 0: `String reverse(String)` can never be `O(1)`.

---

## Notes

### `if` vs `else if` — the rule I was missing

**Ask: if two conditions are both true, do I want both actions to run?**

- **Yes** → separate `if`s
- **No, only one** → `else if`

FizzBuzz wants separate `if`s. At i = 15 both fire, and that's how `"FizzBuzz"` gets built:

```java
if (i % 3 == 0) sb.append("Fizz");
if (i % 5 == 0) sb.append("Buzz");
```

Both read `i`. Neither writes it. Independent tests of unchanging state — safe.

This problem is different. Version 1's branches **share mutable state**: the third condition reads `l` and `r` that the first two just changed. That's the actual red flag — not "two `if`s fired," but "the second test saw data the first test edited."

**Practical framing:**
- Separate `if`s = **accumulating** (collecting flags, appending pieces, tallying)
- `else if` = **dispatching** (picking one action from a menu)

**Self-check:** if you're tracing through to confirm two `if`s can't both fire and break something, that's the signal. Either make them `else if`, or stop letting them touch the same variables.

### Skipping both pointers at once is fine

The instinct to skip both sides in one iteration is correct and saves work. Version 1 wasn't wrong for doing that. The fix isn't to give up the optimization — it's to take it further (skip whole runs) while making the bounds explicit with `l < r` guards. Version 3 is faster *and* easier to verify.

### `Character.isLetter` is broader than "English letter"

Returns `true` for `é`, `ñ`, `日`, `Ω` — any Unicode letter. Safe here only because the constraint caps input at ASCII 122. If that were lifted:

```java
private boolean isEnglishLetter(char c) {
    return (c >= 'a' && c <= 'z') || (c >= 'A' && c <= 'Z');
}
```

Flagging this shows you read the constraints instead of reaching for the familiar library call.

### The ASCII gap trap

Values 91–96 (`[ \ ] ^ _` and backtick) sit **between** `Z` (90) and `a` (97). So a range check like `c >= 'A' && c <= 'z'` wrongly accepts all six. Classic bug on this problem.

### Pattern to reuse

"Two pointers, skip past what doesn't qualify, act only when both sides qualify":

- **LC 345** Reverse Vowels — identical code, swap `isLetter` for `isVowel`
- **LC 125** Valid Palindrome — skip non-alphanumeric, compare instead of swap
- **LC 283** Move Zeroes
- Quicksort's partition step
