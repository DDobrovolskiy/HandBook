https://www.youtube.com/watch?v=b52Qz6qlic0  
https://www.youtube.com/watch?v=-EpP0Vo63FM  
#### text
https://stackoverflow.com/questions/43665090/why-do-we-have-to-use-modifying-annotation-for-queries-in-data-jpa  
https://habr.com/ru/articles/265061/

#### log4j.properties

log4j.logger.org.hibernate=INFO, hb  
log4j.logger.org.hibernate.SQL=DEBUG  
log4j.logger.org.hibernate.type=TRACE  
log4j.logger.org.hibernate.hql.ast.AST=info  
log4j.logger.org.hibernate.tool.hbm2ddl=warm  
log4j.logger.org.hibernate.hql=debug  
log4j.logger.org.hibernate.cache=info  
log4j.logger.org.hibernate.jdbc=debug  

log4j.append.hb=org.apache.log4j.ConsoleAppender  
log4j.append.hb.layout=org.apache.log4j.PatternLayout  
log4j.append.hb.lauout.ConversionPattern=Hibernatelog --> %d{HH:mm:ss} %-5p %c - %m%n  
log4j.append.hb.Threshold=Trace  

#### hibernate.cfg.xml  
``` xml
<property name="show_sql">true</property>
<property name="format_sql">true</property>
<property name="use_sql_comment">true</property>
```

#### application.yml
``` yml
spring:
    profiles: develop
    jpa:
        properties:
            hibernate:
                format_sql: true
        hibernate:
            ddl-auto: update
    reportServer:
        # Ограничение на кол-во ценников при печати, нужно, чтоб юзеры не положили сервер
        priceTagLimit: 10000
        url: ${REPORT_SERVER_URL:http://localhost:8080}
logging:
    level:
        root: info
        org:
            springframework:
                boot: info
            hibernate:
                type:
                    descriptor:
                        sql:
                            BasicBinder: ${CLOUD_TEST_HIBERNATE_LOG_LEVEL:trace}
                SQL: debug
                orm:
                    jpa: debug
                transaction: debug
                security: error
```

#### lifeHack
- ``` findById ``` - для поиска сущностей;
- ``` getById ``` - для простановки сущностей;
