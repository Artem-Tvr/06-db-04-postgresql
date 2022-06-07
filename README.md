# Домашнее задание к занятию "6.4. PostgreSQL"

## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.

Подключитесь к БД PostgreSQL используя `psql`.

Воспользуйтесь командой `\?` для вывода подсказки по имеющимся в `psql` управляющим командам.

**Найдите и приведите** управляющие команды для:

* вывода списка БД - *\l[+]   [PATTERN]    list databases*
* подключения к БД - *\c[onnect] {[DBNAME|- USER|- HOST|- PORT|-] | conninfo}    connect to new database (currently "test_database")*
* вывода списка таблиц - *\dt[S+] [PATTERN] list tables*
* вывода описания содержимого таблиц - *\d[S+] NAME    describe table, view, sequence, or index*
* выхода из psql - \q     quit psql


```
root@cadcixztkv:~/hw-postgresql# docker-compose up
Creating network "hw-postgresql_default" with the default driver
Creating volume "hw-postgresql_data-vol" with default driver
Creating volume "hw-postgresql_backup-vol" with default driver
Creating hw-postgresql_postgres_1 ... done
Attaching to hw-postgresql_postgres_1
postgres_1  | The files belonging to this database system will be owned by user "postgres".
postgres_1  | This user must also own the server process.
postgres_1  |
postgres_1  | The database cluster will be initialized with locale "en_US.utf8".
postgres_1  | The default database encoding has accordingly been set to "UTF8".
postgres_1  | The default text search configuration will be set to "english".
postgres_1  |
postgres_1  | Data page checksums are disabled.
postgres_1  |
postgres_1  | fixing permissions on existing directory /var/lib/postgresql/data/pgdata ... ok
postgres_1  | creating subdirectories ... ok
postgres_1  | selecting dynamic shared memory implementation ... posix
postgres_1  | selecting default max_connections ... 100
postgres_1  | selecting default shared_buffers ... 128MB
postgres_1  | selecting default time zone ... Etc/UTC
postgres_1  | creating configuration files ... ok
postgres_1  | running bootstrap script ... ok
postgres_1  | performing post-bootstrap initialization ... ok
postgres_1  | syncing data to disk ... ok
postgres_1  |
postgres_1  | initdb: warning: enabling "trust" authentication for local connections
postgres_1  | You can change this by editing pg_hba.conf or using the option -A, or
postgres_1  | --auth-local and --auth-host, the next time you run initdb.
postgres_1  |
postgres_1  | Success. You can now start the database server using:

```


```
root@cadcixztkv:~# docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                                       NAMES
71622f52fa94   postgres:13   "docker-entrypoint.s…"   4 minutes ago   Up 4 minutes   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   hw-postgresql_postgres_1
root@cadcixztkv:~# docker exec -ti 71622f52fa94 bash
root@71622f52fa94:/# cd /backups
root@71622f52fa94:/backups# ls -la
total 12
drwxr-xr-x 2 root root 4096 Jun  6 15:41 .
drwxr-xr-x 1 root root 4096 Jun  6 15:38 ..
-rw-r--r-- 1 root root 2082 Jun  6 15:41 test_dump.sql
root@71622f52fa94:/backups# psql -U devops
psql (13.7 (Debian 13.7-1.pgdg110+1))
Type "help" for help.

```



```
root@71622f52fa94:/# psql -U devops
psql (13.7 (Debian 13.7-1.pgdg110+1))
Type "help" for help.

devops=#
```


## Задача 2

Используя `psql` создайте БД `test_database`.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/master/06-db-04-postgresql/test_data).

Восстановите бэкап БД в `test_database`.

Перейдите в управляющую консоль `psql` внутри контейнера.

Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.

Используя таблицу [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы `orders` с наибольшим средним значением размера элементов в байтах.

**Приведите в ответе** команду, которую вы использовали для вычисления и полученный результат.

*Ответ:*

```
root@71622f52fa94:/backups# psql -U devops
psql (13.7 (Debian 13.7-1.pgdg110+1))
Type "help" for help.
devops=# CREATE DATABASE test_database;
CREATE DATABASE

```

```
root@71622f52fa94:/# psql -U devops test_database < /backups/test_dump.sql
SET
SET
SET
SET
SET
 set_config
------------

(1 row)
```


```
devops=# \l
                                  List of databases
      Name       |  Owner   | Encoding |  Collate   |   Ctype    | Access privileges
-----------------+----------+----------+------------+------------+-------------------
 devops          | devops   | UTF8     | en_US.utf8 | en_US.utf8 |
 postgres        | devops   | UTF8     | en_US.utf8 | en_US.utf8 |
 template0       | devops   | UTF8     | en_US.utf8 | en_US.utf8 | =c/devops        +
                 |          |          |            |            | devops=CTc/devops
 template1       | devops   | UTF8     | en_US.utf8 | en_US.utf8 | =c/devops        +
                 |          |          |            |            | devops=CTc/devops
 test_database   | devops   | UTF8     | en_US.utf8 | en_US.utf8 |
 test_database_2 | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
(6 rows)

devops=# \c test_database;
You are now connected to database "test_database" as user "devops".
test_database=# \dt
         List of relations
 Schema |  Name  | Type  |  Owner
--------+--------+-------+----------
 public | orders | table | postgres
(1 row)
```


```
test_database=# ANALYZE VERBOSE public.orders;
INFO:  analyzing "public.orders"
INFO:  "orders": scanned 1 of 1 pages, containing 8 live rows and 8 dead rows; 8 rows in sample, 8 estimated total rows
ANALYZE

```


```
test_database=# select avg_width from pg_stats where tablename='orders';
 avg_width
-----------
         4
        16
         4
(3 rows)

```


## Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и поиск по ней занимает долгое время. Вам, как успешному выпускнику курсов DevOps в нетологии предложили провести разбиение таблицы на 2 (шардировать на orders_1 - price>499 и orders_2 - price<=499).

Предложите SQL-транзакцию для проведения данной операции.

Можно ли было изначально исключить "ручное" разбиение при проектировании таблицы orders?

*Ответ:*

```
test_database=# CREATE TABLE orders_1 ( CHECK ( price > 499 ) ) INHERITS (orders);
CREATE TABLE
test_database=# CREATE TABLE orders_2 ( CHECK ( price <= 499 ) ) INHERITS (orders);
CREATE TABLE
test_database=# CREATE RULE orders_insert_to_1 AS ON INSERT TO orders WHERE ( price > 499 ) DO INSTEAD INSERT INTO orders_1 VALUES (NEW.*);
CREATE RULE
test_database=# CREATE RULE orders_insert_to_2 AS ON INSERT TO orders WHERE ( price <= 499 ) DO INSTEAD INSERT INTO orders_2 VALUES (NEW.*);
CREATE RULE
test_database=#

```

Да, можно было исключить ручное разбиение, если изначально предполагать большое количество данных, разбить таблицы во время проектирования БД.

## Задача 4

Используя утилиту `pg_dump` создайте бекап БД `test_database`.

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?

*Ответ:*

root@71622f52fa94:/# pg_dump -U devops > /backups/my_backup.sql

Для уникальности можно добавить, например, индекс.

```
CREATE INDEX ON orders ((unicum(title)));
```
