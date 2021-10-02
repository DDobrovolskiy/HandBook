Links: https://habr.com/ru/post/347752/

JDK dynamic proxy vs CGLib proxy

Если обратиться к документации то можно найти там следующий текст.
Spring AOP uses either JDK dynamic proxies or CGLIB to create the proxy for a given target object. (JDK dynamic proxies are preferred whenever you have a choice).
If the target object to be proxied implements at least one interface then a JDK dynamic proxy will be used. All of the interfaces implemented by the target type will be proxied. If the target object does not implement any interfaces then a CGLIB proxy will be created.
If you want to force the use of CGLIB proxying (for example, to proxy every method defined for the target object, not just those implemented by its interfaces) you can do so.
To force the use of CGLIB proxies set the value of the proxy-target-class attribute of the <aop:config> element to true
Т.е., согласно документации, для создания прокси объектов может использоваться как JDK так и CGLib, но предпочтение должно отдаваться JDK. И, если класс имеет хотя бы один интерфейс, то именно JDK dynamic proxy и будет использоваться (хотя это можно изменить, явно задав флаг proxy-target-class). При создании прокси объекта с помощью JDK на вход передаются все интерфейсы класса и метод для имплементации нового поведения. В результате получаем объект, который абсолютно точно реализует паттерн Proxy. Все это происходит на этапе создания бинов, поэтому, когда начинается внедрение зависимостей, то в реальности внедрен будет этот самый прокси-объект. И все обращения будут производиться именно к нему. Но выполнив свою часть функционала, он обратиться к объекту исходного класса и передаст ему управление. Если же этот объект сам обратиться к одному из своих методов, то это будет уже прямой вызов без всяких прокси. Собственно именно это поведение и есть то, которое ожидается согласно правильному* ответу.

С этим вроде все понятно, а что же с CGLib? Ведь он создает на самом деле не прокси объект, а наследника класса. И вот тут мой мозг уже просто кричит: СТОП! Ведь тут мы имеем ну просто пример из учебника по ООП.
```java
public class SampleParent {

public void method1() {
System.out.println("SampleParent.method1");
method2();
}

public void method2() {
System.out.println("SampleParent.method2");
}

}
public class SampleChild extends SampleParent {

@Override
public void method2() {
System.out.println("SampleChild.method2");
}

}
```
Где SampleChild – это, по сути, наш прокси-объект. И вот тут я уже начинаю сомневаться даже в ООП (или в своих знаниях о нём). Ведь если мы имеем наследование, то перекрытый метод должен вызываться вместо родительского и тогда поведение будет отличаться от того что мы имеем при использовании JDK dynamic proxy.

Хотя есть еще один вариант, возможно, я неправильно понял, как создаются объекты с помощью CGLib и, на самом деле, они «не совсем наследники», а тоже «какие-будь прокси». И, конечно же, самый простой способ быть уверенным хоть в чем-нибудь – это проверить на простом примере. Вот и создадим еще один маленький проектик.

На этот раз нам Spring уже не нужен, просто создаем простейший maven-проект и добавляем в pom зависимость на CGLib (собственно это и есть всё содержание нашего pom-файла).
```xml
  <dependencies>
  <dependency>
      <groupId>cglib</groupId>
      <artifactId>cglib</artifactId>
      <version>3.2.5</version>
  </dependency>
  </dependencies>
```
Добавим в созданный проект наши два Sample-класса (ну так, на всякий случай, что бы убедить себя, что всё-таки принципы ООП незыблемы) и собственно класс с main() методом, в котором и будем выполнять наши тесты.

```java
package com.example.CGLIBTest;

import java.lang.reflect.Method;

import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

public class CGLIBTestApp {

public static void main(String[] args) {
new SampleChild().method1();

    System.out.println("//------------------------------------------//");
    
    Enhancer enhancer = new Enhancer();
    enhancer.setSuperclass(SampleParent.class);
    enhancer.setCallback(new MethodInterceptor() {
      public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy)
            throws Throwable {
          if(method.getName().equals("method2")) {
            System.out.println("SampleProxy.method2");
            return null;
          } else {
            return proxy.invokeSuper(obj, args);
          }
        }
    });
    ((SampleParent) enhancer.create()).method1();

}

}
```
Первой строчкой вызываем method1() у объекта SampleChild класса (как уже сказал, ну просто что бы быть уверенным…) и далее создаем Enchancer. В объекте Enchancer переопределяем поведение метода method2(). После чего, собственно, создается новый объект и уже у него вызываем method1(). И запускаем.
```
SampleParent.method1
SampleChild.method2
//------------------------------------------//
SampleParent.method1
SampleProxy.method2
```
Фух… Можно выдохнуть. Если верить первым двум строкам вывода, то за последние 20 лет в ООП ничего не изменилось.