# Real‑Time Ad Inventory System with Direct‑Access Table (Inventory‑Only)

*AI‑generated technical overview of published application **20140279051**. Focus: **inventory computation** and **what‑if queries**, not ad serving.*

## Executive Essence

1. **Model future opportunities as a direct‑access table (DAT) of impression slots.**
   Each slot is a discrete attribute combination with an optional **weight** (forecasted count).

2. **Treat each ad (and each query) as an expression over attributes → an impression set.**
   Each has a comparable **value** (e.g., eCPM) and an optional **goal/cap**.

3. **Single‑scan assignment with pointers.**

  * **Link pointer** (tentative): at most one per ad at a time into one eligible slot.
  * **Award pointer** (final): the slot’s winner after resolving contentions by **value**.
  * After an award, the **winner** re‑links forward; any **loser** re‑links forward. Iterators only move forward.

4. **Two‑pass mode for what‑ifs and unsold inventory.**

  * **Pass‑1:** block the query’s footprint; allocate everything else → shows **true unsold** while preserving other buys.
  * **Pass‑2:** block non‑query slots; include the query as a buy → shows **what the query would win**.

Result: **Real‑time visibility** into value‑optimized winners and **unsold** inventory with O(1) per‑slot access and forward‑only re‑linking.

---

## 1) Entities & Definitions

* **Impression slot (slot):** bucketed opportunity defined by attributes (site, placement, time bucket, device, geo, age band, etc.).
  Fields: `forecast` (weight), `remaining`, compact `attr_key`.
* **Impression set (of an ad):** the set of `slot_id`s that satisfy the ad’s predicate.
* **Value:** comparable scalar used to break ties; may include pacing/priority multipliers.
* **Query:** an ad‑shaped request used for **Pass‑1 blocking** and **Pass‑2 competition**.

---

## 2) Direct‑Access Table (DAT)

O(1) per‑slot structure keyed by `slot_id`:

```c
struct DATEntry { int32_t linked_ad; int32_t awarded_ad; };
DATEntry DAT[U]; // U = number of slots
```

Scan slots in numeric order; keep each ad’s iterator **forward‑only**.

---

## 3) Building the Slot Universe

1. Choose binning for each attribute dimension (pre‑bin numeric ranges).
2. Enumerate expected/forecasted combinations for the planning window → dense `slot_id ∈ [0..U-1]`.
3. Initialize `forecast[slot_id]`, `remaining[slot_id]=forecast`, `attr_key`.

> Only materialize combinations you expect to see.

---

## 4) Optional: Inverted Index for Matching (kept simple)

An inverted index **accelerates** matching but is **not required** by the filing.

* **Term:** `attribute=value` (or a binned range).
* **Postings:** the set of `slot_id`s containing that term.

**Compiling a buy/query:** parse its predicate into Boolean ops over term postings (AND/OR/NOT) to produce a **lazy iterator over matching `slot_id`s**. Avoid materializing full per‑buy slot lists—advance the iterator on demand during linking.

> Storage for postings (sorted IDs or compressed lists) is implementation‑specific and out of scope; no bitmap/roaring detail is required.

---

## 5) Assignment: Link → Award → Re‑link

**Ad ordering.** Sort by **value** (optionally break ties by tighter supply or smaller goal); the filing requires only a comparable value.

**Linking (tentative).** Each ad keeps a forward iterator over eligible slots. It tries to place **one link**; if it bumps a lower‑value link, the bumped ad re‑links **forward**.

**Awarding (final).** Scan slots in ID order: award to the linked ad, transfer `take = min(slot.remaining, ad.remaining)`, clear link, then re‑link the winner forward if it still has `remaining > 0`.

**Illustrative pseudocode (minimal):**

```c
bool try_link(Ad &a, Iterator &it, const BlockMask &blocked) {
  while (it.has_next()) {
    uint32_t s = it.next();
    if (blocked.test(s)) continue;
    int32_t cur = DAT[s].linked_ad;
    if (cur == -1 || a.value > ads[cur].value) {
      DAT[s].linked_ad = a.id;
      if (cur != -1) try_link(ads[cur], iters[cur], blocked); // loser re-links forward
      return true;
    }
  }
  return false;
}

void assign_all(vector<Ad> &ads, const BlockMask &blocked) {
  for (Ad &a : ads) if (a.remaining > 0) try_link(a, iters[a.id], blocked);
  for (uint32_t s = 0; s < U; ++s) {
    if (blocked.test(s)) continue;
    int32_t lid = DAT[s].linked_ad;
    if (lid == -1) continue;
    Ad &w = ads[lid];
    if (w.remaining == 0 || remaining[s] == 0) { DAT[s].linked_ad = -1; continue; }
    uint32_t take = min(remaining[s], w.remaining);
    remaining[s] -= take; w.remaining -= take;
    DAT[s].awarded_ad = w.id; DAT[s].linked_ad = -1;
    if (w.remaining > 0) try_link(w, iters[w.id], blocked);
  }
}
```

**Behavioral guarantees:** higher value wins; losers move only forward; caps enforced via `remaining`; partial awards allowed.

---

## 6) Two‑Pass Flow (Queries + Unsold)

```c
Bitmap Q = match(query);                  // query footprint (via index or other matcher)

// Pass‑1: block query slots → allocate the rest
BlockMask pass1 = Q;
reset_DAT_and_remaining();
assign_all(active_ads, pass1);
collect_unsold(); // slots with remaining > 0

// Pass‑2: inside query only; include the query as a buy
BlockMask pass2 = UniverseMinus(Q);
reset_DAT_and_remaining();
vector<Ad> ads2 = active_ads; ads2.push_back(query_as_ad);
assign_all(ads2, pass2);
collect_query_results();
```

Outputs:

* **Unsold inventory** from Pass‑1.
* **What the query would win** from Pass‑2.

---

## 7) Worked Mini‑Example (compact)

* Slots `1..12`, unit weights.
* Ads: `AD1(v=10, {1,3,7,10})`, `AD2(v=15, {2,3,10,12})`, `AD3(v=30, {4,8,9,10})`.
* Query: `Q(v=20, {3,4,5,6})`.

**Pass‑1 (block 3–6):**
Awards → `1→AD1`, `2→AD2`, `7→AD1`, `8→AD3`, `9→AD3`, `10→AD3`, `12→AD2`; `11` unsold.

**Pass‑2 (only 3–6, include Q):**
`3→Q` (beats AD1), `4→AD3` (beats Q), `5→Q`, `6→Q`.

---

## 8) Engineering Notes (kept tight)

* **Sharding:** split by (site, placement, time bucket). Each shard owns its slots, index, and DAT.
* **Lazy sets only:** store predicates + iterators, not per‑buy slot lists.
* **Caps/pacing:** `remaining` enforces caps; pacing/priority can fold into **value** if needed.
* **Incremental updates:** add/remove/update a buy → re‑link it (and any bumped chains); no global recompute.
* **Complexity:** near‑linear in the number of visited edges thanks to O(1) slot access and forward‑only iterators.

---

## 9) What This Is *Not*

* Not a live **ad server**. This computes **inventory & what‑ifs**; outputs can be exported to serving systems.
* Not an LP/Max‑Flow solver; this emphasizes incremental, forward‑only assignment with constant‑time slot access.

---

## 10) Minimal Structures (inventory‑only)

```c
struct Slot     { uint32_t forecast, remaining; uint64_t attr_key; };
struct DATEntry { int32_t  linked_ad, awarded_ad; };
struct Ad {
  int32_t id; float value; uint32_t goal_total, remaining;
  Predicate pred; Iterator it; // iterator over eligible slot_ids
};
// Arrays: Slot slots[U]; DATEntry DAT[U]; uint32_t remaining[U];
```

> Keep hot fields (`remaining`, `linked_ad`, `awarded_ad`) in cache‑friendly arrays; 32‑bit IDs where feasible.

---

### Summary

This design represents inventory as a **DAT of slots**, assigns via **link → award → re‑link** (with forward‑only movement), and uses a **two‑pass** method to surface **unsold inventory** and **query outcomes**. An **inverted index** is an **optional** but practical way to generate lazy iterators over eligible slots—no specific storage format is required. The result is a fast, incremental inventory engine aligned with the application’s core teachings.
