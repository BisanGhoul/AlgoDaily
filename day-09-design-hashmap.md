# Day 9 — Design HashMap

**LeetCode 706** · Easy
**Topics:** hash table · collision resolution · chaining · linked list · hash function

---

## Problem

Design a HashMap without built-in hash libraries. Implement:
- `MyHashMap()` — initialize empty
- `put(key, value)` — insert or update
- `get(key)` — return value or `-1` if not found
- `remove(key)` — delete the key-value pair

```
put(1, 1)    -> {1: 1}
put(2, 2)    -> {1: 1, 2: 2}
get(1)       -> 1
get(3)       -> -1
put(2, 1)    -> {1: 1, 2: 1}
remove(2)    -> {1: 1}
```

**Constraints**
- `0 <= key, value <= 10^6`
- At most 10⁴ calls to `put`, `get`, `remove`

---

## Questions to Ask First

- Is key uniqueness guaranteed? (Yes — one value per key.)
- What does a missing key return? (−1.)
- Can I use built-in `HashMap`? (No — design one.)
- What's the expected ratio of calls to keys? (Sparse — 10⁴ calls, keys up to 10⁶.)

---

## Solutions

### 1 — Direct addressing (exploit the constraints)

```java
public class MyHashMap {
    private int[] map;

    public MyHashMap() {
        map = new int[1000001];
        Arrays.fill(map, -1);
    }

    public void put(int key, int value) {
        map[key] = value;
    }

    public int get(int key) {
        return map[key];
    }

    public void remove(int key) {
        map[key] = -1;
    }
}
```

**Time: all `O(1)` · Space `O(10^6)` — allocated once**

Keys are capped at 10⁶, so use the key itself as the array index. No hashing, no collisions. All operations are single array accesses.

Works and passes, but doesn't answer "design a HashMap" — you exploited the constraint instead of solving the general problem.

### 2 — Hash table with chaining ← the real design

```java
package algodaily;

public class MyHashMap {
    private static final int BUCKETS = 769;  // prime number
    private Bucket[] buckets;

    public MyHashMap() {
        buckets = new Bucket[BUCKETS];
        for (int i = 0; i < BUCKETS; i++) {
            buckets[i] = new Bucket();
        }
    }

    public void put(int key, int value) {
        int index = hash(key);
        buckets[index].put(key, value);
    }

    public int get(int key) {
        int index = hash(key);
        return buckets[index].get(key);
    }

    public void remove(int key) {
        int index = hash(key);
        buckets[index].remove(key);
    }

    private int hash(int key) {
        return key % BUCKETS;
    }

    private static class Bucket {
        private Node head = new Node(0, 0);  // dummy node

        public void put(int key, int value) {
            Node prev = findPrev(key);
            if (prev.next != null && prev.next.key == key) {
                prev.next.value = value;  // update existing
            } else {
                prev.next = new Node(key, value, prev.next);  // insert new
            }
        }

        public int get(int key) {
            Node prev = findPrev(key);
            if (prev.next != null && prev.next.key == key) {
                return prev.next.value;
            }
            return -1;
        }

        public void remove(int key) {
            Node prev = findPrev(key);
            if (prev.next != null && prev.next.key == key) {
                prev.next = prev.next.next;
            }
        }

        private Node findPrev(int key) {
            Node curr = head;
            while (curr.next != null && curr.next.key < key) {
                curr = curr.next;
            }
            return curr;
        }

        private static class Node {
            int key, value;
            Node next;

            Node(int key, int value) {
                this.key = key;
                this.value = value;
            }

            Node(int key, int value, Node next) {
                this.key = key;
                this.value = value;
                this.next = next;
            }
        }
    }
}
```

**Time: `O(1)` average, `O(n)` worst case · Space `O(k)` where k is unique keys stored**

Four key decisions:

1. **Bucket count — 769** (prime). `key % prime` distributes patterned keys better than `key % power-of-2`.

2. **Hash function — `key % BUCKETS`**. Maps any key to a valid bucket index.

3. **Collision resolution — chaining**. Each bucket holds a linked list. Two keys with the same hash coexist in the list.

4. **Dummy head node**. Makes remove uniform — no special case for deleting the first element. Finding the node *before* the target lets you unlink it in one operation.

**`findPrev` walks the chain until it overshoots the target key.** Returns the node *before* where the key should be (or before the end if the key isn't there). All three operations — put, get, remove — use this.

---

## Edge Cases

| Operation | Input | Result | Why |
|---|---|---|---|
| `get` on empty | any key | `-1` ✅ | Dummy head's `next` is null |
| `put` + `get` | same key twice | latest value | `put` overwrites |
| `remove` then `get` | removed key | `-1` ✅ | `findPrev` stops before null |
| Collision | keys 1 and 770 | both stored ✅ | Same bucket, different list nodes |
| `remove` non-existent | any key | no-op ✅ | `if` check fails, nothing happens |
| Duplicate `remove` | same key twice | no-op ✅ | Second call finds nothing |

**Collisions are inevitable.** With 769 buckets and up to 10⁴ keys, pigeonhole guarantee says some collisions occur. The chain handles it.

---

## Complexity Comparison

| Approach | Time | Space | Real design |
|---|---|---|---|
| **Direct addressing** | `O(1)` | `O(10^6)` | ✗ — exploits constraint |
| **Chaining** | `O(1)` avg, `O(n)` worst | `O(k)` | ✅ — actual hash table |

Direct addressing is faster here, but only because keys are bounded. The real design scales to any key range.

---

## Notes

**Why a dummy head node.** Without it, removing the first element is a special case:

```java
if (head.key == key) head = head.next;  // special case
else while (curr.next != null) ...       // general case
```

With a dummy, `findPrev` returns the dummy if the target is first, and `prev.next = prev.next.next` unlinks the first element the same way it unlinks any other.

**Prime bucket count scatters keys better.** `1` and `770` both hash to 1 mod 1000, but mod 769 they go to 1 and 1 (collision) vs other distributions that spread more evenly. Primes near 2× expected unique keys are the standard choice.

**−1 is safe as not-found.** Values are guaranteed non-negative, so −1unambiguously signals "not in map." If values could be negative, use a wrapper or a boolean flag.

**`findPrev` assumes keys in the chain are sorted.** The loop `while (curr.next.key < key)` relies on this. The code maintains it: `put` only inserts into gaps, never out of order. If you remove the ordering assumption, `findPrev` becomes a full linear search — still `O(n)` worst case, but with higher average constants.

**Time complexity in practice.** With 10⁴ calls and 769 buckets, average chain length is 10⁴ / 769 ≈ 13. So average-case all operations are `O(1)` (one hash, then a short chain walk). Worst case is `O(n)` — all keys hash to the same bucket — but vanishingly unlikely with a prime bucket count.

**Pattern to reuse.** Chaining is one collision strategy. Others: open addressing (probe for the next free slot), double hashing (use a second hash function), or cuckoo hashing (multiple tables). Chaining is simplest and what production hash tables use (Java's `HashMap`, Python's `dict`).

**Related problems:** LC 705 (Design HashSet — identical structure, no values), LC 1206 (Design Skiplist — ordered alternative to chaining).
