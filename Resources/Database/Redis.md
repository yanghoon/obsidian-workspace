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

# Getting Started

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

```bash
# docker exec -it my-redis sh
# docker-compose exec redis sh

redis-cli # 127.0.0.1:6379
127.0.0.1:6379> set key1 banana
OK
127.0.0.1:6379> get key1
"banana"
127.0.0.1:6379> get key2
(nil)
127.0.0.1:6379> keys *
1) "key1"

127.0.0.1:6379> set key2 apple
OK
127.0.0.1:6379> dbsize
(integer) 2

127.0.0.1:6379> flushall
OK
127.0.0.1:6379> dbsize
(integer) 0
127.0.0.1:6379> get key1
(nil)
127.0.0.1:6379> get key2
(nil)
127.0.0.1:6379> exit
```
# Data Type

### String
* binary-safe (null char, file, ...)
* Max 512MB
* 