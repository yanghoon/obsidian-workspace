# Concept

* Remote Dictionary Server
* In-memory storage ~~database~~
* Faster than SSD/HDD
* Used to
	* distributed heap, cache, lock, pub/sub(message queue), stream
	* eg. session store, limit rater, job queue
* Key-value data-model
	* hash(fast), partitioning
	* poor range query, no duplication
* Data Type
	* String, Hash, Set, List
