---
title: Spring boot 에서 Multi cache manger 사용하기
date: 2023-01-13
description: "Redis, Caffeine cache Manger"
tags: [Spring boot, Redis, cache, caffeine]
thumbnail: ""
---

## Approach

Spring boot 에서 각기 다른 host 를 가진 Redis 에 캐싱을 해야 한다던지
Local cahce 와 Redis cache 를 각각 handling 해야 할때 사용 하면 좋을 것 같아 글로 적어 보려 합니다.

```gradle
implementation("org.springframework.boot:spring-boot-starter-data-redis")
implementation("org.springframework.boot:spring-boot-starter-web")
implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
implementation("org.jetbrains.kotlin:kotlin-reflect")
implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8")
implementation("com.github.ben-manes.caffeine:caffeine:3.1.2")
testImplementation("org.springframework.boot:spring-boot-starter-test")
```

Redis 와 local cache 에는 Concurent hash map 보다 Read/Write 에서 성능이 좋은 Caffeine cache 를 사용할 예정입니다.


## Bean 구성
```kotlin
@Configuration
class CacheMangerConfiguration(
    @Value("\${redis.host}") private val redisHost: String,
    @Value("\${redis.port}") private val redisPort: Int,
) : CachingConfigurerSupport() {

    @Bean(name = ["cacheManager"])
    @Primary
    override fun cacheManager(): CacheManager {
        val builder = RedisCacheManager.RedisCacheManagerBuilder.fromConnectionFactory((redisConnectionFactory()))
        val configuration = RedisCacheConfiguration.defaultCacheConfig()
            .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(StringRedisSerializer()))
            .entryTtl(Duration.ofMinutes(5))
        builder.cacheDefaults(configuration)
        return builder.build()
    }

    @Bean(name = ["caffeineCacheManager"])
    fun caffeineCacheManager(): CacheManager {
        val cacheManager = CaffeineCacheManager("customers", "orders")
        cacheManager.setCaffeine(
            Caffeine.newBuilder()
                .initialCapacity(200)
                .maximumSize(500)
                .weakKeys()
                .recordStats()
        )
        return cacheManager
    }

    @Bean
    fun redisConnectionFactory(): RedisConnectionFactory {
        val configuration = RedisStandaloneConfiguration(redisHost, redisPort)
        return object : LettuceConnectionFactory(configuration) {
            override fun afterPropertiesSet() {
                super.afterPropertiesSet()
                this.connection
            }
        }
    }
}
```

@Primary annotation 으로 Redis cacheManger 를 Default 로 가져갑니다 CaffeineCacheManger 가 필요한곳에서는 cacheManger = 를 사용하여 선언합니다.

## @Cacheable 적용하기

<br />

```kotlin
@RestController
class HelloController {

    @Cacheable("hello", key = "#name")
    @GetMapping("/hello")
    fun hello(@RequestParam("name") name: String): String {
        println("hello, $name")
        return "hello, $name"
    }

    @Cacheable("order", key = "#id", cacheNames = ["order"], cacheManger = "caffeineCacheManager")
    @GetMapping("/orders")
    fun order(@RequestParam("id") id: Int): String {
        println("Order completed id: $id")
        return "Order completed id: $id"
    }
}
```

hello method 와 order method 를 선언 했습니다
hello method 는 cacheManger 를 선언하지 않아 Primary CacheManger (Redis) 로 cache 가 되고
order method 는 caffeineCacheManger 를 사용하여 local cache 가 됩니다.

key 가 같을 경우 cache 된 값을 return 하게 되어 println 가 찍히지 않습니다.

## 직접 CacheManger 를 지정하기

<br />

```kotlin
class MultipleCacheResolver(
    private val cacheManger: CacheManager,
    private val caffeineCacheManager: CacheManager
) : CacheResolver {
    override fun resolveCaches(context: CacheOperationInvocationContext<*>): List<Cache> {
        val caches = mutableListOf<Cache?>()
        if ("order" == context.method.name) {
            caches.add(caffeineCacheManager.getCache("customers"))
        } else {
            caches.add(cacheManger.getCache("cacheManager"))
        }
        return caches.filterNotNull()
    }
}
```

```kotlin
@Configuration
class CacheMangerConfiguration (
    ...
    @Bean
    override fun cacheResolver(): CacheResolver {
        return MultipleCacheResolver(cacheManager(), caffeineCacheManager())
    }
}
```

method 별로 필요한 cache manger 를 handling 할 수도 있습니다.