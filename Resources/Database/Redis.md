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

## Docker

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
## Java

```gradle
dependencies {
  
}
```

# Data Type

### String
* binary-safe (null char, file, ...)
* Max 512MB

```bash
set key1 val1
get key1

incr key_int # (error) ERR value is not an integer or out of range
decr key_int

mset key1 val1 key2 val2
mget key1 key2 # throughput

```
### List
* Linked-list
* Queue, Stack
* read: O(n), add/remove: O(1)

```bash
LPUSH key1 val1 val2
RPUSH key1 val3

LLEN key1
LRANGE key1 0 -1 # LLEN -1

LPOP key1 1 # default: 1
RPOP key1 1 # default: 1
```

### Set
* Unordered unique values
* Support set operations
* Exactly once flag
* read/add/remove: O(1)

```bash
SADD key1 val1
SREM key1 val1

SCARD key1
SEMEMBERS key1
SISMEMBER key1 val1
```
### Hash
* Like object or map
* Counter
* Used for throughput of object style (vs string type with JSON)

```bash
HSET key1 k1 v1 k2 v2
HGET key1 k1
HDEL key1 k1 k2

HKEYS key1
HMGET key1 k1 k2

HINCRBY key1 k1 1 # default: 1
```
### Sorted Set
* with Score, sorted unique values by asc
* Rank

```bash
ZADD key1 score1 val1 score2 val2
ZREM key1 val1

ZRANGE key1 0 1 # 

ZRANK key1 val1    # asc
ZREVRANG key1 val1 # desc
```
### Bitmap
* Bit array (4,294,967,295 = 2^32-1)
* Bit operation

```bash
SETBIT key1 index [0 | 1]
GETBIT key1 index

BITCOUNT key1
BITOP [AND|OR|XOR] key_result key1 key2
```
### HyperLogLog
* cardinality, probabilistic unique count
* not store actual data. just count
* handle 2^64, max 12 KB, 0.81% error rate

```bash
PFADD key1 val1 val2 val3
PFCOUNT key1
PFMERGE key_result key1 key2
```