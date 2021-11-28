Link: https://www.youtube.com/watch?v=cou_qomYLNU&t=12s&ab_channel=JUG.ru
```java
@Component
public class PeriodicalScopeConfigure implements Scope {
    @Override
    public Object get(String name, ObjectFactory<> ObjectFactory) {
        return ...(logic);
    }
}
```
registration:
```java
public class Foo implements BeanFactoryPostProcessor {
    public void postProcessorBeanFactory(ConfigurationListerinBeanFactory factory) {
        factory.registerScope("nameScope", Foo.class);
    }
}
```