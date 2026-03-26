<div align="center">
  <h1 align="center">Github Profile for Andy Curtis</h1>
  <p align="center">
    <i>Selective works, algorithmic research, and high-performance C infrastructure by Andy Curtis.</i>
  </p>

  <br />

  <img src="https://img.shields.io/badge/C-00599C?style=for-the-badge&logo=c&logoColor=white" alt="C" />
  <img src="https://img.shields.io/badge/C%2B%2B-00599C?style=for-the-badge&logo=c%2B%2B&logoColor=white" alt="C++" />
  <img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python" />
  <img src="https://img.shields.io/badge/Algorithms-141c25?style=for-the-badge" alt="Algorithms" />
</div>

<br />

---

### ✦ Archival Works & Algorithms

A selection of algorithms and systems I've developed over my career, largely in the public domain at this point.

* **Quicksort Improvement (2019)** — Designed an efficient modification to quicksort/introsort. By reusing the pivot sample to detect when input is already sorted, reverse-sorted, or all-equal, the algorithm finishes in *O(n)* with a single verification pass. Yields 5–30× speedups for sorted inputs with no regressions on adversarial cases.
* **Ad Inventory Overlapping Set Problem (2011–2012)** — Designed a real-time ad inventory model to handle millions of buys with overlapping targeting criteria using inverted indices and bitmap intersections. Enabled accurate forecasting and scalable allocation at web scale.
* **Expected Frequency & Long Correlation (2009+)** — An algorithm for search and recommendations leveraging entire user histories. By adjusting local co-occurrence with an expected vs. actual frequency correction, it systematically removes global popularity bias.
* **Click-Based Search & Recommendations (2003–2005)** — A framework leveraging full user sessions (queries + clicks) to learn correlations, improving ranking, personalization, and localization.
* **Efficient Near-Shingling (2001)** — An approach to near-duplicate detection that approximates shingling accuracy while reducing storage and compute costs. Documents are reduced to title + first + 10 longest sentence hashes, indexed in dual forward/inverted indices.
* **EzResult Search Engine (1998–1999)** — A distributed search engine written entirely in C/C++/assembly, independently utilizing inverted indices, tries, and cosine similarity. Supported instant index updates and was acquired in 1999.

<br />

---

### ✦ The C Infrastructure Ecosystem

A modular, multi-tier dependency graph of C libraries built for out-of-core data processing, search, and system reliability. Designed strictly for performance, simplicity, and composability.

<table width="100%" style="border: none;">
  <tr>
    <td width="50%" valign="top">
      <h4>🧱 Foundation & Memory</h4>
      <ul>
        <li><code><a href="https://github.com/contactandyc/a-memory-library">a-memory-library</a></code> Zero-overhead memory pools, auto-growing buffers, and debug-wrapped allocators.</li>
        <li><code><a href="https://github.com/contactandyc/the-macro-library">the-macro-library</a></code> Type-safe C macros for core algorithms (introsort, bsearch, red-black trees, heaps).</li>
        <li><code><a href="https://github.com/contactandyc/a-bitset-library">a-bitset-library</a></code> Expandable bitset structures for setting, querying, and bitwise operations.</li>
      </ul>
      <h4>⚙️ Distributed Processing & I/O</h4>
      <ul>
        <li><code><a href="https://github.com/contactandyc/a-map-reduce-library">a-map-reduce-library</a></code> Single-node, partitioned DAG execution engine for out-of-core data processing and pipelining.</li>
        <li><code><a href="https://github.com/contactandyc/the-io-library">the-io-library</a></code> Record-oriented file processing with transparent compression, partitioning, and sort-merging.</li>
        <li><code><a href="https://github.com/contactandyc/the-lz4-library">the-lz4-library</a></code> Fast LZ4 compression and decompression primitives.</li>
      </ul>
      <h4>🕸️ Networking & Security</h4>
      <ul>
        <li><code><a href="https://github.com/contactandyc/a-curl-library">a-curl-library</a></code> Async event-loop wrapper over libcurl with rate-limiting, backoffs, and dependency scheduling.</li>
        <li><code><a href="https://github.com/contactandyc/a-curl-openai-plugin">a-curl-openai-plugin</a></code> Builder API on the curl event loop for handling OpenAI streams and structured outputs.</li>
        <li><code><a href="https://github.com/contactandyc/an-encryption-library">an-encryption-library</a></code> Secure key generation and in-place AES-GCM encryption/decryption.</li>
      </ul>
    </td>
    <td width="50%" valign="top">
      <h4>🗂️ Parsing & Serialization</h4>
      <ul>
        <li><code><a href="https://github.com/contactandyc/a-json-library">a-json-library</a></code> Fast JSON parser integrated directly with memory pools.</li>
        <li><code><a href="https://github.com/contactandyc/a-json-sax-library">a-json-sax-library</a></code> High-speed, destructive/in-place SAX parser for JSON.</li>
        <li><code><a href="https://github.com/contactandyc/sql-parser-library">sql-parser-library</a></code> Lightweight SQL expression tokenizer, parser, and AST evaluator.</li>
        <li><code><a href="https://github.com/contactandyc/a-json-schema-builder-library">a-json-schema-builder</a></code> Programmatic C helpers for dynamically building JSON schemas.</li>
      </ul>
      <h4>🔎 NLP, Search & ML</h4>
      <ul>
        <li><code><a href="https://github.com/contactandyc/search-index-library">search-index-library</a></code> Compact inverted index engine with BM25 helpers and snippet generation.</li>
        <li><code><a href="https://github.com/contactandyc/embedding-library">embedding-library</a></code> SIMD-accelerated primitives for vector embeddings (dot product, cosine) and int8 quantization.</li>
        <li><code><a href="https://github.com/contactandyc/a-tokenizer-library">a-tokenizer-library</a></code> Tokenization and cursor logic for parsing queries and expressions.</li>
        <li><code><a href="https://github.com/contactandyc/stemmer-library">stemmer-library</a></code> Word stemming with an integrated caching mechanism.</li>
        <li><code><a href="https://github.com/contactandyc/a-sentence-chunker-library">a-sentence-chunker-library</a></code> Fast UTF-8/ASCII sentence segmentation with length constraint re-chunking.</li>
      </ul>
    </td>
  </tr>
</table>

<br />

---

### ✦ Fleet Management & Orchestration

Tools designed to tame the complexity of multi-repo, polyglot ecosystems through declarative configuration and GitOps automation.

* **[scaffold-repo](https://github.com/contactandyc/scaffold-repo)** — A declarative fleet manager and build orchestrator. Resolves dynamic dependency graphs, enforces OSS license compliance, and automates Git branching/releases across dozens of interconnected micro-repos via a single Python CLI.
* **[scaffold-templates](https://github.com/contactandyc/scaffold-templates)** — The centralized Template Registry powering `scaffold-repo`. Defines language stacks (C/CMake, Python), organizational profiles, and dynamic Jinja2 file routing to prevent vendor lock-in.

<br />

---

### ✦ Activity & Statistics

<div align="center">
  <img src="https://ghchart.rshah.org/0057c0/contactandyc" alt="contactandyc's Github chart" />
  <br />
  <br />
  <img src="https://github-readme-stats.vercel.app/api?username=contactandyc&show_icons=true&theme=transparent&hide_border=true&title_color=0057c0&text_color=414755&icon_color=0057c0&hide_rank=true" alt="GitHub Stats" />
</div>
```
