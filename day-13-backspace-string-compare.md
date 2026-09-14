# Day 12 — Backspace String Compare

**LeetCode 844** · Easy
**Topics:** stack · two pointers · reverse traversal · string building

---

## Problem

Given two strings `s` and `t`, return `true` if they are equal when typed into empty text editors. `'#'` means backspace. Backspacing empty text leaves it empty.

```
s = "ab#c", t = "ad#c"   ->  true   (both become "ac")
s = "ab##", t = "c#d#"   ->  true   (both become "")
s = "a#c",  t = "b"      ->  false  ("c" vs "b")
```

**Constraints**
- `1 <= s.length, t.length <= 200`
- Lowercase letters and `'#'` only

**Follow-up:** `O(n)` time and `O(1)` space?

---

## Questions to Ask First

- What happens when backspacing empty text? (Stays empty — no error.)
- Can the whole string be backspaces? (Yes — result is empty.)
- Are both empty results equal? (Yes.)
- Only lowercase letters besides `'#'`? (Yes.)

---

## Solutions

### 1 — Build both strings

```java
package algodaily;

public class BackspaceStringCompare {

    public boolean backspaceCompare(String s, String t) {
        return build(s).equals(build(t));
    }

    private String build(String str) {
        StringBuilder sb = new StringBuilder();
        for (char c : str.toCharArray()) {
            if (c != '#') {
                sb.append(c);
            } else if (sb.length() > 0) {
                sb.deleteCharAt(sb.length() - 1);
            }
        }
        return sb.toString();
    }
}
```

**Time `O(n + m)` · Space `O(n + m)`**

`StringBuilder` acts as the stack — append is push, `deleteCharAt(length - 1)` is pop. The `length() > 0` guard makes backspacing empty text a no-op instead of an exception.

**Trace of `"ab#c"`:**

```
'a'  append  ->  "a"
'b'  append  ->  "ab"
'#'  pop     ->  "a"
'c'  append  ->  "ac"
```

---

### 2 — Build from the right with a skip counter

```java
public boolean backspaceCompare(String s, String t) {
    return build(s).equals(build(t));
}

private String build(String str) {
    StringBuilder sb = new StringBuilder();
    int skip = 0;

    for (int i = str.length() - 1; i >= 0; i--) {
        if (str.charAt(i) == '#') {
            skip++;                      // one more deletion owed
        } else if (skip > 0) {
            skip--;                      // pay off a deletion, discard this char
        } else {
            sb.append(str.charAt(i));    // nothing owed, this char survives
        }
    }
    return sb.toString();
}
```

**Time `O(n + m)` · Space `O(n + m)`**

Same three cases as the two-pointer version, but collecting survivors into a string instead of comparing on the fly. The result comes out reversed — which doesn't matter, since both strings are reversed the same way and `equals` still gives the right verdict.

**Trace of `"ab#c"`:**

```
i=3 'c'  skip=0  ->  keep       sb = "c"
i=2 '#'          ->  skip=1
i=1 'b'  skip=1  ->  skip=0, discard
i=0 'a'  skip=0  ->  keep       sb = "ca"
```

`"ca"` is `"ac"` reversed. Comparing `"ca"` against `"ca"` gives the same answer as `"ac"` against `"ac"`.

**Simpler than the two-pointer version** — one loop, no termination cases, and `equals` handles length mismatches for free. The only thing it gives up is `O(1)` space.

---

### 3 — Two pointers from the right ← the follow-up answer

```java
public boolean backspaceCompare(String s, String t) {
    // walk both strings backward; a '#' deletes leftward,
    // so going backward we meet it before the chars it kills
    int i = s.length() - 1;
    int j = t.length() - 1;

    // "deletions owed" — how many upcoming letters each side must discard
    int skipS = 0, skipT = 0;

    // keep going while EITHER string still has characters.
    // '||' not '&&' — with '&&' we'd stop as soon as one ran out and
    // miss the length check that catches "a" vs "a#"
    while (i >= 0 || j >= 0) {

        // ---- move i back to the next character of s that survives ----
        while (i >= 0) {
            if (s.charAt(i) == '#') {
                skipS++;                 // another deletion owed
                i--;
            } else if (skipS > 0) {
                skipS--;                 // this letter gets eaten
                i--;
            } else {
                break;                   // survivor found — stop ON it, don't consume it
            }
        }

        // ---- same for t ----
        while (j >= 0) {
            if (t.charAt(j) == '#') {
                skipT++;
                j--;
            } else if (skipT > 0) {
                skipT--;
                j--;
            } else {
                break;
            }
        }

        // after both inner loops, each pointer is either ON a survivor
        // or has fallen off the front (-1)

        if (i >= 0 && j >= 0) {
            // both found a survivor — they must match
            if (s.charAt(i) != t.charAt(j)) return false;
        } else if (i >= 0 || j >= 0) {
            // exactly one found a survivor: one string has characters
            // left and the other is exhausted -> different lengths
            return false;
        }
        // neither valid: both ran out together, nothing to compare

        // step past the characters we just compared
        i--;
        j--;
    }
    return true;
}
```

**Time `O(n + m)` · Space `O(1)`**

---

## Why Scan From the Right

Left to right, when you look at a character you don't yet know whether some future `#` will delete it. You have to store it somewhere — hence the stack.

Right to left flips this. A `#` deletes characters to its **left**, so scanning backward you meet the `#` **before** the characters it kills. That turns storage into a counter:

```
see '#'       ->  skip++
see a letter  ->  if skip > 0: skip--, discard it
                  else: this character survives
```

**Trace of `"ab##"`:**

```
i=3 '#'   skip=1
i=2 '#'   skip=2
i=1 'b'   skip>0  ->  skip=1, discard
i=0 'a'   skip>0  ->  skip=0, discard
          nothing survives
```

**Trace of `"ab#c"` vs `"ad#c"`:**

```
i=3 'c' survives    j=3 'c' survives    ->  'c' == 'c' ✓
i=2 '#' skipS=1     j=2 '#' skipT=1
i=1 'b' discarded   j=1 'd' discarded
i=0 'a' survives    j=0 'a' survives    ->  'a' == 'a' ✓
both exhausted  ->  true
```

---

## Edge Cases

| s | t | Result | Why |
|---|---|---|---|
| `"ab#c"` | `"ad#c"` | `true` ✅ | Both reduce to `"ac"` |
| `"ab##"` | `"c#d#"` | `true` ✅ | Both empty |
| `"a#c"` | `"b"` | `false` ✅ | `"c"` vs `"b"` |
| `"#"` | `"#"` | `true` ✅ | Backspace on empty — no crash |
| `"###"` | `""` | `true` ✅ | Excess backspaces are harmless |
| `"a"` | `"a#"` | `false` ✅ | One survivor vs none — the length check catches it |
| `"bxj##tw"` | `"bxo#j##tw"` | `true` ✅ | Multiple backspaces interleaved |
| `"nzp#o#g"` | `"b#nzp#o#g"` | `true` ✅ | Leading char deleted |
| `"y#fo##f"` | `"y#f#o##f"` | `true` ✅ | Different `#` placement, same result |

**`"a"` vs `"a#"` is the case the length check exists for.** One string has a surviving character and the other doesn't — without the `else if (i >= 0 || j >= 0)` branch, the loop would exit and wrongly return `true`.

---

## Complexity Comparison

| Approach | Time | Space | Meets follow-up |
|---|---|---|---|
| 1 — Build forward (stack) | `O(n + m)` | `O(n + m)` | ✗ |
| 2 — Build backward (counter) | `O(n + m)` | `O(n + m)` | ✗ |
| **3 — Two pointers** | **`O(n + m)`** | **`O(1)`** | ✅ |

Solutions 1 and 2 are equivalent in cost. Solution 2 is arguably clearer — a skip counter says "pending deletions" more directly than "undo the last append" — but Solution 1 produces the string in readable order, which helps when debugging.

---

## Notes

**Guard the pop.** `sb.deleteCharAt(sb.length() - 1)` on an empty builder throws `StringIndexOutOfBoundsException`. The problem explicitly says backspacing empty text is a no-op, so the `length() > 0` check is required, not defensive.

**The inner loops need `i >= 0` in the condition.** Without it, `s.charAt(-1)` throws. They must also `break` when a surviving character is found, otherwise they consume it.

**Termination is the fiddly part.** Three cases after both inner loops settle:
- Both pointers valid → compare the characters
- Exactly one valid → different lengths → `false`
- Neither valid → both exhausted → done

Getting only the first two and forgetting the third (or vice versa) is the common bug.

**`while (i >= 0 || j >= 0)`, not `&&`.** With `&&` the loop exits as soon as one string runs out, skipping the length check that catches `"a"` vs `"a#"`.

**Why the reverse scan removes the need for storage.** It's the same idea as any "process in the order that resolves dependencies" trick — forward, a character's fate depends on the future; backward, it depends on the past, which you've already seen.

**Pattern to reuse.** Reverse traversal to avoid buffering also solves: LC 443 (String Compression), LC 88 (Merge Sorted Array — fill from the back to avoid overwriting), LC 71 (Simplify Path — stack version of the same idea).
