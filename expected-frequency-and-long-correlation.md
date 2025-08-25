# Expected Frequency and Long Correlation

An AI based description of an algorithm which enabled me to create great recommendations and better search using spare sessions.  I've found it to work on many different datasets and have been continuously developing the approach.

## Overview

This document describes an approach for improving ranking and relevance in search and recommendation systems by analyzing entire user histories, not just short sessions. The method builds on co-occurrence analysis, but corrects for global popularity bias through an expected-vs-actual frequency adjustment. Extensions using vector similarity allow clustering of queries and URLs to further enhance relevance and diversification.

---

## Problem

Traditional correlation techniques that use full user histories often overweight globally popular items, leading to poor relevance. Short-session correlations avoid this but waste much of the long-term behavioral data available in user histories. The goal is to use **all user data** while still highlighting meaningful, unexpected associations.

---

## Core Algorithm

### Step 1: Local Co-Occurrence

* For each item $A$, compute co-occurrence frequencies with all other items $B$.
* Track both:

    * **User-level frequency:** number of users who interacted with both $A$ and $B$.
    * **Hit-level frequency:** number of total interactions linking $A$ and $B$.
* Compute a **local weight** as a descending linear combination of these two.
* Keep the **top N (≈500)** candidates per item.

### Step 2: Expected Global Frequency

* Gather the **global frequency distribution** of items from the candidate set.
* Sort these global frequencies in descending order.
* Assign each candidate from Step 1 an **expected frequency** based on this distribution.

### Step 3: Unexpectedness Weight

* For each candidate pair ($A, B$):

    * Compute the ratio:

      $$
      w_{unexpected}(A,B) = \frac{expected(B)}{global(B)}
      $$
    * If the ratio ≤ 1, the pair is not very meaningful.
* Combine this ratio with the local weight from Step 1 to produce a **final adjusted weight**.

### Result

Pairs that are **unexpectedly frequent** (compared to global popularity) are surfaced with higher weights, leading to more relevant and less popularity-skewed associations.

---

## Extension: Query/URL Vector Similarity

To further improve results:

1. Represent each query as a vector of weighted URLs:
   $Q_1 = \{(U_1, w_1), (U_2, w_2), ...\}$

2. Define similarity between two queries $Q_1, Q_2$ as:

   $$
   sim(Q_1, Q_2) = \frac{\sum (w_{Q1}(U) \cdot w_{Q2}(U))}{\sqrt{\sum w_{Q1}(U)^2} \cdot \sqrt{\sum w_{Q2}(U)^2}}
   $$

   This is a cosine-like similarity score.

3. Apply the same approach in reverse for URLs sharing queries.

4. Use **maximal matching** to merge near-duplicate queries (e.g., “JFK” vs. “John F Kennedy”) into unified clusters.

---

## Benefits

* **Leverages full histories**: Uses all user data, not just sessions.
* **Corrects popularity bias**: Adjusts for expected frequency so popular items don’t dominate.
* **Improves recall**: Surfaces long-tail but meaningful associations.
* **Enhances diversification**: Clustering reduces duplication and broadens coverage.
* **Compatible with ML models**: Outputs can be fed into RNNs/ANNs or modern ranking pipelines.

---

## Applications

* Search engine ranking (click data, session data).
* Recommendation systems (movies, products, music, etc.).
* Query expansion and clustering.
* User personalization.

---

## Next Steps

* Prototype with publicly available datasets (e.g., MovieLens, Netflix Prize).
* Evaluate relevance via offline metrics (precision\@k, NDCG) and user studies.
* Integrate into search ranking pipelines as a feature alongside neural models.

---

## Summary

This method extracts signal from entire user histories by combining local co-occurrence, expected global frequency adjustment, and vector-based clustering. The result is more relevant, diverse, and personalized search and recommendation outputs.
