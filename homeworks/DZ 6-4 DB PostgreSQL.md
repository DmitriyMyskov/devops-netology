# Домашнее задание к занятию 4. «PostgreSQL»

***
## Задача 1

Используя Docker, поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.

Подключитесь к БД PostgreSQL, используя `psql`.

Воспользуйтесь командой `\?` для вывода подсказки по имеющимся в `psql` управляющим командам.

**Найдите и приведите** управляющие команды для:

- вывода списка БД,
- подключения к БД,
- вывода списка таблиц,
- вывода описания содержимого таблиц,
- выхода из psql.

### Решение:

```shell
dmitriy@dmitriy-lin:~/netology/docker/hw6-4$ cat docker-compose.yml 
version: '3.6'

services:
  postgres:
    image: postgres:13
    container_name: psql_13
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=1qazxsw2
    volumes:
      - /home/dmitriy/netology/docker/hw6-4/data:/var/lib/postgresql/data
      - /home/dmitriy/netology/docker/hw6-4/backup:/mnt/backup
    ports:
      - "5432:5432"
    restart: always
dmitriy@dmitriy-lin:~/netology/docker/hw6-4$ sudo docker exec -it psql_13 bash
root@bd995570b614:/# psql -U postgres -h localhost
psql (13.10 (Debian 13.10-1.pgdg110+1))
Type "help" for help.

postgres=# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
   
-----------+----------+----------+------------+------------+--------------------
---
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres        
  +
           |          |          |            |            | postgres=CTc/postgr
es
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres        
  +
           |          |          |            |            | postgres=CTc/postgr
es
(3 rows)

postgres=# \d pg_database
               Table "pg_catalog.pg_database"
    Column     |   Type    | Collation | Nullable | Default 
---------------+-----------+-----------+----------+---------
 oid           | oid       |           | not null | 
 datname       | name      |           | not null | 
 datdba        | oid       |           | not null | 
 encoding      | integer   |           | not null | 
 datcollate    | name      |           | not null | 
 datctype      | name      |           | not null | 
 datistemplate | boolean   |           | not null | 
 datallowconn  | boolean   |           | not null | 
 datconnlimit  | integer   |           | not null | 
 datlastsysoid | oid       |           | not null | 
 datfrozenxid  | xid       |           | not null | 
 datminmxid    | xid       |           | not null | 
 dattablespace | oid       |           | not null | 
 datacl        | aclitem[] |           |          | 
Indexes:
    "pg_database_datname_index" UNIQUE, btree (datname), tablespace "pg_global"
    "pg_database_oid_index" UNIQUE, btree (oid), tablespace "pg_global"
Tablespace: "pg_global"

postgres=# \q

```

***
## Задача 2

Используя `psql`, создайте БД `test_database`.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/virt-11/06-db-04-postgresql/test_data).

Восстановите бэкап БД в `test_database`.

Перейдите в управляющую консоль `psql` внутри контейнера.

Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.

Используя таблицу [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы `orders` 
с наибольшим средним значением размера элементов в байтах.

**Приведите в ответе** команду, которую вы использовали для вычисления, и полученный результат.

### Решение:

```shell
root@bd995570b614:/# psql -U postgres -h localhost
psql (13.10 (Debian 13.10-1.pgdg110+1))
Type "help" for help.

postgres=# CREATE DATABASE test_database;
CREATE DATABASE
postgres=# \q
root@bd995570b614:/# psql -U postgres test_database < /mnt/backup/test_dump.sql 
SET
SET
SET
SET
SET
 set_config 
------------
 
(1 row)

SET
SET
SET
SET
SET
SET
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
ALTER TABLE
COPY 8
 setval 
--------
      8
(1 row)

ALTER TABLE
root@bd995570b614:/# psql -U postgres -h localhost
psql (13.10 (Debian 13.10-1.pgdg110+1))
Type "help" for help.

postgres=# \c test_database
You are now connected to database "test_database" as user "postgres".
test_database=# \d
              List of relations
 Schema |     Name      |   Type   |  Owner   
--------+---------------+----------+----------
 public | orders        | table    | postgres
 public | orders_id_seq | sequence | postgres
(2 rows)

test_database=# ANALYZE VERBOSE public.orders;
INFO:  analyzing "public.orders"
INFO:  "orders": scanned 1 of 1 pages, containing 8 live rows and 0 dead rows; 8 rows in sample, 8 estimated total rows
ANALYZE
test_database=# SELECT tablename, attname, avg_width FROM pg_stats WHERE tablename='orders';
 tablename | attname | avg_width 
-----------+---------+-----------
 orders    | id      |         4
 orders    | title   |        16
 orders    | price   |         4
(3 rows)

test_database=# SELECT tablename, attname, avg_width FROM pg_stats WHERE tablename='orders' ORDER BY avg_width DESC LIMIT 1;
 tablename | attname | avg_width 
-----------+---------+-----------
 orders    | title   |        16
(1 row)

```

***
## Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам как успешному выпускнику курсов DevOps в Нетологии предложили
провести разбиение таблицы на 2: шардировать на orders_1 - price>499 и orders_2 - price<=499.

Предложите SQL-транзакцию для проведения этой операции.

Можно ли было изначально исключить ручное разбиение при проектировании таблицы orders?

### Решение:
```shell
test_database=# CREATE TABLE orders_1 (CHECK (price > 499)) INHERITS (orders);
CREATE TABLE
test_database=# INSERT INTO orders_1 SELECT * FROM orders WHERE price > 499;
INSERT 0 3
test_database=# CREATE TABLE orders_2 (CHECK (price <= 499)) INHERITS (orders);
CREATE TABLE
test_database=# INSERT INTO orders_2 SELECT * FROM orders WHERE price <= 499;
INSERT 0 5
test_database=# DELETE FROM ONLY orders;
DELETE 8
test_database=# \dt
          List of relations
 Schema |   Name   | Type  |  Owner   
--------+----------+-------+----------
 public | orders   | table | postgres
 public | orders_1 | table | postgres
 public | orders_2 | table | postgres
(3 rows)

test_database=# explain analyze select * from orders where price = 15;
                                                   QUERY PLAN                  
                                 
-------------------------------------------------------------------------------
---------------------------------
 Append  (cost=0.00..15.87 rows=3 width=132) (actual time=0.066..0.073 rows=0 l
oops=1)
   ->  Seq Scan on orders orders_1  (cost=0.00..1.10 rows=1 width=24) (actual t
ime=0.034..0.034 rows=0 loops=1)
         Filter: (price = 15)
   ->  Seq Scan on orders_2  (cost=0.00..14.75 rows=2 width=186) (actual time=0
.024..0.024 rows=0 loops=1)
         Filter: (price = 15)
         Rows Removed by Filter: 5
 Planning Time: 0.882 ms
 Execution Time: 0.150 ms
(8 rows)
```

- Можно ли было изначально исключить ручное разбиение при проектировании таблицы orders?   

Можно, Если знать по какому полю будет распределение.   
*Можем получить ситуацию когда шарды будут заполнены не равномерно.

***
## Задача 4

Используя утилиту `pg_dump`, создайте бекап БД `test_database`.

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?

### Решение:
```shell
root@bd995570b614:/# pg_dump -h localhost -U postgres test_database > /mnt/backup/test_database.sql
root@bd995570b614:/# ls -l /mnt/backup/
total 8
-rw-r--r-- 1 root root 3507 Apr 10 09:33 test_database.sql
-rw-rw-r-- 1 1001 1001 2082 Feb 28 03:20 test_dump.sql

```

- Доработка:  
```
CREATE TABLE public.orders (
    id integer NOT NULL,
    title character varying(80) NOT NULL UNIQUE,
    price integer DEFAULT 0
);

```
PS> По логике "гугл" говорит что для PostgreSQL нужно добавить UNIQUE:   
PostgreSQL provides you with the UNIQUE constraint that maintains the uniqueness of the data correctly.