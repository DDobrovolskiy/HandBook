Очиситить соединения:
``` sql
SELECT pg_terminate_backend(pid)  FROM  pg_stat_activity WHERE pid <> pg_backend_pid() AND datname = 'CloudBack';
```
#### GIT UTILS
``` sql
CREATE EXTENSION IF NOT EXISTS "pgstattuple";
```
https://github.com/dataegret/pg-utils

Оптимизирование:
https://habr.com/ru/articles/488968/  
https://habr.com/ru/companies/postgrespro/articles/349224/  
http://www.gilev.ru/sizetablepostgre/  
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
  
!необходимо создавать индекс по FK!  
```sql
create index *fk* on *pgconf (fk_id)* where *fk_id* is not null;

select * from pg_stats where tablename = 'form';
```
Описание: https://postgrespro.ru/docs/postgresql/9.5/view-pg-stats

Статистика по запросу
```sql
explain(costs off,analyze)
SELECT * FROM form
WHERE state = 'DREFT';
```

```sql
SELECT C.oid,C.relfilenode, nspname, relname AS "relation",
relkind, pg_size_pretty(pg_relation_size(C.oid)) AS "size"
FROM pg_class C
LEFT JOIN pg_namespace N ON (N.oid = C.relnamespace)
WHERE nspname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_relation_size(C.oid) DESC;
```
для конкретной таблицы
```sql
SELECT C.oid,C.relfilenode, nspname, relname AS "relation",
	relkind, pg_size_pretty(pg_relation_size(C.oid)) AS "size"
FROM pg_class C
	LEFT JOIN pg_namespace N ON (N.oid = C.relnamespace)
	LEFT JOIN pg_indexes I ON (I.indexname = C.relname)
WHERE nspname NOT IN ('pg_catalog', 'information_schema')
	AND (I.tablename = 'form' OR C.relname = 'form')
ORDER BY pg_relation_size(C.oid) DESC;
```
INDEX NULL ^ https://github.com/dataegret/pg-utils/blob/master/sql/indexes_with_nulls.sql
