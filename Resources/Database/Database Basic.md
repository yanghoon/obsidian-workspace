# Trend

* multi-core, multi-processor, cheap storages
* scale(mobile)
* elastic, distributed, reactive

# Storage

## Log Structured

* Log
	* data format. append only. duplicated data
* Primary Data
	* the source of truth
* Index
	* data structure for speed up to retrieval
	* or extra data file for that purpose

Hash
* in memory data structure. read/write O(1)
* keep the key and value that represent byte offset in the log file
* need enough memories to store all keys and slow for range query

Log Segment
* fragments of logs divided by specific conditions (size, time, ...)
* compaction : leave only the most recent data in segement
* merge : combine multiple segments as one

SSTable (Sorted-String Table)
* A segment file that have entries sorted by key and also include offset index of those.
* handle large amount of data. (disk merge sort)
* sparse index (small memory)
* data compression (smapl diskspace and bandwith)

LSM Tree (Log-Structured Merge-Tree)
* memtable(sorted in-memory data structure) + sstable(disk)
* handle random write operation as sorted data set
* memtrable is flushed when fixed size is over
* bloom filter ()
## Page Oriented

B-Tree
* read/write with fixed size page(block)
* each page have range of keys (sorted) and references of other pages
* internal page(), leaf page(actual key and values)
* branching factor : count of references in a single page
* balanced tree ()
## Comparation
|  | Log Structured | Page Orignted |
| ---- | ---- | ---- |
| write amplification |  | O |
| write speed | O |  |
| write throughput | O |  |
| compression rate | O |  |
| size |  | O |
| fragmemtation |  | O |
|  |  |  |
| compaction (background) | O |  |
| disk usage - predictable |  | O |
| disk usage - max | O |  |
|  |  |  |
| trasaction |  | O |
|  |  |  |




## Other Index Types

primary key index : unique id
secondary index : non-unique id
clustered index : key with actual data
non-clustered index : heap file(have all values). fast update and avoid data duplication in index
covering index : indexed key + other colums
concatenated index : ordered and combined colums
full-text index : reversed index
fuzzy index : edit distance automata
in-memory database :

# Data Model

## Relational Model


* data = table (row + column)
* tables (relations, entity type)
* columns (attributes, values)
* rows (tuples, instances)
* join : data consistency, normalize(no duplication)
* fixed, hard to scale-out
