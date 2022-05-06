Класс синглетон:

``` java
@Component
public class WriterService {

    @Lookup
    public Writer getPrototypeBean(String str) {
        return new Writer(str);
    }

    public void test() {
        System.out.println(getPrototypeBean(LocalDateTime.now().toString()));
    }
}
```
Класс прототайп

``` java
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class Writer {
    private String data;
    @Autowired
    private Collector collector;

    public Writer(String data) {
        this.data = data;
    }

    @Override
    public String toString() {
        return "Writer{" +
                "data='" + data + '\'' +
                ", collector=" + collector +
                '}';
    }
}
```

Еще один класс прототайп

``` java
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
@Data
public class Collector {
    private LocalDateTime data = LocalDateTime.now();

    @Override
    public String toString() {
        return "Collector{" +
                "data=" + data.toString() +
                '}';
    }
}
```

Проверить:
``` java
@SpringBootApplication(scanBasePackageClasses = ConfigurationLazy.class)
public class App {
    public static void main(String[] args) throws InterruptedException {
        var contex = SpringApplication.run(App.class, args);
        var bean = contex.getBean(WriterService.class);
        bean.test();
        Thread.sleep(1000);
        bean.test();
        contex.stop();
    }
}
```
