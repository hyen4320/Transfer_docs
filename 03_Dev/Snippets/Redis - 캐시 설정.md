---
tags: [snippet, redis, spring]
---

# Redis - Spring Cache 설정

```java
@Configuration
@EnableCaching
public class RedisConfig {

    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory factory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(10))
                .serializeValuesWith(
                    RedisSerializationContext.SerializationPair
                        .fromSerializer(new GenericJackson2JsonRedisSerializer())
                );
        return RedisCacheManager.builder(factory)
                .cacheDefaults(config)
                .build();
    }
}
```

```java
// 사용 예시
@Cacheable(value = "journalist:ranking", key = "'all'")
public List<JournalistDto> getRanking() { ... }

@CacheEvict(value = "journalist:ranking", allEntries = true)
public void updateScore(Long journalistId) { ... }
```

관련: [[tech-stack]], [[002 - Redis 캐시 전략]]
