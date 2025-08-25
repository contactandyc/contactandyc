# Session‑Based Correlation for Search & Discovery

The main idea presented below was that everything in a session is related.  

Based upon the following patents
* [7181447](https://patents.justia.com/patent/7181447) - Methods and systems for conceptually organizing and presenting information
* [7152061](https://patents.justia.com/patent/7152061), [7451131](https://patents.justia.com/patent/7451131), [7739274](https://patents.justia.com/patent/7739274), [7984048](https://patents.justia.com/patent/7984048), [8037087](https://patents.justia.com/patent/8037087), [8065299](https://patents.justia.com/patent/8065299) - Methods and systems for providing a response to a query

## Table of Contents

1. Foundations & Data Model
2. Core Correlation Primitives
   2.1 Q2P (Query→Pick)
   2.2 P2Q (Pick→Query)
   2.3 Q2Q (Query→Query)
   2.4 P2P (Pick→Pick)
3. Scoring, Signals & Penalizations
4. Concatenated Correlations (e.g., QPQ) as Graph/Matrix Ops
5. Query Improvements
   5.1 “Soft” Spelling Correction
   5.2 Equivalent Phrasing (synonyms / same intent)
   5.3 Redundancy Trimming (drop non‑helpful words)
   5.4 Query Suggestions (refinements vs. related)
   5.5 Breadth Control via Q2RP/Q2P Mixing
6. Result Improvements
   6.1 “Similar Results” via P2P
   6.2 “Suggest Queries for this Result” via P2Q
   6.3 Product Mix (images, news, web, …)
7. User Modeling & Personalization
   7.1 User‑to‑User Affinity (U2U)
   7.2 Personalized Re‑ranking
8. Geo‑Localization
   8.1 URL Centroid & Sphere of Influence
   8.2 Jurisdiction‑Based Result Lists (caching by region)
9. System Architecture (Batch + Online)
10. Worked Toy Example (step‑by‑step)
11. Practical Notes (privacy, spam, cold start, evaluation)
12. Minimal Schemas & Pseudocode Index

---

## 1) Foundations & Data Model

**Goal:** Learn from *entire search sessions*—not just one click—to improve ranking, suggestions, spelling, personalization, and localization.

**Key terms**

* **Query (Q):** user’s typed search string.
* **Pick (P):** any user selection (URL, image, video, news result, etc.).
* **Session:** a sequence of search actions from one user within a time window (e.g., 30 minutes of inactivity ends a session).
* **Independent users:** distinct users/devices; we want correlations that appear across different users (reduces noise).

**Minimal event log schema (append‑only):**

```
Event {
  timestamp
  user_id
  session_id
  action_type: {query, click}
  query_text?    // when action_type = query
  result_url?    // when action_type = click
  result_rank?   // 1-based, if click came from SERP
  dwell_ms?      // time until next action (proxy for satisfaction)
  page_type?     // web, image, video, news, etc.
  user_geo?      // (lat, lon) or coarse region (city, state, country)
}
```

> **Principle:** Don’t roll up early. Keep raw logs so you can recompute scores with new heuristics without data loss.

---

## 2) Core Correlation Primitives

These four primitives describe co‑occurrences within sessions.

### 2.1 Q2P (Query→Pick)

**What:** Associate a query with *all* picks that happen in the same session (not only immediate clicks).
**Why:** Users often explore; valuable results may be clicked after reformulations.
**Output:** A weighted association count `score(Q, P)`.

**Basic pseudocode**

```pseudo
function build_Q2P(events):
  // group by session
  sessions = group_by_session(events)

  Q2P = Counter()  // (Q,P) -> weighted count
  for s in sessions:
    queries = [e.query_text for e in s if e.action_type == 'query']
    clicks  = [e for e in s if e.action_type == 'click']

    for q in queries:
      for c in clicks:
        w = edge_weight(q, c, s)  // see Section 3
        Q2P[(q, c.result_url)] += w

  return Q2P
```

**Notes**

* Distinguish **Q2RP** (query→result→pick) when the clicked URL appeared on the SERP for that query; it’s often stronger evidence.
* Penalize repeated clicks from the same user/session; discount ultra‑short dwell.

---

### 2.2 P2Q (Pick→Query)

**What:** For each pick, collect all queries that appeared in the same session.
**Why:** “Which queries lead people to this page?” Useful for query suggestions attached to a URL.
**Output:** `score(P, Q)`.

**Pseudocode:** identical to Q2P with roles swapped.

---

### 2.3 Q2Q (Query→Query)

**What:** Co‑occurrence of queries within sessions (order aware or not).
**Why:** Reveals **refinements**, **equivalents**, and **corrections**.
**Output:** `score(Q1, Q2)`; optionally keep directionality and order.

**Pseudocode (order-aware)**

```pseudo
function build_Q2Q(events):
  sessions = group_by_session(events)
  Q2Q = Counter()  // (Q1,Q2) -> weighted count

  for s in sessions:
    queries = [e.query_text for e in s if e.action_type == 'query']
    for i in range(len(queries)):
      for j in range(len(queries)):
        if i == j: continue
        w = q2q_weight(queries[i], queries[j], i, j, s) // time/order sensitive
        Q2Q[(queries[i], queries[j])] += w

  return Q2Q
```

---

### 2.4 P2P (Pick→Pick)

**What:** Co‑occurrence of clicked results (URLs) within the same session.
**Why:** “Users who clicked this also clicked that” → **similar results**.
**Output:** `score(P1, P2)`; optionally directional and order‑aware.

---

## 3) Scoring, Signals & Penalizations

**Common signals**

* **Dwell time:** proxy for satisfaction; discount if < X seconds.
* **Rank bias:** click at rank 1 is easier than at rank 9; down‑weight by propensity.
* **Time delta:** the farther from the query, the weaker the linkage.
* **Uniqueness:** deduplicate repeats within a session; across users is what really matters.
* **Freshness:** decay older evidence to stay current.
* **Independence:** require evidence from ≥ 2 distinct users to promote an association.

**Example edge weight**

```pseudo
function edge_weight(q, click, session):
  base = 1.0

  // dwell penalty
  if click.dwell_ms < 1000: base *= 0.0  // treat as no evidence

  // rank propensity correction (e.g., inverse propensity)
  if click.result_rank is not None:
    base *= 1.0 / propensity(click.result_rank)  // propensity(1) > propensity(9)

  // time distance penalty (query vs click)
  dt = abs(click.timestamp - time_of(q, session))
  base *= exp(-dt / TAU)

  return base
```

---

## 4) Concatenated Correlations (e.g., QPQ) as Graph/Matrix Ops

**Idea:** Build a graph: `Q` nodes and `P` nodes with weighted edges from Q2P and P2Q. Then derive **indirect** associations by composing edges.

* **QPQ**: queries related to picks related to the original query
  Equivalent to sparse matrix multiply: `M_QQ ≈ M_QP × M_PQ`
* **PQP**: picks related to queries related to the original pick
  `M_PP ≈ M_PQ × M_QP`
* General patterns: QQQ, PPQ, QPP, etc.

**Why:** Indirect paths predict associations we haven’t directly seen yet (helps long tail).

**Pseudocode (sparse)**

```pseudo
// M_QP: map<Q, list<(P, w)>>
// M_PQ: map<P, list<(Q, w)>>
function infer_QPQ(M_QP, M_PQ, min_paths=2):
  M_QQ = Counter()

  for q in M_QP:
    paths = Counter()
    for (p, w_qp) in M_QP[q]:
      for (q2, w_pq) in M_PQ.get(p, []):
        if q2 == q: continue
        paths[q2] += w_qp * w_pq

    // require at least two distinct intermediate picks (independence proxy)
    for q2, score in paths.items():
      if count_distinct_intermediate_picks(q, q2) >= min_paths:
        M_QQ[(q, q2)] += score

  return M_QQ
```

**Normalization tricks**

* Use PMI/lift to favor *surprising* co‑occurrences: `PMI(Q,P) = log( P(Q,P) / (P(Q)P(P)) )`.
* Cap contributions per session/user to avoid heavy‑user bias.

---

## 5) Query Improvements

### 5.1 “Soft” Spelling Correction

**Problem:** Avoid forcing a correction when the misspelling may also be a valid intent (e.g., band names, product codes).
**Approach:** Use **Q2Q** evidence + **lexical distance**. If a suggestion is both conceptually related *and* textually similar (and more frequent), show it as a **soft** correction (present both).

**Algorithm (observed query)**

```pseudo
function soft_spell_suggest(q):
  candidates = neighbors_in_Q2Q(q)  // strong Q2Q links
  scored = []
  for c in candidates:
    ed = edit_distance(q, c)
    if ed <= 2:
      score = alpha * Q2Q_strength(q, c) +
              beta  * frequency(c) -
              gamma * ed
      scored.append((c, score))

  ranked = sort_desc(scored)

  // label which are "likely corrections"
  likely = [c for (c,score) in ranked if
             frequency(c) > frequency(q) and
             tends_to_follow(q, c) and
             produces_more_picks(c, q)]

  return ranked, likely
```

**Cold case (unseen query):**

* Identify suspect token (rare/unseen).
* Look at queries sharing the other tokens; collect common alternatives for the suspect token; score by co‑occurrence + edit distance.
* Fallback to n‑gram language model over known queries.

---

### 5.2 Equivalent Phrasing (same intent, different words)

**Signal:** High **Q2Q** strength both directions + **Q2P** overlap (their clicked results overlap).
**Metric:** `equiv(q1,q2) = min( sym_Q2Q_strength(q1,q2), jaccard(TopPicks(q1), TopPicks(q2)) )`

Use to merge histories, unify reporting, or suggest “Try also: myanmar history” when the user searched “burma history”.

---

### 5.3 Redundancy Trimming

**Idea:** Some tokens add noise (“ohio” in “columbus ohio blue jackets”). If dropping token T **increases** historical satisfaction (via Q2P/Q2RP), suggest/auto‑apply the simpler query.

**Heuristic**

```pseudo
function redundant_tokens(q):
  tokens = tokenize(q)
  redundant = []
  for t in tokens:
    q_simpler = remove_token(q, t)
    if score(Q2P_top(q_simpler)) > score(Q2P_top(q)) * (1 + epsilon):
      redundant.append(t)
  return redundant
```

---

### 5.4 Query Suggestions (refinements vs. related)

* **Refinements:** keep all original terms + add constraints (e.g., “running shoes → running shoes women size 8”). Often found via **Q2Q** with order where `q2` follows `q1` and `tokens(q1) ⊆ tokens(q2)`.
* **Related:** different but topically adjacent (via **QPQ** or **PQP**).

**Pseudocode**

```pseudo
function suggest_queries(q):
  direct = top_k(Q2Q_out[q])
  indirect = top_k(QPQ[q]) // inferred via concatenation
  refinements = [r for r in direct if is_refinement(q, r)]
  related     = dedup([r for r in direct + indirect if not is_refinement(q, r)])
  return refinements, related
```

---

### 5.5 Breadth Control via Q2RP/Q2P Mixing

**Narrow (depth):** emphasize immediate SERP clicks (**Q2RP**).
**Broad (breadth):** emphasize all session clicks (**Q2P**) excluding immediate SERP clicks.

Provide a UI knob or automatic blend:
`final(Q,P) = λ * score_Q2RP(Q,P) + (1-λ) * score_Q2P_nonSERP(Q,P)`

---

## 6) Result Improvements

### 6.1 “Similar Results” via P2P

**Core:** Co‑clicked pages in the same session indicate **similarity**.
**Normalize for popularity** (PMI/lift or cosine).

```pseudo
function similar_results(p):
  neighbors = P2P[p]             // co-click counts
  sims = []
  for p2, c in neighbors:
    sim = pmi_or_cosine(p, p2)   // normalize by popularity
    sims.append((p2, sim))
  return sort_desc(sims)
```

---

### 6.2 “Suggest Queries for this Result” via P2Q

Attach to each result page the top queries that historically led users there.

```pseudo
function queries_for_result(p):
  return top_k(P2Q[p])  // with normalization (lift)
```

---

### 6.3 Product Mix (images, news, web, …)

Treat different result types uniformly as picks. If an image URL dominates Q2P for a query, surface an **image block** early.
**Presentation rule:** rank by score, then group by product for clarity.

---

## 7) User Modeling & Personalization

### 7.1 User‑to‑User Affinity (U2U)

**Idea:** If user U shares a lot of distinctive queries/picks with user V, V’s preferences get more weight for U.

```pseudo
function build_affinity(all_users):
  // represent each user as a tf-idf vector over queries+picks
  V = {}
  for u in all_users:
    V[u] = tfidf_vector(events_of(u))

  A = {} // affinity matrix (sparse)
  for u in all_users:
    for v in neighbors_by_LSH(V[u]):   // or top sims
      A[(u,v)] = cosine(V[u], V[v])
  return A
```

**Distinctiveness:** weight rare queries/picks higher (idf).

---

### 7.2 Personalized Re‑ranking

At serve time:

```pseudo
function personalize(user, query, base_results):
  for r in base_results:
    // boost if similar users liked r for this query or neighbors
    r.score += k1 * sum_over_v( A[(user,v)] * picked(v, r) )
    r.score += k2 * sum_over_v( A[(user,v)] * Q2P_strength(query, r.url) )
  return rerank(base_results)
```

---

## 8) Geo‑Localization

### 8.1 URL Centroid & Sphere of Influence

**Goal:** Prefer **local** results when a URL shows localized appeal.

**Centroid:** location minimizing distance to the URL’s historical pick origins.
**Sphere of influence:** radius where interest stays high; inversely proportional to how localized the URL’s picks are.

```pseudo
function compute_local_profile(url):
  picks = [(lat,lon) for each click of url]
  if picks is sparse: return None

  centroid = geo_weighted_center(picks)  // e.g., geometric median
  spread   = robust_spread(picks)        // e.g., MAD or percentile radius
  radius   = f(spread)                    // smaller spread -> smaller radius

  return {centroid, radius}
```

**Serve‑time boost**

```pseudo
function geo_boost(user_location, url_profile):
  d = haversine(user_location, url_profile.centroid)
  return kernel(d, url_profile.radius)  // e.g., exp(-(d/r)^2)
```

### 8.2 Jurisdiction‑Based Result Lists (caching by region)

For common queries with clear regional splits:

* Build separate ranked lists per jurisdiction (e.g., country/state).
* At serve time, pick the list matching the user’s region or blend nearby lists.

---

## 9) System Architecture (Batch + Online)

**Batch/offline**

1. Ingest logs → sessionize.
2. Build Q2P, P2Q, Q2Q, P2P with scoring.
3. Compute concatenations (QPQ, PQP…) using sparse matrix multiplies.
4. Build user affinity matrix (sparse).
5. Compute geo profiles per URL.
6. Materialize:

    * Query→TopResults (narrow/broad, with product mix)
    * Query→Suggestions (refinements/related)
    * URL→SimilarResults
    * URL→TopQueries
    * Jurisdictional lists (if applicable)

**Online**

* For a query: retrieve base list + suggestions.
* Apply personalization (U2U), geo boosts.
* Blend products and breadth, de‑duplicate, and return.

---

## 10) Worked Toy Example (very small)

**Session logs (3 users):**

| time | user | action | value | extra             |
| ---- | ---- | ------ | ----- | ----------------- |
| t1   | U1   | query  | Q1    |                   |
| t2   | U1   | click  | P5    | rank=2, dwell=20s |
| t3   | U1   | query  | Q2    |                   |
| t4   | U1   | click  | P1    | rank=1, dwell=30s |
| t5   | U1   | click  | P3    | rank=3, dwell=15s |
| t6   | U2   | query  | Q2    |                   |
| t7   | U2   | click  | P4    | rank=1, dwell=25s |
| t8   | U2   | click  | P1    | rank=1, dwell=40s |
| t9   | U2   | query  | Q1    |                   |
| t10  | U2   | click  | P2    | rank=1, dwell=10s |
| t11  | U2   | click  | P3    | rank=2, dwell=18s |
| t12  | U3   | query  | Q3    |                   |
| t13  | U3   | click  | P3    | rank=1, dwell=45s |
| t14  | U3   | query  | Q2    |                   |
| t15  | U3   | click  | P1    | rank=2, dwell=20s |
| t16  | U3   | query  | Q3    |                   |
| t17  | U3   | click  | P5    | rank=1, dwell=35s |

**Takeaways**

* **Q2P(Q1):** {P5,P2,P3,P1} with weights; P1 is strong even though not an immediate click after Q1 (shows the value of full‑session association).
* **Q2Q:** Q1↔Q2 and Q2↔Q3 pairs appear; use this to suggest refinements/equivalents.
* **P2P:** (P1,P3) and (P3,P5) co‑clicked → “similar results” candidates.
* **QPQ for Q1:** multiply Q1→P\* with P\*→Q\* to find additional query suggestions not directly co‑issued in the same session.

---

## 11) Practical Notes

* **Privacy:** Anonymize user IDs, store coarse geos when possible, enforce retention limits, differential privacy noise for aggregates if required.
* **Spam/Robustness:**

    * Limit per‑user contributions; detect bots; require multi‑user evidence; dwell filters; rate limits per source.
* **Cold Start:**

    * Use text/content ranking as a base; rely more on concatenations and product‑type priors until behavioral evidence accrues.
* **Evaluation:**

    * Offline: counterfactual replay, NDCG\@K with propensity correction.
    * Online: A/B of CTR, long‑click rate, reformulation rate, satisfaction surveys.

---

## 12) Minimal Schemas & Pseudocode Index

**Data tables (materialized)**

```
Q2P[(q,p)] -> score
P2Q[(p,q)] -> score
Q2Q[(q1,q2)] -> score (optionally directional)
P2P[(p1,p2)] -> score (optionally directional)

Q_BREADTH[q, λ] -> [p... scores...]   // λ∈[0,1] mix of Q2RP/Q2P
Q_REFINEMENTS[q] -> [q'...]
Q_RELATED[q] -> [q'...]

P_SIMILAR[p] -> [p'...]
P_TOP_QUERIES[p] -> [q'...]

U2U[(u,v)] -> affinity
URL_GEO[p] -> {centroid, radius}
JURIS_LISTS[jurisdiction, q] -> ranked results
```

**Signal helpers**

```pseudo
propensity(rank):              // learned/external estimate
dwell_ok(dwell_ms): dwell_ms >= 1000
decay(age_days): exp(-age_days / HALF_LIFE)
pmi(a,b): log( P(a,b) / (P(a)*P(b)) )

// treat counts as probabilities with normalization
```

**Service pipeline**

```pseudo
function serve(user, query, location):
  base = retrieve_base_results(query)             // from Q_BREADTH + content
  base = blend_products(base)
  base = apply_geo_boosts(base, location, URL_GEO)
  base = personalize(user, query, base)           // U2U
  sugg_ref, sugg_rel = suggest_queries(query)     // Q2Q & QPQ
  return render(base, sugg_ref, sugg_rel)
```

---

## Cheatsheet: When to Use What

| Problem                          | Primary Signal(s)                         |
| -------------------------------- | ----------------------------------------- |
| Rank results for a query         | Q2P (+Q2RP), breadth control              |
| Suggest refinements              | Q2Q (ordered), token subset checks        |
| Suggest related searches         | Q2Q + QPQ (concatenations)                |
| Spelling correction (soft)       | Q2Q + edit distance + frequency           |
| Similar results to a URL         | P2P with normalization (PMI/cosine)       |
| “Users also searched for (page)” | P2Q                                       |
| Personalize for a user           | U2U affinity boosts                       |
| Localize results                 | URL centroid & sphere; jurisdiction lists |

---

### Closing Thought

The unifying theme is **consensus from behavior**: by capturing and correlating *whole sessions* across many independent users, you can infer intent, quality, and relationships that pure text signals miss. The primitives (Q2P, Q2Q, P2Q, P2P), their scoring, and their compositions give you a flexible toolkit for search ranking, suggestions, personalization, and localization.
