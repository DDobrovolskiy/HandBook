Link: https://job4j.ru/profile/exercise/98/task-view/503/solutionId/201943

Servlet 3 и выше могут запускаться без web.xml. Он использует конфигурирование через Java классы.

Создадим класс WebInit.

```java
import org.springframework.web.WebApplicationInitializer;
import org.springframework.web.context.support.AnnotationConfigWebApplicationContext;
import org.springframework.web.servlet.DispatcherServlet;
import ru.job4j.accident.config.WebConfig;
import javax.servlet.ServletContext;
import javax.servlet.ServletRegistration;

public class WebInit implements WebApplicationInitializer {

    public void onStartup(ServletContext servletCxt) {
        AnnotationConfigWebApplicationContext ac = new AnnotationConfigWebApplicationContext();
        ac.register(WebConfig.class);
        ac.refresh();
        DispatcherServlet servlet = new DispatcherServlet(ac);
        ServletRegistration.Dynamic registration = servletCxt.addServlet("app", servlet);
        registration.setLoadOnStartup(1);
        registration.addMapping("/");
    }
}
```
Когда tomcat загружает наше приложение, он ищет класс, который расширяет WebApplicationInitializer.

Tomcat создает контекст Spring и загружает DispatcherServlet.

DispatcherServlet будет обрабатывать все запросы. Он доступен по адресу, указанному в addMapping().

Класс AppConfig.

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.view.InternalResourceViewResolver;
import org.springframework.web.servlet.view.JstlView;

@Configuration
@ComponentScan("ru.job4j.accident")
public class WebConfig {

    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver bean = new InternalResourceViewResolver();
        bean.setViewClass(JstlView.class);
        bean.setPrefix("./WEB-INF/views/");
        bean.setSuffix(".jsp");
        return bean;
    }
}
```
В нем появилась аннотация ComponentScan. Она сканирует проект и загружает бины в контекст.

Внутри этого класса создается объект ViewResolver. Spring использует этот объект для поиска jsp. В нем сразу подключен JSTL.