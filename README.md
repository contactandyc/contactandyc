<div align="center">
  <h1 align="center">Andy Curtis Github Profile</h1>
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

These libraries form a cohesive ecosystem of modern C components—covering memory management, parsing, networking, embeddings, and file I/O—designed strictly for **performance, simplicity, and composability**.

<table>
  <tr>
    <td width="50%" valign="top">
      <h4>🧠 Memory & Foundations</h4>
      <ul>
        <li><code><a href="https://github.com/contactandyc/a-memory-library">a-memory-library</a></code> Core building blocks (alloc, pool, buffer) with zero-overhead release builds.</li>
        <li><code><a href="https://github.com/contactandyc/the-macro-library">the-macro-library</a></code> Type-safe containers and algorithms (sort, bsearch, map, heap).</li>
      </ul>
      <h4>🕸️ Networking & Async</h4>
      <ul>
        <li><code><a href="https://github.com/contactandyc/a-curl-library">a-curl-library</a></code> High-level libcurl abstraction with an event loop, retry semantics, and rate limiting.</li>
        <li><code><a href="https://github.com/contactandyc/a-curl-openai-plugin">a-curl-openai-plugin</a></code> Builder-style API for OpenAI endpoints with streaming SSE.</li>
        <li><code><a href="https://github.com/contactandyc/restinio-c">restinio-c</a></code> C wrapper over C++ Restinio HTTP framework.</li>
      </ul>
    </td>
    <td width="50%" valign="top">
      <h4>🗂️ Parsing & Serialization</h4>
      <ul>
        <li><code><a href="https://github.com/contactandyc/a-json-library">a-json-library</a></code> Fast JSON parser integrated with memory pools.</li>
        <li><code><a href="https://github.com/contactandyc/sql-parser-library">sql-parser-library</a></code> Lightweight SQL expression parser and evaluator.</li>
        <li><code><a href="https://github.com/contactandyc/a-json-schema-builder-library">a-json-schema-builder</a></code> Programmatic JSON Schema builder in C.</li>
      </ul>
      <h4>🔎 NLP, Search & I/O</h4>
      <ul>
        <li><code><a href="https://github.com/contactandyc/embedding-library">embedding-library</a></code> SIMD-accelerated primitives for vector embeddings.</li>
        <li><code><a href="https://github.com/contactandyc/search-index-library">search-index-library</a></code> Compact inverted index engine.</li>
        <li><code><a href="https://github.com/contactandyc/the-io-library">the-io-library</a></code> Record-oriented file processing (map-reduce style).</li>
        <li><code><a href="https://github.com/contactandyc/a-sentence-chunker-library">a-sentence-chunker-library</a></code> Fast UTF-8/ASCII sentence segmentation.</li>
      </ul>
    </td>
  </tr>
</table>

<br />

---

### ✦ Featured Repositories

<div align="center">
  <a href="https://github.com/contactandyc/another-c-library">
    <img src="https://github-readme-stats.vercel.app/api/pin/?username=contactandyc&repo=another-c-library&theme=transparent&hide_border=true&title_color=0057c0&text_color=414755&icon_color=0057c0" alt="another-c-library" />
  </a>
  <a href="https://github.com/contactandyc/the-macro-library">
    <img src="https://github-readme-stats.vercel.app/api/pin/?username=contactandyc&repo=the-macro-library&theme=transparent&hide_border=true&title_color=0057c0&text_color=414755&icon_color=0057c0" alt="the-macro-library" />
  </a>
</div>
<div align="center">
  <a href="https://github.com/contactandyc/scaffold-repo">
    <img src="https://github-readme-stats.vercel.app/api/pin/?username=contactandyc&repo=scaffold-repo&theme=transparent&hide_border=true&title_color=0057c0&text_color=414755&icon_color=0057c0" alt="scaffold-repo" />
  </a>
  <a href="https://github.com/contactandyc/a-json-library">
    <img src="https://github-readme-stats.vercel.app/api/pin/?username=contactandyc&repo=a-json-library&theme=transparent&hide_border=true&title_color=0057c0&text_color=414755&icon_color=0057c0" alt="a-json-library" />
  </a>
</div>

<br />

---

### ✦ Activity & Statistics

<div align="center">
  <img src="https://ghchart.rshah.org/0057c0/contactandyc" alt="contactandyc's Github chart" />
  <br />
  <br />
  <img src="https://github-readme-stats.vercel.app/api?username=contactandyc&show_icons=true&theme=transparent&hide_border=true&title_color=0057c0&text_color=414755&icon_color=0057c0&hide_rank=true" alt="GitHub Stats" />
</div>
