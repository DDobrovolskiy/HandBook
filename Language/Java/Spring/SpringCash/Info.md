```java
@SpringBootApplication
@EnableCaching    //подключение Spring Cache
public class CacheApplication {

    public static void main(String[] args) {
        SpringApplication.run(CacheApplication.class, args);
    }

}
```

```java
@Override
@Cacheable("users")
public User get(Long id) {
        log.info("getting user by id: {}", id);
        return repository.findById(id)
                .orElseThrow(() -> new EntityNotFoundException("User not found by id " + id));
    }
```

Links:
* https://habr.com/ru/post/465667/
* https://www.baeldung.com/spring-cache-tutorial

GitHub:
* https://github.com/promoscow/cache