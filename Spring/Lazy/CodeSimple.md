```java
@Component
@Lazy
public class LazyClass {
    public LazyClass() {
        System.out.println("Create lazy class");
    }

    public void printYouMessage() {
        System.out.println("I`m lazy class!");
    }
}
```

```java
@Component
public class Person {
    @Autowired
    @Lazy
    private LazyClass lazyClass;

    public LazyClass getLazyClass() {
        return lazyClass;
    }
}
```

```java
@SpringBootApplication
public class ApplicationScope {
    public static void main(String[] args) {
        var context = SpringApplication.run(ApplicationScope.class, args);
        Person person = context.getBean(Person.class);
        person.getLazyClass().printYouMessage();
        context.close();
    }
}
```