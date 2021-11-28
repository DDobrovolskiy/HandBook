Откройте файл pom.xml и добавьте зависимости. JDBC - драйвер для работы с postgresql,

Spring-jdbc - это обертка для работы с JDBC.
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>${spring.version}</version>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.2.12</version>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-dbcp2</artifactId>
    <version>2.7.0</version>
</dependency>
```
Файл настроек app.properties

Создайте файл src/main/resources/app.properties
```properties
jdbc.url=jdbc:postgresql://127.0.0.1:5432/accident
jdbc.username=postgres
jdbc.password=password
jdbc.driver=org.postgresql.Driver
```
В нем будут настройки работы с базой.

Настройка пула

Создайте класс JdbcConfig.

В нем нужно создать бин, который будет содержать пул соединений.

package ru.job4j.accident.config;

```java
import org.apache.commons.dbcp2.BasicDataSource;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.transaction.annotation.EnableTransactionManagement;
import javax.sql.DataSource;

@Configuration
@PropertySource("classpath:app.properties")
@EnableTransactionManagement
public class JdbcConfig {

    @Bean
    public DataSource ds(@Value("${jdbc.driver}") String driver,
                         @Value("${jdbc.url}") String url,
                         @Value("${jdbc.username}") String username,
                         @Value("${jdbc.password}") String password) {
        BasicDataSource ds = new BasicDataSource();
        ds.setDriverClassName(driver);
        ds.setUrl(url);
        ds.setUsername(username);
        ds.setPassword(password);
        return ds;
    }

    @Bean
    public JdbcTemplate jdbc(DataSource ds) {
        return new JdbcTemplate(ds);
    }

}
```
Разберем этот код.

@PropertySource("classpath:app.properties")
- эта аннотация говорит Spring считать файл.
  Далее настройки можно получить через аннотацию @Value.

@Value("${jdbc.driver}") String driver,
Метод ds загружает пул соединений.

Метод jdbc создает обертку для работы с базой.

Подключим класс конфигурации.

ac.register(WebConfig.class, JdbcConfig.class);
Репозиторий

Перепишем класс для работы с базой.
```java
package ru.job4j.accident.repository;

import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;
import ru.job4j.accident.model.Accident;

import java.util.List;

@Repository
public class AccidentJdbcTemplate {
private final JdbcTemplate jdbc;

    public AccidentJdbcTemplate(JdbcTemplate jdbc) {
        this.jdbc = jdbc;
    }

    public Accident save(Accident accident) {
        jdbc.update("insert into accident (name) values (?)",
                accident.getName());
        return accident;
    }

    public List<Accident> getAll() {
        return jdbc.query("select id, name from accident",
                (rs, row) -> {
                    Accident accident = new Accident();
                    accident.setId(rs.getInt("id"));
                    accident.setName(rs.getString("name"));
                    return accident;
                });
    }
}
```
@Repository - называется стереотипной аннотацией.