## My open source libraries

These libraries form a cohesive ecosystem of modern C components — covering memory management, parsing, networking, embeddings, indexing, and file I/O — all designed with **performance, simplicity, and composability** in mind.

You can explore them here:

- [**the-macro-library**](https://github.com/contactandyc/the-macro-library): Header-only, code-generation macros for fast, type-safe containers and algorithms (sort, bsearch, map, heap, test harness, timing, utilities).
- [**a-memory-library**](https://github.com/contactandyc/a-memory-library): Core memory building blocks (`aml_alloc`, `aml_pool`, `aml_buffer`) with zero-overhead release builds and rich debug diagnostics.
- [**a-json-library**](https://github.com/contactandyc/a-json-library): High-performance JSON parser/encoder integrated with memory pools for efficient, low-overhead handling.
- [**a-curl-library**](https://github.com/contactandyc/a-curl-library): Event-loop abstraction over libcurl with retry/backoff, rate-limiting, plugins, outputs, and a worker pool.
- [**a-curl-openai-plugin**](https://github.com/contactandyc/a-curl-openai-plugin): Builder-style API for OpenAI endpoints on top of `a-curl-library`, with streaming SSE, structured outputs, and chaining.
- [**a-json-schema-builder-library**](https://github.com/contactandyc/a-json-schema-builder-library): Programmatic JSON Schema construction helpers built on top of `a-json-library`.
- [**a-sentence-chunker-library**](https://github.com/contactandyc/a-sentence-chunker-library): Fast UTF-8/ASCII sentence segmentation with optional re-chunking to enforce length constraints.
- [**embedding-library**](https://github.com/contactandyc/embedding-library): SIMD-accelerated primitives for vector embeddings (dot product, cosine, quantisation) plus efficient int8 embedding tables.
- [**restinio-c**](https://github.com/contactandyc/restinio-c): C wrapper for the Restinio C++ HTTP framework, providing lightweight HTTP/2 servers with callback handlers and Swagger UI integration.
- [**search-index-library**](https://github.com/contactandyc/search-index-library): Compact inverted index engine with BM25 helpers, snippet builders, embedding integration, and AML-friendly memory model.
- [**sql-parser-library**](https://github.com/contactandyc/sql-parser-library): Lightweight SQL expression tokenizer, parser, and evaluator with extensible function specifications and type inference.
- [**the-io-library**](https://github.com/contactandyc/the-io-library): Record-oriented file I/O framework with reducers, partitioners, comparators, and utilities for scalable data pipelines.
- [**scaffold-repo**](https://github.com/contactandyc/scaffold-repo): A batteries-included tool to scaffold opinionated C/C++ repositories, derive CMake builds, manage tests/apps/docs, and enforce OSS hygiene (SPDX, NOTICE, license texts).

### **the-macro-library**

* Header-only, code-generation macros for core algorithms:

    * **macro\_sort**: type-safe introsort with “already sorted/reversed” fast paths.
    * **macro\_bsearch**: binary search variants (lb/ub/first/last).
    * **macro\_map**: intrusive red–black trees.
    * **macro\_heap**: min/max heap ops.
* Extras: unit test harness, timing utils, string→number conversions, bit ops.
* Comparator styles unified across all containers/algos.

---

### **a-memory-library**

* Three low-level building blocks:

    * **aml\_alloc**: malloc wrappers with debug-time leak/double-free detection.
    * **aml\_pool**: zero-overhead arena allocator with string helpers, base64, tokenization.
    * **aml\_buffer**: auto-growing, always NUL-terminated byte/string buffer.
* Release builds are zero-overhead; debug builds add rich diagnostics.
* Tight integration with higher-level libs (JSON, chunker, tokenizer).

---

### **a-json-library**

* Fast JSON parser in C with **memory-pool integration** (via AML).
* Rich conversion helpers (to int, float, bool, string, etc.).
* Flexible object/array scanning, path retrieval, encoding/decoding.
* Error handling (`ajson_is_error`, `ajson_dump_error`).
* Strong integration with **a-memory-library** pools for efficient lifetime mgmt.

---

### **a-curl-library**

* High-level **libcurl abstraction** with an **event loop** built on multi-handle.
* Rich **retry & backoff semantics** (function return codes dictate retry behavior).
* Fine-grained **rate limiting** per logical key (concurrency, RPS, backoff on 429).
* Supports **dependencies between requests** (auth token fetch → dependent calls).
* Integrated **worker pool** for off-loop CPU work.
* Pluggable **output interfaces** (memory, file, embeddings, Pub/Sub, custom).
* Modular **plugins** for APIs (OpenAI, Google Cloud services, GCS, SQL, Spanner).
* Clean C11 API with metrics (completed, failed, retried).

---

### **a-curl-openai-plugin**

* Built directly on **a-curl-library** event loop.
* **Builder-style API** for `/v1/responses`, `/v1/chat/completions`, `/v1/embeddings`.
* **Streaming via SSE** with simple callbacks (text deltas, usage, errors).
* **Structured outputs**: validated JSON Schema with strict mode.
* **Chaining** support (`previous_response_id`).
* Handles **typed message parts** (text, image URLs, file IDs).
* Parsing helpers for usage, errors, incomplete responses.
* C23 / CMake-based, clean separation of sinks (aggregated vs streaming).

---

### **a-json-schema-builder-library**

* Programmatic **JSON Schema builder** in C.
* Supports 2020-12 keywords (`$defs`, `$anchor`, `$dynamic*`).
* Small API for building primitives (object, array, string, number).
* Helpers for schema composition: `anyOf`, `oneOf`, `allOf`, required fields.
* Integrates with **a-json-library** + AML pools.

---

### **a-sentence-chunker-library**

* Fast UTF-8/ASCII **sentence segmentation**.
* Two-phase design: heuristic boundary detection, then optional **re-chunking** to enforce min/max lengths.
* Zero-copy offsets (`start_offset`, `length`) into original text.
* Designed to pair with AML buffers for memory-efficient pipelines.

---

### **embedding-library**

* **SIMD-accelerated** primitives for embeddings:

    * Float32: dot product, cosine similarity.
    * Int16/Int8: quantisation and similarity.
* Auto-dispatch to AVX-512 / AVX2 / NEON.
* **Embedding tables**: efficient storage and retrieval of 512-dim int8 vectors.
* Header-only, portable, high-performance design.

---

### **restinio-c**

* C wrapper over C++ **Restinio** HTTP framework.
* Builds HTTP/HTTP2 servers with thread pooling, keep-alive.
* **Callback-based request handling** with simple C API.
* Built-in **Swagger UI integration** for API docs.
* Resource mgmt with user-defined destroy functions.

---

### **search-index-library**

* Compact inverted index engine in C:

    * Term postings with values, positions, BM25 helpers.
    * Document images & search images (memory-mapped).
* **Hybrid search ready**: per-doc int8 embeddings attached.
* Bit-packed postings with adaptive encoding.
* Includes **snippet utilities** for text windows.
* Strong integration with AML + tokenizer library.

---

### **sql-parser-library**

* Lightweight SQL expression parser/evaluator.
* Builds tokens → AST → evaluable nodes (`sql_node_t`).
* Extensible **function spec system** for arithmetic, string, date/time.
* Type inference + implicit conversions.
* Pool-based memory mgmt via AML.
* Collects warnings/errors during parse/eval.

---

### **the-io-library**

* General-purpose **I/O and file processing framework** in C.
* Supports multiple record layouts: **delimited**, **CSV**, **fixed-size**, and **length-prefixed**.
* Provides **record abstractions** (`io_record_t`) with callbacks for **reducers**, **comparators**, and **partitioners** (map-reduce style).
* File utilities: listing, filtering, partitioning, sorting by metadata, reading/writing, path validation, directory creation.
* **Partitioning helpers** for splitting/sorting by numeric keys (32/64-bit).
* Designed for **scalable data pipelines**, integrates with **a-memory-library** and **the-macro-library** for efficient allocation and sorting.

---

### **scaffold-repo**

* A CLI tool to **stamp opinionated templates into one or many repos** and keep them compliant.
* Scaffolds **CMake projects, apps, tests, docs, Dockerfiles**, and handles **multi-project/monorepo layouts**.
* Provides **OSS hygiene** automation — SPDX headers, NOTICE rebuilds, license verification.
* Can **clone, build, and install external git libraries** in dependency order with safe step sanitization.
* Generates build scripts (`build.sh`) with install, coverage, and clean support.
* Idempotent and configurable — uses Jinja2 templates and a YAML model, with overrides for templates and licensing.

---

**Summary**:
My goal with these repos was to form a **tight ecosystem of C libraries** for building high-performance, low-dependency infrastructure:

* **Memory management & buffers**: `a-memory-library`
* **Parsing & serialization**: `a-json-library`, `a-json-schema-builder-library`, `sql-parser-library`
* **NLP primitives**: `a-sentence-chunker-library`, `embedding-library`
* **Search & indexing**: `search-index-library`
* **Networking & async APIs**: `a-curl-library`, `a-curl-openai-plugin`, `restinio-c`
* **File/record I/O**: `the-io-library` (record-oriented file processing, map-reduce style)
* **Foundational algorithms**: `the-macro-library`
* **Repo management**: `scaffold-repo`
