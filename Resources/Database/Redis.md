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

# Management

```bash
docker run -d --name my-redis -p 6379:6379 redis
```

```yaml
version: '3'

services:
  redis:
    # name: my-redis
    image: redis:latest
    ports:
      - 6379:6379
    # volumes:
    #  - ./data/redis:/etc/redis
```
