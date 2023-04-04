# Домашнее задание к занятию 2. «SQL».

***
## Задача 1

Используя Docker, поднимите инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose-манифест.

### Решение:
```shell
dmitriy@dmitriy-lin:~/netology/docker$ sudo docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

dmitriy@dmitriy-lin:~/netology/docker/hw6-2$ cat docker-compose.yml 
version: '3.6'

volumes:
  data: {}
  backup: {}

services:
  postgres:
    image: postgres:12
    container_name: psql_12
    environment:
      - POSTGRES_USER=postgresdb
      - POSTGRES_PASSWORD=postgresdb
    volumes:
      - /home/dmitriy/netology/docker/hw6-2/data:/var/lib/postgresql/data
      - /home/dmitriy/netology/docker/hw6-2/backup:/data/backup/postgres
    ports:
      - "5432:5432"
    restart: always

    dmitriy@dmitriy-lin:~/netology/docker/hw6-2$ sudo docker-compose up -d
Creating network "hw6-2_default" with the default driver
...
Creating psql_12 ... done

dmitriy@dmitriy-lin:~/netology/docker/hw6-2$ sudo docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                                       NAMES
2898799554ce   postgres:12   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   psql_12

```

***
## Задача 2

В БД из задачи 1: 

- создайте пользователя test-admin-user и БД test_db;
- в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже);
- предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db;
- создайте пользователя test-simple-user;
- предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE этих таблиц БД test_db.

Таблица orders:

- id (serial primary key);
- наименование (string);
- цена (integer).

Таблица clients:

- id (serial primary key);
- фамилия (string);
- страна проживания (string, index);
- заказ (foreign key orders).

Приведите:

- итоговый список БД после выполнения пунктов выше;
- описание таблиц (describe);
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db;
- список пользователей с правами над таблицами test_db.

### Решение:
```shell
dmitriy@dmitriy-lin:~/netology/docker/hw6-2$ sudo psql -h 127.0.0.1 user=postgresdb
Password for user postgresdb: 
psql (12.14 (Ubuntu 12.14-0ubuntu0.20.04.1))
Type "help" for help.

postgresdb=# CREATE USER "test-admin-user" WITH LOGIN;
CREATE ROLE
postgresdb=# CREATE DATABASE test_db;
CREATE DATABASE
postgresdb=# \c test_db
You are now connected to database "test_db" as user "postgresdb".
test_db=# CREATE TABLE orders (id SERIAL PRIMARY KEY, наименование TEXT, цена INT);
CREATE TABLE
test_db=# CREATE TABLE clients (id SERIAL PRIMARY KEY, фамилия TEXT, "страна проживания" TEXT, заказ INT REFERENCES orders (id));
CREATE TABLE
test_db=# CREATE INDEX ON clients ("страна проживания");
CREATE INDEX
test_db=# GRANT ALL ON TABLE clients, orders TO "test-admin-user";
GRANT
test_db=# CREATE USER "test-simple-user" WITH LOGIN;
CREATE ROLE
test_db=# GRANT SELECT,INSERT,UPDATE,DELETE ON TABLE clients,orders TO "test-simple-user";
GRANT
test_db=# \l
                                    List of databases
    Name    |   Owner    | Encoding |  Collate   |   Ctype    |     Access privileges     
------------+------------+----------+------------+------------+---------------------------
 postgres   | postgresdb | UTF8     | en_US.utf8 | en_US.utf8 | 
 postgresdb | postgresdb | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0  | postgresdb | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgresdb            +
            |            |          |            |            | postgresdb=CTc/postgresdb
 template1  | postgresdb | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgresdb            +
            |            |          |            |            | postgresdb=CTc/postgresdb
 test_db    | postgresdb | UTF8     | en_US.utf8 | en_US.utf8 | 
(5 rows)

test_db=# \d+ orders
                                                   Table "public.orders"
    Column    |  Type   | Collation | Nullable |              Default               | Storage  | Stats target | Description 
--------------+---------+-----------+----------+------------------------------------+----------+--------------+-------------
 id           | integer |           | not null | nextval('orders_id_seq'::regclass) | plain    |              | 
 наименование | text    |           |          |                                    | extended |              | 
 цена         | integer |           |          |                                    | plain    |              | 
Indexes:
    "orders_pkey" PRIMARY KEY, btree (id)
Referenced by:
    TABLE "clients" CONSTRAINT "clients_заказ_fkey" FOREIGN KEY ("заказ") REFERENCES orders(id)
Access method: heap

test_db=# \d+ clients
                                                      Table "public.clients"
      Column       |  Type   | Collation | Nullable |               Default               | Storage  | Stats target | Description 
-------------------+---------+-----------+----------+-------------------------------------+----------+--------------+-------------
 id                | integer |           | not null | nextval('clients_id_seq'::regclass) | plain    |              | 
 фамилия           | text    |           |          |                                     | extended |              | 
 страна проживания | text    |           |          |                                     | extended |              | 
 заказ             | integer |           |          |                                     | plain    |              | 
Indexes:
    "clients_pkey" PRIMARY KEY, btree (id)
    "clients_страна проживания_idx" btree ("страна проживания")
Foreign-key constraints:
    "clients_заказ_fkey" FOREIGN KEY ("заказ") REFERENCES orders(id)
Access method: heap

test_db=# SELECT table_name, array_agg(privilege_type), grantee FROM information_schema.table_privileges WHERE table_name = 'orders' OR table_name = 'clients' GROUP BY table_name, grantee ;
 table_name |                         array_agg                         |     grantee      
------------+-----------------------------------------------------------+------------------
 clients    | {INSERT,TRIGGER,REFERENCES,TRUNCATE,DELETE,UPDATE,SELECT} | postgresdb
 clients    | {INSERT,TRIGGER,REFERENCES,TRUNCATE,DELETE,UPDATE,SELECT} | test-admin-user
 clients    | {DELETE,INSERT,SELECT,UPDATE}                             | test-simple-user
 orders     | {INSERT,TRIGGER,REFERENCES,TRUNCATE,DELETE,UPDATE,SELECT} | postgresdb
 orders     | {INSERT,TRIGGER,REFERENCES,TRUNCATE,DELETE,UPDATE,SELECT} | test-admin-user
 orders     | {DELETE,SELECT,UPDATE,INSERT}                             | test-simple-user
(6 rows)

```

***
## Задача 3

Используя SQL-синтаксис, наполните таблицы следующими тестовыми данными:

Таблица orders

|Наименование|цена|
|------------|----|
|Шоколад| 10 |
|Принтер| 3000 |
|Книга| 500 |
|Монитор| 7000|
|Гитара| 4000|

Таблица clients

|ФИО|Страна проживания|
|------------|----|
|Иванов Иван Иванович| USA |
|Петров Петр Петрович| Canada |
|Иоганн Себастьян Бах| Japan |
|Ронни Джеймс Дио| Russia|
|Ritchie Blackmore| Russia|

Используя SQL-синтаксис:
- вычислите количество записей для каждой таблицы.

Приведите в ответе:

    - запросы,
    - результаты их выполнения.

### Решение:
```shell
test_db=# INSERT INTO orders VALUES (1, 'Шоколад', 10), (2, 'Принтер', 3000), (3, 'Книга', 500), (4, 'Монитор', 7000), (5, 'Гитара', 4000);
INSERT 0 5
test_db=# SELECT * FROM orders;
 id | наименование | цена 
----+--------------+------
  1 | Шоколад      |   10
  2 | Принтер      | 3000
  3 | Книга        |  500
  4 | Монитор      | 7000
  5 | Гитара       | 4000
(5 rows)

test_db=# INSERT INTO clients VALUES (1, 'Иванов Иван Иванович', 'USA'), (2, 'Петров Петр Петрович', 'Canada'), (3, 'Иоганн Себастьян Бах', 'Japan'), (4, 'Ронни Джеймс Дио', 'Russia'), (5, 'Ritchie Blackmore', 'Russia');
INSERT 0 5
test_db=# SELECT * FROM clients;
 id |       фамилия        | страна проживания | заказ 
----+----------------------+-------------------+-------
  1 | Иванов Иван Иванович | USA               |      
  2 | Петров Петр Петрович | Canada            |      
  3 | Иоганн Себастьян Бах | Japan             |      
  4 | Ронни Джеймс Дио     | Russia            |      
  5 | Ritchie Blackmore    | Russia            |      
(5 rows)

test_db=# SELECT count(1) FROM clients;
 count 
-------
     5
(1 row)

test_db=# SELECT count(1) FROM orders;
 count 
-------
     5
(1 row)

```

***
## Задача 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys, свяжите записи из таблиц, согласно таблице:

|ФИО|Заказ|
|------------|----|
|Иванов Иван Иванович| Книга |
|Петров Петр Петрович| Монитор |
|Иоганн Себастьян Бах| Гитара |

Приведите SQL-запросы для выполнения этих операций.

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод этого запроса.
 
Подсказка: используйте директиву `UPDATE`.

### Решение:
```shell
test_db=# UPDATE clients SET "заказ"=3 WHERE id=1;
UPDATE 1
test_db=# UPDATE clients SET "заказ"=4 WHERE id=2;
UPDATE 1
test_db=# UPDATE clients SET "заказ"=5 WHERE id=3;
UPDATE 1
test_db=# SELECT * FROM clients WHERE "заказ" IS NOT null;
 id |       фамилия        | страна проживания | заказ 
----+----------------------+-------------------+-------
  1 | Иванов Иван Иванович | USA               |     3
  2 | Петров Петр Петрович | Canada            |     4
  3 | Иоганн Себастьян Бах | Japan             |     5
(3 rows)

```

***
## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).

Приведите получившийся результат и объясните, что значат полученные значения.

### Решение:
```shell
test_db=# EXPLAIN SELECT * FROM clients WHERE "заказ" IS NOT null;
                        QUERY PLAN                         
-----------------------------------------------------------
 Seq Scan on clients  (cost=0.00..18.10 rows=806 width=72)
   Filter: ("заказ" IS NOT NULL)
(2 rows)

```

**cost** - ожидаемые затраты примерно, от и до.   
**rows** - ожидаемое число строк   
**width** - ожидаемый средний размер строки   
Смотрел тут: https://www.postgresql.org/docs/current/using-explain.html

***
## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. задачу 1).

Остановите контейнер с PostgreSQL, но не удаляйте volumes.

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления.

### Решение:
```shell
dmitriy@dmitriy-lin:~/netology/docker/hw6-2$ cat docker-compose.yml 
version: '3.6'

volumes:
  data: {}
  backup: {}

services:
  postgres:
    image: postgres:12
    container_name: psql_12
    environment:
      - POSTGRES_USER=postgresdb
      - POSTGRES_PASSWORD=postgresdb
    volumes:
      - /home/dmitriy/netology/docker/hw6-2/data:/var/lib/postgresql/data
      - /home/dmitriy/netology/docker/hw6-2/backup:/data/backup/postgres
    ports:
      - "5432:5432"
    restart: always

  postgres_copy:
    image: postgres:12
    container_name: psql_12_copy
    environment:
      - POSTGRES_USER=postgresdb
      - POSTGRES_PASSWORD=postgresdb
    volumes:
      - /home/dmitriy/netology/docker/hw6-2/data_2:/var/lib/postgresql/data
      - /home/dmitriy/netology/docker/hw6-2/backup:/data/backup/postgres
    ports:
      - "5433:5432"
    restart: always

dmitriy@dmitriy-lin:~/netology/docker/hw6-2$ sudo docker-compose up -d
Starting psql_12      ... done
Creating psql_12_copy ... done
dmitriy@dmitriy-lin:~/netology/docker/hw6-2$ sudo docker exec -it psql_12 bash
root@2898799554ce:/# cd /data/backup/postgres/
root@2898799554ce:/data/backup/postgres# pg_dump -h localhost -U postgresdb test_db > test_db.sql
root@2898799554ce:/data/backup/postgres# ls -l
total 8
-rw-r--r-- 1 root root 4586 Apr  4 11:49 test_db.sql
root@2898799554ce:/data/backup/postgres# exit
exit
dmitriy@dmitriy-lin:~/netology/docker/hw6-2$ ls -l
итого 16
drwxr-xr-x  2 root             root    4096 апр  4 18:49 backup
drwx------ 19 systemd-coredump root    4096 апр  4 18:47 data
drwx------ 19 systemd-coredump root    4096 апр  4 18:47 data_2
-rw-rw-r--  1 dmitriy          dmitriy  795 апр  4 18:47 docker-compose.yml
dmitriy@dmitriy-lin:~/netology/docker/hw6-2$ ls -l backup/
итого 8
-rw-r--r-- 1 root root 4586 апр  4 18:49 test_db.sql
dmitriy@dmitriy-lin:~/netology/docker/hw6-2$ sudo docker exec -it psql_12_copy bash
root@ea7dec4719fe:/# cd data/backup/postgres/
root@ea7dec4719fe:/data/backup/postgres# ls -l
total 8
-rw-r--r-- 1 root root 4586 Apr  4 11:49 test_db.sql
root@ea7dec4719fe:/data/backup/postgres# psql -h localhost -U postgresdb
psql (12.14 (Debian 12.14-1.pgdg110+1))
Type "help" for help.

postgresdb=# exit
root@ea7dec4719fe:/data/backup/postgres# psql -h localhost -U postgresdb test_db < test_db.sql
psql: error: connection to server at "localhost" (127.0.0.1), port 5432 failed: FATAL:  database "test_db" does not exist
root@ea7dec4719fe:/data/backup/postgres# CREATE DATABASE test_db;
bash: CREATE: command not found
root@ea7dec4719fe:/data/backup/postgres# psql -h localhost -U postgresdb
psql (12.14 (Debian 12.14-1.pgdg110+1))
Type "help" for help.

postgresdb=# CREATE DATABASE test_db;
CREATE DATABASE
postgresdb=# exit
root@ea7dec4719fe:/data/backup/postgres# psql -h localhost -U postgresdb test_db < test_db.sql
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
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
ALTER TABLE
ALTER TABLE
COPY 5
COPY 5
 setval 
--------
      1
(1 row)

 setval 
--------
      1
(1 row)

ALTER TABLE
ALTER TABLE
CREATE INDEX
ALTER TABLE
ERROR:  role "test-admin-user" does not exist
ERROR:  role "test-simple-user" does not exist
ERROR:  role "test-admin-user" does not exist
ERROR:  role "test-simple-user" does not exist
root@ea7dec4719fe:/data/backup/postgres# psql -h localhost -U postgresdb
psql (12.14 (Debian 12.14-1.pgdg110+1))
Type "help" for help.

postgresdb=# CREATE USER "test-simple-user" WITH LOGIN;
CREATE ROLE
postgresdb=# CREATE USER "test-admin-user" WITH LOGIN;
CREATE ROLE
postgresdb=# exit
root@ea7dec4719fe:/data/backup/postgres# psql -h localhost -U postgresdb test_db < test_db.sql
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
ERROR:  relation "clients" already exists
ALTER TABLE
ERROR:  relation "clients_id_seq" already exists
ALTER TABLE
ALTER SEQUENCE
ERROR:  relation "orders" already exists
ALTER TABLE
ERROR:  relation "orders_id_seq" already exists
ALTER TABLE
ALTER SEQUENCE
ALTER TABLE
ALTER TABLE
ERROR:  duplicate key value violates unique constraint "clients_pkey"
DETAIL:  Key (id)=(4) already exists.
CONTEXT:  COPY clients, line 1
ERROR:  duplicate key value violates unique constraint "orders_pkey"
DETAIL:  Key (id)=(1) already exists.
CONTEXT:  COPY orders, line 1
 setval 
--------
      1
(1 row)

 setval 
--------
      1
(1 row)

ERROR:  multiple primary keys for table "clients" are not allowed
ERROR:  multiple primary keys for table "orders" are not allowed
ERROR:  relation "clients_страна проживания_idx" already exists
ERROR:  constraint "clients_заказ_fkey" for relation "clients" already exists
GRANT
GRANT
GRANT
GRANT
root@ea7dec4719fe:/data/backup/postgres# psql -h localhost -U postgresdb
psql (12.14 (Debian 12.14-1.pgdg110+1))
Type "help" for help.

postgresdb=# \l
                                    List of databases
    Name    |   Owner    | Encoding |  Collate   |   Ctype    |     Access privileges     
------------+------------+----------+------------+------------+---------------------------
 postgres   | postgresdb | UTF8     | en_US.utf8 | en_US.utf8 | 
 postgresdb | postgresdb | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0  | postgresdb | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgresdb            +
            |            |          |            |            | postgresdb=CTc/postgresdb
 template1  | postgresdb | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgresdb            +
            |            |          |            |            | postgresdb=CTc/postgresdb
 test_db    | postgresdb | UTF8     | en_US.utf8 | en_US.utf8 | 
(5 rows)

postgresdb=# \c test_db
You are now connected to database "test_db" as user "postgresdb".
test_db=# \d+ orders
                                                   Table "public.orders"
    Column    |  Type   | Collation | Nullable |              Default               | Storage  | Stats target | Description 
--------------+---------+-----------+----------+------------------------------------+----------+--------------+-------------
 id           | integer |           | not null | nextval('orders_id_seq'::regclass) | plain    |              | 
 наименование | text    |           |          |                                    | extended |              | 
 цена         | integer |           |          |                                    | plain    |              | 
Indexes:
    "orders_pkey" PRIMARY KEY, btree (id)
Referenced by:
    TABLE "clients" CONSTRAINT "clients_заказ_fkey" FOREIGN KEY ("заказ") REFERENCES orders(id)
Access method: heap

test_db=# \d+ clients
                                                      Table "public.clients"
      Column       |  Type   | Collation | Nullable |               Default               | Storage  | Stats target | Description 
-------------------+---------+-----------+----------+-------------------------------------+----------+--------------+-------------
 id                | integer |           | not null | nextval('clients_id_seq'::regclass) | plain    |              | 
 фамилия           | text    |           |          |                                     | extended |              | 
 страна проживания | text    |           |          |                                     | extended |              | 
 заказ             | integer |           |          |                                     | plain    |              | 
Indexes:
    "clients_pkey" PRIMARY KEY, btree (id)
    "clients_страна проживания_idx" btree ("страна проживания")
Foreign-key constraints:
    "clients_заказ_fkey" FOREIGN KEY ("заказ") REFERENCES orders(id)
Access method: heap

test_db=# EXPLAIN SELECT * FROM clients WHERE "заказ" IS NOT null;
                       QUERY PLAN                       
--------------------------------------------------------
 Seq Scan on clients  (cost=0.00..1.05 rows=5 width=72)
   Filter: ("заказ" IS NOT NULL)
(2 rows)

test_db=# SELECT * FROM clients WHERE "заказ" IS NOT null;
 id |       фамилия        | страна проживания | заказ 
----+----------------------+-------------------+-------
  1 | Иванов Иван Иванович | USA               |     3
  2 | Петров Петр Петрович | Canada            |     4
  3 | Иоганн Себастьян Бах | Japan             |     5
(3 rows)

test_db=# exit
root@ea7dec4719fe:/data/backup/postgres# exit
exit

```