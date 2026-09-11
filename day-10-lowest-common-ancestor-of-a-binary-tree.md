# Day 10 — Lowest Common Ancestor of a Binary Tree

**LeetCode 236** · Medium · [Link](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/)

## Topics Covered

- Binary tree
- Bottom-up DFS (post-order)
- Recursion returning a node instead of a scalar
- Hash map + set (iterative variant)

## Problem Statement

Given the root of a binary tree and two nodes `p` and `q` in it, return their lowest common ancestor — the deepest node that has both `p` and `q` as descendants. A node counts as a descendant of itself, so if `p` is an ancestor of `q`, the answer is `p`.

Not a BST. Node values are unique, and both `p` and `q` are guaranteed to exist in the tree.

## Questions to Ask First

- Is it a BST? If yes, the value-comparison walk solves it in `O(h)` — different problem (LC 235).
- Are `p` and `q` guaranteed to be in the tree? If not, the recursion silently returns a wrong node and needs found-flags plus a post-check (LC 1644).
- Am I given node references or just values? References allow `==`; values force `.val` comparison.
- Do nodes have parent pointers? If so it becomes an upward two-pointer merge with no root needed (LC 1650).
- Can a node be its own ancestor? Yes here — that's what makes the early return correct.

## Solution

### Variant 1 — Recursive bottom-up DFS (preferred)

Each call returns: the LCA if both targets are in this subtree, the single target if only one is, `null` if neither. The parent only ever checks null vs non-null, so the overloaded meaning never causes a problem. Both sides coming back non-null means the targets split at this node — that's the answer.

```java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if (root == null || root == p || root == q) return root;
        TreeNode l = lowestCommonAncestor(root.left, p, q);
        TreeNode r = lowestCommonAncestor(root.right, p, q);
        if (l != null && r != null) return root;
        return l != null ? l : r;
    }
}
```

The base case returns immediately on hitting a target without searching below it. That is what makes the ancestor case (`p` above `q`) come out right for free.

**Time** `O(n)` — each node visited once.
**Space** `O(h)` — recursion stack; `O(log n)` balanced, `O(n)` on a skewed tree.

### Variant 2 — Iterative with parent map

Walk the tree once recording every node's parent, climb from `p` collecting its ancestors into a set, then climb from `q` and return the first node already in the set. Useful when recursion depth is a concern or when many LCA queries share one tree.

```java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        Map<TreeNode, TreeNode> par = new HashMap<>();
        Deque<TreeNode> st = new ArrayDeque<>();
        par.put(root, null);
        st.push(root);
        while (!par.containsKey(p) || !par.containsKey(q)) {
            TreeNode n = st.pop();
            if (n.left != null) { par.put(n.left, n); st.push(n.left); }
            if (n.right != null) { par.put(n.right, n); st.push(n.right); }
        }
        Set<TreeNode> anc = new HashSet<>();
        for (TreeNode n = p; n != null; n = par.get(n)) anc.add(n);
        for (TreeNode n = q; n != null; n = par.get(n)) if (anc.contains(n)) return n;
        return null;
    }
}
```

**Time** `O(n)`.
**Space** `O(n)` — parent map plus ancestor set.

## Edge Cases

| Case | Input shape | Expected | Handled by |
|---|---|---|---|
| One node is the root | `p == root` | `root` | Base case fires at root |
| `p` is an ancestor of `q` | `p = 5`, `q = 2` under `5` | `5` | Early return, never descends to `q` |
| Targets in opposite subtrees | `p = 6`, `q = 1` | split node | `l != null && r != null` |
| Both targets in the same subtree | both under `5` | node inside that subtree | Null side bubbles the answer up |
| Skewed tree (single chain) | `n` nodes in a line | correct, `O(n)` stack | Works; stack depth is the risk |
| Two-node tree | root + one child | root | Base case + null sibling |
| Target not in the tree | — | undefined | Not handled; needs found-flags (LC 1644) |

## Complexity Summary

| Variant | Time | Space |
|---|---|---|
| Recursive bottom-up DFS | `O(n)` | `O(h)` |
| Iterative parent map | `O(n)` | `O(n)` |

## Notes

- `root == p` is reference equality. Safe here because LeetCode passes actual node references; switch to `.val` if only values are given.
- Return the node, not `node.val` — the signature wants identity.
- Building both root-to-target paths and comparing them also works in `O(n)`, but it's far more code and more to get wrong under pressure.
- Related problems: LC 235 (BST, `O(h)` value walk), LC 1650 (parent pointers, `O(1)` space), LC 1644 (existence not guaranteed), LC 1676 (n targets via a set in the base case).
