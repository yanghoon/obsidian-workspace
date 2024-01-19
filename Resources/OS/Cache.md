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
	* Eviction Policy
* Cache Strategies
	* Cache-Aside (lazy loading, cache first)
	* Write-Through (fresh, latency)
	* Write-Back (lazy write, data loss)