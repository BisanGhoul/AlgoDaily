# Day 2 — Fizz Buzz

**LeetCode 412** · Easy
**Topics:** modular arithmetic · conditional logic · list construction · extensibility

---

## Problem

Given an integer `n`, return a 1-indexed string array `answer` where:

- `"FizzBuzz"` if divisible by both 3 and 5
- `"Fizz"` if divisible by 3
- `"Buzz"` if divisible by 5
- the number as a string otherwise

```
n = 3   ->  ["1","2","Fizz"]
n = 5   ->  ["1","2","Fizz","4","Buzz"]
n = 15  ->  ["1","2","Fizz","4","Buzz","Fizz","7","8","Fizz","Buzz","11","Fizz","13","14","FizzBuzz"]
```

**Constraints:** `1 <= n <= 10^4`

---

## Questions to Ask First

- Is the output 1-indexed or 0-indexed? (Values start at 1; the list itself is 0-indexed. Easy off-by-one.)
- Should the check order matter — must divisible-by-15 be tested before 3 and 5?
- Are more rules likely to be added later? This changes which structure is correct.
- Return `List<String>` or `String[]`?

---

## Solution

```java
package algodaily;

import java.util.ArrayList;
import java.util.List;

public class FizzBuzz {

    public List<String> fizzBuzz(int n) {
        List<String> answer = new ArrayList<>(n);
        for (int i = 1; i <= n; i++) {
            if (i % 15 == 0)     answer.add("FizzBuzz");
            else if (i % 3 == 0) answer.add("Fizz");
            else if (i % 5 == 0) answer.add("Buzz");
            else                 answer.add(String.valueOf(i));
        }
        return answer;
    }
}
```

Order matters: `% 15` must come first. If `% 3` is checked first, every multiple of 15 exits early as `"Fizz"` and `"FizzBuzz"` never appears.

---

## Edge Cases

| Input | Result | Why |
|---|---|---|
| `n = 1` | `["1"]` ✅ | Loop runs once, no divisibility hits |
| `n = 3` | `[...,"Fizz"]` ✅ | First Fizz at the boundary |
| `n = 15` | ends `"FizzBuzz"` ✅ | First FizzBuzz — the case a wrong check order breaks |
| `n = 14` | never hits FizzBuzz ✅ | Confirms 15 isn't triggered early |
| `n = 10^4` | ✅ | 10,000 strings; fine, but pre-size the list |
| `n = 0` or negative | `[]` | Excluded by constraints; loop simply doesn't run |

**Test `n = 15` specifically.** Anything below it passes even with the branches in the wrong order.

---

## Complexity

**Time — `O(n)`**

One pass, `n` iterations. Each does up to three modulo operations (constant) and one list append (amortized constant).

Strictly, converting `i` to a string costs `O(log i)` since it writes one char per digit — so total work is `O(n log n)`. Almost everyone states `O(n)` because digit count is capped at 5 here. Worth knowing the distinction; not worth arguing about.

**Space — `O(n)`**

The output list holds `n` strings. That's required output, not auxiliary — auxiliary space is `O(1)`. Say it that way; it shows you know the difference.

---

## Notes

**The bug that wasn't: `list.add(i - 1, "Fizz")`**

`ArrayList` has two overloads:
- `add(E element)` — appends to the end
- `add(int index, E element)` — *inserts* at `index`, shifting everything after it right

The original called the insert version. It works only because `i - 1` always happens to equal the current size, so `System.arraycopy` shifts zero elements. But it hand-maintains an invariant that `add(E)` maintains for free, and an off-by-one throws `IndexOutOfBoundsException` rather than producing a wrong answer.

Use `add(E)` when appending. Reserve `add(int, E)` for genuine mid-list insertion.

**`String.valueOf(i)` over `i + ""`.** Both work. The concat version routes through `StringConcatFactory` (Java 9+) or a `StringBuilder`; `String.valueOf` calls `Integer.toString` directly and states the intent.

**Pre-size the list.** `new ArrayList<>()` starts at capacity 10 and grows by ~1.5× each time it fills. For n = 10⁴ that's roughly 14 reallocations, each copying the whole backing array. `new ArrayList<>(n)` allocates once.

**`i % 15` vs `i % 3 == 0 && i % 5 == 0`.** Identical, since 3 and 5 are coprime — anything divisible by both is divisible by 15.

---

## The Extensibility Answer

This is what the problem is actually testing. The if-chain hardcodes the *combination* `"FizzBuzz"`, so adding a seventh rule needs `2^7` branches. Build the string instead:

```java
public List<String> fizzBuzz(int n) {
    List<String> answer = new ArrayList<>(n);
    StringBuilder sb = new StringBuilder();

    for (int i = 1; i <= n; i++) {
        sb.setLength(0);                       // reuse, don't reallocate
        if (i % 3 == 0) sb.append("Fizz");
        if (i % 5 == 0) sb.append("Buzz");
        answer.add(sb.isEmpty() ? String.valueOf(i) : sb.toString());
    }
    return answer;
}
```

Each rule is now one independent `if`. Adding "Jazz" for 7 is one line, and `"FizzBuzzJazz"` composes automatically.

Fully data-driven, if pushed further:

```java
int[] divisors = {3, 5, 7};
String[] words  = {"Fizz", "Buzz", "Jazz"};
```

Then rules are configuration, not code. Mentioning this progression — hardcoded → composed → data-driven — is the strongest thing you can do on an otherwise trivial problem.

**Pattern to reuse.** "Compose the answer instead of enumerating combinations" shows up in permission flags, feature toggles, and any rules engine.
