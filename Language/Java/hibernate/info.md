https://www.youtube.com/watch?v=b52Qz6qlic0

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
