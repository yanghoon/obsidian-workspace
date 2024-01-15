# Concept

* Remote Dictionary Server
* In-memory storage ~~database~~
* Fast than SSD/HDD
* Used to
	* distributed cache, lock, pub/sub(message queue), stream
	* eg. session store, limit rater, job queue
* Key-value data-model
	* hash(fast), partitioning
	* poor range query, no duplication
