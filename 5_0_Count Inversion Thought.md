# Count Inversions

> Use the merge step of merge sort to count inversions for free — O(n log n).

**Problem links:**
- [GFG — Count Inversions](https://www.geeksforgeeks.org/problems/inversion-of-array-1587115620/1)
- Related: [LC 493 — Reverse Pairs](https://leetcode.com/problems/reverse-pairs/) (harder variant)

---

## Table of Contents

1. [What is an Inversion?](#1-what-is-an-inversion)
2. [The Brute Force](#2-the-brute-force)
3. [The Brilliant Insight](#3-the-brilliant-insight)
4. [Why Merging Sorted Halves Reveals Inversions](#4-why-merging-sorted-halves-reveals-inversions)
5. [The Magic Formula](#5-the-magic-formula)
6. [Why We Don't Lose Information by Sorting](#6-why-we-dont-lose-information-by-sorting)
7. [The Convention in This Code](#7-the-convention-in-this-code)
8. [The Java Code](#8-the-java-code)
9. [Mapping the Logic to the Code](#9-mapping-the-logic-to-the-code)
10. [Full Trace Example](#10-full-trace-example)
11. [Complexity](#11-complexity)
12. [Common Pitfalls](#12-common-pitfalls)
13. [The "Solve a Different Problem with a Sort" Pattern](#13-the-solve-a-different-problem-with-a-sort-pattern)
14. [My Own Questions & Examples](#14-my-own-questions--examples)

---

## 1. What is an Inversion?

An **inversion** is a pair of indices `(i, j)` where `i < j` but `arr[i] > arr[j]`. In plain English: *"two elements that are out of order relative to a sorted arrangement."*

**Example:** `arr = [2, 4, 1, 3, 5]`

Let's check all pairs `(i, j)` with `i < j`:

| Pair | indices | values | Inversion? |
|:----:|:-------:|:------:|:----------:|
| (0,1) | 0 < 1 | 2, 4 | 2 < 4, no |
| (0,2) | 0 < 2 | 2, 1 | 2 > 1, ✓ |
| (0,3) | 0 < 3 | 2, 3 | 2 < 3, no |
| (0,4) | 0 < 4 | 2, 5 | 2 < 5, no |
| (1,2) | 1 < 2 | 4, 1 | 4 > 1, ✓ |
| (1,3) | 1 < 3 | 4, 3 | 4 > 3, ✓ |
| (1,4) | 1 < 4 | 4, 5 | 4 < 5, no |
| (2,3) | 2 < 3 | 1, 3 | no |
| (2,4) | 2 < 4 | 1, 5 | no |
| (3,4) | 3 < 4 | 3, 5 | no |

**Total: 3 inversions.**

### What inversions tell us

The inversion count measures **how far the array is from being sorted**:
- A sorted array has **0 inversions**.
- A reverse-sorted array of size `n` has the maximum: **n(n-1)/2 inversions**.
- It's also the minimum number of adjacent swaps needed to sort the array via bubble sort.

> 📝 **My notes:**
> - _Inversions are a measure of "disorder"_
> -

---

## 2. The Brute Force

Two nested loops checking every pair:

```java
long count = 0;
for (int i = 0; i < n; i++) {
    for (int j = i + 1; j < n; j++) {
        if (arr[i] > arr[j]) count++;
    }
}
return count;
```

**Complexity:** `O(n²)` time, `O(1)` space.

For `n = 10⁵`, this is `10¹⁰` operations — far too slow. We need O(n log n).

---

## 3. The Brilliant Insight

> **Inversions can be counted as a side-effect of merge sort — for free, in O(n log n).**

This is one of the most elegant algorithm tricks in computer science. The idea:

1. Merge sort already touches every pair of elements (across recursion levels).
2. At each merge step, we have two sorted halves.
3. Whenever an element from the **right** half is picked before the left is exhausted, all remaining left elements are bigger — and each forms an inversion with that right element.
4. We can count these inversions in **one operation per pick**.

The total cost is the same as merge sort: O(n log n). The inversion count comes "for free."

---

## 4. Why Merging Sorted Halves Reveals Inversions

This is the part to really internalize. Let's set up the situation precisely.

Consider a merge step on these two sorted halves:
```
Left half:  [3, 5, 7]   (sorted)
Right half: [2, 4, 6]   (sorted)
```

The original array (before recursion sorted the halves) was something like `[3, 5, 7, 2, 4, 6]`. The left half occupied earlier indices than the right half.

So for any element `L` in the left half and `R` in the right half:
- `L`'s original index < `R`'s original index ✓ (by construction)
- If `L > R` → that's an inversion!

**During the merge, when do we discover `L > R`?** When we're comparing front elements and the right's front is smaller. We pick the right element. But if the right element is smaller than the left's front, it's also smaller than ALL the rest of the left half (because the left is sorted ascending).

> 💡 **Key insight:** picking from the right means EVERY remaining left element forms an inversion with this right element.

### Visual

```
Left:  [ 3,  5,  7 ]   (i pointer at 3)
Right: [ 2,  4,  6 ]   (j pointer at 2)

Compare: 3 vs 2 → 2 wins.
Pick 2 from right.

Now: 2 forms inversion with 3 (yes, 3 > 2)
     2 forms inversion with 5 (yes, 5 > 2)
     2 forms inversion with 7 (yes, 7 > 2)

Count: +3 inversions in one operation.
```

We don't need to manually check each pair — the sorted property of the left half guarantees all remaining elements are bigger.

---

## 5. The Magic Formula

When `arr[i] > arr[j]` during merge (right wins):

```java
invCount += (mid - i);
```

**What does `(mid - i)` represent?**

In this code's convention, the left half is `[left..mid-1]`. So when pointer `i` is somewhere in the left half:
- Elements still in the left half: indices `i, i+1, ..., mid-1`
- Count of those elements: `(mid - 1) - i + 1 = mid - i`

That's exactly how many inversions to add when we pick from the right.

> 📝 **My notes:**
> - _The formula counts the LEFT remainders, not the RIGHT remainders_
> - _Memorize: "right wins → count left remainders"_
> -

---

## 6. Why We Don't Lose Information by Sorting

You might worry: "If we sort the array as we go, don't we lose track of the original order?"

**The answer:** inversions are about **relative order between pairs**. The recursion structure ensures we count each pair exactly when it matters and exactly once.

### Where each inversion gets counted

Consider any pair `(i, j)` with `i < j`. They start in the same array. As recursion divides:

- **Case 1:** Both `i` and `j` end up in the same half at some level. Then they'll both be present in some smaller subarray, and will eventually be compared in some merge.

- **Case 2:** `i` and `j` end up in different halves at some level — `i` in the left half, `j` in the right half. They'll be compared during the merge of those two halves.

In both cases, the pair is examined during exactly one merge — when they're first in opposite halves of a merge step. That's where their inversion status (if any) gets counted.

### Why sorting after counting is safe

Once we've counted inversions for a particular merge level, the relative order of those elements no longer matters for **future** merges. Future merges deal with pairs that span across larger halves — pairs we haven't seen yet. As long as each level's elements are sorted within their range, future merges work correctly.

> 💡 **The pattern:** count first (using current ordering), then sort (for the next level's benefit). This is why the inversion count is added BEFORE the swap/copy in the merge.

> 📝 **My notes:**
> - _Each pair is "judged" exactly once — at the merge level where they're in opposite halves_
> -

---

## 7. The Convention in This Code

> ⚠️ **Heads up:** this implementation uses a slightly different convention from "standard" merge sort.

**Standard convention:**
- Left half: `[s..mid]` (inclusive)
- Right half: `[mid+1..e]` (inclusive)
- `merge(arr, s, mid, e)` is called with three boundaries

**This code's convention:**
- Left half: `[left..mid-1]` (inclusive)
- Right half: `[mid..right]` (inclusive)
- `merge(arr, temp, left, mid, right)` where `mid` is the START of the right half
- The driver passes `mid + 1` as `mid`, but inside merge, that value (called `mid` locally) is the start of the right half

So in the merge function:
- `i` walks from `left` to `mid - 1` (the left half)
- `j` walks from `mid` to `right` (the right half)

This explains the loop conditions `i <= mid - 1` and `j <= right`, and the formula `mid - i` (which counts elements from `i` to `mid - 1`).

> 📝 **My notes:**
> - _Trace through carefully — the `mid` boundary is different from standard merge sort_
> - _Either convention works; just be consistent_
> -

---

## 8. The Java Code

```java
public class CountInversions {

    static long merge(long[] arr, long[] temp, int left, int mid, int right) {
        int i = left;     // pointer for left half [left..mid-1]
        int j = mid;      // pointer for right half [mid..right]
        int k = left;     // pointer for temp buffer
        long invCount = 0;

        while (i <= mid - 1 && j <= right) {
            if (arr[i] <= arr[j]) {
                temp[k++] = arr[i++];
            } else {
                temp[k++] = arr[j++];
                invCount += (mid - i);   // ← count inversions here
            }
        }

        // Drain remaining elements
        while (i <= mid - 1) temp[k++] = arr[i++];
        while (j <= right)   temp[k++] = arr[j++];

        // Copy back to original array
        for (int x = left; x <= right; x++) {
            arr[x] = temp[x];
        }
        return invCount;
    }

    static long mergeSort(long[] arr, long[] temp, int left, int right) {
        long invCount = 0;
        if (right > left) {
            int mid = (left + right) / 2;
            invCount += mergeSort(arr, temp, left, mid);
            invCount += mergeSort(arr, temp, mid + 1, right);
            invCount += merge(arr, temp, left, mid + 1, right);
        }
        return invCount;
    }

    public static long inversionCount(long[] arr, int n) {
        long[] temp = new long[n];
        return mergeSort(arr, temp, 0, n - 1);
    }
}
```

> 💡 **Why `long` for the count?** With `n = 10⁵` and a reverse-sorted array, the count is about `5 × 10⁹` — overflows `int`. Always use `long` for inversion counts.

---

## 9. Mapping the Logic to the Code

### The driver
```java
long[] temp = new long[n];
return mergeSort(arr, temp, 0, n - 1);
```
Allocate one shared temp buffer (more efficient than allocating per recursion). Start the recursion on the full array.

### The recursive divide
```java
if (right > left) {
    int mid = (left + right) / 2;
    invCount += mergeSort(arr, temp, left, mid);
    invCount += mergeSort(arr, temp, mid + 1, right);
    invCount += merge(arr, temp, left, mid + 1, right);
}
```

**Translation:** *"Recursively count inversions in the left half. Then in the right half. Then count cross-inversions during merge. Sum them all."*

The total inversion count splits into **three disjoint groups**:
1. Inversions within the left half (counted by recursive call on left)
2. Inversions within the right half (counted by recursive call on right)
3. Inversions where one element is in the left, the other in the right (counted during merge)

Every pair belongs to exactly one of these groups → no double counting, no misses.

### The key counting line
```java
else {
    temp[k++] = arr[j++];
    invCount += (mid - i);
}
```

**Translation:** *"Right wins. Pick from right. Count: `mid - i` elements remain in the left half (all bigger than the picked right element), so add that many inversions."*

> 📝 **My notes:**
> - _The three-group decomposition is the secret sauce_
> -

---

## 10. Full Trace Example

**Input:** `arr = [2, 4, 1, 3, 5]`, n = 5
**Expected:** 3 inversions

### Recursion tree
```
mergeSort(0, 4)
├── mergeSort(0, 2)        ← [2, 4, 1]
│   ├── mergeSort(0, 1)    ← [2, 4]
│   │   ├── mergeSort(0,0) → 0 inversions
│   │   ├── mergeSort(1,1) → 0 inversions
│   │   └── merge(0,1,1)   → merges [2] and [4]
│   ├── mergeSort(2, 2)    → 0 inversions
│   └── merge(0,2,2)       → merges [2,4] and [1]
├── mergeSort(3, 4)        ← [3, 5]
│   ├── mergeSort(3,3) → 0
│   ├── mergeSort(4,4) → 0
│   └── merge(3,4,4)   → merges [3] and [5]
└── merge(0,3,4)            → merges [1,2,4] and [3,5]
```

### Merge 1: merge(arr, temp, 0, 1, 1)
Left = `[2]`, Right = `[4]`. (`i = 0`, `j = 1`, `mid = 1`)

- `arr[0]=2 ≤ arr[1]=4` → take 2. No inversion.
- Drain right: take 4.

Result: `[2, 4]`. Inversions added: **0**.

### Merge 2: merge(arr, temp, 0, 2, 2)
Left = `[2, 4]`, Right = `[1]`. (`i = 0`, `j = 2`, `mid = 2`)

| Step | i | j | arr[i] | arr[j] | Result | Inversions |
|:----:|:-:|:-:|:------:|:------:|:-------|:----------:|
| 1 | 0 | 2 | 2 | 1 | 2 > 1 → take 1, count = mid - i = 2 - 0 = **2** | +2 |
| 2 | 0 | 3 | — | — | j > right, drain left | — |
| 3 | drain | | | | take 2, take 4 | — |

Result: `[1, 2, 4]`. Inversions added: **2** (pairs: `(2,1)`, `(4,1)`)

### Merge 3: merge(arr, temp, 3, 4, 4)
Left = `[3]`, Right = `[5]`. Trivial — no inversion.

### Merge 4 (final): merge(arr, temp, 0, 3, 4)
Left = `[1, 2, 4]`, Right = `[3, 5]`. (`i = 0`, `j = 3`, `mid = 3`)

| Step | i | j | arr[i] | arr[j] | Result | Inversions |
|:----:|:-:|:-:|:------:|:------:|:-------|:----------:|
| 1 | 0 | 3 | 1 | 3 | 1 ≤ 3 → take 1 | 0 |
| 2 | 1 | 3 | 2 | 3 | 2 ≤ 3 → take 2 | 0 |
| 3 | 2 | 3 | 4 | 3 | 4 > 3 → take 3, count = mid - i = 3 - 2 = **1** | +1 |
| 4 | 2 | 4 | 4 | 5 | 4 ≤ 5 → take 4 | 0 |
| 5 | drain | | | | take 5 | 0 |

Result: `[1, 2, 3, 4, 5]`. Inversions added: **1** (pair: `(4, 3)`)

### Total
0 (merge 1) + 2 (merge 2) + 0 (merge 3) + 1 (merge 4) = **3** ✓

> 📝 **My notes:**
> - _The 3 inversions are: (2,1), (4,1), (4,3) — exactly what we found in section 1_
> - _Try tracing [5, 4, 3, 2, 1] (reverse sorted, max inversions = 10)_
> -

---

## 11. Complexity

- **Time:** `O(n log n)` — same as merge sort. Counting is folded into the existing work.
- **Space:** `O(n)` — for the temp buffer (allocated once at the top, reused).

For `n = 10⁵`:
- Brute force: `10¹⁰` ops → too slow
- This algorithm: `~10⁶` ops → instant

This is a **dramatic** improvement that comes from a tiny algorithmic insight.

---

## 12. Common Pitfalls

- ❌ **Using `int` instead of `long` for the count.** Reverse-sorted array of size 10⁵ has ~5×10⁹ inversions. Overflows `int`!
- ⚠️ **Wrong formula for the inversion count.** It's `(mid - i)` in this convention, NOT `(right - i)` or `(j - i)`. Must count remaining LEFT elements, not right.
- ⚠️ **Mixing up the two conventions.** This code uses `mid` as the start of the right half (left half is `[left..mid-1]`). Standard merge sort uses `mid` as end of left. Be consistent.
- ⚠️ **Counting AFTER swapping/sorting.** Inversions must be counted while the relative order is still meaningful — i.e., during the merge comparison, not after.
- ⚠️ **Allocating `temp` inside merge().** Wasteful. Allocate once at the top level and pass the same buffer down.
- ⚠️ **Checking `right >= left` instead of `right > left`.** `>=` would cause infinite recursion on a single-element range; `>` correctly stops at the base case.

> 📝 **My notes:**
> - _What edge cases tripped me up?_
> -

---

## 13. The "Solve a Different Problem with a Sort" Pattern

Counting inversions teaches a powerful template: **piggyback work onto a sort**. The sort visits every element in a structured way; we hijack that traversal to compute something else.

### Other problems using this pattern

| Problem | What you compute | Where in the sort |
|:--------|:----------------|:------------------|
| **Count inversions** (this) | # pairs out of order | During merge comparison |
| Reverse pairs (LC 493) | # pairs where `arr[i] > 2*arr[j]` | Extra pass before merge |
| Count of smaller after self (LC 315) | For each `i`, count `j > i` with `arr[j] < arr[i]` | Index tracking during merge |
| Maximum sum subarray of size K | — | Variation of partition |
| K-th smallest element | Pick the K-th during quickselect | Partition step |
| Closest pair of points | Min distance | During merge in 2D divide-and-conquer |
| Shortest path in DAG | Distances in topological order | During topological sort |

### The general technique

1. Identify what you want to count or compute.
2. Find a sort or traversal that visits the relevant element groups.
3. Add a constant amount of work at each step of the sort to update your answer.
4. Total complexity stays the same as the underlying sort.

This "trojan horse" approach turns many `O(n²)` problems into `O(n log n)`.

> 📝 **My notes:**
> - _When you see "count pairs satisfying X", think merge sort_
> -

---

## 14. My Own Questions & Examples

Use the space below to write down your own questions, edge cases to think about, or alternative approaches you want to remember.

### Questions

-
-
-

### Edge cases to test

- [ ] Empty array: `[]` → 0
- [ ] Single element: `[5]` → 0
- [ ] Sorted: `[1, 2, 3, 4, 5]` → 0 (fewest)
- [ ] Reverse sorted: `[5, 4, 3, 2, 1]` → 10 (n(n-1)/2 = 5·4/2 = 10, the max)
- [ ] All same: `[3, 3, 3, 3]` → 0 (equal elements aren't inversions because we use `≤`)
- [ ] Two elements: `[2, 1]` → 1
- [ ] Large array (n = 10⁵, random) — should run in well under 1 second
- [ ] _Add your own:_

### Variations to try

- _Count "specific" inversions: pairs where `arr[i] > 2·arr[j]`_ (LeetCode 493 — Reverse Pairs)
- _For each element, count smaller elements to its right_ (LC 315 — uses index tracking)
- _Count inversions using Fenwick / BIT tree_ — alternative O(n log n) approach
- _Count pairs with given difference_ — sort + two pointers
- _Your idea:_

### Related problems

- [ ] [GFG — Inversion of Array](https://www.geeksforgeeks.org/problems/inversion-of-array-1587115620/1)
- [ ] [LC 493 — Reverse Pairs](https://leetcode.com/problems/reverse-pairs/) (Hard)
- [ ] [LC 315 — Count of Smaller Numbers After Self](https://leetcode.com/problems/count-of-smaller-numbers-after-self/) (Hard)
- [ ] [LC 327 — Count of Range Sum](https://leetcode.com/problems/count-of-range-sum/) (Hard, uses similar merge counting)
- [ ] [GFG — Count pairs with given sum](https://www.geeksforgeeks.org/problems/count-pairs-with-given-sum5022/1)

---

## Pattern Recognition: "Hard Problem + Standard Algorithm = Elegant Solution"

The lesson here is bigger than just inversions. When you face a problem that seems to require examining all pairs (`O(n²)`), ask:

> *"Is there a divide-and-conquer or sorting algorithm that already touches all the elements I care about? Can I piggyback my counting onto that?"*

This thinking unlocks many "hard" problems. Examples:

| Hard Problem | Standard Algorithm | Insight |
|:-------------|:-------------------|:--------|
| Count inversions | Merge sort | Merge step compares cross-half pairs |
| Closest pair of points | Divide and conquer | Recursive halving of plane |
| Maximum subarray | Divide and conquer | "Crossing" subarrays during merge |
| Counting smaller right | Merge sort | Track original indices during merge |
| Quickselect (kth element) | Quicksort partition | Skip irrelevant half |

Once you internalize this, you'll see opportunities to apply it everywhere.

---

_Last updated: <!-- add date here -->_
