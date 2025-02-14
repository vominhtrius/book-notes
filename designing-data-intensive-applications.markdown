## Designing Data-Intensive Applications

by Martin Kleppmann

### Chapter 1: Reliable, Scalable, and Maintainable Applications

* Many applications are data-intensive and not compute intensive, meaning the biggest problems are usually the amount of data, the complexity of data, and the speed at which it is changing.

#### Thinking About Data Systems

* No single tool can meet all data processing and storage needs. Instead work is broken down into tasks that can be performed efficiently on a single tool, and application code stitches those tools together.
* When you create an API in front of several tools, you have created a new, special-purpose data system from smaller, general-purpose components.
* This book focuses on three main concerns: reliability, scalability, and maintainability.

#### Reliability

* Reliability means the system continues to work correctly, even when things go wrong.
* Things that can go wrong are called *faults*, and systems that anticipate faults and can cope with them are called *fault-tolerant* or *resilient*.
* A *fault* is when one component of the system deviates from its spec, whereas a *failure* is when the system as a whole stops providing the required service to the user.
* You cannot reduce the possibility of a fault to zero, and so you must design fault-tolerance mechanisms that prevent faults from causing failures.

##### Hardware Faults

* Hard disks have a MTTF of 10 to 50 years, and so on a storage cluster with 10,000 disks, we should expect on average one disk to die per day.

##### Software Errors

* Systemic errors, such as leap second bugs or cascading failures, are correlated across nodes and so tend to cause more system failures than uncorrelated hardware faults.

##### Human Errors

* Allow quick recovery from human errors, such as making it fast to rollback configuration changes, and to roll out code gradually.
* Set up detailed and clear monitoring, such as performance metrics and error rates.

#### Scalability

* Scalability is the term we use to describe a system's ability to cope with increased load.

##### Describing Load

* Load can be described with a few numbers that we call *load parameters*. The best choice of parameters depends on the architecture of your system.
* For Twitter, the distribution of followers per user is a key load parameter for describing scalability, as it determines the fan-out load.

##### Describing Performance

* You can look at what happens when load increases in two ways:
  * When you increase a load parameter and keep system resources fixed, how is the performance of your system affected?
  * When you increase a load parameter, how much do you need to increase the resources if you want to keep performance unchanged?
* Response time what the client sees, including the time to process the request (the service time) and network delays and queuing delays.
* Latency is the duration that a request is waiting to be handled – during which it is *latent*, awaiting service.
* The mean is not a good metrics to describe the "typical" response time, as it doesn't tell you how many users experienced that delay. Prefer percentiles instead.
* High percentiles of response times, or tail latencies, are important because they directly affect users' experience of the service.
* Reducing response times at high percentiles is difficult because they are easily affected by random events outside your control, and the benefits are diminishing.
* When artificially generating load to test scalability, the client must send requests independently of the response time, or else it will artificially keep the queues shorter than in reality, thereby skewing the measurements.
* Even if you call multiple backend services in parallel, the end-user request must wait for the slowest of the parallel calls to complete.
* Averaging percentiles, such as when combining data from several machines, is mathematically meaningless. The right way of aggregating response time data is to add the histograms.

##### Approaches for Coping with Load

* Scaling up, or vertical scaling, is moving to a more powerful machine. Scaling out, or horizontal scaling, distributes the load across multiple smaller machines.
* An architecture that scales well for a particular application is built around assumptions of operations will be common and which will be rare – the load parameters.
* In an early-stage startup it's more important to be able to iterate quickly on product features than to scale beyond some hypothetical future load.

#### Maintainability

* The majority of the cost of software is not in its initial development, but its ongoing maintenance.
* Three important design principles for maintainability are:
  * Operability: Make it easy for the operations teams to keep the system running smoothly.
  * Simplicity: Make it easy for new engineers to understand the system by removing complexity.
  * Evolvability (or extensibility): Make it easy for engineers to adapt the system to unanticipated use cases as requirements change.

##### Operability: Making Life Easy for Operations

* Good operations can work around the limitations of bad software, but good software cannot run reliably with bad operations.
* A good operations team works to preserve the organization's knowledge about the system, even as individuals come and go.
* Data systems with good operability provide good default behavior but allow overriding, and exhibit predictable behavior to minimize surprises.

##### Simplicity: Managing Complexity

* Making a system simpler does not necessarily mean reducing its functionality; it can also mean removing *accidental complexity*.
* Accidental complexity is complexity that is not inherent in the problem that the software solves, but arises only from the implementation.
* Good abstractions are one of the best tools for removing accidental complexity by hiding implementation details, but finding good abstractions is very hard.

##### Evolvability: Making Change Easy

* Simple and easy-to-understand systems are usually easier to modify than complex ones.

#### Summary

* Functional requirements are what an application should do.
* Nonfunctional requirements are general properties like security, reliability, compliance, scalability, compatibility, and maintainability.

### Chapter 2: Data Models and Query Languages

* Data models are the most important part of developing software, because they affect not only how the software is written, but how we think about the problem that we're solving.
* Layered software abstractions allow different groups of people to work together effectively.

#### Relational Model Versus Document Model

##### The Birth of NoSQL

* The term NoSQL has been retroactively reinterpreted as _Not Only SQL_.

##### The Object-Relational Mismatch

* Impedance mismatch is the disconnect between objects in the application code and the database model of tables, rows, and columns.
* A JSON representation of a document-oriented database has better locality than an equivalent multi-table schema.

##### Many-to-One and Many-to-Many Relationships

* In a document-oriented database, if information is duplicated and that information is changed, then all redundant copies need to be updated. Normalization is the key idea behind removing such duplication.

##### Are Document Databases Repeating History?

###### The relational model

* A key insight of the relational model was that you only need to build a query optimizer once, and then all database clients can benefit from it.

###### Comparison to document databases

* For many-to-one and many-to-many relationships, a related item is referenced by a *foreign key* in the relational model, and a *document reference* in the document model.

##### Relational Versus Document Databases Today

* The document data model offers greater schema flexibility, better performance from locality, and for some applications it is closer to the data structures used by the application.
* The relational model offers better support for joins, and many-to-one and many-to-many relationships.

###### Which data model leads to simpler application code?

* If your application data has a document-like structure (i.e. a tree of one-to-many relationships, where typically the entire tree is loaded at once), then a document model is likely a good idea.
* But if your application uses many-to-many relationships, the document model becomes less appealing.

###### Schema flexibility in the document model

* Document databases are sometimes called *schemaless*, but that's misleading – there is an implicit schema assumed by the application, but not enforced by the database.
* A more accurate term is *schema-on-read* in contrast with *schema-on-write*.
* Schema-on-read is similar to dynamic (runtime) type checking in programming languages, whereas schema-on-write is similar to static (compile-type) type checking.
* In cases where all records are expected to have the same structure, schemas are a useful mechanism for documenting and enforcing a data structure.

###### Data locality for queries

* The locality advantage only applies if you need large parts of the document at the same time.

#### Query Languages for Data

* An imperative language tells the computer to perform certain operations in a certain order.
* In a declarative query language like SQL or relational algebra, you specify the pattern of the data you want, but not *how* to achieve that goal.
* A declarative query language also hides implementation details of the database engine, allowing it to add performance improvements without requiring changes to queries.
* Declarative languages are more amenable to parallel execution because they specify the pattern of the results, but not the algorithm used to determine them.

##### MapReduce Querying

* MapReduce is based on a `map` (or `collect`) and `reduce` (or `fold` or `inject`) functions that exist in many functional languages.
* The `map` and `reduce` functions must be pure, thereby allowing the database to run them anywhere, and to rerun them on failure.

#### Graph-Like Data Models

##### Property Graphs

* A property graph model, each vertex and edge has a collection of properties (i.e. key-value pairs), and each edge has a label to describe the relationship between its vertices.
* To get the set of incoming and outgoing edges for a vertex, you can query the `edges` table by `head_vertex` or `tail_vertex` respectively.

##### Graph Queries in SQL

* In a relational database, you usually know in advance which joins you need in your query.
* In a graph query, you may need to traverse a variable number of edges before you find the vertex you're looking for – i.e. the number of joins is not fixed in advance.

##### Triple-Stores and SPARQL

* In a triple-store, all information is stored in the form of very simple three-part statements: (_subject_, _predicate_, _object_).
* The object is one of two things:
  * A value in a primitive data type, and so the predicate and object are equivalent to the key and value of a property on the subject vertex. (E.g. (_lucy_, _age_, _33_) is like a vertex `lucy` with properties `{"age": 33}`.)
  * Another vertex in the graph, and so the predicate is an edge, the subject is the tail vertex, and the object is the head vertex. (E.g. (_alice_, _married_to_, _bob_) has an edge _married_to_ between _alice_ and _bob_.)

##### The Foundation: Datalog

* Datalog generalizes the triple-store model, writing the triple as _predicate(subject, object)_.
* A Datalog rule applies if the system can find a match for _all_ predicates on the right side of the `:-` operator.
* When a rule applies, it's as though the left side of the `:-` was added to the database, with variables replaced by the values they matched.

### Chapter 3: Storage and Retrieval

* There is a big difference between storage engines optimized for transactional workloads and those optimized for analytics.

#### Data Structures That Power Your Database

* *Log* is defined as an append-only sequence of records. It doesn't have to be human-readable, and instead might be binary.
* Well-chosen indexes speed up read queries, but every index slows down writes, because the index also needs to be updated every time data is written.

##### Hash Indexes

* If new and updated values are blindly written to a log, that log can be divided into segments. Compacting a segment keeps only the most recent value for each key, discarding the older ones.
* Since compaction makes segments much smaller (assuming a key is updated several times in one segment), compaction can also merge several segments together.
* A background thread can merge and compact segments. While it happens, the database can still serve reads from old segment files, and write requests to the latest segment file.
* A *tombstone* marks a deleted key. When segments are merged, the tombstone tells the merging process to discard all previous values for the key.
* Data file segments are append-only and otherwise immutable, and so they can be read concurrently by multiple threads.
* Append-only design is advantageous in several ways:
  * Appending and segment merging are sequential write operations, which are generally much faster than random writes.
  * Crash recovery are much simpler if segment files are append-only or immutable, e.g. you avoid crashing while overwriting a value and leaving old and new data spliced together.
  * Merging old segments avoids the problem of data files getting fragmented over time.

##### SSTables and LSM-Trees

* Segment files sorted by key is a format called *Sorted String Table*, or *SSTable* for short.
* This has several advantages over log segments with hash indexes:
  * Merging segments is efficient even if the files exceed available memory, by using an algorithm similar to mergesort.
  * Your in-memory index of keys to file offsets can be sparse, and so the total size of your keys can exceed available memory.
  * If your in-memory index of keys is sparse, reads may scan over several key-value pairs on disk anyway, which can be compressed. This reduces disk space and I/O bandwidth use.

###### Constructing and maintaining SSTables

* Writes update a *memtable*, or an in-memory balanced tree. When its size exceeds some threshold, it can be written to disk as an SSTable file.
* Read requests first query the memtable and then query on-disk segments in order from newest to oldest.
* We can append every update to a log before updating the memtable. The log is not in sorted order, but can be used to restore the memtable after a crash.

###### Making an LSM-tree out of SSTables

* Storage engines based on the principle of merging and compacting sorted files are called LSM (Log-Structured Merge) storage engines.

###### Performance optimizations

* An LSM storage engine can use bloom filters to avoid querying the memtable and all segments for a key that does not exist.
* Since data is stored in sorted order, you can efficiently perform range queries, and because disk writes are essential the LSM-tree can support remarkably high write throughput.

##### B-Trees

* B-trees remain the standard index implementation in almost all relational databases, and many non-relational databases use them too.
* B-trees break down the database into fixed-size *blocks* or *pages* (usually 4KB) and read or write one page at a time. This maps closely to the underlying disk hardware.
* The number of references to child pages in one page of the B-tree is called its *branching factor*, and is typically several hundred.

###### Making B-trees reliable

* To make the database resilient to crashes, many B-tree implementations also persist to an append-only log called the *write-ahead log* (WAL, or *redo log*).
* When the database comes back up after a crash, the write-ahead log is used to restore the B-tree back to a consistent state.

###### B-tree optimizations

* In pages in the interior of a tree, we do not need to store full keys, but only enough information to act as boundaries between key ranges.
* B-trees try to ensure leaf pages are in sequential order on disk. This is easier for LSM-trees, which rewrite large segments of the storage in one go during merging.

##### Comparing B-Trees and LSM-Trees

* LSM-trees are typically faster for writes, whereas B-trees are thought to be faster for reads.

###### Advantages of LSM-trees

* *Write amplification* is where one write results in multiple writes over the database's lifetime, such as when compacting and merging SSTables.
* Write amplification is of concern with SSDs, which can only overwrite blocks a limited number of times before wearing out.
* LSM-trees typically sustain higher write throughput than B-trees because they sometimes have lower write-amplification, and because they sequentially write data.

###### Downsides of LSM-trees

* The bigger the database gets, the more disk bandwidth is required for compaction. But this bandwidth is limited and can increase response times at higher percentiles.
* Relational databases implement transaction isolation using locks on ranges of keys. Because each key exists at exactly one place in a B-tree index, those locks can be attached directly to the tree.

##### Other Indexing Structures

###### Storing values within the index

* With secondary indexes, each index references a location in the *heap file* that belongs to the corresponding row.
* A *clustered index* stores the indexed row directly within an index, to avoid the performance penalty upon reads of hopping from the index to the heap file.
* A compromise between a clustered index and a non-clustered index is a *covering index*, which stores *some* of the table's columns within the index (the index *covers* the query).

###### Multi-column indexes

* Multi-dimensional indexes allow querying several columns at once, which is important for geospatial data. But more commonly indexes like R-trees are used.

###### Full-text search and fuzzy indexes

* In Lucene, the in-memory index is a finite state automaton over the characters in the keys, similar to a trie.
* This automaton can be transformed into a *Levenshtein automaton*, which supports efficient search for words within a given edit distance.

###### Keeping everything in memory

* In-memory databases are faster because they avoid the overheads of encoding in-memory data structures in a form that can be written to disk.
* In-memory databases are not faster because they don't need to read from disk – the operating system caches recently used disk blocks in memory anyway.

#### Transaction Processing or Analytics?

* *Transaction processing* just means allowing clients to make low-latency reads and writes, as opposed to *batch-processing* jobs which only run periodically.
* OLTP, or *online transaction processing*, is the access pattern for "interactive applications" driven by the user's input.
* OLAP, or *online analytic processing*, scans over a huge number of records, reading only a few columns per record, and computes aggregates (e.g. sum, count, or average).
* The separate database on which OLAP queries are run is called the *data warehouse*.

##### Data Warehousing

* *Extract-Transform-Load* or ETL is the process of extracting data from the OLTP database, transforming it into an analysis-friendly schema, and loading it into the data warehouse.

###### The divergence between OLTP databases and data warehouses

* OLTP and OLAP are increasingly becoming two separate storage and query engines, which happen to be accessible through a common SQL interface.

##### Stars and Snowflakes: Schemas for Analytics

* Many data warehouses are used in a formulaic style known as *star schema* (also known as *dimensional modeling*).
* The center of the schema is a *fact table* where each row represents an event that occurred at a particular time.
* Columns in the fact table are attributes or foreign keys into *dimension tables*, which represent the	*who*, *what*, *where*, *when*, *why*, and *how* of the event.
* The name *star schema* comes from visualizing the fact table in the middle, connected to its surrounding dimension tables like rays in a star.
* A variation of this is the *snowflake schema*, where dimensions are further broken down into subdimensions.
* In a typical data warehouse, tables are often very wide: fact tables have over 100 columns, and sometimes several hundred.

#### Column-Oriented Storage

* Although fact tables are often over 100 columns wide, a typical data warehouse query only accesses 4 or 5 of them at one time.
* *Column-oriented storage* stores all the values from each column together, instead of all the values from each row together.
* The column-oriented storage layout relies on each column file containing the rows in the same order.

##### Column Compression

* We can convert a column with *n* distinct values into n separate bitmaps, with one bitmap for each distinct value, and one bit for each row.
* If *n* is large, then the bitmaps will be sparse, and the bitmaps can additionally be run-length encoded to make them very compact.
* An `IN` SQL query can be implemented by a bitwise *OR* of bitmaps, while an `AND` SQL query can be implemented by a bitwise *AND* of bitmaps.

###### Memory bandwidth and vectorized processing

* *Vectorized processing* is where operators like bitwise *AND* and *OR* operate on chunks of compressed column data directly.

##### Sort Order in Column Storage

* In column store, the ordering of rows is irrelevant, and so inserting a new row requires simply appending to each of the column files.
* If after choosing an order, the primary sort column does not have many distinct values, then a simple run-length encoding can compress it efficiently.
* Run-length encoding compresses the first sort key best, as subsequent sort keys will not have such long runs of repeated values.

###### Several different sort orders

* In a column store, there normally aren't pointers to data elsewhere, only columns containing values.

##### Writing to Column-Oriented Storage

* Unlike B-trees, which use an update-in-place approach, LSM-trees allow inserting a row with compressed columns without rewriting all the column files.
* When using an LSM-tree, it doesn't matter whether the in-memory store is row-oriented or column-oriented.

##### Aggregation: Data Cubes and Materialized Views

* A materialized view is an actual copy of the query results written to disk, while a virtual view is just a shortcut for writing queries.
* A *data cube* or *OLAP cube* is a materialized view. It is a grid of aggregates grouped by different dimensions.
* The advantage of a materialized data cube is that certain queries become very fast because they have effectively been precomputed.
* The disadvantage is that a data cube doesn't have the same flexibility as querying the raw data.

#### Summary

* Analytic workloads require sequentially scanning across a large number of rows, and so indexes are much less relevant.
* Instead it is more important to encode data compactly, in order to minimize the amount of data that the query needs to read from disk.

### Chapter 4: Encoding and Evolution

* Backward compatibility means newer code can read data written by older code. Forward compatibility means older code can read data written by newer code.
* Forward compatibility is trickier because it requires older code to ignore additions made by a newer version of the code.

#### Formats for Encoding Data

##### Language-Specific Formats

* Using a language's built-in encoding functionality is bad from interoperability, security, efficiency, and versioning perspectives.

##### JSON, XML, and Binary Variants

* JSON distinguishes strings and numbers, but it doesn't distinguish integers and floating-point numbers, and it doesn't specify a precision.
* Base64-encoding a binary string to use it in XML or JSON increases the data size by 33%.
* Simply having different organizations agree on a single format is far more important than the aesthetics or efficiency of a format.

###### Binary encoding

* Because JSON and XML don't prescribe a schema, they need to include all the object field names within the encoded data.
* It's not clear whether the small space reduction by MessagePack, a binary encoding for JSON, is worth the loss of human-readability.

##### Thrift and Protocol Buffers

* Confusingly, Thrift has two binary encoding formats, called *BinaryProtocol* and *CompactProtocol*.
* Field tags are like aliases for fields – they are a compact way of saying what field we're talking about without spelling out its name.
* Variable length integers use the top bit of each byte to specify whether there are still more bytes to come.
* With variable length integers, the numbers -64 to 63 are encoded in one byte, -8192 to 8191 are encoded in two bytes, etc. Bigger numbers use more bytes.
* The `required` keyword of Protocol Buffers enables a runtime check that fails if the field is not set, which is useful for catching bugs.

###### Field tags and schema evolution

* With Thirft and Protocol Buffers, you can change a field name, but changing its tag renders all existing encoded data invalid.
* If old code reads data written by new code, including a new field with an unrecognized tag, it can simply ignore that field. This maintains forward compatibility.

###### Datatypes and schema evolution

* With Protocol Buffers, it's okay to change an `optional` (single-valued) field into a `repeated` (multi-valued) field.

##### Avro

* Avro has two schema languages: Avro IDL is intended for human editing, and a JSON equivalent that is more easily machine-readable.
* An encoded byte string specifies nothing to identify fields or their data types. It simply consists of values concatenated together.
* Consequently the binary data can only be decoded correctly if the code reading the data is using the *exact same schema* as the code that wrote the data.

###### The writer's schema and the reader's schema

* While decoding data, Avro resolves the differences between the *writer's schema* and the *reader's schema*, and translates read data from the former to the latter. This enables schema evolution.

###### Schema evolution rules

* Adding a field that has no default value breaks backward compatibility. Removing a field that has no default value breaks forward compatibility.
* Avro doesn't have `optional` or `required` keywords like Thrift and Protocol buffers, but instead allows defining union types with `null` as a value.

###### Dynamically generated schemas

* Because an Avro schema doesn't define tag numbers, it is friendlier to dynamically generated schemas, e.g. creating an Avro schema from a relational database schema.

##### The Merits of Schemas

* Database vendors provide a driver (e.g. using the ODBC or JDBC APIs) that decode responses from a database's network protocol into in-memory data structures.
* Schema evolution allows the same flexibility as schemaless/schema-on-read JSON databases, while also providing better guarantees about your data and better tooling.

#### Modes of Dataflow

##### Dataflow through Databases

* In an environment where the application is changing, it's likely that some processes accessing the database will be running newer code and some will be running older code.
* Consequently a value in the database might be written by a newer version of the code, and then later read by an older version. And so forward compatibility is required.
* If an application decodes a database value into model objects, and then later re-encodes those model objects, the unknown fields may be lost in that translation process.

###### Different values written at different times

* Data outlives code: While deploys may replace older code with newer code within minutes, data in the database may have last been encoded and written years ago.
* Rewriting (migrating) data into a new schema is possible, but it is expensive on a large data set, and so most databases avoid it if possible.
* Schema evolution allows a database to appear as if it was encoded with a single schema, although the underlying storage may contain records encoded with various historical schema versions.

##### Dataflow Through Services: REST and RPC

* The application-specific APIs of services provide encapsulation, by restricting what clients can and cannot do.
* A key design goal of service-oriented/microservices architecture is to make the application easier to change and maintain by making services independently deployable and evolvable.

###### Web services

* The two popular approaches to web services are REST and SOAP.
* REST emphasizes data formats, using URLs for identifying resources and using HTTP features for cache-control, authentication, and content-type negotiation.
* SOAP is an XML protocol for API requests, and comes with a sprawling set of related standards (the *web standards framework*, or *WS-*) that add various features.
* The API of a SOAP web service is described using an XML-based language called the Web Services Descriptive Language, or WSDL. WSDL enables code generation so that a client can access a remote service using local classes and method calls.
* WSDL is not designed to be human-readable, and SOAP messages are often too complex to construct manually, so tooling, code generation, and IDEs fill in the gaps.
* Despite the ostensible standards, interoperability between different vendors' implementations often causes problems in practice.

###### The problems with remote procedure calls (RPCs)

* The RPC model tries to make a request to a remote service look the same as calling a method within the same process – an abstraction called *location transparency*.
* Variable latency, timeouts, retries and idempotence semantics, and serialization all mean there's no point in location transparency. Calling a remote service is fundamentally different.

###### Current directions for RPC

* The new generation of RPC frameworks is more explicit about the fact that a remote request is different from a local function call.
* RESTful APIs have the advantage of being good for experimentation and debugging, support by all mainstream programming languages and platforms, and a vast ecosystem of available tools.
* For these reasons, REST is used for public APIs, while RPC frameworks mostly focus on requests between services owned by the same organization.

###### Data encoding and evolution for RPC

* Between services using RPC, we assume servers will be updated first, and clients second. You therefore need backward compatibility on requests, and forward compatibility on responses.
* The backward and forward compatibility properties of an RPC scheme are inherited from whatever encoding it uses, such as Thrift or gRPC.

##### Message-Passing Dataflow

* In *asynchronous message-passing* systems, messages are delivered to an intermediary called a *message broker* which stores the message temporarily.
* Message brokers can act as a buffer if the recipient is unavailable or overloaded, redeliver messages to crashed processes, deliver messages to multiple recipients, and decouple producers and consumers.

###### Message brokers

* A process sends a message to a named *queue* or *topic*, and the broker ensures delivery of the message to one or more *consumers* or *subscribers* of that queue or topic.

###### Distributed actor frameworks

* In the *actor model*, logic is encapsulated by actors that communicate via asynchronous messages, where delivery of each message is not guaranteed (even within the same process).
* In a *distributed actor framework*, location transparency works better than with RPC, because the actor model already assumes that messages may be lost.
* A distributed actor framework essentially integrates a message broker and the actor programming model into a single framework.
