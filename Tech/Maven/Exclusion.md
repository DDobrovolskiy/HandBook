Исключить зависимости, транзитивные
```xml
<dependency>
            <groupId>test</groupId>
            <artifactId>test</artifactId>
            <version>1.2.17</version>
            <exclusions>
                <exclusion>
                    <groupId>test</groupId>
                    <artifactId>test</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```