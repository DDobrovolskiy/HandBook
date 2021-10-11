```properties
spring.main.lazy-initialization=true
```


Package code
```java
@Component
public class Person {

    @Autowired
    private Message message;
    @Autowired
    @ITwoAn
    private Inter inter;

    public Person() {
        System.out.println("New Person");
    }
    
    public void print() {
        System.out.println(inter.getName());
    }
}
```
```java
@Component
@Scope("prototype")
public class Message {
    private String hello = "Hello";

    public Message() {
        System.out.println("Create Message");
    }
}
```
```java
public interface Inter {
    String getName();
}
```
```java
@Component("iTwo")
public class ITwo implements Inter {
    @IOneAn
    private Inter inter; //Lazy!!!

    public ITwo() {
        System.out.println("Create ITwo");
    }

    @Override
    public String getName() {
        return "ITwo";
    }
}
```
```java
@Component("iOne")
public class IOne implements Inter {

    public IOne() {
        System.out.println("Create IOne");
    }

    @Override
    public String getName() {
        return "IOne";
    }
}
```
Package Test
```java
@Configuration
@ComponentScan(lazyInit = true)
public class ConfigurationTest {
}
```
```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = ConfigurationTest.class)
public class PersonTest {
    @Autowired
    private Person person;

    @Test
    public void test() {
        person.print();
    }
}
```
Return:  
...  
~~Create IOne~~  
Create ITwo  
New Person  
Create Message  
...  
ITwo  