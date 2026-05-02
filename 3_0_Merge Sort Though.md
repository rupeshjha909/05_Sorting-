# Merge Sort

> The classic Divide-and-Conquer sorting algorithm — O(n log n) guaranteed.

**Reference links:**
- [GFG — Merge Sort](https://www.geeksforgeeks.org/merge-sort/)
- [Wikipedia — Merge Sort](https://en.wikipedia.org/wiki/Merge_sort)

---

## Table of Contents

1. [The One Big Idea](#1-the-one-big-idea)
2. [Divide and Conquer in Three Steps](#2-divide-and-conquer-in-three-steps)
3. [The Merge Function — The Heart of the Algorithm](#3-the-merge-function--the-heart-of-the-algorithm)
4. [The Recursion — How Splitting Works](#4-the-recursion--how-splitting-works)
5. [The Java Code](#5-the-java-code)
6. [Mapping the Logic to the Code](#6-mapping-the-logic-to-the-code)
7. [Full Trace Example](#7-full-trace-example)
8. [Why O(n log n)?](#8-why-on-log-n)
9. [Common Pitfalls](#9-common-pitfalls)
10. [Merge Sort vs Other Sorts](#10-merge-sort-vs-other-sorts)
11. [My Own Questions & Examples](#11-my-own-questions--examples)

---

## 1. The One Big Idea

Merge sort is built on one beautifully simple observation:

> **Merging two already-sorted arrays is fast and easy. So if I can split my problem into two sorted halves, I can solve it.**

This is **divide and conquer** in its purest form. We don't sort the array directly — we trust recursion to give us sorted halves, then we just merge them.

### The recursive promise

The recursive call promises: *"Give me a range, and I'll return it sorted."* So when we write:

```java
mergeSort(a, s, mid);       // ← trust this to sort the left half
mergeSort(a, mid + 1, e);   // ← trust this to sort the right half
merge(a, s, e);             // ← now combine two sorted halves
```

We don't have to think about how the halves get sorted — recursion handles it. We only have to write the merge step correctly.

> 📝 **My notes:**
> - _The "leap of faith" in recursion: trust the smaller cases to work_
> -

---

## 2. Divide and Conquer in Three Steps

Every divide-and-conquer algorithm follows this pattern:

| Step | What it does | In merge sort |
|:----:|:-------------|:--------------|
| **Divide** | Break the problem into smaller subproblems | Split array into two halves at `mid` |
| **Conquer** | Solve each subproblem (usually recursively) | Recursively sort each half |
| **Combine** | Merge the solutions back together | Merge two sorted halves |

The base case is when the subproblem is trivially solvable — here, when the range has 0 or 1 element (`s >= e`), it's already sorted.

---

## 3. The Merge Function — The Heart of the Algorithm

Suppose you're given two sorted halves and want to combine them into one sorted whole. Use **two pointers**:

```
Left half  (sorted):  [1, 2, 5, 8]
Right half (sorted):  [3, 4, 7, 9]

Pointer i starts at left[0] = 1
Pointer j starts at right[0] = 3
Output buffer is empty.

Step 1: 1 ≤ 3 → take 1, advance i
Step 2: 2 ≤ 3 → take 2, advance i
Step 3: 5 > 3 → take 3, advance j
Step 4: 5 > 4 → take 4, advance j
Step 5: 5 ≤ 7 → take 5, advance i
Step 6: 8 > 7 → take 7, advance j
Step 7: 8 ≤ 9 → take 8, advance i
Step 8: only right has elements → drain it: 9

Result: [1, 2, 3, 4, 5, 7, 8, 9]
```

### Why this works

Both halves are sorted. So the smallest unprocessed element of the union is **always at one of the two pointers**. We just compare and take the smaller. After 8 picks, all elements are taken in sorted order.

### Why we need the temp buffer

We can't merge in-place into the original array because writing would overwrite values we haven't read yet. Instead, we write into a temporary buffer, then copy back.

> 📝 **My notes:**
> - _The merge step is O(n) — every element is touched exactly once_
> - _Two pointers + comparison = the classic "merge" pattern, used in many problems_
> -

---

## 4. The Recursion — How Splitting Works

The split itself is trivial: just compute `mid = (s + e) / 2` and recurse on the two halves.

```
                [5, 2, 8, 1, 9, 3, 7, 4]
                  /                    \
            [5, 2, 8, 1]            [9, 3, 7, 4]
              /      \                /        \
          [5, 2]   [8, 1]         [9, 3]    [7, 4]
           / \      / \            / \        / \
         [5][2]  [8][1]         [9][3]      [7][4]
          ↓ merge ↓               ↓ merge ↓
         [2, 5]  [1, 8]         [3, 9]    [4, 7]
              ↓ merge ↓            ↓ merge ↓
          [1, 2, 5, 8]           [3, 4, 7, 9]
                  ↓ merge ↓
            [1, 2, 3, 4, 5, 7, 8, 9]
```

### The base case

```java
if (s >= e) return;
```

When the range has 0 or 1 element, there's nothing to sort. This stops the recursion. Without a base case, we'd recurse infinitely.

---

## 5. The Java Code

```java
public class MergeSort {

    static void merge(int[] a, int s, int e) {
        int mid = (s + e) / 2;
        int i = s;          // pointer for left half
        int j = mid + 1;    // pointer for right half
        int k = s;          // pointer for temp buffer

        int[] temp = new int[a.length];

        // Pick smaller of the two front elements
        while (i <= mid && j <= e) {
            if (a[i] <= a[j])
                temp[k++] = a[i++];
            else
                temp[k++] = a[j++];
        }

        // Drain whatever remains in the left half
        while (i <= mid)
            temp[k++] = a[i++];

        // Drain whatever remains in the right half
        while (j <= e)
            temp[k++] = a[j++];

        // Copy merged result back into original array
        for (int x = s; x <= e; x++) {
            a[x] = temp[x];
        }
    }

    static void mergeSort(int[] a, int s, int e) {
        if (s >= e)         // base case: 0 or 1 element
            return;

        int mid = (s + e) / 2;
        mergeSort(a, s, mid);          // sort left half
        mergeSort(a, mid + 1, e);      // sort right half
        merge(a, s, e);                // combine
    }

    // Convenience wrapper
    public static void sort(int[] a) {
        if (a == null || a.length <= 1) return;
        mergeSort(a, 0, a.length - 1);
    }
}
```

---

## 6. Mapping the Logic to the Code

### The driver

```java
mergeSort(a, s, mid);
mergeSort(a, mid + 1, e);
merge(a, s, e);
```

**Translation:** *"First sort the left half. Then sort the right half. Then merge them. (And trust recursion to handle the sorting!)"*

### Setup in merge

```java
int i = s;          // walks through left half [s..mid]
int j = mid + 1;    // walks through right half [mid+1..e]
int k = s;          // writes into temp[s..e]
```

Three pointers. `i` and `j` read from the original. `k` writes to the temp buffer.

### The main merge loop

```java
while (i <= mid && j <= e) {
    if (a[i] <= a[j])
        temp[k++] = a[i++];
    else
        temp[k++] = a[j++];
}
```

**Translation:** *"While both halves still have elements, take the smaller front element and advance its pointer."*

The loop ends when **one half is exhausted**.

### Drain the remainder

```java
while (i <= mid) temp[k++] = a[i++];
while (j <= e)   temp[k++] = a[j++];
```

When one half runs out, the other half is **already sorted** (it was sorted by the recursive call). Just copy what's left.

> 💡 **Only ONE of these loops will actually run** in any given call — whichever half wasn't exhausted in the main loop.

### Copy back

```java
for (int x = s; x <= e; x++) {
    a[x] = temp[x];
}
```

The merge wrote into `temp[s..e]`. Copy those values back into `a[s..e]` so subsequent recursive calls see the merged result.

> 📝 **My notes:**
> - _Why use `<=` and not `<` in `a[i] <= a[j]`? For STABILITY — keeps relative order of equal elements_
> -

---

## 7. Full Trace Example

**Input:** `a = [5, 2, 8, 1, 9, 3, 7, 4]`

### The recursion tree (call order, depth-first)

```
mergeSort(a, 0, 7)
├── mergeSort(a, 0, 3)
│   ├── mergeSort(a, 0, 1)
│   │   ├── mergeSort(a, 0, 0)  → base case, return
│   │   ├── mergeSort(a, 1, 1)  → base case, return
│   │   └── merge(a, 0, 1)      → [2, 5]
│   ├── mergeSort(a, 2, 3)
│   │   ├── mergeSort(a, 2, 2)  → base case
│   │   ├── mergeSort(a, 3, 3)  → base case
│   │   └── merge(a, 2, 3)      → [1, 8]
│   └── merge(a, 0, 3)          → [1, 2, 5, 8]
├── mergeSort(a, 4, 7)
│   ├── mergeSort(a, 4, 5)
│   │   └── ... merge(a, 4, 5)  → [3, 9]
│   ├── mergeSort(a, 6, 7)
│   │   └── ... merge(a, 6, 7)  → [4, 7]
│   └── merge(a, 4, 7)          → [3, 4, 7, 9]
└── merge(a, 0, 7)              → [1, 2, 3, 4, 5, 7, 8, 9]
```

### Trace of the final merge

`merge(a, 0, 7)` operates on a (now-sorted-in-halves) array `[1, 2, 5, 8, 3, 4, 7, 9]`, mid=3.

| Step | i | j | k | a[i] | a[j] | Action | temp so far |
|:----:|:-:|:-:|:-:|:----:|:----:|:-------|:-----------:|
| 1 | 0 | 4 | 0 | 1 | 3 | 1 ≤ 3, take a[i]=1 | [1] |
| 2 | 1 | 4 | 1 | 2 | 3 | 2 ≤ 3, take a[i]=2 | [1, 2] |
| 3 | 2 | 4 | 2 | 5 | 3 | 5 > 3, take a[j]=3 | [1, 2, 3] |
| 4 | 2 | 5 | 3 | 5 | 4 | 5 > 4, take a[j]=4 | [1, 2, 3, 4] |
| 5 | 2 | 6 | 4 | 5 | 7 | 5 ≤ 7, take a[i]=5 | [1, 2, 3, 4, 5] |
| 6 | 3 | 6 | 5 | 8 | 7 | 8 > 7, take a[j]=7 | [1, 2, 3, 4, 5, 7] |
| 7 | 3 | 7 | 6 | 8 | 9 | 8 ≤ 9, take a[i]=8 | [1, 2, 3, 4, 5, 7, 8] |
| 8 | 4 | 7 | 7 | — | — | i > mid, drain right | [1, 2, 3, 4, 5, 7, 8, 9] |

Then copy temp[0..7] back into a[0..7]. **Final:** `[1, 2, 3, 4, 5, 7, 8, 9]` ✓

> 📝 **My notes:**
> - _Try tracing [4, 3, 2, 1] (reverse sorted)_
> - _Try tracing [1, 2, 3, 4] (already sorted) — notice it still does all the work_
> -

---

## 8. Why O(n log n)?

The complexity comes from two sources:

### Depth of recursion: log n

Each call splits the range in half. Starting from size `n`, we go: `n → n/2 → n/4 → ... → 1`. The number of halvings is **log₂(n)**.

### Work per level: n

At every level of recursion, the total work (across all calls at that level) is `O(n)` — because every element is touched exactly once during the merges.

```
Level 0:  one merge of size n          → O(n) work
Level 1:  two merges of size n/2 each  → O(n) work total
Level 2:  four merges of size n/4 each → O(n) work total
...
Level log n: trivial (size 1)          → O(n) work total
```

**Total:** `O(n) × log n = O(n log n)`.

### Space: O(n)

The temp buffer is sized `n`. Plus `O(log n)` for the recursion stack. Net: **O(n)**.

| Case | Time | Space | Notes |
|:-----|:----:|:-----:|:------|
| Best | O(n log n) | O(n) | Even on sorted input, all the splits/merges happen |
| Average | O(n log n) | O(n) | Always |
| Worst | O(n log n) | O(n) | Guaranteed — no bad cases |

This is merge sort's **superpower**: it's O(n log n) **regardless** of input. Quicksort can degrade to O(n²) on bad inputs; merge sort never does.

---

## 9. Common Pitfalls

- ❌ **Using a fixed-size temp array like `int temp[100]`.** This crashes for arrays larger than 100. Use `new int[a.length]` or pass a shared buffer.
- ⚠️ **Computing `mid = (s + e) / 2` with very large `s` and `e`.** Could overflow. Use `s + (e - s) / 2` for safety.
- ⚠️ **Off-by-one in the recursive call.** It's `mergeSort(a, mid+1, e)`, NOT `mergeSort(a, mid, e)` — otherwise infinite recursion when range has 2 elements.
- ⚠️ **Using `<` instead of `<=` in the merge comparison.** Breaks stability (changes the relative order of equal elements).
- ⚠️ **Forgetting to copy temp back to `a`.** The merge writes to temp; without copying back, subsequent recursive calls work on stale data.
- ⚠️ **Allocating the temp buffer inside `merge()` on every call.** Wasteful — allocates O(n) memory at every recursion level. Better: allocate once at the top level and pass it down.

> 📝 **My notes:**
> - _What edge cases tripped me up?_
> -

---

## 10. Merge Sort vs Other Sorts

| Sort | Best | Average | Worst | Space | Stable | In-place |
|:-----|:----:|:-------:|:-----:|:-----:|:------:|:--------:|
| Insertion | O(n) | O(n²) | O(n²) | O(1) | ✅ | ✅ |
| **Merge** | **O(n log n)** | **O(n log n)** | **O(n log n)** | **O(n)** | **✅** | **❌** |
| Quick | O(n log n) | O(n log n) | O(n²) | O(log n) | ❌ | ✅ |
| Heap | O(n log n) | O(n log n) | O(n log n) | O(1) | ❌ | ✅ |

### When merge sort is the right choice

✅ **Guaranteed O(n log n)** — even adversarial input can't slow it down.
✅ **Stable sort** — preserves relative order of equal elements (important for sorting by multiple keys).
✅ **External sorting** — for huge files that don't fit in memory, merge sort handles streaming gracefully.
✅ **Linked lists** — merge sort is the natural choice for linked lists since random access isn't needed.
✅ **Used in production** — Java's `Arrays.sort` for objects, Python's Timsort, etc., are based on merge sort.

### When to prefer alternatives

❌ **Quicksort** — when you need in-place sorting and average performance is enough.
❌ **Heap sort** — when you need O(n log n) worst-case AND O(1) space.
❌ **Insertion sort** — for small arrays (< 20 elements). Many libraries combine insertion sort + merge sort (Timsort) for the best of both worlds.

---

## 11. My Own Questions & Examples

Use the space below to write down your own questions, edge cases to think about, or alternative approaches you want to remember.

### Questions

-
-
-

### Edge cases to test

- [ ] Empty array: `[]` → `[]`
- [ ] Single element: `[5]` → `[5]`
- [ ] Two elements: `[2, 1]` → `[1, 2]`
- [ ] Already sorted: `[1, 2, 3, 4, 5]` → unchanged (but still does all the recursion!)
- [ ] Reverse sorted: `[5, 4, 3, 2, 1]` → `[1, 2, 3, 4, 5]`
- [ ] All duplicates: `[3, 3, 3, 3]` → unchanged, stable
- [ ] Negative numbers: `[3, -1, 0, -5, 2]`
- [ ] _Add your own:_

### Variations to try

- _Iterative bottom-up merge sort_ — avoids recursion, builds up from size 1, 2, 4, ...
- _In-place merge_ — clever but complicated; usually not worth the complexity
- _Sort a linked list_ — merge sort is the standard choice
- _Count inversions while sorting_ — classic interview question
- _Your idea:_

### Related problems (uses the merge step)

- [ ] [LC 88 — Merge Sorted Array](https://leetcode.com/problems/merge-sorted-array/)
- [ ] [LC 21 — Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/)
- [ ] [LC 23 — Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/)
- [ ] [LC 148 — Sort List](https://leetcode.com/problems/sort-list/) (linked list version)
- [ ] [LC 493 — Reverse Pairs](https://leetcode.com/problems/reverse-pairs/) (count inversions during merge)
- [ ] [GFG — Count Inversions](https://www.geeksforgeeks.org/problems/inversion-of-array-1587115620/1)

---

## Pattern Recognition: Divide and Conquer

Merge sort introduces the powerful **divide-and-conquer** template:

```java
solve(problem):
    if (problem is small enough)
        return baseCase(problem)
    
    divide problem into smaller subproblems
    
    for each subproblem:
        recursive_result = solve(subproblem)
    
    return combine(recursive_results)
```

This template applies to many problems:

| Problem | Divide | Conquer | Combine |
|:--------|:-------|:--------|:--------|
| **Merge sort** | Split in half | Sort each half | Merge sorted halves |
| Quick sort | Partition by pivot | Sort each side | (Already in place — no combine) |
| Binary search | Pick midpoint | Search one half | (No combine — answer found) |
| Closest pair (geometry) | Split by x-coordinate | Find closest in each half | Check pairs across split |
| Strassen's matrix multiply | Split into quadrants | Recursive multiplications | Combine submatrices |
| Karatsuba multiplication | Split number digits | 3 sub-multiplications | Combine with shifts |

Once you see this pattern, you'll recognize it everywhere.

---

## Pattern Recognition: Your Sorting Toolkit

| Sort | Approach | Best for |
|:-----|:---------|:---------|
| Insertion | Build sorted region by inserting | Small or nearly-sorted data |
| **Merge** | **Divide and conquer + merge** | **Stable O(n log n), large data** |
| Quick | Partition + recurse | In-place average-case fastest |
| Heap | Build heap, extract max | Worst-case O(n log n), in-place |

---

_Last updated: <!-- add date here -->_
