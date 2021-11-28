#Spring Cloud  
Создадим модуль конфигурационного сервера на основе Spring Boot приложения:
```java
@EnableConfigServer
@SpringBootApplication
public class ConfigServerApplication {
public static void main(String[] args) {
SpringApplication.run(ConfigServerApplication.class, args);
}
}
```
Аннотация @EnableConfigServer делает все дело, превращая простейшее Spring Boot приложение в
сервер раздачи конфигураций. Под капотом @EnableConfigServer скрывается то, как зависимость
подтягивает Tomcat (с возможностью выбора другого контейнера) и сервлет, отдающий конфигурации
по HTTP-протоколу.
Нам потребуются следующие зависимости:
```xml
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-config-server</artifactId>
</dependency>
```
В свойствах модуля необходимо указать версию Spring Cloud:
```xml
<properties>
...
<spring-cloud.version>Dalston.SR3</spring-cloud.version>
</properties>
```
Файл свойств модуля (application.properties):
```properties
# по умолчанию клиент обращается 8888 порт
server.port=8888
# локальный вариант хранения (без Git-репозитория)
# spring.profiles.active=native
# путь к ней внутри classpath.
# реально конфиги будут деплоиться в облако вместе с сервисом
# просто и понятно, но оборотной стороной будет
# потребность в редеплое приложения при изменениях настроек
# spring.cloud.config.server.native.search-locations=classpath:/config
# используем Git, в облаке так удобнее, не требуется редеплой, только рестарт
spring.cloud.config.server.git.uri=${GIT_URI}
spring.cloud.config.server.git.search-paths=${GIT_SEARCH_PATHS}
```
В Git-репозитории (локальном или удаленном) необходимо разместить (add) и закоммитить (commit)
файл конфигурации:
```properties
# указываем конфигурационный параметр приложения, в данном случае — строку
config.name=Config Server App From Git application.properties
```
Запускаем сервер конфигураций.
