# Day 4 — Valid Anagram

**LeetCode 242** · Easy · Accepted 55/55
**Topics:** hash map · frequency counting · direct addressing · boxing vs primitives

---

## Problem

Given two strings `s` and `t`, return `true` if `t` is an anagram of `s`.

```
s = "anagram", t = "nagaram"  ->  true
s = "rat",     t = "car"      ->  false
```

**Constraints**
- `1 <= s.length, t.length <= 5 * 10^4`
- Both consist of lowercase English letters

**Follow up:** what if the inputs contain Unicode characters?

---

## Questions to Ask First

- Is the character set bounded? (Yes — lowercase English only. This changes the optimal solution.)
- Case sensitive? Should whitespace count?
- Can I mutate the inputs? (Determines whether sorting in place is allowed.)
- Is this called once, or repeatedly against the same `s`? (Repeated calls favour precomputing `s`'s frequency map.)
- Unicode possible? (The stated follow-up.)

---

## Solution

### Version 1 — HashMap (submitted, accepted)

```java
package algodaily;

import java.util.HashMap;
import java.util.Map;

public class ValidAnagram {

    public boolean isAnagram(String s, String t) {
        if (s.length() != t.length()) return false;

        Map<Character, Integer> map = new HashMap<>();

        for (int i = 0; i < s.length(); i++) {
            char ch = s.charAt(i);
            map.put(ch, map.getOrDefault(ch, 0) + 1);
        }

        for (int i = 0; i < t.length(); i++) {
            char ch = t.charAt(i);
            if (!map.containsKey(ch)) return false;
            map.put(ch, map.get(ch) - 1);
            if (map.get(ch) == 0) map.remove(ch);
        }

        return map.isEmpty();
    }
}
```

Count every char in `s`, then decrement while walking `t`. A missing key means `t` has a character `s` doesn't — not an anagram.

The early length check is the right call: different lengths can never be anagrams, and it lets the rest of the logic assume equal lengths.

**Result:** 17ms (beats 16.89%), 46.70MB (beats 9.07%). Correct and optimal complexity, but heavy constant factors — see notes.

### Version 2 — `int[26]` ← optimal for these constraints

```java
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;

    int[] counts = new int[26];
    for (int i = 0; i < s.length(); i++) {
        counts[s.charAt(i) - 'a']++;
        counts[t.charAt(i) - 'a']--;
    }

    for (int c : counts) {
        if (c != 0) return false;
    }
    return true;
}
```

Because lengths are already equal, both strings can be walked in a **single pass** — increment for `s`, decrement for `t`. If every bucket lands on zero, the frequencies matched.

`s.charAt(i) - 'a'` maps `'a'`→0, `'b'`→1, … `'z'`→25. Same direct-addressing trick as the boolean array in Day 1: stop storing the value, use it as an index.

### Version 3 — sorting

```java
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    char[] a = s.toCharArray(), b = t.toCharArray();
    Arrays.sort(a);
    Arrays.sort(b);
    return Arrays.equals(a, b);
}
```

`O(n log n)` — worse, but two lines and no auxiliary structure. Worth naming as the "if memory were tight" alternative.

### Follow-up — Unicode

`char` is a 16-bit UTF-16 code unit, not a character. Emoji and anything outside the Basic Multilingual Plane are stored as **surrogate pairs** — two `char`s that only mean something together. Counting by `char` splits them and gives wrong answers.

Count by **code point** instead:

```java
public boolean isAnagram(String s, String t) {
    if (s.codePointCount(0, s.length()) != t.codePointCount(0, t.length())) return false;

    Map<Integer, Integer> map = new HashMap<>();
    s.codePoints().forEach(cp -> map.merge(cp, 1, Integer::sum));

    return t.codePoints().allMatch(cp -> {
        Integer count = map.get(cp);
        if (count == null) return false;
        if (count == 1) map.remove(cp); else map.put(cp, count - 1);
        return true;
    }) && map.isEmpty();
}
```

`int[26]` is impossible here — Unicode has ~1.1 million code points, so direct addressing would need a 1.1M-element array. This is exactly the tradeoff from Day 1: **bounded value range → array; unbounded → hash map.**

---

## Edge Cases

| Input | Result | Why |
|---|---|---|
| `"a"`, `"a"` | `true` ✅ | Minimum length |
| `"a"`, `"ab"` | `false` ✅ | Length guard catches it immediately |
| `"ab"`, `"ba"` | `true` ✅ | Simple reorder |
| `"aacc"`, `"ccac"` | `false` ✅ | Same char *set*, different counts — the case a `Set` would wrongly pass |
| `"rat"`, `"car"` | `false` ✅ | `'r'` missing from `t`'s perspective |
| `"aaa"`, `"aaa"` | `true` ✅ | All duplicates |
| 5×10⁴ chars | ✅ | Linear, fine |
| `null` | ❌ NPE | Excluded by constraints |
| `"ab😀"` variants | ❌ with `char` | Needs the code-point version |

**Test `"aacc"` vs `"ccac"`.** It's the case that separates frequency counting from set membership. Both strings have the same distinct characters — only the counts differ.

---

## Complexity

Let `n = s.length()`.

**Time — `O(n)`**

Two passes of `n` iterations, each doing `O(1)` average hash operations. The final `isEmpty()` is `O(1)`.

The `int[26]` version is also `O(n)` but with a single pass and a fixed 26-element sweep at the end — same class, much smaller constants.

**Space — `O(1)`**

The map holds at most 26 distinct keys regardless of input size. A 5×10⁴-character string still produces a 26-entry map. Bounded by the alphabet, not by `n`.

Say this precisely in an interview: `O(k)` where `k` is the alphabet size, which is `O(1)` for a fixed alphabet. For the Unicode version it's genuinely `O(min(n, 1.1M))`.

---

## Notes

### `return map.isEmpty()` is always `true` at that point

Given the length guard, this line can only ever return `true`:

- The map's counts sum to exactly `s.length()` after the first loop.
- Zero-count keys are removed, so every remaining count is ≥ 1 — a decrement can never go negative.
- The second loop runs `t.length()` times and decrements the total by exactly 1 each time.
- Since `s.length() == t.length()`, the total reaches 0, meaning every key was removed.

So reaching that line implies an empty map. `return true` is equivalent.

Not a bug, and keeping it is defensible as a safety net. But knowing *why* it's redundant is the point — it means you understand the invariant your own code maintains.

### Why the memory beat only 9%

Every `char` gets boxed into a `Character` object and every count into an `Integer`. Each is a heap allocation with an object header (~16 bytes) versus 4 bytes for a raw `int`. Plus `HashMap.Node` wrappers, hash computation per operation, and pointers scattered across the heap causing cache misses.

`int[26]` is 104 contiguous bytes the CPU prefetches in one go. Zero boxing, zero hashing. Same `O(n)`, dramatically better constants.

### The recurring lesson: bounded values → index, don't hash

Third time this has come up:

| Problem | Bound | Structure |
|---|---|---|
| Day 1 — Intersection | values `0..1000` | `boolean[1001]` |
| Day 4 — Anagram | lowercase `a-z` | `int[26]` |
| Unicode follow-up | unbounded | `HashMap` |

**Whenever the constraints bound the *values* (not just the length), consider indexing by value instead of hashing.** The moment that bound disappears, fall back to a hash map.

### Single pass when lengths are equal

Once the length guard passes, you don't need separate loops. Increment for `s` and decrement for `t` in the same iteration. Halves the loop overhead and is arguably clearer about what's being compared.

### Pattern to reuse

Frequency counting with a fixed-size array solves: Group Anagrams (LC 49 — use the count array as a map key), Find All Anagrams in a String (LC 438 — sliding window over the counts), First Unique Character (LC 387), Ransom Note (LC 383).

LC 438 is the natural next step — same counting, plus a sliding window.
