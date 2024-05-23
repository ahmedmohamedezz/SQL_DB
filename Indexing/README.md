# Indexing in SQL

- indexes: used to speed up the retrieval of records in response to certain search conditions

- the index structures are additional files on disk that provide secondary access paths

- the same table can have more than 1 index to multiple fields

- types of indexes are based on ordered files (single-level indexes) and use tree data structures (multilevel indexes, B+-trees) to organize the index

---

### Single-Level Ordered Indexes

- the idea behind an ordered index is similar to that behind the index used in a textbook, which lists important terms at the end of the book in alphabetical order along with a list of page numbers where the term appears in the book

- the field used as index is called the indexing field (or attribute)

- the values in the index are ordered so that we can do a binary search on the index

- the index file is typically much smaller than the data file

### 1 Primary Indexes

- a primary index is an ordered file whose records are of fixed length with two fields (key, pointer)

  - key field must be unique & can be sorted

- The total number of entries in the index is the same as the number of disk blocks in the ordered data file

- Indexes can also be characterized as dense or sparse. A dense index has an index entry for every search key value (and hence every record) in the data file. A sparse (or nondense) index, on the other hand, has index entries for only some of the search values

- primary index is sparse if it points to blocks of records, not each record

  - index to all names starting with 'a', another for 'b', ...etc

- A major problem with a primary index—as with any ordered file—is insertion and deletion of records.

---

#### Example

- Suppose that we have an ordered file with r = 300,000 records stored on a disk with block size B = 4,096 bytes.5 File records are of fixed size and are unspanned, with record length R = 100 bytes. The blocking factor for the file would be bfr = ⎣(B/R)⎦ = ⎣(4,096/100)⎦ = 40 records per block. The number of blocks needed for the file is b = ⎡(r/bfr)⎤ = ⎡(300,000/40)⎤ = 7,500 blocks.
- A binary search on the data file would need approximately ⎡log2 b⎤= ⎡(log2 7,500)⎤ = 13 block accesses.
- Now suppose that the ordering key field of the file is V = 9 bytes long, a block pointer is P = 6 bytes long, and we have constructed a primary index for the file. The size of each index entry is Ri = (9 + 6) = 15 bytes, so the blocking factor for the index is bfri = ⎣(B/Ri)⎦ = ⎣(4,096/15)⎦ = 273 entries per block.
- The total number of index entries ri is equal to the number of blocks in the data file, which is 7,500. The number of index blocks is hence bi = ⎡(ri/bfri)⎤ = ⎡(7,500/273)⎤ = 28 blocks. To perform a binary search on the index file would need ⎡(log2 bi)⎤ = ⎡(log2 28)⎤ = 5 block accesses.
- To search for a record using the index, we need one additional block access to the data file for a total of 5 + 1 = 6 block accesses—an improvement over binary search on the data file, which required 13 disk block accesses.

---

### 2 Clustering Indexes

- If file records are physically ordered on a nonkey field—which does not have a distinct value for each record—that field is called the clustering field and the data fileis called a clustered file

- the index structure is the same as primary index (fixed len + 2 fields)

- There is one entry in the clustering index for each distinct value of the
  clustering field

- index key is the value of the record, index pointer points to the **first** record that has this value

---

### 3 Secondary Indexes

- A secondary index provides a secondary means of accessing a data file for which some primary access already exists

- The data file records could be ordered, unordered, or hashed.

- The secondary index may be created on a field that is a candidate
  key and has a unique value in every record, or on a nonkey field with duplicate values.

- same index structure (ordered file, 2 fields)

- if the index key is a candidate key (unique values), then the index is **dense**

- note that if the key part in the index can be sorted & reference a unique valued field in the data file (table)

  - we can sort key in the index, no matter it's sorted in data file or not

- if the secondary index on a field that can have dupilcates & non-ordered, we have several options here

  - index can have duplicate key values for each record => dense index
  - using variable length records in the index, and store list of pointers for each key value
  - the most commonly used approach is to keep the records of fixed length, and for unique values of the indexing field, **BUT** add extra level for list of pointers to records of that key value

- example on the last option
  - index <1, 0x33>, pointer points to list of locations in the extra level
  - 0x33 => [0x1, 0x15, 0x10] list of record addresses

---

#### Readings

- [Datacamp](https://www.datacamp.com/tutorial/introduction-indexing-sql)
- [GFG](https://www.geeksforgeeks.org/indexing-in-databases-set-1/)
- [Medium](https://user3141592.medium.com/single-vs-composite-indexes-in-relational-databases-58d0eb045cbe)

---

### Multilevel Indexes

- we can construct a multi-level index by considering the data file as the first level.
- primary index of the first level => second level
- primary index of the second level => third level

- we are still faced with the problems of dealing with index insertions and deletions, because all index levels are physically ordered files

- To retain the benefits of using multilevel indexing while reducing index insertion and deletion problems, designers adopted a multilevel index called a **_dynamic multilevel index_** that leaves some space in each of its blocks for inserting new entries and uses appropriate insertion/deletion algorithms for creating and deleting new index blocks when the data file grows and shrinks

- dynamic multilevel index is implemented using B-trees & **B+-trees**

- B-tree nodes are kept between 50 and 100 percent full, and pointers to the data blocks are stored in both internal nodes and leaf nodes of the B-tree structure. In Section 17.3.2 we discuss B+-trees, a variation of B-trees in which pointers to the data blocks of a file are stored only in leaf nodes

- we need to make the tree balanced, so we:

  - avoid skewness of the tree (chain case)
  - make the search speed uniform

- we also need to minimize the level of the tree

---

#### B -Trees

- The B-tree has additional constraints that ensure that the tree is always
  balanced and that the space wasted by deletion is minimized

- can be used to create an index for non-oredered key field

---

#### B+ -Trees

- data pointers are stored only at the leaf nodes of the tree

- The leaf nodes have an entry for every value of the search field, along with a data pointer to the record (or to the block that contains this record)
  if the search field is a key field.

- For a nonkey search field, the pointer points to a block containing pointers to the data file records, creating an extra level of indirection.

- usually you can traverse leaf nodes in order (left to right), through special pointers in the leafes
  - note that leaf node structure is not the same as other nodes
  - leaf nodes have pointers to (data nodes, or next leaf node)
  - other nodes have pointers to other tree nodes in different levels

- since no data in internal nodes in the tree, we can use more pointers to tree nodes, thus less tree levels & improved search time

---

### Indexes on Multiple Keys

- if a combination of fields are queried frequently, they maybe both used as a key of index

- if the data in not large, you may use 1 index to either of the fields of the query
  - if you want to query no. of employee in dept 'x', having age >= 'y'
  - you may use 1 index to age, or dept to initialy filter results, then brute force the result to find records satisfying the other condition

- 
