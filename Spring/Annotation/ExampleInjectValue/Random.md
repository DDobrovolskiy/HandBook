Link: https://habr.com/ru/post/222579/

На первом этапе будет создана аннотация, которой будут помечаться поля класса, в которые нужно проинжектить значение.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface InjectRandomInt {
int min() default 0;
int max() default 10;
}
```

По умолчанию, диапазон случайных чисел будет от 0 до 10.

Затем, нужно создать обработчик этой аннотации, а именно реализацию BeanPostProcessor для обработки аннотации InjectRandomInt.
```java
@Component
public class InjectRandomIntBeanPostProcessor implements BeanPostProcessor {

    private static final Logger LOGGER = LoggerFactory.getLogger(InjectRandomIntBeanPostProcessor.class);

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {

        LOGGER.info("postProcessBeforeInitialization::beanName = {}, beanClass = {}", beanName, bean.getClass().getSimpleName());

        Field[] fields = bean.getClass().getDeclaredFields();

        for (Field field : fields) {
            if (field.isAnnotationPresent(InjectRandomInt.class)) {
                field.setAccessible(true);
                InjectRandomInt annotation = field.getAnnotation(InjectRandomInt.class);
                ReflectionUtils.setField(field, bean, getRandomIntInRange(annotation.min(), annotation.max()));
            }
        }

        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    private int getRandomIntInRange(int min, int max) {
        return min + (int)(Math.random() * ((max - min) + 1));
    }
}
```

Код данного BeanPostProcessor достаточно прозрачен, поэтому мы не будем на нем останавливаться, но тут есть один важный момент.

BeanPostProcessor обязательно должен быть бином, поэтому мы его либо помечаем аннотацией @Component, либо регестрируем его в xml конфигурации как обычный бин.

Первая группа разработчиков свою задачу выполнила. Теперь вторая группа может использовать эти наработки.
```java
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class MyBean {

    @InjectRandomInt
    private int value1;

    @InjectRandomInt(min = 100, max = 200)
    private int value2;

    private int value3;

    @Override
    public String toString() {
        return "MyBean{" +
                "value1=" + value1 +
                ", value2=" + value2 +
                ", value3=" + value3 +
                '}';
    }
}
```

В итоге все бины типа MyBean, получаемые из контекста, будут создаваться с уже проинициализированными полями value1 и value2. Также тут стоить отметить, этап на котором будет происходить инжект значений в эти поля будет зависеть от того какой @ Scope у вашего бина. SCOPE_SINGLETON — инициализация произойдет один раз на этапе поднятия контекста. SCOPE_PROTOTYPE — инициализация будет выполняться каждый раз по запросу. Причем во втором случае ваш бин будет проходить через все BeanPostProcessor-ы что может значительно ударить по производительности.