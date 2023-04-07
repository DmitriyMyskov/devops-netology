# Домашнее задание к занятию 3. «MySQL»

***
## Задача 1

Используя Docker, поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/virt-11/06-db-03-mysql/test_data) и 
восстановитесь из него.

Перейдите в управляющую консоль `mysql` внутри контейнера.

Используя команду `\h`, получите список управляющих команд.

Найдите команду для выдачи статуса БД и **приведите в ответе** из её вывода версию сервера БД.

Подключитесь к восстановленной БД и получите список таблиц из этой БД.

**Приведите в ответе** количество записей с `price` > 300.

В следующих заданиях мы будем продолжать работу с этим контейнером.

### Решение:

Используя Docker, поднимите инстанс MySQL (версию 8).

```shell
dmitriy@dmitriy-lin:~/netology/docker/hw6-3$ cat docker-compose.yml 
version: '3.6'

services:
  mysql:
    image: mysql:8
    container_name: mysql_8
    environment:
      - MYSQL_ROOT_PASSWORD=qazwsxedc
    volumes:
      - /home/dmitriy/netology/docker/hw6-3/data:/var/lib/mysql
      - /home/dmitriy/netology/docker/hw6-3/backup:/mnt/backup
    ports:
      - "3306:3306"
    restart: always
dmitriy@dmitriy-lin:~/netology/docker/hw6-3$ sudo docker-compose up -d
Creating network "hw6-3_default" with the default driver
Pulling mysql (mysql:8)...
8: Pulling from library/mysql
328ba678bf27: Pull complete
f3f5ff008d73: Pull complete
dd7054d6d0c7: Pull complete
70b5d4e8750e: Pull complete
cdc4a7b43bdd: Pull complete
3e9c0b61a8f3: Pull complete
806a08b6c085: Pull complete
021b2cebd832: Pull complete
ad31ba45b26b: Pull complete
0d4c2bd59d1c: Pull complete
148dcef42e3b: Pull complete
Digest: sha256:f496c25da703053a6e0717f1d52092205775304ea57535cc9fcaa6f35867800b
Status: Downloaded newer image for mysql:8
Creating mysql_8 ... done

```

Изучите бэкап БД и восстановитесь из него.

```shell
dmitriy@dmitriy-lin:~/netology/docker/hw6-3$ sudo docker exec -it mysql_8 bash
bash-4.4# mysql -h localhost -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.32 MySQL Community Server - GPL

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> CREATE DATABASE test_db;
Query OK, 1 row affected (0.01 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test_db            |
+--------------------+
5 rows in set (0.02 sec)

mysql> exit
Bye
bash-4.4# mysql -u root -p test_db < /mnt/backup/test_dump.sql 
Enter password:

```

Используя команду `\h`, получите список управляющих команд.

```shell
bash-4.4# mysql -h localhost -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 8.0.32 MySQL Community Server - GPL

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> \h

For information about MySQL products and services, visit:
   http://www.mysql.com/
For developer information, including the MySQL Reference Manual, visit:
   http://dev.mysql.com/
To buy MySQL Enterprise support, training, or other products, visit:
   https://shop.mysql.com/

List of all MySQL commands:
Note that all text commands must be first on line and end with ';'
?         (\?) Synonym for `help'.
clear     (\c) Clear the current input statement.
connect   (\r) Reconnect to the server. Optional arguments are db and host.
delimiter (\d) Set statement delimiter.
edit      (\e) Edit command with $EDITOR.
ego       (\G) Send command to mysql server, display result vertically.
exit      (\q) Exit mysql. Same as quit.
go        (\g) Send command to mysql server.
help      (\h) Display this help.
nopager   (\n) Disable pager, print to stdout.
notee     (\t) Don't write into outfile.
pager     (\P) Set PAGER [to_pager]. Print the query results via PAGER.
print     (\p) Print current command.
prompt    (\R) Change your mysql prompt.
quit      (\q) Quit mysql.
rehash    (\#) Rebuild completion hash.
source    (\.) Execute an SQL script file. Takes a file name as an argument.
status    (\s) Get status information from the server.
system    (\!) Execute a system shell command.
tee       (\T) Set outfile [to_outfile]. Append everything into given outfile.
use       (\u) Use another database. Takes database name as argument.
charset   (\C) Switch to another charset. Might be needed for processing binlog with multi-byte charsets.
warnings  (\W) Show warnings after every statement.
nowarning (\w) Don't show warnings after every statement.
resetconnection(\x) Clean session context.
query_attributes Sets string parameters (name1 value1 name2 value2 ...) for the next query to pick up.
ssl_session_data_print Serializes the current SSL session data to stdout or file

For server side help, type 'help contents'

```

Найдите команду для выдачи статуса БД и **приведите в ответе** из её вывода версию сервера БД.

```shell
mysql> \s
--------------
mysql  Ver 8.0.32 for Linux on x86_64 (MySQL Community Server - GPL)

Connection id:		10
Current database:	
Current user:		root@localhost
SSL:			Not in use
Current pager:		stdout
Using outfile:		''
Using delimiter:	;
Server version:		8.0.32 MySQL Community Server - GPL
Protocol version:	10
Connection:		Localhost via UNIX socket
Server characterset:	utf8mb4
Db     characterset:	utf8mb4
Client characterset:	latin1
Conn.  characterset:	latin1
UNIX socket:		/var/run/mysqld/mysqld.sock
Binary data as:		Hexadecimal
Uptime:			5 min 37 sec

Threads: 2  Questions: 41  Slow queries: 0  Opens: 156  Flush tables: 3  Open tables: 74  Queries per second avg: 0.121
--------------

```

Подключитесь к восстановленной БД и получите список таблиц из этой БД.

```shell
mysql> use test_db
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+-------------------+
| Tables_in_test_db |
+-------------------+
| orders            |
+-------------------+
1 row in set (0.01 sec)

```

**Приведите в ответе** количество записей с `price` > 300.

```shell
mysql> SELECT * FROM orders WHERE price > 300;
+----+----------------+-------+
| id | title          | price |
+----+----------------+-------+
|  2 | My little pony |   500 |
+----+----------------+-------+
1 row in set (0.00 sec)

```

***
## Задача 2

Создайте пользователя test в БД c паролем test-pass, используя:

- плагин авторизации mysql_native_password
- срок истечения пароля — 180 дней 
- количество попыток авторизации — 3 
- максимальное количество запросов в час — 100
- аттрибуты пользователя:
    - Фамилия "Pretty"
    - Имя "James".

Предоставьте привелегии пользователю `test` на операции SELECT базы `test_db`.
    
Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES, получите данные по пользователю `test` и 
**приведите в ответе к задаче**.

### Решение:

```shell
mysql> CREATE USER 'test'@'localhost'
    -> IDENTIFIED WITH mysql_native_password BY 'test-pass'
    -> WITH MAX_QUERIES_PER_HOUR 100
    -> PASSWORD EXPIRE INTERVAL 180 DAY
    -> FAILED_LOGIN_ATTEMPTS 3
    -> ATTRIBUTE '{"fname": "James", "lname": "Pretty"}';
Query OK, 0 rows affected (0.04 sec)

mysql> GRANT SELECT ON test_db.* TO 'test'@'localhost';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> select * from information_schema.user_attributes where user='test';
+------+-----------+---------------------------------------+
| USER | HOST      | ATTRIBUTE                             |
+------+-----------+---------------------------------------+
| test | localhost | {"fname": "James", "lname": "Pretty"} |
+------+-----------+---------------------------------------+
1 row in set (0.01 sec)

```

***
## Задача 3

Установите профилирование `SET profiling = 1`.
Изучите вывод профилирования команд `SHOW PROFILES;`.

Исследуйте, какой `engine` используется в таблице БД `test_db` и **приведите в ответе**.

Измените `engine` и **приведите время выполнения и запрос на изменения из профайлера в ответе**:
- на `MyISAM`,
- на `InnoDB`.

### Решение:

```shell
mysql> SET profiling = 1;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> SHOW PROFILES;
+----------+------------+----------------------------------------+
| Query_ID | Duration   | Query                                  |
+----------+------------+----------------------------------------+
|        1 | 0.00038600 | SET profiling = 1                      |
|        2 | 0.00381250 | SELECT * FROM orders WHERE price > 300 |
+----------+------------+----------------------------------------+
2 rows in set, 1 warning (0.00 sec)

mysql> SELECT TABLE_NAME, ENGINE FROM information_schema.TABLES WHERE TABLE_SCHEMA = 'test_db';
+------------+--------+
| TABLE_NAME | ENGINE |
+------------+--------+
| orders     | InnoDB |
+------------+--------+
1 row in set (0.01 sec)

mysql> alter table orders engine = myisam;
Query OK, 5 rows affected (0.05 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> alter table orders engine = innodb;
Query OK, 5 rows affected (0.07 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> show profiles;
+----------+------------+-----------------------------------------------------------------------------------------+
| Query_ID | Duration   | Query                                                                                   |
+----------+------------+-----------------------------------------------------------------------------------------+
|        1 | 0.00038600 | SET profiling = 1                                                                       |
|        2 | 0.00381250 | SELECT * FROM orders WHERE price > 300                                                  |
|        3 | 0.00645025 | SELECT TABLE_NAME, ENGINE FROM information_schema.TABLES WHERE TABLE_SCHEMA = 'test_db' |
|        4 | 0.05026175 | alter table orders engine = myisam                                                      |
|        5 | 0.07118100 | alter table orders engine = innodb                                                      |
+----------+------------+-----------------------------------------------------------------------------------------+
5 rows in set, 1 warning (0.00 sec)

```

***
## Задача 4 

Изучите файл `my.cnf` в директории /etc/mysql.

Измените его согласно ТЗ (движок InnoDB):

- скорость IO важнее сохранности данных;
- нужна компрессия таблиц для экономии места на диске;
- размер буффера с незакомиченными транзакциями 1 Мб;
- буффер кеширования 30% от ОЗУ;
- размер файла логов операций 100 Мб.

Приведите в ответе изменённый файл `my.cnf`.

### Решение:

Для того чтобы после перезагрузки или пересоздания контейнера файл конфигурационный файл оставался модифицированным вносим изменение в конфигурацию докера:

```shell
dmitriy@dmitriy-lin:~/netology/docker/hw6-3$ cat docker-compose.yml 
version: '3.6'

services:
  mysql:
    image: mysql:8
    container_name: mysql_8
    environment:
      - MYSQL_ROOT_PASSWORD=qazwsxedc
    volumes:
      - /home/dmitriy/netology/docker/hw6-3/data:/var/lib/mysql
      - /home/dmitriy/netology/docker/hw6-3/backup:/mnt/backup
      - /home/dmitriy/netology/docker/hw6-3/mysql_conf/my.cnf:/etc/my.cnf

    ports:
      - "3306:3306"
    restart: always

dmitriy@dmitriy-lin:~/netology/docker/hw6-3$ sudo docker-compose up -d
Recreating mysql_8 ... done
dmitriy@dmitriy-lin:~/netology/docker/hw6-3$ sudo docker exec -it mysql_8 bash
bash-4.4# cat /etc/my.cnf
[mysqld]
skip-host-cache
skip-name-resolve
datadir=/var/lib/mysql
socket=/var/run/mysqld/mysqld.sock
secure-file-priv=/var/lib/mysql-files
user=mysql

pid-file=/var/run/mysqld/mysqld.pid
[client]
socket=/var/run/mysqld/mysqld.sock

!includedir /etc/mysql/conf.d/

# Скорость IO важнее сохранности данных
innodb_flush_method = O_DSYN
# Компрессия
innodb_file_per_table = 1
# Размер буффера с незакомиченными транзакциями 1 Мб
innodb_log_buffer_size = 1M
# Буффер кеширования
innodb_buffer_pool_size = 1G
# Размер файла логов операций
innodb_log_file_size = 100M
bash-4.4# exit
exit

```