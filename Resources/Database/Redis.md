#cache
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

### Docker Compose (Replication)

```yaml
version: '3'

services:
  redis-master:
    image: bitnami/redis:latest
    container_name: redis-master
    environments
      - REDIS_REPLICATION_MODE=master
      # - ALLOW_EMPTY_PASSWORD=yes
    ports:
      - 6379:6379
    # volumes:
    #  - ./data/redis:/etc/redis
  redis-replica-1:
    image: bitnami/redis:latest
    container_name: redis-replica-1
    environments
      - REDIS_REPLICATION_MODE=slave
      - REDIS_MASTER_HOST=redis-master
      # - ALLOW_EMPTY_PASSWORD=yes
    ports:
      - 6380:6379
```
### Docker Compose (Sentinel)

```

```yaml
version: '3'

services:
  redis-master:
    image: bitnami/redis:latest
    container_name: redis-master
    environments
      - REDIS_REPLICATION_MODE=master
      # - ALLOW_EMPTY_PASSWORD=yes
    ports:
      - 6379:6379
    # volumes:
    #  - ./data/redis:/etc/redis
  redis-replica-1:
    image: bitnami/redis:latest
    container_name: redis-replica-1
    environments
      - REDIS_REPLICATION_MODE=slave
      - REDIS_MASTER_HOST=redis-master
      # - ALLOW_EMPTY_PASSWORD=yes
    ports:
      - 6380:6379
  redis-sentinel-1:
    image: bitnami/redis-sentinel:latest
    container_name: sentinel1
    environment:
      - REDIS_MASTER_HOST=redis-master
      - REDIS_MASTER_PORT=6379
      - REDIS_MASTER_SET=mymaster
      - REDIS_SENTINEL_QUORUM=2
      - REDIS_SENTINEL_DOWN_AFTER_MILLISECONDS=3000
    ports:
      - 26379:26379
    depends_on:
      - redis-master
      - redis-replica-1
  redis-sentinel-2:
    image: bitnami/redis-sentinel:latest
    container_name: sentinel2
    environment:
      - REDIS_MASTER_HOST=redis-master
      - REDIS_MASTER_PORT=6379
      - REDIS_MASTER_SET=mymaster
      - REDIS_SENTINEL_QUORUM=2
      - REDIS_SENTINEL_DOWN_AFTER_MILLISECONDS=3000
    ports:
      - 26380:26379
    depends_on:
      - redis-master
      - redis-replica-1
  redis-sentinel-3:
    image: bitnami/redis-sentinel:latest
    container_name: sentinel3
    environment:
      - REDIS_MASTER_HOST=redis-master
      - REDIS_MASTER_PORT=6379
      - REDIS_MASTER_SET=mymaster
      - REDIS_SENTINEL_QUORUM=2
      - REDIS_SENTINEL_DOWN_AFTER_MILLISECONDS=3000
    ports:
      - 26381:26379
    depends_on:
      - redis-master
      - redis-replica-1
```
## Java

Spring Data Redis
* Redis client: Lettuce
* StringRedisTemplate, ValueOperations<?, ?>

```java
/* buidl.gradle */
dependencies {
  implementation 'org.springframework.boot:spring-boot-starter-data-redis'
  implementation 'org.springframework.session:spring-session-data-redis'
}

/* application.yaml */
spring:
  redis:
    host: localhost
    port: 6379

/* Java Code */
ValueOperations<String, String> ops = StringRestTemplate.opsForValue();
ops.set("key", "val");
ops.get("key"); // val
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

ZCOUNT key1 0 100 # score_min  score_max
ZRANGE key1 0 100 # index_from index_to

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
# Pub/Sub
* Message loss, push to all online, at-most-once

# Redis Streams
## Event-Driven
* Service
	* Each application, own release, **same database**
	* Database (join)
* Micsoservice
	* Each application, own release, **each database**
	* API (gateway, rpc)
	* Distribution problems
		* consistency (transaction)
		* complexity (testing, monitoring)
* Low coupling/high cohesion (only message broker), scalable, elastic
* Hard to trace, test, debug

## Concept
* append-only (log)
* data layout
	* `stream = (key, [entry])`
	* `entry = (id, [field: value, field: value, ...])`
* Consumer group
	* consumers in the same group read distinct entries

```bash
# XADD [key] [id] [field-value]
redis> XADD user-notifications * user-a hi user-b hello
"<timestamp>-<version-count>" # entry id

# XRANGE [key] [start] [end]
redis> XRANGE user-notifications - + # find all
1) 1) "<timestamp>-version-count>" # entry, entry id
   2) 1) "user-a" # values, field
      2) "hi"     # value
      3) "user-b" # field
      4) "hello"  # value
2) 1) "<timestamp>-version-count>" # entry, entry id
   2) 1) "user-c" # values, field
      2) "nice"   # value
```

```bash
# XREAD BLOCK [milliseconds] STREAMS [key] [id]
redis> XREAD BLOCK 0 STREAMS user-notifications 0
1) 1) "user-notifications" # stream key
   2) 1) 1) "<timestamp>-version-count>" # stream, entry, entry id
         2) 1) "user-a" # values, field
            2) "hi"     # value
            3) "user-b" # field
            4) "hello"  # value
      2) 1) "<timestamp>-version-count>" # entry, entry id
         2) 1) "user-c" # values, field
            2) "nice"   # value

# Subscribe new entries ($: from now on)
redis> XREAD BLOCK 0 STREAMS user-notifications $
```

```bash
# XGROUP CREATE [key] [group-name] [id] ["MKSTREAM"]
redis> XGROUP CREATE user-notifications group1 $

# XREADGROUP GROUP [group-name] [consumer-anme] COUNT [count] STREAMS [key] [id]
# (>: not yet read by another consumer)
redis> XREADGROUP GROUP group1 consumer1 COUNT 1 STREAMS user-notifications >
```
# Persistence
## RDB (Redis Database)
* Snapshot Backup (eg. dump.rdb)
* Schedule or Manual
* At child process (fork), small size, fast recovery
* Data loss, Durability(crash during backup), Unpredictability(high cpu usage, long execution time)

```bash
# Config Template (https://redis.io/docs/management/config/)
# redis.conf

# Redis Database (Snapshotting)
# every 60s or 10 items changed
save 60 10

dbfilename dump.rdb
```

```bash
# redis-cli
redis> bgsave
Background saving started
```
## AOF (Append Only File)
* Log, replay to recovery, human readable(not matched memory)
* Large size(duplication, not only latest), lower speed backup/recovery
* fsync flag (OS)
	* always : safe, slow
	* everysec : like rdb (recommand)
	* no : by OS, fast, not safe
* Tuning
	* Log rewriting(compaction)
	* Multi Part AOF
		* base (latest rewrite, rdb)
		* incremental(changed from base)
		* manifest(files metadata)

```systemd
save "" # disable rdb

appendonly yes
```

## Replication
* Scalability, Read Throughput, Availability
* Single leader replication
* Configure at replica node
* MUST enable persistence of msater node (replicate empty data)

```bash
replicaof 127.0.0.1 6379

# ** Redis Logs **
# Master  - Synchronization with replica <ip>:<port> succeeded
# Replica - MASTER <-> REPLICA sync started
# Replica - MASTER <-> REPLICA sync: Finished with success
```
## Redis Sentinel
* Auto-failover daemon
* Monitoring, alerts, auto-failover, redis proxy
* Quorum (over 3 nodes)
	* SDOWN(Subject down) : by 1 sentinel node
	* ODOWN(Objective down) : by quorum sentinel nodes

![[Pasted image 20240121121509.png]]

```bash
# redis-cli -p 26379
redis> info sentinel
# Sentinel
sentinel_master:1
sentinel_tilt:0
sentinel_tilt_since_seconds:-1
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,adderess=<ip>:6379,slaves=1,sentinels=3
```

```java
/* application.yaml */
spring:
  redis:
    sentinel:
      master: mymaster
      nodes: 127.0.0.1:26379,127.0.0.1:26380,127.0.0.1:26381
```
# Redis Cluster

> Scalability
> * Ability to handle incresed loads (with more resources)
> * Scale-Out
> 	* multiple node, distributed system
> 	* partial failure(network), eventual consistency, load balancing(discovery), complexity
## Concept
* Replication(async), scale-out, auto-failover
* Availability/partition tolerance > consistency
* Characteristic
	* full-mesh, cluster-bus (port)
	* gossip protocol (broadcast optimize)
	* hash slot (partition key, 16k entries, 14 bits)
	* DB0 (vs 16 database)
	* limited multi key operations
		* hash tags
		* `MSET {tag:val}:key1 val1 {tag:v}:key2 val2`
	* request redirect (no proxy)
		* client should know/connect all servers
		* `MOVED <hash-slot> <ip>:<port>`
* vs Sentinel
	* no partitioning
	* no monitoring node
	* do multi key operations
	* for small scale
## Consistent Hashing
* node counts (key  % n) : changed new or remove node
* hash slot : fixed hash lengh (16384)
* pseudo code (TODO)
## Auto-Failover
* Quonum master group + Has replica of failure master
* Another replica is migrated
## Configure
### Config

```bash
cluster-enabled      <yes/no>            # (default: no)
cluster-config-file  <filename>
cluster-node-timeout <milliseconds>
cluster-replica-validity-factor <factor> # cluster-node-timeout * factor
cluster-migration-barrier     <count>    # minimum replica count of each master
cluster-require-full-coverage <yes/no>   # (default: yes)
                                        # allow write when all hash slot is not healthy
cluster-all-reads-when-down   <yes/no>   # (default: no)
                                        # allow read when all hash slot is not healthy 
```
### Commands

```bash
redis-cli \
	--cluster create \
		localhost:7000 localhost:7001 localhost:7002 \
		localhost:7003 localhost:7004 localhost:7005 \
	--cluster-replicas 1
# >>> Performing hash slots allocation on 6 nodes...
# Master[0] -> Slots 0 - 5460
# Master[1] -> Slots 5461 - 10922
# Master[2] -> Slots 10923 - 16383
# Adding replica localhost:7004 to localhost:7000
# Adding replica localhost:7005 to localhost:7001
# Adding replica localhost:7003 to localhost:7002
# ...
# [OK] All nodes agree about slots configurations.
# ...
# [OK] all 16384 slots covered.
```

```bash
# redis-cli -p 7000
redis> cluster nodes
# hash-key-of-node, ip-port-of-node
redis> set aa bb
OK
redis> set aaa dd
(error) MODED 10439 loclahost:7001

# redis-cli -p 7001
redis> set aaa dd
OK

# redis-cli -p 7003
redis> set aaa dd
(error) MODED 10439 loclahost:7001
redis> get aaa
(error) MODED 10439 loclahost:7001
redis> readonly
OK
redis> get aaa
dd
```

```bash
redis-cli \
  --cluster add-node localhost:7006 localhost:7001

redis-cli \
  --cluster add-node localhost:7007 localhost:7006 \
  --cluster-slave
```
### Cluster with Spring

```yaml
spring:
  redis:
    cluster:
      nodes:
		redis:7000,redis:7001,redis:7002,redis:7003,redis:7004,redis:7005
```
# Performance
## Eviction
* Redis for cache
* allkeys-lru : basic for cache (recommend)
* allkeys-random : equal distribution
* volatile* : handle by application

```bash
maxmemory 100mb  # (default: 3GB - 32bit, 0 - 64bit)
maxmemory-policy noeviction
  # noeviction:   read(success), write(error)
  # allkeys-lru:  least recently used (oldest)
  # allkeys-lfu:  least frequently used (infrequently)
  # volatile-lru: LRU with only TTL
  # volatile-lfu: LFU with only TTL
  # allkeys-random: 
  # volatile-random: 
  # volatile-ttl: Remain short TTL with only TTL
```
## Tuning
* Testing
	* `redis-benchmark [-h host] [-p port] [-c clients] [-n requests] [-t command]`
	* metrics
		* throughput
		* avg, min, p50, p95, p99, max
	* run benchmark at remote client (for network)
* Parameters
	* Network bandwidth & latency
	* Cpu clock & cpu cache
	* RAM speed & bandwidth (over 10 KB data)
	* Network disks, IOPS, hypervisor versions
* Slowlog
	* without I/O time

```bash
rdbcompression <yes/no> # (default: yes) save disk usage. more cpu usage
rdbchecksum    <yes/no> # (default: yes) validation. 10% overhead
save <time> <change-counts> # Schedule of RBD

slowlog-log-slower-than 10000 # 10000 us = 10 ms
slowlog-max-len         128
```

```bash
# SHOWLOG
# redis-cli
redis> SLOWLOG LEN
(integer) 1
redis> SLOWLOG GET <index>
index) 1) (integer) <slowlog_sequence>
       2) (integer) <timestamp>
       3) (integer) <elapsed>
       4) 1) "COMMAND"
          2) "ARG1"
          3) "ARG2"
	   5) "client_ip:port"
	   6) "client_name"
redis> SLOWLOG RESET
```
# Use Case
## Spring Session

```java
/* build.gradle */
dependencies {
  implementation 'org.springframework.boot:spring-boot-web'
  implementation 'org.springframework.boot:spring-boot-data-redis'
  implementation 'org.springframework.session:spring-session-data-redis'
}

/* application.yaml */
spring:
  session:
    storage-type: redis

/* Controller.java */
@GetMapping
public void api(HttpSession session) { ... }
```
## RedisTemplate

```java
/* build.gradle */
dependencies {
  implementation 'org.springframework.boot:spring-boot-data-redis'
}

/* application.yaml */
spring:
  redis:
    host: localhost
    port: 6379

/* Service.java */
final int TIMEOUT = 5;
final StringRedisTemplate redisTemplate;

public String getData(String key) {
	// Cache-Aside
	// 1. Lookup Cache
	ValueOperations<String, String> ops = redisTemplate.opsForValue();
	String val = ops.get(key);
	
	// 2. Update Cache
	if (val == null) {
		val = getDataFromSource(key);
		ops.set("data:" + key, val, TIMEOUT, TimeUnit.SECONDS);
	}
	
	// 3. 
	return val;
}

private String getDataFromSource(String key) { ... }
```
## Spring Cache
* @Cacheable
* @CachePut
* @CacheEvict

```java
/* build.gradle */
dependencies {
  implementation 'org.springframework.boot:spring-boot-data-redis'
}

/* application.yaml */
spring:
  cache:
    type: redis
  redis:
    host: localhost
    port: 6379

/* CacheConfig.java */
@EnableCaching
@Configuration
public class CacheConfig {
	@Bean
	public CacheManager cacheManager(RedisConnectionFactory factory) {
		var defaultConfig = RedisCacheConfiguration.defaultConfig()
			.disabledCachingNullValues()
			.entryTtl(Duration.ofSeconds(10)) // default
			.computePrefixWith(CacheKeyPrefix.simple())
			.serializeKeyWith(
				RedisSerializationContext
					.SerializationPair
					.fromSerializer(new StringRedisSerializer())
			);
		
		var configMap = new HashMap<String, RedisCacheConfiguration>();
		configMap.put("data", RedisCacheConfiguration
			.defaultConfig()
			.entryTtl(Duration.ofSeconds(10))); // specific ttl
		
		return RedisCacheManager
				.RedisCacheManagerBuilder
				.fromConnectionFactory(factory)
				.cacheDefaults(defaultConfig)
				.withInitialCacheConfigurations(configMap)
				.build();
	}
}

/* Service.java */
final CacheService service;

public String getData(String key) {
	return service.getDataFromSource(key);
}

/* CachedService.java */
@Cacheable(cacheNames = "data", key = "#key") // data::{key}
private String getDataFromSource(String key) { ... }
```

## ZSetOperations (Sorted Set)

```java
/* build.gradle */
dependencies {
  implementation 'org.springframework.boot:spring-boot-data-redis'
}

/* Service.java */
final String KEY = "key";
final RedisTemplate redisTemplate;

public void setScore(String id, in socore) {
	ZSetOperations zops = redisTempalte.opsForZSet();
	zops.add(KEY, id, score);
}

public Set<String> getTopRank(int length) {
	ZSetOperations zops = redisTempalte.opsForZSet();
	// Set<String> rank = zops.reverseRange(KEY, 0, end - 1);
	// return new ArrayList<String>(rank);
	return zops.reverseRange(KEY, 0, length - 1); // desc, [start, end]
}

public Long getRanking(String id) {
	ZSetOperations zops = redisTempalte.opsForZSet();
	return zops.reverseRank(KEY, id); // desc
}
```
## Pub/Sub

```java
@Configuration
public class RedisConfig {
	// @MessageListenerAdapter
	@Bean
	RedisMessageListenerContainer redisContainer(RedisConnectionFactory factory) {
		var container = new RedisMessageListenerContainer();
		container.setConnectionFactory(factory);
		return container
	}
}

@Service
public class Service implements MessageListener {
	final RedisMessageListnerContainer container;
	final RedisTemplate<String, String> redisTemplate
	
	public void subscribe(String topic) {
		container.addMessageListener(this, new ChannelTopic(topic));
		
		Scanner in = new Scanner(System.in);
		while(in.hasNextLine()) {
			String line = in.nextLine();
			switch(line) {
				// redis> PUBLISH topic_key message
				case "q" -> break;
				default -> {
					redisTemplate.convertAndSend(topic, line);
				}
			}
		}
		
		container.removeMessageListener(this);
	}
	
	public void onMessage(Message message, byte[] pattern) {
		// SUBSCRIBE topic_key
		System.out.println("Message: " + message.toString());
	}
}

@SpringBootApplication
public class App implements CommandLineRunner {
	final Service service;
	
	public static vodi main(String[] args) {
		SpringApplication.run(App.class, args);
	}
	
	public void run(String... args) {
		service.subscribe("topic");
	}
}
```