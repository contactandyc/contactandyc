# Real-Time Ad Inventory System with Inverted Index

The following is an AI overview on the patent application in the link.  The ideas below represent determining ad inventory and isn't meant for serving.  It isn't too much of a stretch to convert this to an ad serving technology.

## Overview

This document describes how to design and implement a real-time advertisement inventory system capable of handling millions of ad buys with overlapping targeting criteria. It is based on representing ad inventory as sets of impression slots, then using inverted indices and efficient allocation algorithms to scale.

- [20140279051](https://patents.justia.com/patent/20140279051)
---

## 1. Problem Definition

Digital media providers have a finite number of ad impression opportunities ("slots"). Advertisers purchase these slots by specifying targeting criteria (e.g., demographics, context, geography, time). Each purchase (ad buy) is essentially a set of eligible slots.

Challenges:

* **Overlapping sets:** Multiple buys compete for the same slots.
* **Different values:** Buys pay different CPMs; higher CPM buys should win first.
* **Caps/quotas:** Some buys only want a fixed number of impressions.
* **Queries/what-ifs:** Media providers must simulate how new buys would fit into current inventory.

---

## 2. Representing Impressions as Slots

### Slot Model

Each impression opportunity is bucketed into a **slot** defined by attributes:

* Example attributes: site, placement, time bucket, device, gender, age range, hair color, weight range, geo.

```c
struct Slot {
  uint32_t forecast;   // forecasted impressions in window
  uint32_t remaining;  // decremented during allocation
  uint64_t attr_key;   // packed attribute codes (or side table reference)
};
```

A dense slot\_id (`0..U-1`) indexes slots.

---

## 3. Inverted Index

### Terms

A **term** = one attribute/value or bin, e.g.:

* `gender=F`, `age=35-39`, `site=example.com`, `time=2025-08-25T12`.

### Postings

For each term, maintain the set of slots that contain that term. Store as:

* **Roaring bitmap** (default): compressed, fast set operations.
* **List\<uint32\_t>** for sparse terms.
* **Bitset** for very dense terms.

### Memory Considerations

* Forward slot table: `U * ~16–32 bytes`.
* Inverted index: depends on term density. Hybrid (list/bitmap/bitset) keeps usage reasonable.
* Buys: store as predicate trees + iterators; do not materialize full slot sets.

---

## 4. Compiling Ad Buys

Each buy’s predicate is compiled into Boolean operations over postings:

* Example: **Women 35–40 blond weight≥200** →

  ```
  match = postings[gender=F] ∧ postings[age=35-39] ∧ postings[hair=blond] ∧ postings[weight≥200]
  ```
* Matching is done via bitmap intersections/unions.

Buys are represented as:

```c
struct Buy {
  uint32_t id;
  float cpm;
  uint32_t goal_total;   // impressions cap
  uint32_t remaining;
  Predicate pred;        // compiled predicate
  BitmapIterator iter;   // lazy iterator over slots
};
```

---

## 5. Allocation Algorithm

### Overview

The system assigns slots to buys using a **link → award → re-link** process:

1. **Link:** Tentatively connect each buy to the first eligible slot it can fill.
2. **Award:** Traverse slots; award to the highest-value linked buy.
3. **Re-link:** Losing buys advance to their next eligible slots.

This continues until all slots are processed or buys are capped.

### Two-Pass Allocation

* **Pass 1:** Block out slots in a query’s footprint. Allocate remaining slots to current buys. This shows true unsold inventory.
* **Pass 2:** Open only the query’s footprint, add the query as a buy, and re-run allocation. This shows what the query would win.

### Ordering Buys

Sort buys before allocation to minimize conflicts:

* Score = `CPM × (exclusivity/supply)^α`, where exclusivity measures how rare a buy’s slots are.
* High-CPM, rare buys processed first; broad cheap buys last.

---

## 6. Pseudocode

### Linking

```c
bool try_link(Buy& b, BitmapIterator& it, BlockMask& blocked) {
  while (it.has_next()) {
    SlotID s = it.next();
    if (blocked.test(s)) continue;
    if (DAT[s].linked == -1 || b.cpm > buys[DAT[s].linked].cpm) {
      int bumped = DAT[s].linked;
      DAT[s].linked = b.id;
      if (bumped != -1)
        try_link(buys[bumped], get_iterator(buys[bumped]), blocked);
      return true;
    }
  }
  return false;
}
```

### Awarding

```c
void assign_all(vector<Buy>& buys, BlockMask& blocked) {
  for (Buy& b : buys) try_link(b, get_iterator(b), blocked);
  for (SlotID s = 0; s < U; ++s) {
    if (blocked.test(s)) continue;
    int lb = DAT[s].linked;
    if (lb == -1) continue;
    Buy& w = buys[lb];
    uint32_t take = min(remaining[s], w.remaining);
    remaining[s] -= take;
    w.remaining -= take;
    DAT[s].awarded = w.id;
    DAT[s].linked = -1;
    if (w.remaining > 0) try_link(w, get_iterator(w), blocked);
  }
}
```

---

## 7. Complexity

* **Matching:** Bitmap ops ≈ O(posting size). Efficient with Roaring.
* **Allocation:** Each buy iterator advances forward only. Complexity ≈ O(total conflicts).
* **Memory:** Scales with slot universe + postings, not buys × slots.

---

## 8. Practical Notes

* **Sharding:** Partition by site/placement/time to cut problem size.
* **Pacing:** Use token buckets per time slice to avoid caps filling too early.
* **Unsold inventory:** Directly observable from slots with `remaining > 0` after allocation.
* **What-ifs:** Queries are buys; run two-pass allocation to see their effect.

---

## 9. Summary

A scalable ad inventory system requires:

1. **Slot binning**: represent impressions as discrete bins.
2. **Inverted index**: map attributes to slots for efficient buy matching.
3. **Lazy iterators**: avoid materializing full sets for millions of buys.
4. **Link → award → re-link** allocation: ensures high-value, capped buys get priority.
5. **Two-pass planning**: reveals both true unsold inventory and query outcomes.

This approach handles millions of overlapping buys efficiently in both memory and compute.
