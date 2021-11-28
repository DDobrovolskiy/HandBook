```xml
<packaging>war</packaging>
```
Запуск сервера jetty:run
```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.eclipse.jetty</groupId>
                <artifactId>jetty-maven-plugin</artifactId>
                <version>11.0.7</version>
            </plugin>
        </plugins>
    </build>
```