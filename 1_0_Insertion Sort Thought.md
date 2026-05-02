# Insertion Sort

> The "playing cards" sort — simple, intuitive, and surprisingly useful.

**Reference links:**
- [GFG — Insertion Sort](https://www.geeksforgeeks.org/insertion-sort/)
- [Wikipedia — Insertion Sort](https://en.wikipedia.org/wiki/Insertion_sort)

---

## Table of Contents

1. [The Intuition — Sorting Cards in Your Hand](#1-the-intuition--sorting-cards-in-your-hand)
2. [The Invariant](#2-the-invariant)
3. [The Algorithm in Plain English](#3-the-algorithm-in-plain-english)
4. [The Java Code](#4-the-java-code)
5. [Mapping the Logic to the Code](#5-mapping-the-logic-to-the-code)
6. [Why the Inner Loop is Tricky](#6-why-the-inner-loop-is-tricky)
7. [Full Trace Example](#7-full-trace-example)
8. [Complexity](#8-complexity)
9. [When to Use Insertion Sort](#9-when-to-use-insertion-sort)
10. [Common Pitfalls](#10-common-pitfalls)
11. [Insertion Sort vs Other Sorts](#11-insertion-sort-vs-other-sorts)
12. [My Own Questions & Examples](#12-my-own-questions--examples)

---

## 1. The Intuition — Sorting Cards in Your Hand

Imagine you're dealt cards one at a time, and you want to keep them sorted in your hand:

```
Hand: empty
Deal: 5  →  Hand: [5]
Deal: 2  →  Hand: [2, 5]      (slid 2 in front of 5)
Deal: 8  →  Hand: [2, 5, 8]   (8 is bigger than everything, goes at end)
Deal: 1  →  Hand: [1, 2, 5, 8] (slid 1 to the very front)
Deal: 3  →  Hand: [1, 2, 3, 5, 8] (slid 3 between 2 and 5)
```

That's exactly what insertion sort does. The "hand" is the **sorted left portion** of the array, and we're inserting one new element from the unsorted right portion at each step.

> 📝 **My notes:**
> - _This is how most people naturally sort small things in real life_
> -

---

## 2. The Invariant

The single most important idea behind insertion sort:

> **At the start of iteration `i`, the subarray `a[0..i-1]` is sorted.**

This is called a **loop invariant** — a property that's guaranteed to hold every time the outer loop starts a new iteration.

```
Initial:  a[0..0] is trivially sorted (single element)
After i=1: a[0..1] sorted
After i=2: a[0..2] sorted
...
After i=n-1: a[0..n-1] sorted ← that's the entire array!
```

Each iteration **extends the sorted region by one element**. After `n-1` iterations, the whole array is sorted.

---

## 3. The Algorithm in Plain English

For each index `i` from 1 to `n-1`:
1. **Pick up** `a[i]` and remember it as `e`.
2. **Walk backwards** through the sorted left portion using pointer `j`.
3. **Shift** every element greater than `e` one slot to the right.
4. **Drop** `e` into the empty slot.

The "empty slot" is the position where the shifting stopped — that's `j + 1` after the inner loop ends.

---

## 4. The Java Code

```java
public static void insertionSort(int[] a) {
    int n = a.length;
    for (int i = 1; i < n; i++) {
        int e = a[i];           // pick up the next card
        int j = i - 1;          // start from the last sorted position

        // Shift larger elements to the right
        while (j >= 0 && a[j] > e) {
            a[j + 1] = a[j];
            j--;
        }

        a[j + 1] = e;           // drop e into the empty slot
    }
}
```

---

## 5. Mapping the Logic to the Code

### Outer loop
```java
for (int i = 1; i < n; i++)
```
**Translation:** *"For each new element starting from index 1 (since index 0 alone is trivially sorted)..."*

### Pick up the element
```java
int e = a[i];
```
**Translation:** *"Save the value at position `i` because we're about to overwrite that slot during shifting."*

This is critical! Without saving `e`, the original value would be lost when we shift.

### Walk backwards
```java
int j = i - 1;
```
**Translation:** *"Start comparing with the rightmost element of the sorted portion."*

### Shift right
```java
while (j >= 0 && a[j] > e) {
    a[j + 1] = a[j];
    j--;
}
```
**Translation:** *"While there's still a sorted element to my left AND it's bigger than `e`, shift it one slot to the right and keep walking left."*

### Drop the element
```java
a[j + 1] = e;
```
**Translation:** *"`j` is now either -1 (walked off the edge) or pointing to an element that's `≤ e`. Either way, `j + 1` is exactly where `e` belongs."*

> 📝 **My notes:**
> - _Why `j + 1`? Because after the loop, j moved one step too far to the left_
> -

---

## 6. Why the Inner Loop is Tricky

The inner loop has **two exit conditions** — beginners often miss this:

### Exit 1: `j < 0`
We walked off the left edge of the array. This means `e` is **smaller than every element in the sorted portion**. It should go at index 0. After the loop, `j = -1`, so `j + 1 = 0`. ✓

### Exit 2: `a[j] <= e`
We found an element that's NOT bigger than `e`. It (and everything to its left) is sorted AND ≤ `e`. So `e` belongs right after it: index `j + 1`. ✓

Both cases are handled by the same final line `a[j + 1] = e`. That's the elegance.

### Visual of both exits

```
Case 1: e is smaller than everything
  Before:  [ 3, 5, 7, 9 | e=2 ]
  Shift:   [ _, 3, 5, 7, 9 ]    j walked to -1
  Place:   [ 2, 3, 5, 7, 9 ]    a[0] = 2 ✓

Case 2: e fits in the middle
  Before:  [ 1, 4, 7, 9 | e=5 ]
  Shift:   [ 1, 4, _, 7, 9 ]    j stopped at index 1 (a[1]=4 ≤ 5)
  Place:   [ 1, 4, 5, 7, 9 ]    a[2] = 5 ✓
```

> 📝 **My notes:**
> - _The short-circuit `&&` is essential — without it, j=-1 would cause a[-1] access_
> -

---

## 7. Full Trace Example

**Input:** `a = [5, 2, 8, 1, 3]`

### Iteration i = 1 (e = 2)
```
Start:  [5, 2, 8, 1, 3]    e=2, j=0
Check:  a[0]=5 > 2 → shift
After:  [5, 5, 8, 1, 3]    j=-1, exit loop
Place:  [2, 5, 8, 1, 3]    a[0]=2
```

### Iteration i = 2 (e = 8)
```
Start:  [2, 5, 8, 1, 3]    e=8, j=1
Check:  a[1]=5 > 8? No → exit immediately
Place:  [2, 5, 8, 1, 3]    a[2]=8 (unchanged)
```

### Iteration i = 3 (e = 1)
```
Start:  [2, 5, 8, 1, 3]    e=1, j=2
Check:  a[2]=8 > 1 → shift  →  [2, 5, 8, 8, 3]   j=1
Check:  a[1]=5 > 1 → shift  →  [2, 5, 5, 8, 3]   j=0
Check:  a[0]=2 > 1 → shift  →  [2, 2, 5, 8, 3]   j=-1
Place:  [1, 2, 5, 8, 3]    a[0]=1
```

### Iteration i = 4 (e = 3)
```
Start:  [1, 2, 5, 8, 3]    e=3, j=3
Check:  a[3]=8 > 3 → shift  →  [1, 2, 5, 8, 8]   j=2
Check:  a[2]=5 > 3 → shift  →  [1, 2, 5, 5, 8]   j=1
Check:  a[1]=2 > 3? No → exit
Place:  [1, 2, 3, 5, 8]    a[2]=3
```

**Final:** `[1, 2, 3, 5, 8]` ✓

> 📝 **My notes:**
> - _Try tracing [9, 8, 7, 6, 5] (worst case — already reversed)_
> - _Try tracing [1, 2, 3, 4, 5] (best case — already sorted)_
> -

---

## 8. Complexity

| Case | Time | When |
|:-----|:----:|:-----|
| Best | `O(n)` | Already sorted — inner loop exits immediately every time |
| Average | `O(n²)` | Random order |
| Worst | `O(n²)` | Reverse sorted — every element shifts all the way to the front |

**Space:** `O(1)` — sorts in place, no extra arrays.

### Why O(n²) in worst case

For a reverse-sorted array `[5, 4, 3, 2, 1]`:
- i=1: 1 shift
- i=2: 2 shifts
- i=3: 3 shifts
- i=4: 4 shifts

Total: `1 + 2 + 3 + ... + (n-1) = n(n-1)/2 ≈ n²/2` operations.

### Why O(n) in best case

For a sorted array `[1, 2, 3, 4, 5]`:
- Every iteration's first comparison `a[j] > e` is false
- Inner loop exits immediately, doing zero shifts
- Just `n-1` outer loop iterations → `O(n)`

This makes insertion sort **adaptive** — it runs fast on nearly-sorted data.

---

## 9. When to Use Insertion Sort

Despite its O(n²) worst case, insertion sort is genuinely useful in real systems:

✅ **Small arrays (n ≤ 10-20).** The constant factor is so small that it beats more complex sorts. Many production sorts (like Java's `Arrays.sort` for primitives) switch to insertion sort for small sub-arrays.

✅ **Nearly sorted data.** With only `k` out-of-place elements, runtime is `O(nk)` — much better than the O(n log n) of merge/quicksort for small `k`.

✅ **Streaming / online sorting.** When data arrives one element at a time, insertion sort handles it naturally — each new element is just inserted into the sorted portion.

✅ **Stable sort.** Equal elements keep their original relative order (because we use `>`, not `>=`).

❌ **Avoid for large unsorted arrays.** Use merge sort, quicksort, or library functions instead.

> 📝 **My notes:**
> - _Look up "Timsort" — it combines insertion sort with merge sort_
> -

---

## 10. Common Pitfalls

- ❌ **Forgetting to save `e` before shifting.** Without `int e = a[i]`, the value at `a[i]` would be overwritten on the first shift.
- ⚠️ **Using `>=` instead of `>` in the comparison.** This breaks stability — equal elements would shift past each other unnecessarily, and the relative order of equal elements would change.
- ⚠️ **Forgetting `j >= 0` before `a[j] > e`.** The order matters! Java short-circuits `&&`, so `j >= 0` must come first to avoid `ArrayIndexOutOfBoundsException` when `j = -1`.
- ⚠️ **Starting outer loop at `i = 0`.** That would compare a[0] with itself or read a[-1]. Always start at `i = 1`.
- ⚠️ **Using `a[j+1] = e` inside the inner loop.** That would overwrite the shifted value. The placement happens ONCE, after the loop.

> 📝 **My notes:**
> - _What edge cases tripped me up?_
> -

---

## 11. Insertion Sort vs Other Sorts

| Sort | Best | Average | Worst | Space | Stable | In-place |
|:-----|:----:|:-------:|:-----:|:-----:|:------:|:--------:|
| **Insertion** | O(n) | O(n²) | O(n²) | O(1) | ✅ | ✅ |
| Bubble | O(n) | O(n²) | O(n²) | O(1) | ✅ | ✅ |
| Selection | O(n²) | O(n²) | O(n²) | O(1) | ❌ | ✅ |
| Merge | O(n log n) | O(n log n) | O(n log n) | O(n) | ✅ | ❌ |
| Quick | O(n log n) | O(n log n) | O(n²) | O(log n) | ❌ | ✅ |
| Heap | O(n log n) | O(n log n) | O(n log n) | O(1) | ❌ | ✅ |

### Why insertion beats bubble in practice

Both are O(n²), but insertion sort:
- Does **fewer comparisons on average** (it stops as soon as the slot is found)
- Has **simpler inner loop** with better cache behavior
- Is the only O(n²) sort that's actually used in production code (for small subarrays)

---

## 12. My Own Questions & Examples

Use the space below to write down your own questions, edge cases to think about, or alternative approaches you want to remember.

### Questions

-
-
-

### Edge cases to test

- [ ] Empty array: `[]` → `[]`
- [ ] Single element: `[5]` → `[5]`
- [ ] Already sorted: `[1, 2, 3, 4, 5]` → unchanged, O(n) time
- [ ] Reverse sorted: `[5, 4, 3, 2, 1]` → worst case
- [ ] All same: `[3, 3, 3, 3]` → no shifts at all
- [ ] Two elements: `[2, 1]` → swap to `[1, 2]`
- [ ] Negative numbers: `[3, -1, 0, -5]`
- [ ] _Add your own:_

### Variations to try

- _Sort in descending order_ — change `a[j] > e` to `a[j] < e`
- _Use binary search to find the insertion point_ — "Binary Insertion Sort", reduces comparisons but not shifts
- _Sort a linked list_ — even more elegant since shifting becomes pointer manipulation
- _Your idea:_

### Related problems

- [ ] [LC 147 — Insertion Sort List](https://leetcode.com/problems/insertion-sort-list/)
- [ ] [LC 75 — Sort Colors](https://leetcode.com/problems/sort-colors/)
- [ ] [LC 912 — Sort an Array](https://leetcode.com/problems/sort-an-array/)
- [ ] [GFG — Binary Insertion Sort](https://www.geeksforgeeks.org/binary-insertion-sort/)

---

## Pattern Recognition: Where This Fits

Adding to your sorting toolkit:

| Sort | Approach | Best for |
|:-----|:---------|:---------|
| **Insertion** | **Build sorted portion one element at a time by inserting** | **Small or nearly-sorted data** |
| Selection | Repeatedly find min and swap to front | Educational only |
| Bubble | Repeatedly swap adjacent out-of-order pairs | Educational only |
| Merge | Divide, sort halves, merge | Stable O(n log n) |
| Quick | Partition around pivot, recurse | Average-case fastest |
| Heap | Build heap, extract max repeatedly | Worst-case O(n log n), in-place |

The insertion-sort mindset — **"maintain a sorted region and grow it"** — also shows up in problems like:
- Online median (use two heaps)
- Insertion into sorted linked list
- Patience sort (longest increasing subsequence)

---

_Last updated: <!-- add date here -->_
