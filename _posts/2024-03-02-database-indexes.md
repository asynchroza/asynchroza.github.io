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
<img src="assets/img/clustered-indexes.png" alt="Clustered Indexes" width="500"/>
<a href="https://www.scaler.com/topics/clustered-index-in-sql/">SQL Server Clustered Indexes</a>
</div>

Hence, if you execute the following query:

```sql
SELECT * FROM users WHERE user_id = 123456
```

The database engine will look into the index, locate the `user_id` (primary key) and use the pointer to
access the corresponding row in the `USERS` table.

Depending on the type of index used, querying might happen differently in the background.
But for the most part indexes require a data structure called B-tree.

B-trees provide sorted data for easier searching, sequential access and removals.

<div style="display:flex; flex-direction:column; justify-content: center; align-items: center; margin-bottom: 10px;">
<img src="assets/img/porgramiz-btree-transparent.webp" alt="B-tree" width="500"/>
<a href="https://www.programiz.com/dsa/b-tree">B-tree Example</a>
</div>

## Why are not all columns indexed?

Aside from the required extra memory, it costs time and processing power to update indexes. For every index on a table, whenever a record is written, an additional write operation to the index table will occur. This can result in prolonged table locking.

## Different types of indexes

> TBA

### Clustered and Non-Clustered Index

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
