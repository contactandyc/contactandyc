# Efficient Near-Shingling with Longest-Sentence Hashing and Dual Indices

## Overview

This approach proposes a method for high-efficiency text similarity detection that approximates the accuracy of traditional shingling methods while significantly reducing computational and storage costs. It leverages **title + longest-sentence hashing**, **dual forward/inverted indices**, and **cosine weighting with precomputed norms**.

## Key Concepts

### 1. Title + Longest-Sentence Hashing

* Documents are segmented into sentences.
* The following are selected as anchors for hashing:

    * The **document title**
    * The **first sentence**
    * The **10 longest remaining sentences**
* Each selected sentence is hashed using a fast function such as **CRC32**.
* While CRC32 introduces a possibility of collisions, the probability is extremely low. For a collision to matter, **multiple hashes would need to overlap simultaneously**, making the risk negligible in practice even at large corpus scale.
* Benefits:

    * Captures both introductory and content-rich material.
    * Reduces redundant features.
    * Provides multiple robust anchors per document.

### 2. Dual Forward/Inverted Indices

* **Forward index**: Maps documents → their sentence hashes and weights.
* **Inverted index**: Maps hashes → the set of documents that contain them.
* Benefits:

    * Fast candidate retrieval.
    * Efficient filtering before deeper similarity calculations.

### 3. Cosine Weighting with Precomputed Norms

* Cosine similarity is computed only for candidate pairs from the inverted index.

Cosine formula:

$$
\cos(d_1, d_2) = \frac{\sum_{h \in H_{d1} \cap H_{d2}} w_{d1,h} \cdot w_{d2,h}}{\|d_1\| \cdot \|d_2\|}
$$

* **Intersection only**: Only shared hashes contribute (found via inverted index).
* **Precomputed norms**: For each document, the forward index stores the norm $\|d\| = \sqrt{\sum w^2}$, where weights correspond to sentence lengths or rarity-adjusted values.
* **Dot product efficiency**: Since most documents have a limited set of hashes (e.g., title, first sentence, and 10 longest), cosine similarity reduces to a handful of multiplications.

---

## Without Inverted Indices

If only a forward index is used:

* To compute similarity, each new document must be compared against **all other documents**.
* This requires intersecting every document’s set of hashes with every other document’s set.
* Complexity is **O(N²)** for N documents.
* Storage is simpler but runtime cost is prohibitive for large-scale datasets.

By contrast, the inverted index makes similarity search **sublinear** by narrowing comparisons only to documents sharing hashes.

---

## Distributed Partitioning of Inverted Indices

* Inverted indices can be **sharded by document** (e.g., split corpus into n partitions).
* Each machine holds a portion of the forward + inverted indices.
* When comparing, each partition independently computes cosine similarities using its subset.
* This means each machine only compares **1/nth of the dataset**, enabling near-linear scalability with the number of machines.
* Aggregation step collects top candidates from each partition.

---

## Optimizations

* **Precomputed norms**: Store the square root of the sum of squared weights (where weight = sentence length or adjusted IDF-like weight). This removes denominator recomputation during cosine calculation.
* **Rare-sentence weighting**: Downweight boilerplate or generic sentences (e.g., disclaimers).
* **Anchor diversity**: Using the title, first sentence, and longest sentences balances topical, introductory, and semantically dense signals.

---

## Workflow

1. **Preprocessing**

    * Extract title, first sentence, and 10 longest sentences.
    * Compute CRC32 hashes.
    * Store hashes + weights in forward and inverted indices.
    * Precompute document norm (√(Σ weights²)).

2. **Candidate Generation**

    * Query document → retrieve candidates from inverted index.

3. **Cosine Similarity**

    * Compute dot product over shared hashes.
    * Normalize using pre-stored norms.

4. **Ranking & Filtering**

    * Rank candidates by cosine score.
    * Apply thresholds as needed.

---

## Advantages Over Shingling

* **Efficiency**: 12 hashes per document instead of thousands of shingles.
* **Scalability**: Dual indices + distributed partitioning → sublinear search.
* **Accuracy**: Anchors approximate shingle coverage, enhanced by weighting.
* **Storage Reduction**: Compact indices vs. massive shingle tables.

---

## Applications

* Large-scale plagiarism detection.
* Near-duplicate filtering in search engines.
* Deduplication in content management.
* Real-time similarity detection in chat/moderation systems.

---

**Conclusion**: By combining title + longest-sentence CRC32 hashing, dual indices, precomputed norms, and distributed partitioning, this method achieves **near-shingling accuracy** with **far greater efficiency and scalability**, while keeping hash collision risks negligible.
