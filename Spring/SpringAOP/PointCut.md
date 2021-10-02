Links:
https://habr.com/ru/post/347752/

Наша аннотация (поскольку, как я уже сказал, нам интересны не транзакции, а то, как будет обрабатываться такая ситуация, то мы создаём свою аннотацию):
```java
package com.example.AOPTest;

import static java.lang.annotation.ElementType.METHOD;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

import java.lang.annotation.Retention;
import java.lang.annotation.Target;

@Retention(RUNTIME)
@Target(METHOD)
public @interface Annotation1 {
}
```
И аспект
```java
package com.example.AOPTest;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class MyAspect1 {

@Pointcut("@annotation(com.example.AOPTest.Annotation1)")
public void annotated() {}

@Before("annotated()")
public void printABit() {
System.out.println("Aspect1");
}
}
```
Собственно создали аспект, который будет привязываться к методам, имеющим аннотацию @Annotation1. Перед исполнением таких методов в консоль будет выводиться текст “Aspect1”.
Обращу внимание, что сам класс также аннотирован как Component. Это необходимо для того, что бы Spring мог найти этот класс и создать на его основе бин.

А теперь уже можно добавить и наш класс.
```java
package com.example.AOPTest;

import org.springframework.stereotype.Service;

@Service
public class MyServiceImpl {

@Annotation1
public void method1() {
System.out.println("method1");
method2();
}

@Annotation1
public void method2() {
System.out.println("method2");
}
}
```
С целями, задачами и инструментами определились. Можно приступать к экспериментам.
