---
title: How do indexes actually work?
date: 2024-03-02
categories: [Databases]
tags: [databases, indexes]
pin: true
math: true
mermaid: true
---

## What's an index?

In simple terms, database indexes are data structures which allow for efficient reading.
You can imagine them as lookup tables which store pointers (in most cases) to the location of the searched row in the table.

<div style="display:flex; flex-direction:column; justify-content: center; align-items: center; margin-bottom: 10px;">
<img src="assets/img/clustered-indexes.png" alt="Clustered Indexes" width="500" id="clustered-index-example"/>
<a href="https://www.scaler.com/topics/clustered-index-in-sql/">Scaler: SQL Server Clustered Indexes</a>
</div>

Hence, if you execute the following query:

```sql
SELECT * FROM users WHERE user_id = 123456
```

The database engine will look into the index, locate the `user_id` (primary key) and use the pointer to
access the corresponding row in the `USERS` table.

Depending on the type of index used, querying might happen differently in the background.
But for the most part indexes require a data structure called B-tree.

B-trees provide sorted data (database records are not guaranteed to be sorted) for easier searching, access and removals.

<div style="display:flex; flex-direction:column; justify-content: center; align-items: center; margin-bottom: 10px;">
<img src="assets/img/porgramiz-btree-transparent.webp" alt="B-tree" width="500"/>
<a href="https://www.programiz.com/dsa/b-tree">Programiz: B-tree</a>
</div>

## Why are not all columns indexed?

Aside from the required extra memory, it costs time and processing power to update indexes. For every index on a table, whenever a record is written, an additional write operation to the index table will occur. This can result in prolonged table locking.

## Different types of indexes

Some of the most common types of indexes are [Clustered index and Non-clustered index](https://learn.microsoft.com/en-us/sql/relational-databases/indexes/clustered-and-nonclustered-indexes-described?view=sql-server-ver16), [Hash index](), [B-tree index](), [Unique index](), [Full-text index]()

### Clustered index

Clustered indexes sort the data rows based on their primary keys and store it accordingly. Therefore, the only way to have a sorted table in any SQL database is
by having a clustered index. Behaviour is different between different databases, but there are management systems such Microsoft SQL Server which primary key constraint
creates a clustered index by default. What's so special about clustered indexes is that data they _point_ to is stored in the same order phisically on the disk as it is in the cluster table.

<div style="display:flex; flex-direction:column; justify-content: center; align-items: center; margin-bottom: 10px;">
<img src="assets/img/example-clustered-index.jpeg" alt="Clustered Index" width="500"/>
<a href="https://www.geeksforgeeks.org/difference-between-clustered-and-non-clustered-index/">GFG: Difference Between Clustered and Non-Clustered Indexes</a>
</div>

Initially, it can be perplexing to determine the targets of these pointers. To clarify, on the left side, there's a cluster table which stores the primary keys of the rows as the keys and the corresponding pointers to physical blocks as values.

Let's consider that we are searching for the row with PK = 2. We search the clustered table on the left to locate the key `2`. Once found, we look up its pointer value, which directly guides us to the physical location where the rows with that key are stored. To be honest, the example above is not ideal, or it is only as good as it can be, given that there isn't a single example conforming to a proper "algorithm" for selecting pointers to the physical blocks. If you compare this example with the [first image on the page](#clustered-index-example), you will notice that the algorithm for pointer selection is somewhat different. Additionally, there appears to be a mistake in the selection of the 5th key in the first image. Therefore, it's advisable not to focus too much on these images but rather to remember that clustered indexes are sorted in the same way as data is physically stored.

### Non-clustered index

> TBA

### Comparison of clustered and non-clustered indexes

This is a really nice table I found on [GFG](https://www.geeksforgeeks.org/difference-between-clustered-and-non-clustered-index/):

| Feature                   | Clustered Index                                    | Non-Clustered Index                                           |
| ------------------------- | -------------------------------------------------- | ------------------------------------------------------------- |
| Speed                     | Faster                                             | Slower                                                        |
| Memory Usage              | Requires less memory for operations                | Requires more memory for operations                           |
| Data Storage              | Main data is in the clustered index                | Index is a copy of data in a non-clustered index              |
| Number of Indexes Allowed | Only one per table                                 | Multiple per table                                            |
| Disk Storage              | Stores data on disk                                | Does not inherently store data on disk                        |
| Storage of Data in Index  | Stores pointers to blocks, not data                | Stores both value and a pointer to the actual row             |
| Leaf Nodes                | Actual data in leaf nodes                          | Leaf nodes do not contain actual data, only included columns  |
| Order of Data             | Clustered key defines order within the table       | Index key defines order within the index                      |
| Physical Order on Disk    | Physically reordered to match the index            | Logical order does not necessarily match the physical order   |
| Size                      | Large (especially for the primary clustered index) | Comparatively smaller, especially for the non-clustered index |
| Default for Primary Keys  | Primary keys are default clustered indexes         | Composite keys with unique constraints act as non-clustered   |

### Hash index

> TBA

### Unique index

> TBA

### Full-text index

> TBA
