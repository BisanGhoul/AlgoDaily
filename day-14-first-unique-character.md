# Day 13 — First Unique Character in a String

**LeetCode 387** · Easy
**Topics:** hash map · frequency counting · insertion order · direct addressing

---

## Problem

Find the first non-repeating character in `s` and return its index. Return `-1` if none exists.

```
"leetcode"      ->  0   ('l')
"loveleetcode"  ->  2   ('v')
"aabb"          ->  -1
```

**Constraints**
- `1 <= s.length <= 10^5`
- Lowercase English letters only

---

## Questions to Ask First

- Return the index or the character itself? (Index.)
- What if every character repeats? (Return −1.)
- Is the alphabet bounded? (Yes — lowercase only. Opens up `int[26]`.)
- Case sensitive? (Only lowercase, so it doesn't come up.)
- Called once, or repeatedly on the same string? (Repeated calls would favour caching the counts.)

---

## Solutions

### 1 — Count, then scan the string ← recommended

```java
package algodaily;

import java.util.HashMap;
import java.util.Map;

public class FirstUniqueCharacter {

    public int firstUniqChar(String s) {
        Map<Character, Integer> counts = new HashMap<>();
        for (int i = 0; i < s.length(); i++) {
            counts.merge(s.charAt(i), 1, Integer::sum);
        }

        for (int i = 0; i < s.length(); i++) {
            if (counts.get(s.charAt(i)) == 1) return i;
        }
        return -1;
    }
}
```

**Time `O(n)` · Space `O(1)`** — at most 26 keys

Pass one counts every character. Pass two walks the **string**, so `i` is the real index. Insertion order is irrelevant here, which is why a plain `HashMap` suffices.

---

### 2 — LinkedHashMap + `indexOf`

```java
public int firstUniqChar(String s) {
    Map<Character, Integer> counts = new LinkedHashMap<>();
    for (int i = 0; i < s.length(); i++) {
        counts.merge(s.charAt(i), 1, Integer::sum);
    }

    for (Map.Entry<Character, Integer> e : counts.entrySet()) {
        if (e.getValue() == 1) {
            return s.indexOf(e.getKey());
        }
    }
    return -1;
}
```

**Time `O(n)` · Space `O(1)`**

`LinkedHashMap` preserves order of first insertion, so iterating it finds the first unique character directly. `indexOf` then recovers its position.

`indexOf` is safe here only because the character is unique — its first occurrence is its only occurrence.

---

### 3 — `int[26]` ← fastest

```java
public int firstUniqChar(String s) {
    int[] counts = new int[26];
    for (int i = 0; i < s.length(); i++) {
        counts[s.charAt(i) - 'a']++;
    }

    for (int i = 0; i < s.length(); i++) {
        if (counts[s.charAt(i) - 'a'] == 1) return i;
    }
    return -1;
}
```

**Time `O(n)` · Space `O(1)`**

`s.charAt(i) - 'a'` maps `'a'`→0 … `'z'`→25. Same direct-addressing trick as Days 1 and 4 — no boxing, no hashing, contiguous memory.

---

### 4 — `indexOf` / `lastIndexOf` over the alphabet

```java
public int firstUniqChar(String s) {
    int result = Integer.MAX_VALUE;
    for (char c = 'a'; c <= 'z'; c++) {
        int first = s.indexOf(c);
        if (first != -1 && first == s.lastIndexOf(c)) {
            result = Math.min(result, first);
        }
    }
    return result == Integer.MAX_VALUE ? -1 : result;
}
```

**Time `O(26n)` = `O(n)` · Space `O(1)`**

If a character's first and last occurrence are the same position, it appears exactly once. Loops the alphabet — 26 iterations — rather than the string.

---

## Edge Cases

| Input | Result | Why |
|---|---|---|
| `"leetcode"` | `0` ✅ | First character is unique |
| `"loveleetcode"` | `2` ✅ | `'v'` — first two repeat |
| `"aabb"` | `-1` ✅ | Nothing unique |
| `"a"` | `0` ✅ | Single character |
| `"aabcc"` | `2` ✅ | **The case that breaks map-position counting** |
| `"z"` | `0` ✅ | Last letter of the alphabet — `'z' - 'a'` = 25, in bounds |
| `"aab"` | `2` ✅ | Unique character at the end |
| All identical | `-1` ✅ | |
| 10⁵ chars | ✅ | Linear |

---

## Notes

### Don't count your position in the map

The tempting shortcut is to track an index while iterating the map and return it when a count of 1 shows up. That gives the position **in the map**, not in the string.

`"aabcc"` builds:

```
a:2  b:1  c:2
 0    1    2    ← map position
```

`'b'` sits at map position 1, but its string index is 2. The map collapses duplicates, so positions shift — `'a'` occupies two string slots and one map slot.

Either scan the original string (Solution 1) or use `indexOf` to recover the real position (Solution 2).

### `LinkedHashMap` only helps if you use the order

Storing counts and then scanning the string makes insertion order irrelevant — `HashMap` is the right choice. `LinkedHashMap` earns its place only when you iterate the map itself to find "first," as in Solution 2.

### `charAt` over `toCharArray()`

`toCharArray()` allocates a full copy — 10⁵ characters copied for nothing. `charAt` reads in place. Same complexity, free to avoid. The second loop needs `i` anyway, so an indexed loop is natural in both passes.

### Watch `Integer` comparison

`e.getValue() == 1` works because the right side is an `int` literal, which forces unboxing. But `Integer == Integer` compares **references**, and Java only caches −128 to 127:

```java
Integer a = 1000, b = 1000;
a == b          // false
a.equals(b)     // true
```

Use `.equals()` or compare against a primitive.

### Two passes, not one

You can't answer in a single pass — a character at index 0 might repeat at index 10⁵. You must see the whole string before knowing what's unique. Two passes is optimal.

### Why `int[26]` wins

Every `Character` key and `Integer` value is a heap allocation with an object header. `int[26]` is 104 contiguous bytes with no boxing and no hash computation. Fourth time this tradeoff has come up:

| Day | Problem | Bound | Structure |
|---|---|---|---|
| 1 | Intersection | values `0..1000` | `boolean[1001]` |
| 4 | Valid Anagram | lowercase | `int[26]` |
| 9 | Design HashMap | keys `0..10^6` | direct index or buckets |
| 13 | First Unique Char | lowercase | `int[26]` |

### Pattern to reuse

"Count everything, then find the first match" also solves: LC 383 (Ransom Note), LC 451 (Sort Characters By Frequency), LC 1189 (Maximum Number of Balloons), LC 2062 (Count Vowel Substrings).
