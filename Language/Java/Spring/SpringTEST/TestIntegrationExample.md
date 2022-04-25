```java
import com.github.springtestdbunit.annotation.DatabaseOperation;
import com.github.springtestdbunit.annotation.DatabaseSetup;
import com.github.springtestdbunit.annotation.DatabaseTearDown;
import com.github.springtestdbunit.annotation.ExpectedDatabase;
import com.github.springtestdbunit.assertion.DatabaseAssertionMode;
import dir.app.domain.Book;
import org.junit.Assert;
import org.junit.Test;
import org.junit.jupiter.api.DisplayName;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.ImportAutoConfiguration;
import org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration;
import org.springframework.boot.test.autoconfigure.jdbc.AutoConfigureTestDatabase;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.boot.test.autoconfigure.orm.jpa.TestEntityManager;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.context.TestExecutionListeners;
import org.springframework.test.context.jdbc.Sql;
import org.springframework.test.context.jdbc.SqlConfig;
import org.springframework.test.context.jdbc.SqlGroup;
import org.springframework.test.context.jdbc.SqlMergeMode;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

/**
 * @author ddobrovolskiy
 * @since 22.04.2022
 */

@ActiveProfiles("test")
@RunWith(SpringJUnit4ClassRunner.class)
/*
@DataJpaTest предоставляет некоторые стандартные настройки, необходимые для тестирования уровня сохраняемости:

настройка H2, базы данных в памяти
установка Hibernate, Spring Data и DataSource
выполнение @EntityScan
включение ведения журнала SQL
Для выполнения операций с БД нам нужны некоторые записи, которые уже есть в нашей базе данных.
Чтобы настроить эти данные, мы можем использовать TestEntityManager.

Spring Boot TestEntityManager  — это альтернатива стандартному JPA EntityManager , который предоставляет методы,
обычно используемые при написании тестов.
 */
@DataJpaTest
/*
Учтите, что при тестировании с помощью @DataJpaTest используются встроенные СУБД в оперативной памяти.
Для тестирования же с настоящей базой данных необходимо снабдить класс теста
аннотацией @AutoConfigureTestDatabase(replace=Replace.NONE).
 */
@AutoConfigureTestDatabase(replace= AutoConfigureTestDatabase.Replace.NONE)
@ImportAutoConfiguration(FlywayAutoConfiguration.class)
/*
Этот DatabaseSetup можно аннотировать как для класса, так и для метода, который играет аналогичную роль
с Junit @BeforeClass и @Before.
По умолчанию операция CLEAN_INSERT будет выполняться во время инициализации, что означает, что при импорте данных
в DataSet.xml сначала будут очищены данные в задействованных таблицах.
DatabaseSetup может иметь несколько аннотаций и организован @DatabaseSetups. Для получения подробной информации
обратитесь к предыдущему случаю в этой статье.
 */
@DatabaseSetup("sampleDataSet.xml")
/*
Аннотация @DatabaseTearDown используется для сброса контекста базы данных после завершения теста.
Подобно @DatabaseSetup, его можно использовать в классах или методах или вложить,
и указать соответствующий режим работы данных DBUnit.
 */
@DatabaseTearDown(value={"classpath:data-set.xml"}, type= DatabaseOperation.DELETE)
/*
Как следует из названия, после выполнения тестового примера утверждается, что данные в соответствующей таблице
в базе данных согласуются с "expectedData.xml". Это утверждение имеет те же свойства, что и два предыдущих.
Оно может быть аннотировано для класса или метода, а несколько @ExpectedDatabases также могут использоваться
для формирования сложного утверждения.
 */
@ExpectedDatabase(value = "expectedData.xml", assertionMode= DatabaseAssertionMode.NON_STRICT)
/*
@Sql используется для аннотирования тестового класса или метода тестирования для настройки сценариев SQL,
которые будут запускаться в данной базе данных во время интеграционных тестов.
 */
@Sql("/test-schema.sql")
//или
@Sql(
        scripts = "/test-schema.sql",
        config = @SqlConfig(commentPrefix = "`", separator = "@@") // Задайть префикс комментария и разделитель в сценариях SQL.
)
/*
@SqlMergeMode используется для аннотирования тестового класса или тестового метода, чтобы настроить,
объединяются ли объявления @Sql уровня метода с объявлениями @Sql уровня класса. Если @SqlMergeMode не объявлен
в тестовом классе или тестовом методе, по умолчанию будет использоваться режим слияния OVERRIDE. В режиме OVERRIDE
объявления @Sql на уровне метода будут эффективно переопределять объявления @Sql на уровне класса.

Обратите внимание, что объявление @SqlMergeMode на уровне метода переопределяет объявление на уровне класса.
 */
@SqlMergeMode(SqlMergeMode.MergeMode.MERGE) // Установите режим слияния @Sql на MERGE для всех тестовых методов в классе.
@TestExecutionListeners(
        value = { CustomTestExecutionListener.class },
        mergeMode = TestExecutionListeners.MergeMode.MERGE_WITH_DEFAULTS)
public class BookRepositoryIntegrationTest {

    public BookRepositoryIntegrationTest() {
        System.out.println("--- CONSTRUCTOR 1 ---");
    }

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private BookRepository bookRepository;

    @Test
    @DisplayName("DISPLAY NAME")
    @Sql({"/test-schema.sql", "/test-user-data.sql"}) // Запустить два сценария для этого теста.
    @SqlGroup({ // Объявить группу сценариев SQL.
            @Sql("/employee_truncate-test-data-001.sql"), //Порядок важен
            @Sql("/user-test-data-001.sql")
    })
    @SqlMergeMode(SqlMergeMode.MergeMode.MERGE) // Установить режим слияния @Sql на MERGE для определенного метода тестирования.
    public void whenFindByName_thenReturnBook() {
        System.out.println("--- RUN TEST ---");
        // given
        Book garryPotter = new Book("Garry Potter");
        entityManager.persist(garryPotter);
        entityManager.flush();

        // when
        Book found = bookRepository.findByName(garryPotter.getName());

        // then
        Assert.assertEquals(found.getName(), garryPotter.getName());
        System.out.println("--- NEXT TEST ---");
    }

    @Test
    public void whenFindByName_thenReturnBook2() {
        System.out.println("--- RUN NEXT TEST ---");
        // given
        Book garryPotter = new Book("Garry Potter");
        entityManager.persist(garryPotter);
        entityManager.flush();

        // when
        Book found = bookRepository.findByName(garryPotter.getName());

        // then
        Assert.assertEquals(found.getName(), garryPotter.getName());
        System.out.println("--- STOP TEST ---");
    }
}
```
