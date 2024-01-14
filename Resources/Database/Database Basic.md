# Storage

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
* Segment file that have sorted entries by key and 

LSM Tree (Log-Structured Merge-Tree)