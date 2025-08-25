# Search Engine Architecture

An AI based description of the search engine I built in 1999 [patent - 6278992](https://patents.justia.com/patent/6278992).  The engine was written entirely in C/C++/Assembly with html/javascript for the frontend.  Every component including the service architecture, the distributed mechanisms, the distributed datastore, html parsers, query parsers, indexing approaches, relevance algorithms, etc were written by me.  When I wrote the search engine, I had no prior experience working with a software engineer (and didn't while writing the engine).  This is evident by how I named things.  I also barely attended any college.  This search engine was written starting in early 1998 and was sold in November 1999.  I met with Yahoo! in May, 1999 and as part of the acquisition effort, they asked me to start the search engine over again.  I restarted the search crawl on [May 26th, 1999 and stopped it on June 17th, 1999](https://web.archive.org/web/19991012071857/http://ezresult.com/stat.htm) after getting significant interest from many internet companies.  The engine grew at a rate of ~2.3 million pages per day, but was also refreshing the "core" pages which amounted at the time to about 500-700k pages (can't remember exactly).

---

## 1. Overview

This architecture describes a distributed Internet search engine system that utilizes a **trie-based indexing structure** combined with **duplicate segment lists** and **bitmap-compressed index nodes**. To optimize memory and retrieval speed, the system also supports a **hybrid model**, handing off indexing to a B-tree or other structure after a fixed character depth. The design focuses on scalability, efficiency, and support for rapid updates.

---

## 2. Core Components

### 2.1 Record File

* Stores raw documents (pages, metadata, etc.)
* Each record has a logical position (record ID) used as a retrieval reference

### 2.2 Index Arrays (Trie Segments)

* Each segment is an array of slots, where each slot corresponds to a character defined by a mask (e.g., digits `0-9`, alphabet `A-Z`)
* Slot states:

    * **Empty (0)** – unused
    * **Record address (+)** – points directly to a record
    * **Pointer (-)** – points to another index array at the next character level
    * **TC (terminating character)** – signals the end of a string, stores a reference value
    * **UNK (unknown)** – handles non-standard characters

### 2.3 Duplicate Segments

* A postings list structure that stores multiple record addresses for identical strings
* Implemented as a wide singly linked list with optional:

    * **Count of occurrences**
    * **Pointer to last node**
    * **Sorted flag** (enables efficient boolean operations)

### 2.4 Compressed Index Nodes

* Optional optimization for sparse levels of the trie
* Implemented with:

    * **Bitmap portion** (one bit per possible character)
    * **Pointer array** (stores pointers corresponding to set bits)
    * **Overflow blocks** if more pointers are needed
* Reduces memory footprint while preserving lookup efficiency

### 2.5 Hybrid Indexing

* **First portion of string** indexed via trie arrays
* **Remaining portion** indexed by an alternate structure (e.g., B-tree, hash)
* Benefits:

    * Tries optimize prefix and wildcard queries
    * B-trees reduce memory requirements for long suffixes

---

## 3. Distributed System Architecture

### 3.1 Servers

1. **URL Server**

    * Indexes and manages URL strings
    * Associates each URL with metadata (popularity, last updated, etc.)

2. **Get Page Server (Crawler)**

    * Retrieves pages from the web using URLs
    * Feeds content to processing pipelines

3. **Process Page Servers**

    * *Part 1:* Parse documents, extract links, metadata, attributes
    * *Part 2:* Tokenize text into words, generate word lists, assign relevance scores

4. **Word Index Server**

    * Uses trie to assign unique numeric IDs to words
    * Stores term-to-ID mappings for fast retrieval

5. **Split Server**

    * Shards incoming documents across **PAD triplets**
    * Handles load balancing and rebalancing

6. **PAD Triplets** (per shard)

    * **Page Server** – stores document-level metadata (title, description, size, etc.)
    * **Archive Server** – stores per-document word lists and relevances
    * **Duplicate Server** – stores per-word postings lists (documents containing that word)

### 3.2 Query Processing Flow

1. User enters query terms
2. Word Index Server resolves words → term IDs
3. Duplicate Server retrieves postings lists per term
4. Boolean operations combine postings
5. Results ranked (relevance, freshness, popularity)
6. Page Server retrieves metadata/snippets for display

---

## 4. Key Features

* **Prefix & wildcard support** via trie descent
* **Boolean set operations** optimized with sorted duplicate lists
* **Bitmap-compressed nodes** for memory-efficient indexing
* **Hybrid indexing** combining tries with B-trees/hashes
* **Distributed architecture** for scale (PAD triplets, split servers)
* **Support for rapid updates and deletions** (duplicates and archives handle versioning)

### 4.1 Relevance Model — The Normal English Document Algorithm

The engine employed a custom scoring model called the **Normal English Document (NED) algorithm**. It was designed to reward documents that reflected natural English usage and penalize keyword stuffing or unnatural repetition.

**Core principles:**

* **Anchor on rare terms:** The rarest query term received full weight; subsequent terms were scaled relative to it based on IDF ratios.
* **Normal distribution expectation:** Word frequencies were compared to expected distributions for well-written documents. Terms used too often (beyond the “normal band”) were penalized.
* **Positional salience:** First occurrences, especially in titles or early body text, carried higher weight than later repeats.
* **Spam resistance:** Documents with unnatural repetition lost credit, keeping scores aligned with user perception.

This approach produced results that were intuitive for users: rare and meaningful words mattered most, early mentions mattered more, and simple repetition could not inflate relevance.

---

## 4.2 Distributed Messaging System

Another innovation in the system was its **distributed architecture messaging layer**. Each server instance was given a unique name and communicated using a message-passing framework. Messages could be automatically re-routed to alternate servers if one was unavailable, ensuring fault tolerance. This allowed the search engine to operate autonomously in a distributed environment. In practice, the system could continue crawling, indexing, and answering queries while I was away—even once while I was on an airplane, the engine continued to grow without manual intervention.

---

## 5. Advantages

* Scales to billions of documents without excessive hardware requirements
* Efficient in both **insertions** (append records) and **queries** (trie descent)
* Flexible hybrid model adapts to varying string lengths
* Distributed design enables horizontal scaling
* Spam-resistant relevance ranking aligned with human expectations

---

## 6. Comparison to Modern Systems

* Similar in concept to Lucene/Elasticsearch (inverted index + postings lists)
* Trie-based dictionary structure resembles modern FST (finite state transducer) dictionaries
* Distributed PAD model parallels sharding/replication in Solr/Elasticsearch
* NED algorithm conceptually anticipates modern relevance models that integrate term rarity, position, and anti-spam signals

---

## 6.1 Historical UI

The front-end of the search engine, branded **EZResult**, was simple and powerful. Screenshots preserved in the Wayback Machine (1999) show features such as:

* A clean search box with options to match titles only or exact words.
* Support for advanced operators (`t:` for titles, `d:` for domains, `|` for multiple queries, `.` for exact matches).
* Instantaneous page addition/update forms so site owners could refresh their presence in seconds.
* Public statistics page tracking index growth from **0 pages (May 26, 1999)** to **69+ million pages (October 1999)**.
* Marketing claims of scalability to one trillion pages, parallel data processing, and support for special characters (`cd-rom`, `555-1212`, `$9.99`).

This transparent and user-centric interface complemented the innovative backend architecture, giving users both **control** and **confidence** in the search results.

---

## 7. Conclusion

This architecture anticipated many features of modern large-scale search systems. Its combination of **trie indexing, duplicate/postings management, bitmap compression, hybrid B-tree integration**, and a **modular distributed server model** made it a forward-looking approach to Internet-scale search. The addition of the **Normal English Document algorithm** provided a relevance model that was user-aligned and resistant to keyword spam—an innovation that placed the system’s results second only to Google at the time.
