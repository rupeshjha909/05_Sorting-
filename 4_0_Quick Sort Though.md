# Quicksort

> Divide-and-conquer with a clever twist: do all the work in the **divide** step, and the **combine** step disappears.

**Reference links:**
- [GFG — QuickSort](https://www.geeksforgeeks.org/quick-sort/)
- [Wikipedia — Quicksort](https://en.wikipedia.org/wiki/Quicksort)

---

## Table of Contents

1. [Quicksort vs Merge Sort — The Inversion](#1-quicksort-vs-merge-sort--the-inversion)
2. [The One Big Idea — Partitioning](#2-the-one-big-idea--partitioning)
3. [The Lomuto Partition Scheme](#3-the-lomuto-partition-scheme)
4. [Understanding the Two Pointers](#4-understanding-the-two-pointers)
5. [The Final Swap — Why It Works](#5-the-final-swap--why-it-works)
6. [The Recursion](#6-the-recursion)
7. [The Java Code](#7-the-java-code)
8. [Mapping the Logic to the Code](#8-mapping-the-logic-to-the-code)
9. [Full Trace Example](#9-full-trace-example)
10. [Why O(n log n) Average — and O(n²) Worst](#10-why-on-log-n-average--and-on-worst)
11. [Pivot Choice Matters](#11-pivot-choice-matters)
12. [Common Pitfalls](#12-common-pitfalls)
13. [Quicksort vs Other Sorts](#13-quicksort-vs-other-sorts)
14. [My Own Questions & Examples](#14-my-own-questions--examples)

---

## 1. Quicksort vs Merge Sort — The Inversion

Both are divide and conquer, but they invert where the work happens:

| | Merge Sort | Quicksort |
|:---|:---|:---|
| **Divide** | Split blindly at midpoint (cheap) | **Partition around pivot (expensive)** |
| **Conquer** | Recursively sort halves | Recursively sort halves |
| **Combine** | **Merge sorted halves (expensive)** | Nothing! Already in place (cheap) |

> 💡 **The key insight:** if you do the partition step well, the combine step disappears. After partition, the array is "loosely sorted" — small things on the left, big things on the right — and recursion finishes the job.

This is why quicksort is **in-place**: there's no merge buffer, no copying back. Everything happens by swapping in the original array.

> 📝 **My notes:**
> - _Merge sort: easy split, hard merge. Quicksort: hard split, no merge._
> -

---

## 2. The One Big Idea — Partitioning

Pick any element as the **pivot**. Rearrange the array so that:

- Everything ≤ pivot is on the **left**
- Pivot lands in its **final correct position**
- Everything > pivot is on the **right**

After partitioning, the pivot is **done forever** — it will never move again. That's the magic.

```
Before: [3, 7, 1, 8, 2, 5]    pivot = 5

After:  [3, 1, 2 | 5 | 7, 8]
         ≤ pivot   ↑   > pivot
              pivot in final position
```

The two sides aren't sorted yet, but the pivot is exactly where it belongs in the final sorted array. Now recursion handles the two sides independently.

---

## 3. The Lomuto Partition Scheme

There are several ways to partition (Lomuto, Hoare, three-way, etc.). The code above uses the **Lomuto** scheme — the simplest and most common in textbooks.

**The strategy:**
1. Pick the **last element** `a[e]` as the pivot.
2. Walk through `a[s..e-1]` with two pointers `i` and `j`.
3. Whenever you find something ≤ pivot, "move it to the left zone" by swapping.
4. At the end, swap the pivot into its final position.

---

## 4. Understanding the Two Pointers

This is the part most people find confusing. Let me make it crystal clear.

- **`j`** is the **scanner** — walks left to right examining every element.
- **`i`** is the **boundary marker** — tracks the rightmost index of the "≤ pivot" zone.

The invariant at any point during the loop:

```
[s ......... i] [i+1 ......... j-1] [j ......... e-1] [e]
   ≤ pivot         > pivot              unscanned       pivot
```

So `i` lags behind `j`. Whenever `j` finds an element ≤ pivot, we:
1. Extend the "small" zone: `i++`
2. Bring the small element into the zone: `swap(a[i], a[j])`

If `a[j] > pivot`, we just advance `j` — the element joins the "> pivot" zone naturally because we don't move it.

### Why this works

Every time we swap, we're moving a "small" element into the small zone (now of size `i - s + 1`). The element that was at `a[i]` (which was `> pivot`) gets pushed to where the small one was — it's still in the `> pivot` zone, just at a new position. The invariant holds.

> 📝 **My notes:**
> - _Visualize i as a "wall" being pushed forward as more smalls are found_
> -

---

## 5. The Final Swap — Why It Works

When the loop ends, the array looks like this:

```
[s ......... i] [i+1 ......... e-1] [e]
   ≤ pivot         > pivot            pivot
```

The pivot is sitting at the end, but it should be **between** the two zones. So we swap it with `a[i+1]` (the first element of the `> pivot` zone):

```
After swap:
[s ......... i] [i+1] [i+2 ......... e]
   ≤ pivot     pivot       > pivot
```

We then return `i + 1` — the pivot's final, never-changing index.

### What happens to the element that was at `a[i+1]`?

It was `> pivot`, so moving it to position `e` (where the pivot used to be) keeps the right side correct: everything from `i+2` to `e` is still `> pivot`. ✓

> 📝 **My notes:**
> - _Why `i+1` and not `i`? Because a[i] is ≤ pivot — putting pivot there would put a smaller element to its right_
> -

---

## 6. The Recursion

Once we have the pivot's final position `p`:

```java
quickSort(a, s, p - 1);   // sort everything ≤ pivot
quickSort(a, p + 1, e);   // sort everything > pivot
```

We don't include `p` itself — it's already in place forever.

The base case is `s >= e`: a range with 0 or 1 element is trivially sorted.

```
                [3, 7, 1, 8, 2, 5]
                       ↓ partition (pivot=5)
                [3, 1, 2 | 5 | 7, 8]
                /                  \
        [3, 1, 2]                [7, 8]
            ↓ partition (pivot=2)    ↓ partition (pivot=8)
        [1 | 2 | 3]              [7 | 8]
        /        \                /
      [1]       [3]             [7]    (singletons, base case)
                ↓ everything is in place ↓
                [1, 2, 3, 5, 7, 8]
```

Notice there's **no merge step** — the pivots were placed correctly during partitioning, and the recursive sorts handle their sides independently.

---

## 7. The Java Code

```java
public class QuickSort {

    static int partition(int[] a, int s, int e) {
        int pivot = a[e];
        int i = s - 1;     // boundary of "≤ pivot" zone (empty initially)

        for (int j = s; j <= e - 1; j++) {
            if (a[j] <= pivot) {
                i++;
                // swap a[i] and a[j]
                int temp = a[i];
                a[i] = a[j];
                a[j] = temp;
            }
        }

        // Place pivot in its final position
        int temp = a[i + 1];
        a[i + 1] = a[e];
        a[e] = temp;

        return i + 1;       // pivot's final index
    }

    static void quickSort(int[] a, int s, int e) {
        if (s >= e) return;
        int p = partition(a, s, e);
        quickSort(a, s, p - 1);
        quickSort(a, p + 1, e);
    }

    public static void sort(int[] a) {
        if (a == null || a.length <= 1) return;
        quickSort(a, 0, a.length - 1);
    }
}
```

---

## 8. Mapping the Logic to the Code

### Pivot selection
```java
int pivot = a[e];
```
**Translation:** *"Use the last element as our pivot."*

This is the simplest choice. We'll discuss why it's risky in [Section 11](#11-pivot-choice-matters).

### Initial state
```java
int i = s - 1;
```
**Translation:** *"The 'small' zone is empty — `i` points one before the start of the range."*

### The scan loop
```java
for (int j = s; j <= e - 1; j++) {
    if (a[j] <= pivot) {
        i++;
        swap(a[i], a[j]);
    }
}
```

**Translation:** *"For each element in the range (excluding the pivot at the end), if it's ≤ pivot, extend the small zone and bring this element into it."*

If `a[j] > pivot`, we do nothing — `j` simply advances and the element stays in the `> pivot` zone.

### The final swap
```java
int temp = a[i + 1];
a[i + 1] = a[e];
a[e] = temp;
return i + 1;
```

**Translation:** *"Move the pivot from `a[e]` into the slot just after the small zone. Return its final position."*

### The recursive driver
```java
int p = partition(a, s, e);
quickSort(a, s, p - 1);
quickSort(a, p + 1, e);
```

**Translation:** *"Partition gives us the pivot's final position `p`. Recursively sort everything before `p` and everything after `p`. The pivot itself is done."*

> 📝 **My notes:**
> - _Notice we never recurse on p itself — it's already correctly placed_
> -

---

## 9. Full Trace Example

**Input:** `a = [3, 7, 1, 8, 2, 5]`

### Top-level call: partition(a, 0, 5), pivot = a[5] = 5

Initial: `i = -1`, `j` starts at 0.

| j | a[j] | a[j] ≤ 5? | Action | Array after |
|:-:|:----:|:---------:|:-------|:-----------|
| 0 | 3 | ✓ | i=0, swap a[0]↔a[0] | `[3, 7, 1, 8, 2, 5]` |
| 1 | 7 | ✗ | skip | `[3, 7, 1, 8, 2, 5]` |
| 2 | 1 | ✓ | i=1, swap a[1]↔a[2] | `[3, 1, 7, 8, 2, 5]` |
| 3 | 8 | ✗ | skip | `[3, 1, 7, 8, 2, 5]` |
| 4 | 2 | ✓ | i=2, swap a[2]↔a[4] | `[3, 1, 2, 8, 7, 5]` |

Final swap: `a[i+1]=a[3]=8 ↔ a[5]=5` → `[3, 1, 2, 5, 7, 8]`

Return 3 (pivot's final position).

### Recursive call: quickSort(a, 0, 2) on `[3, 1, 2]`

partition(a, 0, 2), pivot = a[2] = 2.

| j | a[j] | ≤ 2? | Action | Array |
|:-:|:----:|:----:|:-------|:------|
| 0 | 3 | ✗ | skip | `[3, 1, 2]` |
| 1 | 1 | ✓ | i=0, swap a[0]↔a[1] | `[1, 3, 2]` |

Final swap: `a[1]=3 ↔ a[2]=2` → `[1, 2, 3]`

Return 1. Recurse on (0, 0) and (2, 2) — both base cases.

### Recursive call: quickSort(a, 4, 5) on `[7, 8]`

partition(a, 4, 5), pivot = a[5] = 8.

| j | a[j] | ≤ 8? | Action | Array |
|:-:|:----:|:----:|:-------|:------|
| 4 | 7 | ✓ | i=4, swap a[4]↔a[4] | `[7, 8]` |

Final swap: `a[5]=8 ↔ a[5]=8` → `[7, 8]`

Return 5. Recurse on (4, 4) and (6, 5) — both base cases.

**Final array:** `[1, 2, 3, 5, 7, 8]` ✓

> 📝 **My notes:**
> - _Try tracing [4, 3, 2, 1] (reverse sorted — worst case for last-element pivot)_
> - _Try tracing [1, 2, 3, 4] (already sorted — also worst case)_
> -

---

## 10. Why O(n log n) Average — and O(n²) Worst

Quicksort's complexity depends entirely on **how balanced the partitions are**.

### Best / Average: O(n log n)

If each partition splits the array roughly in half:
- Recursion depth: `log n` levels
- Work per level: `O(n)` (each element scanned once during partition)
- Total: `O(n log n)`

```
Level 0: 1 partition of size n      → O(n)
Level 1: 2 partitions of size n/2   → O(n)
Level 2: 4 partitions of size n/4   → O(n)
...
log n levels × O(n) per level = O(n log n)
```

### Worst case: O(n²)

If each partition is **maximally unbalanced** (one side has 0 elements, the other has `n-1`):

```
Level 0: 1 partition of size n     → O(n)
Level 1: 1 partition of size n-1   → O(n-1)
Level 2: 1 partition of size n-2   → O(n-2)
...
Total: n + (n-1) + (n-2) + ... + 1 = n(n+1)/2 = O(n²)
```

**When does this happen?** With the "last element as pivot" strategy:
- Already sorted input: `[1, 2, 3, 4, 5]` → pivot is always the largest → all elements go left
- Reverse sorted: `[5, 4, 3, 2, 1]` → pivot is always the smallest → all elements go right

Both cases produce maximally unbalanced partitions every time.

### Space: O(log n) average, O(n) worst

The recursion stack. Average case has log n depth, but worst case has n depth. (Real-world quicksort implementations recurse on the smaller side and iterate on the larger to keep stack depth bounded.)

---

## 11. Pivot Choice Matters

The "last element as pivot" choice in this code is simple but vulnerable. Better choices:

### Random pivot
```java
int randomIdx = s + new Random().nextInt(e - s + 1);
swap(a[randomIdx], a[e]);
// proceed as before
```
This makes worst case **astronomically unlikely** — an adversary can't pick a bad input because the pivot is randomized.

### Median-of-three
Pick the median of `a[s]`, `a[mid]`, `a[e]` as the pivot. Almost always good in practice. Used in many production sorts.

### First or middle element
Same problem as last element — gives O(n²) on certain inputs.

### Why this matters in practice
Most production quicksort implementations (like Java's `Arrays.sort` for primitives) use **dual-pivot quicksort with median-of-three**, plus they switch to insertion sort for small subarrays. This combines the strengths of multiple algorithms.

> 📝 **My notes:**
> - _Always use random or median-of-three for production code_
> -

---

## 12. Common Pitfalls

- ❌ **Always picking the last element as pivot.** Causes O(n²) on sorted/reverse-sorted input — extremely common in practice.
- ⚠️ **Off-by-one in the loop bound.** Use `j <= e - 1` (NOT `j <= e`) — otherwise we'd swap with the pivot itself during the scan.
- ⚠️ **Swapping when `a[i] == a[j]` (i.e., self-swap).** This is harmless but wastes operations. Some implementations check `i != j`.
- ⚠️ **Recursing on the pivot.** Use `quickSort(a, s, p-1)` and `quickSort(a, p+1, e)`, NOT `(s, p)` and `(p, e)` — including `p` causes infinite recursion.
- ⚠️ **Stack overflow on adversarial input.** With pure last-element pivot and a sorted input of size 10⁵, recursion depth becomes 10⁵ — JVM stack overflow. Use random/median pivot.
- ⚠️ **Confusing `i = s - 1` start.** This is intentional — it represents an empty "small" zone before any element has been examined.

> 📝 **My notes:**
> - _What edge cases tripped me up?_
> -

---

## 13. Quicksort vs Other Sorts

| Sort | Best | Average | Worst | Space | Stable | In-place |
|:-----|:----:|:-------:|:-----:|:-----:|:------:|:--------:|
| Insertion | O(n) | O(n²) | O(n²) | O(1) | ✅ | ✅ |
| Merge | O(n log n) | O(n log n) | O(n log n) | O(n) | ✅ | ❌ |
| **Quick** | **O(n log n)** | **O(n log n)** | **O(n²)** | **O(log n)** | **❌** | **✅** |
| Heap | O(n log n) | O(n log n) | O(n log n) | O(1) | ❌ | ✅ |

### When to use quicksort

✅ **Average performance is king** — quicksort has the smallest constants in practice. Often beats merge sort on real data despite having the same big-O.
✅ **In-place sorting required** — uses no extra buffer (just stack space).
✅ **Cache-friendly** — sequential access patterns work well with CPU caches.
✅ **Production sorting of primitives** — Java's `Arrays.sort(int[])` uses dual-pivot quicksort.

### When to avoid quicksort

❌ **Stability required** — quicksort isn't stable. Use merge sort.
❌ **Worst-case guarantees needed** — heap sort or merge sort give O(n log n) always.
❌ **Adversarial inputs possible** — e.g., user-controlled input. Use a randomized pivot or different algorithm.

### Quicksort vs Merge sort — head to head

| Criterion | Winner |
|:----------|:------:|
| In-place memory | Quicksort |
| Worst-case time | Merge sort |
| Stability | Merge sort |
| Cache behavior | Quicksort |
| Linked lists | Merge sort |
| Average performance | Quicksort (smaller constants) |
| Predictability | Merge sort |

---

## 14. My Own Questions & Examples

Use the space below to write down your own questions, edge cases to think about, or alternative approaches you want to remember.

### Questions

-
-
-

### Edge cases to test

- [ ] Empty array: `[]` → `[]`
- [ ] Single element: `[5]` → `[5]`
- [ ] Two elements: `[2, 1]` → `[1, 2]`
- [ ] Already sorted: `[1, 2, 3, 4, 5]` → unchanged (worst case with last-element pivot!)
- [ ] Reverse sorted: `[5, 4, 3, 2, 1]` → also worst case
- [ ] All same: `[3, 3, 3, 3]` → behavior depends on partition scheme
- [ ] Negative numbers: `[3, -1, 0, -5, 2]`
- [ ] Large random array: 10⁵ elements
- [ ] _Add your own:_

### Variations to try

- _Random pivot_ — swap a random element to position `e` before partitioning
- _Median-of-three pivot_ — picks the median of first, middle, last
- _Hoare partition scheme_ — different two-pointer approach, more efficient
- _Three-way partition (Dutch flag)_ — handles duplicates efficiently
- _Iterative quicksort using an explicit stack_
- _Hybrid quicksort + insertion sort for small subarrays_
- _Quickselect_ — find kth smallest in O(n) average using partition
- _Your idea:_

### Related problems

- [ ] [LC 215 — Kth Largest Element](https://leetcode.com/problems/kth-largest-element-in-an-array/) (uses Quickselect)
- [ ] [LC 75 — Sort Colors](https://leetcode.com/problems/sort-colors/) (Dutch flag / 3-way partition)
- [ ] [LC 912 — Sort an Array](https://leetcode.com/problems/sort-an-array/)
- [ ] [LC 973 — K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/) (Quickselect)
- [ ] [GFG — Quick Sort on Linked List](https://www.geeksforgeeks.org/quicksort-on-singly-linked-list/)

---

## Pattern Recognition: The Power of Partitioning

The partition step itself is a powerful pattern beyond just sorting. It's used in:

| Problem | How partition helps |
|:--------|:-------------------|
| **Kth smallest/largest** (Quickselect) | Partition once, recurse on the side containing rank K — O(n) average |
| **Dutch national flag** (sort 0s, 1s, 2s) | Three-way partition in one pass |
| **Move zeroes to end** | Partition by "is zero or not" |
| **Sort by parity** | Partition by even/odd |
| **Wiggle sort** | Partition + interleave |

Once you understand partitioning, many "rearrangement" problems become trivial.

---

## Pattern Recognition: Your Sorting Toolkit

| Sort | Approach | Best for |
|:-----|:---------|:---------|
| Insertion | Build sorted region by inserting | Small or nearly-sorted data |
| Merge | Divide and conquer + merge | Stable O(n log n), large data |
| **Quick** | **Partition + recurse** | **In-place, average-case fastest** |
| Heap | Build heap, extract max | Worst-case O(n log n), in-place |

You've now seen all four major comparison-based sorts. Each has its place — understanding their tradeoffs is more valuable than memorizing any one.

---

_Last updated: <!-- add date here -->_
