```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
```
```java
@SpringBootApplication
public class ApplicationScope {
    public static void main(String[] args) {
        var context = SpringApplication.run(ApplicationScope.class, args);
        Person person = context.getBean(Person.class);
        person.print();
    }
}
```
```java
@Aspect
@Component
public class MyAspect {
    @Before("execution(public void ru.job4j.scope.scope.Person.print())")
    public void getName() {
        System.out.println("ASPECT");
    }
}
```
```java
@Component
public class Person {
    public void print() {
        System.out.println(inter.getName());
    }
}
```