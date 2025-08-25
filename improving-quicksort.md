# A Pre-Partition Trigger for Quicksort/Introsort

An AI overview of an improvement I made to quicksort.  I wanted to test my theories of inventing new things and thought I'd take a very well known solved problem and see if I could improve it.  It took a few months, but here it is!

## Abstract

We add a small pre-check to quicksort/introsort that quickly detects if the input is already **sorted**, **reverse-sorted**, or **all equal**, and then finishes in **O(n)** (reversing in place if descending). The check reuses the pivot sample, adds a few **cheap probes** to skip wasted work on nearly sorted data, and performs **at most one** linear scan at the root. Overhead on random data is essentially constant; gains on ordered inputs are large.

---

## Motivation

Quicksort is great for random inputs but wastes effort when the array is already ordered. Real datasets often arrive sorted (timestamps, IDs) or reversed. Detecting this before partitioning avoids doing a full `O(n log n)` sort.

Two goals shaped this design:

1. **Almost free** on random inputs.
2. **No regressions** on hard cases; worst-case remains `O(n log n)` (introsort fallback).

---

## Technique Overview

The trigger runs **once at the root**:

1. **Check the pivot sample.**
   While picking the pivot (median-of-k), test if the sample looks sorted ascending or descending. This costs 0–2 extra comparisons.

2. **Cheap probes.**
   If the sample looks ordered, test a few evenly spaced pairs near the ends (4–8 checks). Any disorder aborts the fast path immediately.

3. **Optional full scan.**
   Only if the probes pass, scan the whole array once.

    * Ascending/equal ⇒ return immediately.
    * Descending/equal ⇒ reverse in place, then return.

4. **Else: normal quicksort.**

Because a random sample looks ordered with probability about `2/k!`, the full scan almost never runs unnecessarily.

---

## Detailed Steps

### Preconditions

Only run this check for `n ≥ 17`. Smaller arrays go straight to insertion sort, with a quick reverse test.

### Anchors and Direction Guard

Check `lo = A[0]`, `mid = A[n/2]`, `hi = A[n−1]`.

* If `hi < lo`, candidate descending → verify `lo ≥ mid ≥ hi`.
* Else candidate ascending → verify `lo ≤ mid ≤ hi`.
  Failure ⇒ skip fast path.

### Probes

Pick stride `δ` and test pairs:

| `n` range | stride `δ` | probes |
| --------- | ---------- | ------ |
| `< 32`    | `n >> 2`   | 4      |
| 32–255    | `n >> 3`   | 6      |
| `≥ 256`   | `n >> 4`   | 8      |

Any failure ⇒ fallback.

### Full Scan

* Ascending: scan left→right. If disorder found ⇒ fallback.
* Descending: scan right→left. If disorder found ⇒ fallback; else reverse.

On success, **return** immediately.

### Normal Fallback

If any check fails, proceed as usual: pick pivot (ninther/5-point), partition, recurse with introsort depth limit and insertion sort cutoff.

---

## Implementation Notes

* `macro_check_sorted(...)` implements the trigger and returns early if sorted/reversed.
* `macro_introsort(...)` calls it once at the top; on success, returns; else continues with normal sorting.
* Reversal mutates the ends safely because the caller exits immediately on success.

---

## Cost & Behavior

* **Overhead (random input):** \~0–2 extra comparisons + 4–8 probes.
* **Full scan:** at most once, and very rarely triggered on random data.
* **Best cases:** O(n), reverse costs \~n/2 swaps.
* **Typical/worst cases:** unchanged from introsort.

---

## Correctness & Semantics

* Comparator must be a **strict weak ordering**. With only `<` used, **all-equal** inputs are treated as sorted.
* Behavior with NaNs or non-transitive comparators follows the comparator’s semantics.
* The sort remains **unstable**. Reverse is done in place.

---

## Results (64-bit integers)

| Case              | `qsort`  | `macro_sort` | Speedup   |
| ----------------- | -------- | ------------ | --------- |
| Ordered (n=1000)  | 4056 ns  | 561 ns       | **7.2×**  |
| All equal         | 2545 ns  | 489 ns       | **5.2×**  |
| Reverse           | 21440 ns | 720 ns       | **29.8×** |
| Slightly shuffled | 3945 ns  | 4840 ns      | 0.83×     |
| Random            | 55227 ns | 9079 ns      | **6.1×**  |

---

## Pseudocode (read-only view of the trigger)

```c
bool prepartition_trigger(Item *A, size_t n, Less less) {
    if (n < 17) return false;                    // tiny segments handled elsewhere
    Item *lo = &A[0], *mid = &A[n/2], *hi = &A[n-1];

    // Direction guard using anchors
    bool desc = less(*hi, *lo);
    if (desc) {
        if (less(*lo, *mid) || less(*mid, *hi)) return false;
    } else {
        if (less(*mid, *lo) || less(*hi, *mid)) return false;
    }

    // Sparse probes
    size_t delta = (n < 32) ? (n >> 2) : (n < 256) ? (n >> 3) : (n >> 4);
    if (!sentries_pass(A, n, delta, less, desc)) return false;

    // Full scan
    if (!desc) {
        for (size_t i = 0; i + 1 < n; ++i) if (less(A[i+1], A[i])) return false;
        return true;                              // ascending/equal
    } else {
        for (size_t i = n - 1; i > 0; --i) if (less(A[i-1], A[i])) return false;
        reverse_in_place(A, n);                   // confirmed descending
        return true;
    }
}
```

---

## Takeaway

A simple **pre-partition trigger** with a few probes and one optional full scan:

* Detects already sorted or reversed arrays,
* Converts them to **O(n)** handling,
* Keeps overhead constant on random inputs,
* Leaves worst-case behavior unchanged.
