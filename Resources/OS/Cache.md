# Concept
* Temporary storage to reduce latency between devices latencies
	* Reduce latencies, usage, bottleneck
	* Consistency problems (Invalidation)
* Examples
	* CPU : L1, L2, L3
	* Web Browser : Static resources, ETag
	* Database : query results
* Terms
	* Cache Hit
	* Cache Miss
	* Cache Strategies
		* Cache-Aside (lazy loading, cache first)
		* Read-Through
		* Write-Through (fresh, latency up)
		* Write-Back (lazy write, data loss)
		* Write-Around
	* Eviction Policy
		* TTL(Time-to-Live)
		* LRU(Least Recently Used) : Oldest
		* LFU(Least Frequently Used) : Most hits
		* FIFO(First in First out) : Queue