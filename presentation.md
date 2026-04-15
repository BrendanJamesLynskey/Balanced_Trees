# Balanced Trees

**Computer Science Fundamentals Series**

AVL trees · Red-Black trees · Rotations · 2-3 trees · Balance factor · Persistent trees

*Mid-level software engineer track -- 20 slides*

---

## Table of Contents

1. [Why Balancing Matters](#slide-02--why-balancing-matters)
2. [BST Recap & the Degenerate Case](#slide-03--bst-recap--the-degenerate-case)
3. [AVL Trees -- Balance Factor](#slide-04--avl-trees--balance-factor)
4. [AVL Rotations -- LL & RR](#slide-05--avl-rotations--ll--rr)
5. [AVL Rotations -- LR & RL](#slide-06--avl-rotations--lr--rl)
6. [AVL Insertion & Deletion](#slide-07--avl-insertion--deletion)
7. [Red-Black Trees -- Properties](#slide-08--red-black-trees--properties)
8. [Red-Black Trees -- Colouring Rules](#slide-09--red-black-trees--colouring-rules)
9. [Red-Black Insertion -- Cases & Uncle Colour](#slide-10--red-black-insertion--cases--uncle-colour)
10. [Red-Black Deletion](#slide-11--red-black-deletion)
11. [AVL vs Red-Black -- When to Choose Which](#slide-12--avl-vs-red-black--when-to-choose-which)
12. [2-3 Trees](#slide-13--2-3-trees)
13. [2-3-4 Trees](#slide-14--2-3-4-trees)
14. [Left-Leaning Red-Black Trees](#slide-15--left-leaning-red-black-trees)
15. [AA Trees](#slide-16--aa-trees)
16. [Scapegoat Trees](#slide-17--scapegoat-trees)
17. [Weight-Balanced Trees](#slide-18--weight-balanced-trees)
18. [Persistent Balanced Trees](#slide-19--persistent-balanced-trees)
19. [Summary & Comparison Table](#slide-20--summary--comparison-table)

---

## Slide 02 -- Why Balancing Matters

### The promise of binary search trees

A BST offers `O(log n)` search, insert, and delete -- but only when the tree is balanced. The height determines the worst-case cost of every operation.

- **Balanced tree** -- height `O(log n)`, operations are fast
- **Degenerate tree** -- height `O(n)`, operations degrade to a linked list
- Balancing guarantees a logarithmic height bound regardless of insertion order
- Without balancing, sorted input produces the worst case every time

### The cost of imbalance

| Scenario | Height | Search cost |
|----------|--------|-------------|
| Perfectly balanced (n = 1M) | ~20 | 20 comparisons |
| Random insertion (n = 1M) | ~40 | 40 comparisons |
| Sorted insertion (n = 1M) | 1,000,000 | 1M comparisons |

> **Key insight:** self-balancing trees pay a small rebalancing cost per mutation to guarantee `O(log n)` height at all times. The amortised overhead is negligible compared to the worst-case degenerate scenario.

---

## Slide 03 -- BST Recap & the Degenerate Case

### Binary Search Tree invariant

For every node `x`: all keys in the left subtree are less than `x.key`, and all keys in the right subtree are greater.

```
Balanced BST:          Degenerate BST (sorted insert):

       50                  10
      /  \                   \
    30    70                  20
   / \   / \                   \
  20 40 60 80                   30
                                 \
                                  40
                                   \
                                    50
```

### Why sorted input kills a plain BST

Inserting `[10, 20, 30, 40, 50]` into a naive BST produces a right-skewed chain. Every operation becomes `O(n)`.

- **Search** -- must traverse the entire chain
- **Insert** -- always appends at the deepest point
- **Delete** -- restructuring offers no height reduction

> Self-balancing trees solve this by restructuring after each mutation. The restructuring mechanisms differ -- rotations, colour flips, node splits -- but the goal is always the same: keep `h = O(log n)`.

---

## Slide 04 -- AVL Trees -- Balance Factor

### Definition

An AVL tree (Adelson-Velsky & Landis, 1962) is a BST where, for every node, the heights of the left and right subtrees differ by at most 1.

### Balance factor

```
BF(node) = height(left subtree) - height(right subtree)
```

| Balance factor | Meaning |
|---------------|---------|
| `BF = -1` | Right-heavy by one level -- acceptable |
| `BF =  0` | Perfectly balanced at this node |
| `BF = +1` | Left-heavy by one level -- acceptable |
| `BF = -2` or `+2` | **Violation** -- requires rotation to fix |

### Height guarantee

An AVL tree with `n` nodes has height at most `1.44 * log2(n)`. This is tighter than Red-Black trees, making AVL trees faster for lookup-heavy workloads.

> AVL trees are **strictly balanced** -- they tolerate less imbalance than Red-Black trees. This means more rotations on insert/delete, but faster searches.

---

## Slide 05 -- AVL Rotations -- LL & RR

### Single rotations

When the imbalance is on the outer edge (left-left or right-right), a single rotation restores balance.

### LL Rotation (Right Rotation)

```
  BF=+2                  BF=0
    z                      y
   / \       rotate       / \
  y   T4    right(z)     x   z
 / \        -------->   / \ / \
x   T3                T1 T2 T3 T4
/ \
T1 T2
```

Node `y` becomes the new root. `z` becomes `y`'s right child. `T3` moves from `y`'s right to `z`'s left.

### RR Rotation (Left Rotation)

```
BF=-2                   BF=0
  z                       y
 / \       rotate        / \
T1  y     left(z)       z   x
   / \    -------->    / \ / \
  T2  x             T1 T2 T3 T4
     / \
    T3 T4
```

Mirror of LL. Node `y` becomes root. `z` becomes `y`'s left child.

> Single rotations run in `O(1)` -- they reassign at most three pointers and update two heights.

---

## Slide 06 -- AVL Rotations -- LR & RL

### Double rotations

When the imbalance is on the inner edge (left-right or right-left), a single rotation is insufficient. Two rotations are required.

### LR Rotation (Left-Right)

```
Step 1: Left rotate y         Step 2: Right rotate z

    z            z                    x
   / \          / \                  / \
  y   T4  ->  x   T4   ->         y   z
 / \          / \                 / \ / \
T1  x        y   T3             T1 T2 T3 T4
   / \      / \
  T2 T3    T1 T2
```

First rotate left at `y` (making `x` the left child of `z`), then rotate right at `z`.

### RL Rotation (Right-Left)

Mirror of LR. First rotate right at `y`, then rotate left at `z`.

```
Step 1: Right rotate y        Step 2: Left rotate z

  z              z                    x
 / \            / \                  / \
T1  y    ->   T1   x     ->       z   y
   / \            / \             / \ / \
  x   T4        T2  y           T1 T2 T3 T4
 / \               / \
T2 T3             T3 T4
```

> **How to pick the rotation:** check the balance factor of the violating node and its heavy child. Same sign -> single rotation. Different signs -> double rotation.

---

## Slide 07 -- AVL Insertion & Deletion

### Insertion algorithm

1. Standard BST insert at the correct leaf position
2. Walk back up the path to the root, updating heights
3. At the first node with `|BF| = 2`, perform the appropriate rotation (LL, RR, LR, or RL)
4. At most **one rotation** (single or double) is needed per insertion

### Deletion algorithm

1. Standard BST delete (handle 0, 1, or 2 children; in-order successor for 2-child case)
2. Walk back up the path, updating heights
3. At each node with `|BF| = 2`, perform the appropriate rotation
4. Unlike insertion, deletion may require **O(log n) rotations** -- one at each ancestor

### Complexity

| Operation | Average | Worst case |
|-----------|---------|------------|
| Search | `O(log n)` | `O(log n)` |
| Insert | `O(log n)` | `O(log n)` -- 1 rotation |
| Delete | `O(log n)` | `O(log n)` -- up to `O(log n)` rotations |
| Space | `O(n)` | `O(n)` -- one height/BF per node |

> AVL trees guarantee at most one rotation on insert but may need multiple rotations on delete. This asymmetry is why Red-Black trees are sometimes preferred for write-heavy workloads.

---

## Slide 08 -- Red-Black Trees -- Properties

### Definition

A Red-Black tree is a BST augmented with a colour bit per node (red or black) that satisfies five properties guaranteeing approximate balance.

### The five properties

1. **Every node is either red or black**
2. **The root is black**
3. **Every leaf (NIL sentinel) is black**
4. **If a node is red, both its children are black** (no two consecutive red nodes)
5. **All paths from any node to its descendant NIL leaves contain the same number of black nodes** (black-height)

### Height guarantee

A Red-Black tree with `n` internal nodes has height at most `2 * log2(n + 1)`. Looser than AVL's `1.44 * log2(n)`, but still `O(log n)`.

### Black-height

The **black-height** of a node is the number of black nodes on any path from that node (exclusive) down to a NIL leaf. Property 5 ensures this is well-defined.

> Red-Black trees trade slightly worse search performance for fewer rotations on insert/delete -- at most 2 rotations per insert, at most 3 per delete.

---

## Slide 09 -- Red-Black Trees -- Colouring Rules

### Why colours?

The colour constraints encode balance information implicitly. Property 4 (no consecutive reds) and Property 5 (equal black-height) together force the longest path to be at most twice the shortest.

### Intuition via 2-3-4 mapping

Every Red-Black tree is isomorphic to a 2-3-4 tree:

| 2-3-4 node | Red-Black equivalent |
|------------|---------------------|
| 2-node | Single black node |
| 3-node | Black node with one red child |
| 4-node | Black node with two red children |

### Colour constraints at a glance

- **New nodes are always inserted as red** -- this preserves black-height
- **Red violations** (two consecutive reds) are fixed by recolouring or rotations
- **Black violations** only arise during deletion and require more complex fixups

> The colour scheme is not arbitrary -- it is a compact encoding of the 2-3-4 tree structure into a binary tree. Understanding this mapping makes Red-Black tree operations far more intuitive.

---

## Slide 10 -- Red-Black Insertion -- Cases & Uncle Colour

### Insertion procedure

1. Standard BST insert; colour the new node **red**
2. If the parent is black, done -- no violation
3. If the parent is red, check the **uncle** (parent's sibling)

### Case analysis (parent is red)

| Case | Uncle colour | Fix |
|------|-------------|-----|
| **Case 1** | Uncle is **red** | Recolour parent and uncle to black, grandparent to red. Move violation up to grandparent and repeat. |
| **Case 2** | Uncle is **black**, node is inner child | Rotate node to outer position (converts to Case 3) |
| **Case 3** | Uncle is **black**, node is outer child | Rotate grandparent, swap colours of parent and grandparent. Done. |

### Rotation count

- **At most 2 rotations** per insertion (Case 2 + Case 3)
- Recolouring (Case 1) may propagate up `O(log n)` times but involves no rotations
- After any rotation, the fix-up terminates immediately

> Compare with AVL: AVL does at most 1 rotation on insert but must always walk to the root updating heights. Red-Black insertion can terminate early when no further violations exist.

---

## Slide 11 -- Red-Black Deletion

### Overview

Deletion is the most complex Red-Black operation. Removing a black node reduces the black-height on that path, creating a "double-black" deficit.

### Procedure

1. Standard BST delete (find in-order successor if needed)
2. If the removed/replaced node was red, done -- black-height unchanged
3. If black, the replacement node carries an extra "black" (double-black)
4. Fix up the double-black by case analysis

### Double-black fix-up cases

| Case | Sibling colour | Sibling's children | Fix |
|------|---------------|-------------------|-----|
| **Case 1** | Red | -- | Rotate sibling up, recolour. Converts to Case 2, 3, or 4. |
| **Case 2** | Black | Both black | Recolour sibling red, push double-black up to parent. Repeat. |
| **Case 3** | Black | Near child red, far child black | Rotate sibling's near child up, recolour. Converts to Case 4. |
| **Case 4** | Black | Far child red | Rotate sibling up, transfer colours. Remove double-black. Done. |

### Complexity

- **At most 3 rotations** per deletion
- Recolouring may propagate `O(log n)` times (Case 2 bubbling)

> Deletion case analysis is symmetric for left/right. Many implementations use a unified "fix_double_black" function that checks direction and mirrors the logic.

---

## Slide 12 -- AVL vs Red-Black -- When to Choose Which

### Head-to-head comparison

| Property | AVL | Red-Black |
|----------|-----|-----------|
| Height bound | `1.44 log n` | `2 log n` |
| Search speed | Faster (shorter height) | Slightly slower |
| Insert rotations | At most 1 | At most 2 |
| Delete rotations | Up to `O(log n)` | At most 3 |
| Storage overhead | Height or BF per node | 1 bit (colour) per node |
| Implementation complexity | Moderate | Higher |

### When to choose AVL

- **Read-heavy workloads** where search performance dominates
- **Databases and file systems** where lookups vastly outnumber mutations
- When you need the tightest possible height guarantee
- Educational contexts -- rotations are easier to reason about

### When to choose Red-Black

- **Write-heavy workloads** with frequent insertions and deletions
- **Standard library implementations** (C++ `std::map`, Java `TreeMap`, .NET `SortedDictionary`)
- When bounded worst-case rotations per operation matter
- Kernel data structures (Linux CFS scheduler uses Red-Black trees)

> In practice, the performance difference is small. Red-Black trees won the standard library battle largely because of their O(1) rotation guarantee per mutation.

---

## Slide 13 -- 2-3 Trees

### Definition

A 2-3 tree is a perfectly balanced search tree where every internal node has either 2 or 3 children, and all leaves are at the same depth.

### Node types

| Node type | Keys | Children | Search behaviour |
|-----------|------|----------|-----------------|
| **2-node** | 1 key | 2 children | Left < key < Right |
| **3-node** | 2 keys (a, b) | 3 children | Left < a, Middle between a and b, Right > b |

### Insertion

1. Search down to the correct leaf
2. Add the key to the leaf node
3. If the node overflows (becomes a 4-node with 3 keys), **split**: push the middle key up to the parent
4. Splitting may cascade up to the root; if the root splits, tree height increases by 1

### Properties

- **Perfect balance** -- all leaves at the same level, always
- Height: `O(log n)` with base between 2 and 3
- Splits maintain balance without rotations
- Conceptual foundation for Red-Black trees and B-trees

> 2-3 trees are rarely implemented directly in practice -- they serve as the conceptual model behind Red-Black trees (which are their binary representation) and B-trees (which generalise to higher branching factors).

---

## Slide 14 -- 2-3-4 Trees

### Extension of 2-3 trees

A 2-3-4 tree adds a **4-node** (3 keys, 4 children) to the repertoire. This directly corresponds to a Red-Black tree.

### Node types

| Node type | Keys | Children |
|-----------|------|----------|
| 2-node | 1 | 2 |
| 3-node | 2 | 3 |
| 4-node | 3 | 4 |

### Insertion strategies

- **Bottom-up:** insert, then split overflowing nodes back up the tree
- **Top-down:** pre-emptively split any 4-node encountered on the way down -- guarantees the leaf has room, so no back-tracking needed

### Equivalence to Red-Black trees

```
2-node:    [B]           Black node, no red children
3-node:    [B]-R         Black node with one red child
4-node:  R-[B]-R         Black node with two red children
```

Top-down 2-3-4 insertion maps directly to top-down Red-Black insertion with colour flips (splitting 4-nodes = flipping a black node with two red children).

> Sedgewick's *Algorithms* teaches Red-Black trees entirely through this 2-3-4 mapping. Understanding the isomorphism makes Red-Black rotations intuitive rather than magical.

---

## Slide 15 -- Left-Leaning Red-Black Trees

### Sedgewick's simplification (2008)

A Left-Leaning Red-Black (LLRB) tree adds one extra invariant: **red links lean left**. This eliminates half the cases in standard Red-Black tree operations.

### Extra invariant

- If a node has exactly one red child, it must be the **left** child
- This restricts 3-nodes to a single orientation, reducing case analysis

### Operations simplified

| Operation | Standard RB cases | LLRB cases |
|-----------|-------------------|------------|
| Insert | 6 cases (3 + mirror) | 3 cases |
| Delete | 8+ cases | ~4 cases |
| Code lines (typical) | 150--200 | 40--60 |

### Core helpers

```
rotateLeft(h)    -- fix right-leaning red link
rotateRight(h)   -- temporarily create right-leaning red
flipColours(h)   -- split a 4-node (both children red)
```

### Correspondence

LLRB trees are isomorphic to **2-3 trees** (not 2-3-4), because the left-leaning constraint forbids 4-nodes from persisting -- they are always split immediately.

> LLRB trees are the best choice for teaching and for implementations where code simplicity matters more than raw performance. The ~50-line implementation is remarkably elegant.

---

## Slide 16 -- AA Trees

### Arne Andersson's simplification (1993)

An AA tree is a Red-Black tree with an additional constraint: **right children cannot be red**. Equivalently, only right-leaning "horizontal" links at the same level are allowed.

### Level-based formulation

Instead of colours, AA trees use a **level** number per node:

- Leaf nodes have level 1
- A left child's level is strictly less than its parent's level
- A right child's level is equal to or one less than its parent's level
- A right grandchild's level is strictly less than its grandparent's level

### Two operations only

| Operation | What it does |
|-----------|-------------|
| **skew** | Right rotation to eliminate a left horizontal link |
| **split** | Left rotation to eliminate two consecutive right horizontal links (and increment level) |

### Insert and delete

- **Insert:** BST insert, then `skew` and `split` back up the path
- **Delete:** BST delete, then decrease levels if needed, then `skew` and `split` back up

> AA trees are arguably the simplest balanced BST to implement correctly. The two-operation approach (skew + split) eliminates the complex case analysis of Red-Black trees.

---

## Slide 17 -- Scapegoat Trees

### Lazy rebalancing

Scapegoat trees do not maintain any auxiliary information (no heights, no colours, no levels). Instead, they detect and fix imbalance lazily.

### How it works

1. **Insert:** standard BST insert; if the new node's depth exceeds `log_{1/alpha}(n)`, find a **scapegoat** ancestor
2. **Scapegoat:** the highest node where `size(child) > alpha * size(node)` -- this subtree is too unbalanced
3. **Rebuild:** flatten the scapegoat's subtree into a sorted array, then rebuild as a perfectly balanced BST
4. **Delete:** mark-and-lazy-rebuild -- when too many nodes are deleted, rebuild the entire tree

### The alpha parameter

| Alpha | Behaviour |
|-------|-----------|
| `0.5` | Perfectly balanced -- rebuild often |
| `0.75` | Good balance vs rebuild trade-off (common default) |
| `1.0` | Never rebalance -- degenerate BST |

### Complexity

- **Search:** `O(log n)` worst case
- **Insert:** `O(log n)` amortised (`O(n)` worst case for a single rebuild)
- **Delete:** `O(log n)` amortised
- **Space:** `O(n)` -- no extra metadata per node

> Scapegoat trees are useful when node metadata is expensive (embedded systems, cache-sensitive designs) or when amortised cost is acceptable.

---

## Slide 18 -- Weight-Balanced Trees

### Balance by subtree size

Weight-balanced trees (BB[alpha] trees, Nievergelt & Reingold, 1973) maintain balance based on the **size** (number of nodes) of subtrees rather than height.

### Weight-balance condition

For every node, the weight of each child must be at least a fraction `alpha` of the node's total weight:

```
size(left)  >= alpha * size(node)
size(right) >= alpha * size(node)
```

Typical `alpha` values: `2/11` to `1/4`. The bound `alpha <= 1 - 1/sqrt(2) ~ 0.293` ensures rotations always restore balance.

### Advantages

- **Order-statistic operations** come for free -- each node already stores subtree size
- `rank(x)`, `select(k)`, `range_count(lo, hi)` are all `O(log n)`
- Well-suited for persistent/functional implementations
- Merging and splitting are efficient

### Used in practice

- Haskell's `Data.Map` and `Data.Set` use weight-balanced (size-balanced) trees
- Good choice when order-statistic queries are frequent
- Natural fit for functional programming (persistent structure, no mutation needed)

> Weight-balanced trees rarely appear in imperative standard libraries but dominate the functional programming world. If you need `select(k)` alongside standard map operations, they are the natural choice.

---

## Slide 19 -- Persistent Balanced Trees

### Functional data structures

A **persistent** data structure preserves all previous versions after mutation. Instead of modifying nodes in place, each update creates a new path from the modified node to the root -- path copying.

### Path copying

```
Version 1:           Version 2 (insert 25):

      30                  30'
     / \                 / \
   20   40    ->       20'  40
  / \                 / \
10   25(?)          10   25  <- new node

Shared nodes: 10, 40 (unchanged)
New nodes: 30', 20', 25
```

- Only `O(log n)` nodes are copied per operation (the root-to-leaf path)
- Space per update: `O(log n)` additional nodes
- All previous roots remain valid and accessible

### Balanced tree choices for persistence

| Tree type | Persistence suitability |
|-----------|------------------------|
| **Weight-balanced** | Excellent -- Haskell's default; natural path-copying semantics |
| **Red-Black** | Good -- used in Clojure, Scala; colour metadata is just one bit |
| **AVL** | Good -- slightly more copying due to tighter balance |
| **2-3 trees** | Excellent -- clean functional implementations |

### Applications

- **Version control** -- git-like history for in-memory data
- **Undo/redo** -- O(1) rollback by keeping the old root
- **Concurrent reads** -- readers use old versions while a writer builds a new one; no locks needed
- **Temporal databases** -- query the state of data at any past point in time

> Persistent balanced trees are the backbone of functional language standard libraries. Clojure's sorted maps, Haskell's `Data.Map`, and Scala's `TreeMap` all use persistent balanced trees internally.

---

## Slide 20 -- Summary & Comparison Table

### Comparison of balanced tree variants

| Tree | Height bound | Insert rotations | Delete rotations | Extra storage | Best for |
|------|-------------|------------------|------------------|---------------|----------|
| **AVL** | `1.44 log n` | <=1 | <=O(log n) | BF or height | Read-heavy |
| **Red-Black** | `2 log n` | <=2 | <=3 | 1 bit colour | General-purpose |
| **LLRB** | `2 log n` | <=2 | <=2 | 1 bit colour | Simple implementation |
| **AA** | `2 log n` | <=1 split | <=O(log n) | Level integer | Teaching, simplicity |
| **2-3** | `log_2 n` to `log_3 n` | Splits up | Merges up | Multi-key nodes | Conceptual foundation |
| **2-3-4** | `log_2 n` to `log_4 n` | Splits up | Merges up | Multi-key nodes | RB tree mapping |
| **Scapegoat** | `log_{1/a} n` | None (amort rebuild) | None (amort rebuild) | None | Minimal metadata |
| **Weight-balanced** | `O(log n)` | <=1 | <=1 | Subtree size | Order-statistic, FP |

### Key takeaways

- All balanced BSTs guarantee `O(log n)` search, insert, and delete
- AVL is best for lookup-dominated workloads; Red-Black for mutation-heavy ones
- 2-3 and 2-3-4 trees are the conceptual foundation -- Red-Black trees are their binary encoding
- LLRB and AA trees dramatically simplify implementation at minimal performance cost
- Scapegoat trees trade worst-case guarantees for zero per-node metadata
- Weight-balanced trees excel in functional programming and order-statistic queries
- Persistent balanced trees enable immutable, versioned data with `O(log n)` overhead per update

### Recommended reading

| Source | Description |
|--------|------------|
| **Sedgewick & Wayne** | *Algorithms, 4th ed.* -- LLRB trees, 2-3 trees, Red-Black mapping |
| **CLRS** | *Introduction to Algorithms* -- definitive Red-Black and AVL treatment |
| **Okasaki** | *Purely Functional Data Structures* -- persistent trees, path copying |
| **Andersson** | "Balanced Search Trees Made Simple" (1993) -- AA trees |
| **Galperin & Rivest** | "Scapegoat Trees" (1993) -- lazy rebalancing |
