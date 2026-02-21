# Chapter 2: Data Models and Query Languages

> Most applications are built by layering one data model on top of another. For each layer, the question is, how is it represented in terms of the next layer?

Each layer hides the complexity of the layers below it by providing a clean data model.

## Relational model vs Document model

In relational mode, data is organized into relations, where each relation is an unoredered collection of tuples.

Relational model has its origins in business data processing in the 1960s and '70s. But it has dominated and outlived many other different data models.&#x20;

Much of what we see on the web is still powered by relational databases. Examples:

* WordPress, which powers over 43% of the websites on the Internet, uses MySQL - [source](https://learn.wordpress.org/tutorial/the-wordpress-database/)

NoSQL = Not Only SQL

Driving forces behind the adoption of NoSQL:

* Need for greater scalability requirements such as very large datasets and very high write throughput.
* Preference for free and open software over commercial db products
* Specialized query operations not well-supported by the relational model
* Desire for more dynamic and expressive data model.

Impedance mismatch: The . For example, ORM frameworks such as ActiveRecord, Hibernate.

| Relational data model                                                                                                                                                                                                                              | Document data model                                                                                                                                                                                                                                                                                |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Schema-on-write: structure of the data is explicit and the database ensures all written data conforms to it.                                                                                                                                       | <p>Schema-on-read: structure of the data is implicit and only interpreted when the data is read.<br><br>So not really schemaless as many sources claim.</p>                                                                                                                                        |
| Need for a translation layer between the objects in the application code and relational database model of tables, rows and columns (impedance mismatch).                                                                                           | Data representation is usually closer to the data structures in application layer, thus reducing the impedance mismatch.                                                                                                                                                                           |
| <p>Lower data locality with multi-table schema, needing joins to stitch together all the information.<br><br>Modern RDBMS such as Spanner and Oracle's multi-table cluster index do provide better locality by grouping related data together.</p> | <p>Data locality is generally better due to the document being mostly self-contained with support for nested fields -> better performance.<br><br>Such an advantage only applies if the application often needs to access the entire document or large parts of the document at the same time.</p> |
| Lower duplication with the help of normalization, leading to consistent representation of duplicate data.                                                                                                                                          | Data may be mostly denormalized with self-containedness of the document (denormalized), thus requiring additional work for ensuring duplicate data is consistent across all the documents.                                                                                                         |
| Many-to-one and many-to-many relationships are resolved with a foreign key using a join or follow-up queries.                                                                                                                                      | Many-to-one and many-to-many relationships are resolved with a document reference using a join or follow-up queries.                                                                                                                                                                               |
| Many-to-one and many-to-many relationships have better support with joins.                                                                                                                                                                         | <p>Many-to-one and many-to-many relationships have weaker support for joins and application code may need to handle such capability which is usually slower than the specialized code inside the RDBMS. </p><p></p><p>But only relevant if the application has such interconnectivity.</p>         |
| If the application has a document-like structure, application code may become complicated and schemas cumbersome due to "shredding"of data.                                                                                                        | If the application has a document-like structure, this model may better represent it without needing complicated application code for handling it.                                                                                                                                                 |

Despite these differences, RDBMS and document databases are converging in functionality, with RDBMS supporting document formats such as XML, JSON, and document dbs supporting relational-like joins, resolving document references.

## Query languages for data

Imperative query language: tells the computer to perform certain data operations in a certain order.

Declarative query language: specify the pattern of the data you want - what conditions the result must meet, how you want the data to be transformed - but not how to achieve that goal.

