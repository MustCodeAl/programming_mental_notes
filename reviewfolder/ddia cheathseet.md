# Chapter 1: Reliable, Scalable, and Maintainable Applications

- Data Systems
  - Dimensions to consider when thinking about data systems: access patterns, performance characteristics, implementations.
  - Modern data systems often blur the lines between databases, caches, streams, etc.
- Reliability
  - Systems should perform the expected function at a given level of performance, and be tolerant to faults and user mistakes
  - **Fault**: One component of a system deviating from its spec. Prefer tolerating faults over preventing them (except for things like security issues). Faults stem from hardware failures, software failures, and human error (in a study, config errors caused most outages).
  - **Failure**: The system as a whole not working
- Scalability
  - Describe load using **load parameters** (eg. requests/s, # of users, etc.). Use this to reason about 2 questions:
    1. If you increase the load parameter and keep resources (CPU, memory, etc.) constant, how is performance affected?
    2. If you increase the load parameter, how many more resources do you need to keep performance constant?
  - Report perf for a percentile (eg. p50, p99) depending on use case
- Maintainability
  - **Operability**. The ability for engineers & operators to monitor and debug a system, upgrade libraries & dependencies, audit the system, do capacity planning, and document the system. To do this well, a system should be introspectable, integrate with standard tools, avoid dependencies on specific machines, be well documented, have good defaults but be configurable, and overall, be predictable.
  - **Simplity**. Large state spaces, tight coupling, hacks, inconsistencies, etc. are bad.
  - **Evolvability**. Assume that requirements change. Use the right building blocks to make it easy to adapt to these new requirements.
- **Discussion questions**:
  1. What are the most common types of faults in your experience (config error, human error, breaking API changes, infra issues, etc.)?
  2. What are common load parameters that we deal with on the server and UI?
  3. What tradeoffs do we make (ie. do we often optimize towards a particular choice in that tradeoff?)
  4. What do we consider to be good reliability, and how do we measure that?
  5. How do we deal with faults vs. failures? (eg. how does proper modularization help us?)
  6. Operability is an interesting angle to consider when designing a system. What kinds of operability do we care about in practice (eg. logging to debug failures, standard tools to make it easier for engineers to onboard, writing docs, etc.)?
  
# Chapter 2: Data Models & Query Languages

- Data models are often composed and layered on top of one another
- **Object-relational mismatch**: One pain point is the translation layer between OO/functional languages and relational databases (often resolved by ORMs)
- Like data systems, data models have been converging over time (document support in SQL; joins in document DBs; etc.)
- Types of data models:
  - **Relational DBs** (eg. MySQL, PostgreSQL): Data is stored in unordered collections of tuples:

    ```
    data Tuple = (ID, Data | ID)
    ```

    - Decouples queries from how to execute them (managed by query planner & evaluator)
    - Decouples queries from access paths (if you add a new index, you don't need to update your query)
    - Uses **schema-on-write**. ie. code that does writing needs to know the schema. Like static type checking.
  - **NoSQL** (eg. MongoDB): In 2009, came out of the need for greater scalability, preference for OSS, query operations that were unsupported by relational model (map reduce?), and a desire for more free-form data models
    - Uses **schema-on-read**. ie. schema isn't defined ahead of time, but need to be assumed by application code that's doing the querying. Like dynamic type checking.
    - Criticisms:
      1. Applications need joins for many-to-many queries. If they're not supported by the DB, it just means people will have to do it in app code instead, which is bad.
      2. It makes data models less extensible in the long run, where you need normalization
      3. Writing to big documents is perf-intensive
  - **Network DBs** (eg. CODASYL). Generally not used anymore
  - **Graph DBs** (eg. Neo4J, Ents, Datomic). Two ways to implement it: as a property graph, or as a triple-store:

    ```
    # Property graph

    type Vertex = {id: ID, outgoing: Edge[], incoming: Edge[], properties: {[string]: Data}}
    type Edge = {id: ID, tail: Vertex, head: Vertex, label: string, properties: {[string]: Data}}
    
    # Triple-store
    
    type Data = string | number | boolean | ...
    type Triple = {subject: Atom, predicate: Relation, object: Atom | Data}
    ```
    - Key properties: Any vertex can have an edge to any other vertex (no schema); traversing a graph is efficient; a given pair of vertices can have >1 edge between them, representing >1 kind of relationship. Number of joins is not fixed in advance, as part of the query.
- Declarative queries are appealing because they describe what the intended result is, without saying how to get there (leaving this up to the implementation). It also makes constraints explicit (eg. "does order matter?"). Finally, it's easily parallelizable.
  - Kleppmann gave the example of CSS selectors, which was excellent (CSS selector vs. XPath vs. doing the same thing in JS)
- MapReduce: A way to issue highly scalable queries, using a more structured query language
- Datalog: The spiritual predecessor to graph query languages (also Prolog, IO, Erlang, etc.)
- **Discussion questions**:
  1. What DBs do we have at work that follow these paradigms?
  2. What are some of the reasons we use a Graph DB as a core abstraction (scalability across engineers & data, ease of data migrations, easier deployments, etc.)
  3. When is a better time to validate data against the schema (on read or on write). What are some of the advantages/disadvantages?
  4. Where does GraphQL fit in?
  5. What are other examples of the declarative vs. imperative tradeoff in our app code? When is one easier vs. the other? (eg. performance-critical applications, UI rendering code, aggregation code, etc.)
  
# Chapter 3: Storage and Retrieval

- Data is broadly stored in one of two kinds of databases: OLTP or OLAP. Typical differences:

| | **OLTP** (OnLine Transaction Processing) | **OLAP** (OnLine Analytics Processing)
|---|---|---
| Type | Relational (row-oriented) or Document | Columnar (column-oriented)
| User | End user | Analyst, Data Scientist
| Type of data | Latest | All historical data
| Read pattern | Small number of records, fetched by key | Aggregate over many records
| Write pattern | Random-access, low latency | ETL or event stream
| Perf bottleneck | Disk seek time | Disk bandwidth
| Indexing data structure | Log-structured (SSTable, LSM) or Update-in-place (B-tree) | LSM
| Examples | MySQL, LevelDB, Cassandra, HBase, Lucene | Hive, Presto, Spark, Redshift

- OLTP databases
  - Use **indexes** to speed up reads. Don't index everything by default, since updating indexes slows down writes
  - Can store on-disk (and cache in memory), or be fully in-memory (faster due to not having to encode data for writing to disk)
  - Angles to consider when implementing indexing schemes: file format (eg. binary), deleting records (delete vs. tombstone), crash recovery (how to rebuild an in-memory index from disk, partially-written records (use checksums to verify), concurrency control (a bigger issue for mutable indices like B-trees)
  - Types of indexes:
    - **Log-Structured Merge Trees** (LSM-trees) are composed of **Sorted String Tables** (SSTables). They're an index where the DB maintains a sorted list of primary keys, chunked as variable-sized segments. Backed by Red-Black or AVL trees. Uses MergeSort to combine segments. To look up a key, you look up its closest neighbor in the index, then do a linear scan till you find it. Often uses Bloom Filters to quickly bail out of queries for missing keys. Often faster for writes. Used by LevelDB, RocksDB.
    - **B-Trees**. Like LSM-trees, maintains a sorted list of primary keys. Keys are chunked into fixed-size blocks/pages, where a tree of size N has depth O(log N). Indices are overwritten on update, often using copy-on-write. Often faster for reads. Used by most relational DBs (eg. MySQL)
    - **R-Trees** are used for geospatial indexes (eg. PostGIS)
    - **Fuzzy indexes** are used for fulltext search, often backed by a trie or [Levenshtien Automaton](https://stackoverflow.com/questions/24411745/levenshtein-automata) (eg. Lucene, ElasticSearch)
  - Storing indexes:
    - **Heap Files** are used to store data for an index (index key -> pointer to heap file -> value in heap file)
    - In a **Clustered Index**, data is stored directly in the index instead of in a Heap File (eg. MySQL's InnoDB)
    - In a **Covering Index**, some data is stored in the index, and some in Heap Files
    - In a **Concatenated Index**, we index on multiple columns by concatenating their keys
  
      ```
      type Index = {[IndexKey: string]: HeapFileKey}
      type HeapFile = {[HeapFileKey: string]: Value}
      ```
    
- OLAP databases
  - Get their data from OLTP databases, either via batch uploads or streaming events
  - Orginizes data into a **star** or **snowflake** schema. In both, events are stored in tables prefixed `fact_`, with foreign keys into slower-moving tables prefixed `dim_` (representing who, what, where, etc.)
  - Usually stores data as columns: instead of colocating data from each row, colocate data in each column. Useful when you query a column at a time, rather than all the columns in a row.
  - Uses LSM trees for indexes (can't use B-trees if columns are compressed). Because of this, writes can be slow
  - Perf optimizations:
    - Compresses columns using bitmaps (1 bitmap per distinct value in a column, with 1 bit per row) + run-length encoding
    - Precomputes aggregates using **Data Cubes**, **Materialized Views**
    - Sorts rows (being careful to use the same sort order for every column), often by date (`ds`). Sometimes there can be secondary and tertiary sort orders, with diminishing returns.

# Chapter 4: Encoding and Evolution

- **Backwards compatability**: Newer code can read data that was written by older code.
- **Forwards compatability**: Older code can read data that was written by newer code.
- Data is usually encoded differently in memory (optimized for efficient access), vs on disk (optimized for size). To convert between the two, you **encode**/**decode** the data (or: **serialize**/**parse**, **marshall**/**unmarshall**)
- Formats for encoding data:
  - Key considerations when choosing a data format:
    - Backwards/forwards compatability
    - Support across different languages/clients
    - Size/efficiency
    - Expressiveness
    - Language-specific formats (often built into the stdlib): language-specific, sercurity issues, often unversioned, often inefficient
    - Need for explicit versioning
    - Need for documentation
    - Need for static types
  - Language-agnostic formats: JSON, XML, CSV, binary formats (BSON, BISON, etc.). JSON, XML, CSV have poor support for encoding numbers, don't support binary sequences (requiring Base64 encoding, which increases size), poor schema support. Binary JSON encodings don't reduce file size by much (20-30%).
  - Formats with stronger guarantees: Protobuf, Thrift, Avro. You define schemas in IDL files, and use a library to codegen bindings for your language of choice.
  
  | Format | Created By | Encoding Formats | Schema Support? | Backwards/Forwards Compatability |
  |----|----|----|----|----|
  | Avro | Apache | Binary | Yes. Supports `union`, `null` | Yes. Writer/Reader schemas are auto-translated
  | Protocol Buffers | Google | Binary | Yes. Supports `repeated` | Yes. Can change field names, but can only add fields. New fields must be optional.  |
  | Thrift | Facebook | Binary | Yes. Supports nested lists | Yes. Can change field names, but can only add fields. New fields must be optional. |

- Data flow:
  - *Data outlives code*: While code is updated often, some data in your DB might be years old. It's critical that you can continue to read + parse this data, ideally without paying the cost of expensive data migrations.
  - Ways to send data:
  
  | Protocol | Data format | Schema |
  |---|---|---|
  | REST | JSON | Often no schema. Can be codegenned, eg. using Swagger |
  | SOAP | XML | Yes, using WSDL |
  | RPC | Binary (eg. gRPC uses Protobuf) | Yes |
  | GraphQL | JSON | Yes |
  
  - RPC is fundamentally different from local function calls: networks introduce frequent errors (drops, timeouts, etc.) and have unpredictable latency; networks can time out, a failure mode that local fn calls don't have; idempotence is an issue for network requests; network requests require fully serialized data; data in network calls often has to be parsed differently for different languages (eg. Java vs. JS)
  - For RPC servers to support multiple versions, they need backward compatability on requests, and forward compatability on responses.
  - **Service-Oriented Architecture (SOA)**: An architecture where systems are composed of many services (that communicate using RPC, REST, etc.), that are independently managed & deployed
- **Discussion questions**:
  1. What data encoding & network formats do we use? (for APIs, services, logging, etc.)
  2. How does GraphQL fit into all this? What tradeoffs does it make compared to REST, SOAP, and RPC?
  3. Where do backward and forward-compatability come up in practice, for clients and servers? (GraphQL, SPIN, EntSchema, etc.)
  4. Where does idempotency come up in practice?
  5. How are local function calls vs. remote calls reflected in the UX (optimistic responses, loading states, etc.)
  
# Chapter 5: Replication

- Kinds of scaling:
  1. **Vertical scaling**: Giving a machine more CPU, disk, etc. (problem: cost grows faster than linearly, so 2x more expensive machine != 2x more power)
  2. **Horizontal scaling** ("shared-nothing architecture"): Scaling across more machines ("nodes")
- Why do we want replication? Availability, offline operation, latency, scalability
- **Eventual consistency**: multiple replicas eventually converge on the same state for a given record at some point
- **Concurrency**: multiple clients writing to a DB, without knowledge of what the other clients are writing
- DB nodes are organized into **leaders** and **followers**
- Architectures for replication:
  - **Single-leader**. Writes go to the leader node, and syndicated to follower nodes. Reads go to follower nodes. When a leader goes down, a new one elected via consensus. Used by default in most DBs.
  - **Multi-leader**. Writes go one of the leader nodes (often, one per data center). Reads to to follower nodes. Used by some large-scale DB deployments.
  - **Leaderless**. Reads and writes go to several nodes at once. Every value has a version number, and when a client reads a version that it knows is stale, it writes back to the node that gave that value with a more up-to-date value. Used by exotic DBs (eg. Cassandra, Dynamo, CRDTs)
- Other considerations for replication:
  - Synchronous vs. asynchronous: former is durable, latter performs better. Hybrid of one of former + a bunch of the latter used in practice, to balance durability & perf
  - Setting up new followers: sync a snapshot from a leader, then request all updates since the snapshot
  - Catch-up recovery: from a leader, request all updates since the snapshot
  - Failover: if a leader fails, nodes elect a new leader, usually based on who has the most up-to-date data
  - Replication format: you can replicate data between nodes with using:
    a. Statement-based replication (eg. evaluate `INSERT` statements on each node)
    b. Write-ahead log replication, syncing LSM-trees and B-trees directly
    c. Logical replication, using a special engine-agnostic data structure to represent records (most common)
    d. Trigger-based replication, for more custom replication schemes (eg. for handling conflicts)
 - Replication lag can causes inconsistent reads. Some guarantees that can help:
   - **Read-After-Write consistency**: After a client writes a piece of data, they should see that version of the data on subsequent reads
   - **Monotonic reads**: After a client has seen a piece of data, it should never see an older version of that same data
   - **Consistent prefix reads**: For some kinds of data, local ordering is critical (eg. a message and its response)
 - A **conflict resolution** strategy is necessary when clients are writing concurrently. Common strategies:
   - **Last Write Wins**: Easy to implement, but causes data loss.
 - **Discussion questions**:
   1. What are instances of last-write-wins in our day-to-day work? (eg. multiple users editing the Pending Posts queue)
   2. What are places where conflicts don't happen? (eg. multiple people posting to a feed)
   3. What are the different ways merge conflicts handled by the systems we use? (Mercurial? Google Docs/Quip? Multiple admins editing the group cover photo? Multiple teams working on similar features? People disagreeing in a meeting?)
   4. How do we think about consistency vs. durability vs. perf on clients? What are the different kinds of consistency? (eg. multiple UI elements showing the same value, staying in sync with the server, getting the latest updates from other clients, etc.)
   
# Chapter 6: Partitioning

- Goal of partitioning: improve scalability
- General architecture:
  - Data is split into partitions (`M` data -> 1 partition)
  - Partitions are hosted on DB nodes (`N` partitions -> 1 node)
  - Service router to route clients to the right partitions (1 request -> 1 partition)
- Ways to partition:
  - **Key range partitioning**: Partitions are decided by key (eg. keys 1-10 are on partition A, 11-20 are on B, etc.). Dynamically rebalance as keys/partition grows (similar process to B-trees)
  - **Hash partitioning**: Partitions are decided by hash of a key (eg. hash(1) = A, hash(2) = B, etc.). Typically, you create a bunch of partitions upfront and assign a bunch per node. As load grows, move entire partitions at a time from one node to another, ideally with a human in the loop to confirm the move (since it can be very resource-intensive)
- Secondary indices can be **document-partitioned** (colocted with primary key) or **term-partitioned** (secondary indices are partitioned separately from the data)

# Chapter 7: Transactions

- **Transaction ("tx")**: Real-world reads and writes are complex. Things can go wrong half-way through for a lot of reasons (DB hardware/software failure, application failure, network interrupt, concurrency issues, race conditions, etc.). A transaction is a way for an application to group a set of reads/writes into a logical unit. Either the whole unit succeeds (**commit**), or fails and can be retried (**abort** + **rollback**).
  - Goal: Make product engineers' lives easier with increased *safety guarantees*
  - There are lots of ways to define a transaction
- **Atomicity, Consistency, Isolation, Durability ("ACID")**: The set of safety guarantees that transactions provide. Often mushy in practice.
  - **Atomicity**: The ability to abort a transaction on error and have all writes from that transaction discarded.
  - **Consistency**: Data-model-specific invariants must always hold true. Not really related to the DB, more about product logic.
  - **Isolation**: Each transaction can pretend it's the only running transaction. Even if executed concurrently, the end result is the same as if executed sequentially.
  - **Durability**: Once a transaction is committed, its data will not be lost.
- **Basically Available, Soft state, Eventual consistency ("BASE")**: A set of weaker guarantees offered by systems that are not ACID-compliant
- Retrying aborted transactions is tricky in practice (eg. what if a tx actually succeeded, but then failed to report that it succeeded?)
- Kinds of isolation levels:

|                           | Read Committed | Snapshot Isolation                 | Serializable
|---------------------------|----------------|------------------------------------|-------------------------
| Also known as             |                | Repeatable read (PostgreSQL, MySQL), serializable (Oracle)                    | 
| Race conditions prevented | Dirty read, Dirty Write | All of Read Committed, plus: Read skew, Lost updates | All of Snapshot Isolation, plus: Write skew
| Implementation            | Row-level locks for writes, serve old values while writes are in progress for reads | Row-level locks for writes, Multi-Version Concurrency Control (MVCC) for reads | Executing transactions in serial order (often using stored procedures, and only possible when data fits in memory), Two-phase locking ("2PL"), Serializable Snapshot Isolation ("SSI")

- Kinds of race conditions:
  - **Dirty read**: `B` reads `A`'s writes before they have been committed
  - **Dirty writes**: `B` overwrites `A`'s writes before they have been committed
  - **Read skew** (aka. *nonrepeatable reads*): A reads different data at different points in time in one query
  - **Lost updates**: `B` overwrites `A`'s committed writes. Caused by concurrent read-update-write cycles. Prevented automatically, or by row-level read locks (`SELECT FOR UPDATE`)
  - **Write skew**: `A` writes to the database, but the premise of the write was no longer true by the time the write happened
  - **Phantom reads**: `A` writes to the database based on a search condition, but that condition was no longer true by the time the write happened
- **Discussion questions**:
  - What kinds of race conditions do we run into when using GraphQL mutations?
  - What are ways to design GraphQL mutations to avoid some of these race conditions? (hint: see the [Mutation design](https://www.internalfb.com/intern/wiki/Graphql-mutations/mutation-call-design/) wiki)
  - Do we run into any of these conditions when using Relay/GraphServices?
  - How do we think about transactionality and idempotency on the client?
  - How do we handle retries on the client? What are the most common causes of faults?
  - How do we think about transactionality for reads (loosely), and rendering partially-null data on clients?
  - How do we think about side effects (like logging) and transactions?
  
# Chapter 8: The Trouble With Distributed Systems

- Reliable systems can be built on top of unreliable infra (eg. TCP over IP, CPU registers & cosmic rays, etc.)
- In general, the approach is to approach reliability probabilistically (there's no such thing as 100% reliable, but we can reach 99% etc. in practice)
- Common sources of **partial failures** in distributed systems:
  1. **Network weather** can cause messages to be delayed, queued indefinitely, or lost
     - These can be hard to detect, since a node might have crashed, or be temporarily unresponsive, or can't send responses, etc.
  2. **Clocks** are often out of sync between nodes (eg. Google assumes 6ms drift for clocks synced with NTP every 30s), and may jump forward/backward anytime
     - **Time of day clocks** returns an absolute moment in time. Frequently synced with NTP server to stay accurate.
     - **Monotonic clocks** are useful for measuring durations.
     - **Logical clocks** increment counters to avoid synchronizing (eg. [Lamport Timestamps](https://en.wikipedia.org/wiki/Lamport_timestamp), [Vector Clocks](https://en.wikipedia.org/wiki/Vector_clock))
     - Synchronizing clocks is expensive at scale. Best pracice is to assume the reported time is accurate at a specific confidence interval
  3. **Execution pauses** are possible even in a single thread
     - Typical culprits: GC, VM suspension, client suspension, OS context switch, synchonous I/O, disk paging
- Types of *system models* for timing:
  1. Some systems we use are **asynchronous**, in that a fixed number of resources (eg. network) are distributed dynamically based on in-the-moment load (eg. the web)
  2. Some systems are **synchronous**, so that when a message is created, bandwidth is pre-allocated & reserved for it end-to-end (eg. telephones).
  3. Most systems are **partially synchronous** (somewhere in between)
- **Unbounded delays** are the norm. Some specialty systems (eg. for satellites) offer stronger guarantees & bounds
- In distributed systems, the truth is often decided by consensus (eg. clock times, leader nodes, etc.)
- **Byzantine Fault**: A fault where a node in a distrbuted system lies or breaks the protocol
- Types of *system models* for faults:
  1. **Crash-stop faults**: The only way for a node to fail is by crashing (eg. Erlang + OTP?)
  2. **Crash-recovery faults**: Nodes may crash and recover later; storage should be preserved (often a good model)
  3. **Byzantine faults**: Nodes may do anything, including trying to trick & decieve other nodes
- Kinds of guarantees we can design systems for:
  - **Safety**: A kind of property that, once broken, the damage can't be undone/reversed.
  - **Liveliness**: A kind of property that, when broken, is expected to be un-broken sometime later.
  
# Chapter 9: Consistency and Consensus

- Kinds of distributed consistency guarantees:
  - **Linearizability** (aka. *atomic consistency*, *strong consistency*, *immediate consistency*, *external consistency*): From a client's POV, the DB should behave as if it were a single node (both for reads and writes). Requires total ordering of all writes.
    -  Kills perf/reliability in practice: if a node is down, no nodes can process requests
  - **Causality**: From a client's POV, non-concurrent reads and writes are linearizable. More practical to implement.
     - One kind of constraint this guarantee can't model is *global uniqueness* guarantees (this requires linearizability)
- To support these guarantees, we need:
  - Monotonic counters. A common implementation is **Lamport Timestamps**.
  - **Total Order Broadcast**: A way to reliably broadcast messages in total order
- Famous theorems are not useful in practice:
  - **CAP Theorem**: The tradeoff isn't Consistency vs. Availability vs. Partition-tolerance. It's actually "Consistent or Available under Partition"
  - **FLP Result**: No algorithm can reach consensus if any node can crash. Only true for the *Asynchronous Model* (no clocks, no timeouts)
- **Two-Phase Commit** (2PC): An algorithm for atomically commiting a transaction over multiple nodes, by first preparing, then committing. Implemented by most DBs. Downsides: perf, relies on central coordinator.
- Popular consensus algorithms: Viewsampled Replication (VSR), Paxos, Raft, Zab. Often require >50% consensus.
- **Discussion questions**:
  - Where do we run into consistency issues in practice?
  - Where do we use consensus in practice? Service discovery?
  - What kinds of counters do we use?
  - What kinds of guarantees do we need in practice, as product engineers?