# Fractional Knapsack — Detailed Logic Walkthrough

This document explains the Java code for the **Fractional Knapsack** problem in depth — the problem itself, the greedy intuition, why the chosen comparator works, a line-by-line walk through the algorithm, a worked example, edge cases, complexity, and common pitfalls.

---

## 1. The Problem

You are given:

- A knapsack with a maximum weight capacity `W`.
- `n` items, where each item `i` has a `weight[i]` and a `value[i]`.

You can take **fractions** of items (unlike the 0/1 knapsack, where you must take an item whole or leave it). The goal is to maximize the **total value** placed in the knapsack without exceeding capacity `W`.

The output is the **maximum total value** as a `double`, because partial items lead to fractional values.

### Why "fractional" changes everything

In the 0/1 knapsack, each item is atomic — take it or leave it — and the problem becomes NP-hard in general (solvable in pseudo-polynomial time via DP). The moment you allow fractions, a **greedy** strategy becomes optimal, and the problem drops to `O(n log n)` (dominated by sorting).

---

## 2. The Key Insight: Value-to-Weight Ratio

Think of each item not by its raw value, but by its **value density** — how much value you get per unit of weight:

```
ratio_i = value_i / weight_i
```

This is the "bang for the buck." If the knapsack can only hold so much weight, you want every unit of weight you spend to buy you as much value as possible. So:

> **Greedy rule:** Always fill the knapsack with the item that has the highest value/weight ratio first. When that item is exhausted, move to the next-highest ratio. If only part of an item fits, take the fraction that fits.

### Why this greedy choice is optimal (intuition)

Suppose an optimal solution `O` does **not** include the item with the highest ratio, but instead uses some other item `X` of lower ratio. Swap out a small amount of `X`'s weight for the same amount of the higher-ratio item. The total weight stays the same, but the total value goes **up** (because we replaced lower-density weight with higher-density weight). This contradicts `O` being optimal. So an optimal solution must prefer higher-ratio items — exactly what the greedy algorithm does.

This argument is called an **exchange argument**, and it's the standard proof technique for greedy algorithms.

---

## 3. Walking Through the Code

### 3.1 The Comparator

```java
static class ItemComparator implements Comparator<Item> {
    @Override
    public int compare(Item a, Item b) {
        double r1 = (double) a.value / a.weight;
        double r2 = (double) b.value / b.weight;
        return Double.compare(r2, r1); // Descending order (highest ratio first)
    }
}
```

What this does, piece by piece:

- **`Comparator<Item>`** — a custom rule for ordering `Item` objects. `Arrays.sort` will use this rule instead of the default natural ordering.
- **`(double) a.value / a.weight`** — the cast to `double` is critical. If `value` and `weight` are `int`, then `a.value / a.weight` performs **integer division** and throws away the fractional part. For example, `value=7`, `weight=2` would give `3` instead of `3.5`. The cast forces floating-point division.
- **`Double.compare(r2, r1)`** — this is the line that decides the order. Let's break it down in plain English below.

#### Understanding `Double.compare(r2, r1)` simply

Java's `Arrays.sort` follows a simple convention when it calls your `compare(a, b)` method:

| What `compare(a, b)` returns | What sort thinks | Result |
|------------------------------|------------------|--------|
| **Negative** number (e.g. -1) | `a` should come **before** `b` | `a` placed first |
| **Zero** (0)                  | `a` and `b` are equal           | order unchanged |
| **Positive** number (e.g. +1) | `a` should come **after** `b`   | `b` placed first |

So whoever "looks smaller" to the compare method gets placed first.

##### The simple way: write it like `r1 - r2` (ascending) or `r2 - r1` (descending)

Yes — you can absolutely write it the subtraction way. For ints, this is the most common shorthand:

```java
// ASCENDING (smallest first)
return a.weight - b.weight;   // if a < b, this is negative → a goes first ✓

// DESCENDING (largest first)
return b.weight - a.weight;   // if a > b, b-a is negative → b goes first, meaning a (the larger) goes last... wait
                              // actually: if a > b, then b - a < 0 → b first? No.
                              // Re-check: return value negative means FIRST argument first.
                              // Here first arg is `a`. b - a < 0 when a > b → a comes first ✓ (larger first = descending)
```

Mental shortcut to remember:
- `return a - b;` → **ascending** (small first)
- `return b - a;` → **descending** (large first)

##### So why doesn't the code just write `return (int)(r2 - r1);`?

Because `r1` and `r2` are `double`s (decimals), and casting their difference to `int` is **dangerous**:

```java
double r1 = 5.0;
double r2 = 5.4;
int result = (int)(r2 - r1);   // (int)(0.4) = 0 → sort thinks they're EQUAL!
```

The fractional part gets chopped off, so two items with clearly different ratios look identical to the sort. Your ordering breaks silently.

`Double.compare(r2, r1)` solves this by safely comparing the two doubles and returning **just a sign** (-1, 0, or +1) without ever losing the decimal information.

##### Why `(r2, r1)` and not `(r1, r2)`?

`Double.compare(x, y)` returns:
- negative if `x < y`
- positive if `x > y`

Substituting `x = r2`, `y = r1`:
- returns negative when `r2 < r1` — meaning item `a` (with ratio `r1`) has the **bigger** ratio — and a negative return tells sort "put `a` first." ✓ Larger ratio comes first → **descending**, exactly what we want.

If we had written `Double.compare(r1, r2)` instead, we'd get ascending order (smallest ratio first), which would defeat the whole greedy strategy.

##### Equivalent forms — pick whichever you find clearest

All three of these produce the same descending-by-ratio sort:

```java
// Form 1: original code
return Double.compare(r2, r1);

// Form 2: same idea, just flipped arguments and negated
return -Double.compare(r1, r2);

// Form 3: subtraction style — works for doubles ONLY if you DON'T cast to int
//         Returns a double, so the compare method's signature would need to change,
//         which is why this style is avoided for doubles. For ints it's fine.
//         (Shown only for understanding — don't use this in the actual code.)
//         return (r2 - r1) > 0 ? 1 : (r2 - r1) < 0 ? -1 : 0;
```

**Summary:** `Double.compare(r2, r1)` is just a safe, precision-preserving way of saying "sort by ratio, biggest first." Think of it as the `double` cousin of `b - a` for ints.

### 3.2 The Main Method

```java
static double fractionalKnapsack(int w, Item arr[], int n) {
    Arrays.sort(arr, new ItemComparator());

    double res = 0.0;
    for (int i = 0; i < n; i++) {
        if (arr[i].weight <= w) {
            res += arr[i].value;
            w -= arr[i].weight;
        } else {
            res += (arr[i].value * w) / (double) arr[i].weight;
            break;
        }
    }
    return res;
}
```

#### Step 1 — Sort

```java
Arrays.sort(arr, new ItemComparator());
```

After this line, `arr` is reordered so that the item with the highest value/weight ratio sits at index `0`, the second-highest at index `1`, and so on. This is the foundation of the whole algorithm — every subsequent decision relies on this ordering.

#### Step 2 — Initialize Accumulator

```java
double res = 0.0;
```

`res` will hold the running total value. It's a `double` because the last item may be taken fractionally, producing a non-integer value.

#### Step 3 — The Greedy Loop

```java
for (int i = 0; i < n; i++) {
```

Iterate through items in the sorted (descending-ratio) order.

##### Case A — The whole item fits

```java
if (arr[i].weight <= w) {
    res += arr[i].value;
    w -= arr[i].weight;
}
```

If the current item's weight is less than or equal to the **remaining** capacity `w`, take the whole item:

- Add its full `value` to `res`.
- Subtract its `weight` from the remaining capacity `w`.

Note that `w` is being reused — it started as total capacity and is now being decremented to track remaining capacity. (A common code-clarity improvement is to use a separate variable like `remaining`, but the logic is the same.)

##### Case B — Only a fraction fits

```java
} else {
    res += (arr[i].value * w) / (double) arr[i].weight;
    break;
}
```

If the current item is too heavy to fit whole, take the largest fraction that still fits. The remaining capacity is `w` units of weight, so we take `w` units out of `arr[i].weight` total — that's a fraction of `w / arr[i].weight` of the item.

The value gained is:

```
arr[i].value × (w / arr[i].weight)
```

The code writes this as `(arr[i].value * w) / (double) arr[i].weight`, with the `(double)` cast on the denominator to force floating-point division.

Then **`break;`** — and this is important. Once we've taken a fraction of any item, the knapsack is **exactly full**. No further items can fit (any item we take from now on would either need to fit in zero remaining capacity, which is impossible for a positive-weight item, or we'd be re-fractioning, which we already did optimally). Breaking out is both correct and a small efficiency win.

#### Step 4 — Return

```java
return res;
```

`res` now holds the maximum achievable value.

---

## 4. Worked Example

Suppose `W = 50` and we have three items:

| Item | Value | Weight | Ratio (value/weight) |
|------|-------|--------|----------------------|
| A    | 60    | 10     | 6.0                  |
| B    | 100   | 20     | 5.0                  |
| C    | 120   | 30     | 4.0                  |

### After sorting by ratio descending

The order is already `A, B, C` (ratios 6.0, 5.0, 4.0).

### Loop iterations

- **i = 0 (item A):** weight 10 ≤ remaining 50 → take whole.
  `res = 60`, remaining `w = 50 − 10 = 40`.

- **i = 1 (item B):** weight 20 ≤ remaining 40 → take whole.
  `res = 60 + 100 = 160`, remaining `w = 40 − 20 = 20`.

- **i = 2 (item C):** weight 30 > remaining 20 → take a fraction.
  Fraction added = `120 × 20 / 30 = 80`.
  `res = 160 + 80 = 240`. Break.

**Answer:** 240.

### Sanity check — what if we'd been greedy by raw value instead?

By raw value, item C (120) is "biggest." Take C whole → uses 30 weight, gives 120 value, leaves 20 capacity. Then take half of B → adds 50. Total = 170. Worse than 240. This is exactly why we sort by **ratio**, not value.

---

## 5. Edge Cases and Pitfalls

### Empty array (`n == 0`)
The loop never executes, `res` stays at `0.0`. Correct.

### Capacity is zero (`w == 0`)
First iteration's `arr[0].weight <= 0` is false (assuming positive weights). Goes into the else branch and adds `arr[0].value * 0 / arr[0].weight = 0`, then breaks. `res = 0`. Correct.

### Total weight of all items ≤ capacity
Every item fits whole, the loop completes naturally without ever hitting the `else`, and `res` is the sum of all values. Correct.

### Item with weight 0
**This is a real pitfall.** The ratio computation `value / weight` would divide by zero, producing `Infinity` (since these are `double`s, not ints — no exception, just `Double.POSITIVE_INFINITY`). The sort would still place these items first, and the line `arr[i].weight <= w` would be `0 <= w`, true, so the item would be taken whole, contributing its value without consuming capacity. The fractional-take branch would also break because of division by zero giving `NaN` or `Infinity`. **If your inputs may include zero-weight items, handle them as a special case** before sorting, or guarantee they don't appear.

### Negative weights or values
Not meaningful for the problem. The algorithm doesn't guard against them and will produce nonsense if given them.

### Integer overflow in `arr[i].value * w`
If both are large `int`s, `arr[i].value * w` can overflow before being divided. Java doesn't throw on integer overflow — it silently wraps. The fix is to cast earlier, e.g. `((double) arr[i].value * w) / arr[i].weight`. For typical problem sizes this rarely matters, but it's a latent bug for large inputs.

### Mutating the input array
`Arrays.sort` sorts in place, so the caller's `arr` is reordered as a side effect. If the caller needs the original order, they need to pass a copy.

---

## 6. Complexity Analysis

| Step | Time | Space |
|------|------|-------|
| Sorting | O(n log n) | O(log n) for `Arrays.sort` on objects (uses TimSort) |
| Greedy loop | O(n) | O(1) |
| **Total** | **O(n log n)** | **O(log n)** (excluding the input array) |

The sort dominates. The greedy pass is a single linear scan with no nested work, so it's effectively free compared to the sort.

---

## 7. Why This Greedy Doesn't Work for 0/1 Knapsack

It's tempting to apply the same trick to the 0/1 knapsack, but it fails. Example: `W = 10`, items:

| Item | Value | Weight | Ratio |
|------|-------|--------|-------|
| A    | 6     | 6      | 1.00  |
| B    | 5     | 5      | 1.00  |
| C    | 5     | 5      | 1.00  |

Greedy by ratio (with ties broken arbitrarily) might take A (value 6, weight 6), then can't fit any more 5-weight items in the remaining 4 → total 6. Optimal is B + C = 10. The difference is that fractional knapsack can always **fill exactly to W** with a partial last item, while 0/1 may leave wasted capacity. Without fractions, you sometimes need to skip a higher-ratio item to make room for a combination that fills the knapsack better — and that's the kind of combinatorial decision that needs DP, not greed.

---

## 8. Summary

The algorithm is short, but every line is doing meaningful work:

1. **Define what "good" means** — value per unit weight (the ratio).
2. **Sort items so the best ones come first** — descending by ratio, using a comparator that uses `Double.compare` to avoid precision bugs.
3. **Greedily pack** — take whole items as long as they fit, then take a fraction of the next one to fill the remaining capacity exactly.
4. **Stop as soon as the knapsack is full** — there's nothing left to do.

The correctness follows from the exchange argument: replacing low-ratio weight with high-ratio weight never makes the solution worse, so an optimal solution must prefer high-ratio items — which is precisely the greedy choice.
