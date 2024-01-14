# Storage

* Log
	* data format. append only. duplicated data
* Primary Data
	* the source of truth
* Index
	* data structure for speed up to retrieval
	* or extra data file for that purpose

Hash
* in memory. read/write O(1)

Log Segment
* fragments of logs divided by specific conditions (size, time, ...)
* compaction