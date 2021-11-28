```java
import java.lang.reflect.Constructor;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class Context {
private Map<String, Object> els = new HashMap<String, Object>();

    public void reg(Class cl) {
        Constructor[] constructors = cl.getDeclaredConstructors();
        if (constructors.length > 1) {
            throw new IllegalStateException("Class has multiple constructors : " + cl.getCanonicalName());
        }
        Constructor con = constructors[0];
        List<Object> args = new ArrayList<Object>();
        for (Class arg : con.getParameterTypes()) {
            if (!els.containsKey(arg.getCanonicalName())) {
                throw new IllegalStateException("Object doesn't found in context : " + arg.getCanonicalName());
            }
            args.add(els.get(arg.getCanonicalName()));
        }
        try {
            els.put(cl.getCanonicalName(), con.newInstance(args.toArray()));
        } catch (Exception e) {
            throw new IllegalStateException("Coun't create an instance of : " + cl.getCanonicalName(), e);
        }
    }

    public <T> T get(Class<T> inst) {
        return (T) els.get(inst.getCanonicalName());
    }
}
```
Разберем этот класс.

1. Карта с объектами. В ней мы будем хранить проинициализированные объекты.
```java
   private Map<String, Object> els = new HashMap<String, Object>();
```
2. Метод get возвращает проинициализированный объект.
```java
public <T> T get(Class<T> inst) {
```
Через рефлексию можно получить имя класса.
```java
inst.getCanonicalName()
```
3. Метод регистрации классов.
```java
public void reg(Class cl) {
```
Сначала нужно получить все конструкторы класса. Если их больше 1, то мы не знаем как загружать этот класс и кидаем исключение.
```java
        Constructor[] constructors = cl.getDeclaredConstructors();
        if (constructors.length > 1) {
            throw new IllegalStateException("Class has multiple constructors : " + cl.getCanonicalName());
        }
```
Когда мы нашли конструктор, мы собираем аргументы этого конструктора и ищем уже зарегистрированные объекты, чтобы внедрить их в конструтор.
```java
        Constructor con = constructors[0];
        List<Object> args = new ArrayList<Object>();
        for (Class arg : con.getParameterTypes()) {
            if (!els.containsKey(arg.getCanonicalName())) {
                throw new IllegalStateException("Object doesn't found in context : " + arg.getCanonicalName());
            }
            args.add(els.get(arg.getCanonicalName()));
        }
```
Последний этап - это создание объекта и добавление его в карту.
```java
els.put(cl.getCanonicalName(), con.newInstance(args.toArray()));
```

Создадим класс ru.job4j.di.Main. В нем попробуем использовать наш DI.

```java
public class Main {
public static void main(String[] args) {
Context context = new Context();
context.reg(Store.class);
context.reg(StartUI.class);

        StartUI ui = context.get(StartUI.class);

        ui.add("Petr Arsentev");
        ui.add("Ivan ivanov");
        ui.print();
    }
}
```
Здесь видно, что мы сами не создаем объекты классов Store StartUI за нас это делает Context.