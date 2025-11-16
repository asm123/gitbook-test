# Chapter 3: Storage and Retrieval

## Terminology

* Write amplification: One write to the database resulting in multiple writes to the disk over the course of the database's lifetime.

## B-Trees vs LSM-Trees

**B-Tree**

1. Faster for reads
2. Higher write amplification than LSM-Tree
3. Has to overwrite several pages in the tree
4. Can lead to disk space fragmentatuon
5. Query response time is generally predictable
6. Compaction may not keep up with high write throughput -> unmerged segments on disk increase -> slowed down reads
7. Used by: [MySQL](https://dev.mysql.com/doc/refman/8.0/en/mysql-indexes.html), [PostgreSQL](https://www.postgresql.org/docs/current/btree-implementation.html), [Oracle](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/managing-indexes.html#GUID-2CE1BB91-3EFA-450D-BD31-0C961549F0C2).

**LSM-Tree**

1. Faster for writes
2. Able to sustain higher write throughput due to:
   * lower write amplification
   * sequentially write compact SSTable files
3. Produces smaller files on disk
4. Lower storage overhead with leveled compaction
5. Compaction process can interfere with the performance of ongoing reads and writes -> response time for queries can sometimes be high
6. Each key exists in only one place in index -> can offer strong transaction isolation semantics
7. Used by: [Cassandra](https://docs.datastax.com/en/cassandra-oss/3.x/cassandra/dml/dmlManageOndisk.html), [CockroachDB](https://www.cockroachlabs.com/docs/v23.1/architecture/storage-layer#pebble), [Ressi for Google Spanner](https://static.googleusercontent.com/media/research.google.com/en/pubs/archive/46103.pdf).

## Other indexing structures

* Secondary index
* Storing values within the index
  * Storing actual row - Clustered index - store indexed row within the index. MySQL InnoDB.
  * Reference to the row stored in a file called _heap file_ - unordered, append-only, tracking delete rows
  * Covering index / index with included columns - store some of a table's columns within the index.
* Multi-column index
  * Concatenated index - combine several fields into one key by appending one column to another
* Fuzzy index
  * Full-text search - Lucene
* In-memory database
  * Write a log of changes to disk by writing periodic snapshots to disk or by replicating the in-memory state to other machines.
  * Anti-caching - evicting the least recently used data from memory to disk when there is not enough memory, load back into memory when accessed again in future.
  * NVM - non-volatile memory
  *

## OLTP and OLAP

* OLTP = online transaction processing = small number of records per query, fetched by key
* OLAP = online analytic processing = aggregate over large number of records
* Data warehouse contains a read-only copy of the data in all the OLTP systems.
  * ETL = Extract-Transform-Load - extract data from OLTP Databases, transform into analysis-friendly schema, clean up, load into data warehouse
* Schemas for analytics
  * Star schema / dimensional modelling
    * Fact table - each row represents an event that occurred at a particular time
    * Dimension table - table for foreign key references to other tables, typically columns in fact tables
  * Snowflake schema - similar to star schema, dimensions are further broken down into subdimensions.
    * More normalized than star schemas.

## Column-oriented storage

* Store all the values from each column together instead of storing all the values form one row together.
* Each column is stored in a separate file.
* Examples: Cassandra, HBase
* Column compression
  * Bitmap encoding - column with n distinct values is condided as n separate bitmaps - one bitmap for each distinct value, one bit for each row.
  * Vectorized processing - bitwise operators on chunks of compressed column data
* Aggregation
  * Materialized views - actual copy of query results with materialized aggregates
  * Data cube / OLAP cube - grid of aggregates grouped by different dimensions

## Further reading

* [Bitcask: A Log-Structured Hash Table for Fast Key/Value Data](https://riak.com/assets/bitcask-intro.pdf)
* [Dremel: Interactive Analysis of Web-Scale Datasets](https://research.google/pubs/pub36632/)
* [LevelDB Implementation Notes](https://github.com/google/leveldb/blob/main/doc/impl.md)
* [Bigtable: A Distributed Storage System for Structured Data](https://research.google/pubs/pub27898/)
* [The Log-Structured Merge-Tree (LSM-Tree)](https://www.cs.umb.edu/~poneil/lsmtree.pdf)
* [Hacking Lucene - The Index Format](https://massiveprogramming.blogspot.com/2014/10/hacking-lucene-index-format-hacker-labs.html)
* [Space/Time Trade-offs in Hash Coding with Allowable Errors](https://people.cs.umass.edu/~emery/classes/cmpsci691st/readings/Misc/p422-bloom.pdf)
* [The Advantages of an LSM vs a B-Tree](http://smalldatum.blogspot.co.uk/2016/01/summary-of-advantages-of-lsm-vs-b-tree.html)
* [Burst Tries: A Fast, Efficient Data Structure for String Keys](http://citeseer.ist.psu.edu/viewdoc/summary?doi=10.1.1.18.3499)
* [Fast String Correction with Levenshtein Automata](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.16.652)
* [Anti-Caching: A New Approach to Database Management System Architecture](http://www.vldb.org/pvldb/vol6/p1942-debrabant.pdf)
* [The Traditional RDBMS Wisdom Is (Almost Certainly) All Wrong](http://slideshot.epfl.ch/talks/166)
* [Dremel made simple with Parquet](https://blog.twitter.com/engineering/en_us/a/2013/dremel-made-simple-with-parquet)
* [The Design and Implementation of Modern Column-Oriented Database Systems](http://cs-www.cs.yale.edu/homes/dna/papers/abadi-column-stores.pdf)
