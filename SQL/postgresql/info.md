Очиситить соединения:
``` sql
SELECT pg_terminate_backend(pid)  FROM  pg_stat_activity WHERE pid <> pg_backend_pid() AND datname = 'CloudBack';
```
Оптимизирование:
https://habr.com/ru/articles/488968/  
 - pg_stat_statements  
 - pgbadger

Размер таблиц и индексов

``` sql
SELECT pg_size_pretty( pg_total_relation_size( 'form' ) ); 
SELECT pg_size_pretty( pg_relation_size( 'form' ) );

SELECT nspname || '.' || relname AS "relation",
    pg_size_pretty(pg_total_relation_size(C.oid)) AS "total_size",
	pg_size_pretty(pg_relation_size(C.oid)) AS "size_only_table",
	pg_size_pretty(pg_total_relation_size(C.oid) - pg_relation_size(C.oid)) AS "size_index"
  FROM pg_class C
  LEFT JOIN pg_namespace N ON (N.oid = C.relnamespace)
  WHERE nspname NOT IN ('pg_catalog', 'information_schema')
  ORDER BY pg_total_relation_size(C.oid) DESC
  LIMIT 20;
  ```
  
