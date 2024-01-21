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
	* everysec : like rdb
	* no : by OS, fast, not safe

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