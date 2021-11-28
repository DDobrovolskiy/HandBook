```
mvn clean packege -Pdev
```

```xml
    <profiles>
        <profile>
            <id>dev</id>
            <build>
                <resources>
                    <resource>
                        <directory>${basedir}/src/main/java/resource2</directory>
                    </resource>
                </resources>
            </build>
        </profile>
        <profile>
            <id>test</id>
            <build>
                <resources>
                    <resource>
                        <directory>${basedir}/src/main/java/resource3</directory>
                    </resource>
                </resources>
            </build>
        </profile>
    </profiles>
```

Либо использовать profiles.xml или settings.xml

```xml
<profilesXml xmlns="http://maven.apache.org/PROFILES/1.0.0">
    <profiles>
        <profile>
            <id>prod</id>
        </profile>
    </profiles>
</profilesXml>
```

Активирован по умолчанию
```xml
<profilesXml xmlns="http://maven.apache.org/PROFILES/1.0.0">
    <profiles>
        <profile>
            <id>prod</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>
    </profiles>
</profilesXml>
```

Активация по системе ОС
```xml
    <profiles>
        <profile>
            <id>prod</id>
            <activation>
                <os>
                    <family>unix</family>
                </os>
            </activation>
        </profile>
    </profiles>
```