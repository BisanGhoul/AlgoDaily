# Day 0 — Reverse a String

**Topics:** two pointers · strings · string immutability · in-place array manipulation

---

## Problem

Reverse a string without using the built-in reverse method.

```
reverseString("jake")            ->  "ekaj"
reverseString("reverseastring")  ->  "gnirtsaesrever"
```

**Constraints:** no `#reverse()` / `[::-1]`. Target `O(n)` time, `O(1)` space.

---

## Questions to Ask First

- How should `null` be handled — return `null`, return `""`, or throw?
- ASCII only, or full Unicode? (Matters: emoji are stored as surrogate pairs.)
- Can I return a new `String`, or must it be reversed in place? Determines whether `O(1)` space is possible in Java.
- Does "no built-in reverse" also rule out `StringBuilder.reverse()`?

---

## Solution

```java
package algodaily;

public class ReverseString {

    public static void main(String[] args) {
        System.out.println(reverse("hello"));
    }

    public static String reverse(String str) {
        int left = 0, right = str.length() - 1;
        char[] revArr = str.toCharArray();

        while (left < right) {
            char temp = revArr[left];
            revArr[left] = revArr[right];
            revArr[right] = temp;
            left++;
            right--;
        }

        return new String(revArr);
    }
}
```

One pointer at each end, swap, step both inward. Stop when they meet or cross — every pair has been swapped exactly once.

```
h e l l o    left=0 right=4  ->  swap h/o
o e l l h    left=1 right=3  ->  swap e/l
o l l e h    left=2 right=2  ->  stop
```

---

## Edge Cases

| Input                  | Result     | Why                                                                     |
| ---------------------- | ---------- | ----------------------------------------------------------------------- |
| `""`                   | `""` ✅    | `right = -1`, so `left < right` is false — loop never runs              |
| `"a"`                  | `"a"` ✅   | `left == right == 0`, loop never runs                                   |
| `"aa"`                 | `"aa"` ✅  | One swap, identical chars, no visible change                            |
| Odd length             | ✅         | Middle char untouched — already in its final position                   |
| `null`                 | ❌ NPE     | Thrown by `str.length()` before anything else runs                      |
| `"ab😀"`               | ❌ Garbage | Surrogate pair gets split and flipped                                   |
| Palindrome `"racecar"` | ✅         | Returns identical string — easy to mistake for a no-op bug when testing |

---

## Complexity

**Time — `O(n)`**

`toCharArray()` copies n chars, the loop runs n/2 iterations of constant work, `new String()` copies n chars again. Three linear passes → `O(n)`. The loop being n/2 rather than n is a constant factor and drops out.

**Space — `O(n)`, not `O(1)`**

`revArr` holds n chars and `new String(revArr)` allocates n more. Java's `String` is immutable, so any method with signature `String reverse(String)` _must_ allocate — `O(1)` is unreachable here.

The swapping algorithm itself is `O(1)` auxiliary. Given a mutable `char[]`, only `temp`, `left`, and `right` are extra:

```java
public static void reverse(char[] arr) {
    int left = 0, right = arr.length - 1;
    while (left < right) {
        char temp = arr[left];
        arr[left] = arr[right];
        arr[right] = temp;
        left++;
        right--;
    }
}
```

> **Say this in an interview:** "The algorithm is `O(1)` auxiliary space, but Java's immutable strings force an `O(n)` copy. Give me a `char[]` and it's `O(1)` for real."

---

## Notes

**Missing null guard.** `reverse(null)` throws NPE on `str.length()`. Add a check and pick a contract deliberately:

```java
if (str == null) throw new IllegalArgumentException("input must not be null");
```

**Unicode trap.** `toCharArray()` splits into UTF-16 code units. Characters outside the BMP are surrogate pairs; reversing flips them and produces garbage. Naming this in an interview is usually enough — you rarely have to fix it.

**Never build the result with `+=`.** `result += str.charAt(i)` in a loop is `O(n²)` — every concat reallocates and recopies. Most common Java interview performance mistake.

**Pattern to reuse.** Converging pointers also solves: palindrome check, two-sum on sorted array, container with most water, reverse vowels only.
