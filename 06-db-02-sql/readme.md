# Домашнее задание к занятию "6.2. SQL"

## Введение

Перед выполнением задания вы можете ознакомиться с 
[дополнительными материалами](https://github.com/netology-code/virt-homeworks/tree/master/additional/README.md).

## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose манифест.

```
user@WS-045:~$ docker run --rm --name pg12 -e POSTGRES_PASSWORD=test -d -p 5432:5432 -v vol-pg-data:/var/lib/postgresql/data -v vol-pg-backup:/var/lib/postgresql/backup postgres:12
d74b983666a914c56d4767e046acc1c2b56a4f857a13eaf2dcc08119bfdb5333
user@WS-045:~$ psql -h 127.0.0.1 -U postgres
Password for user postgres: 
psql (12.9 (Ubuntu 12.9-0ubuntu0.20.04.1), server 12.10 (Debian 12.10-1.pgdg110+1))
Type "help" for help.

postgres-# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges   
-----------+----------+----------+------------+------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
(3 rows)
```

## Задача 2

В БД из задачи 1: 
- создайте пользователя test-admin-user и БД test_db
```
postgres=# CREATE USER "test-admin-user" WITH SUPERUSER;
CREATE ROLE
postgres=# CREATE DATABASE test_db;
CREATE DATABASE
```
- в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже)
```
postgres=# \connect test_db
psql (12.9 (Ubuntu 12.9-0ubuntu0.20.04.1), server 12.10 (Debian 12.10-1.pgdg110+1))
You are now connected to database "test_db" as user "postgres".

test_db=# CREATE TABLE orders
(
id integer PRIMARY KEY,
name text,
price integer
);
CREATE TABLE
test_db=# CREATE TABLE clients
(
id integer PRIMARY KEY,
lastname text,
country text,
ord integer REFERENCES orders(id)
);
CREATE TABLE
```
- предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db
```
test_db=# GRANT ALL PRIVELEGES ON DATABASE test_db to test-admin-user
```
- создайте пользователя test-simple-user  
```
test_db-# CREATE USER "test-simple-user";
```
- предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE данных таблиц БД test_db
```
test_db=# GRANT SELECT, INSERT, UPDATE, DELETE ON TABLE orders to "test-simple-user";
GRANT
test_db=# GRANT SELECT, INSERT, UPDATE, DELETE ON TABLE clients to "test-simple-user";
GRANT
```


Таблица orders:
- id (serial primary key)
- наименование (string)
- цена (integer)

Таблица clients:
- id (serial primary key)
- фамилия (string)
- страна проживания (string, index)
- заказ (foreign key orders)

Приведите:
- итоговый список БД после выполнения пунктов выше
```
test_db=# \l
                                     List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |       Access privileges        
-----------+----------+----------+------------+------------+--------------------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres                   +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres                   +
           |          |          |            |            | postgres=CTc/postgres
 test_db   | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =Tc/postgres                  +
           |          |          |            |            | postgres=CTc/postgres         +
           |          |          |            |            | "test-admin-user"=CTc/postgres
(4 rows)

test_db=# \dt
          List of relations
 Schema |  Name   | Type  |  Owner   
--------+---------+-------+----------
 public | clients | table | postgres
 public | orders  | table | postgres
(2 rows)

test_db=# \du
                                       List of roles
    Role name     |                         Attributes                         | Member of 
------------------+------------------------------------------------------------+-----------
 postgres         | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 test-admin-user  | Superuser                                                  | {}
 test-simple-user |                                                            | {}
```
- описание таблиц (describe)
```
test_db=# \d orders
               Table "public.orders"
 Column |  Type   | Collation | Nullable | Default 
--------+---------+-----------+----------+---------
 id     | integer |           | not null | 
 name   | text    |           |          | 
 price  | integer |           |          | 
Indexes:
    "orders_pkey" PRIMARY KEY, btree (id)
Referenced by:
    TABLE "clients" CONSTRAINT "clients_ord_fkey" FOREIGN KEY (ord) REFERENCES orders(id)

test_db=# \d clients
               Table "public.clients"
  Column  |  Type   | Collation | Nullable | Default 
----------+---------+-----------+----------+---------
 id       | integer |           | not null | 
 lastname | text    |           |          | 
 country  | text    |           |          | 
 ord      | integer |           |          | 
Indexes:
    "clients_pkey" PRIMARY KEY, btree (id)
Foreign-key constraints:
    "clients_ord_fkey" FOREIGN KEY (ord) REFERENCES orders(id)
```
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db
```
test_db=# SELECT * FROM information_schema.table_privileges WHERE grantee in ('test-admin-user', 'test-simple-user');
```
- список пользователей с правами над таблицами test_db
```
 grantor  |     grantee      | table_catalog | table_schema | table_name | privilege_type | is_grantable | with_hierarchy 
----------+------------------+---------------+--------------+------------+----------------+--------------+----------------
 postgres | test-simple-user | test_db       | public       | orders     | INSERT         | NO           | NO
 postgres | test-simple-user | test_db       | public       | orders     | SELECT         | NO           | YES
 postgres | test-simple-user | test_db       | public       | orders     | UPDATE         | NO           | NO
 postgres | test-simple-user | test_db       | public       | orders     | DELETE         | NO           | NO
 postgres | test-simple-user | test_db       | public       | clients    | INSERT         | NO           | NO
 postgres | test-simple-user | test_db       | public       | clients    | SELECT         | NO           | YES
 postgres | test-simple-user | test_db       | public       | clients    | UPDATE         | NO           | NO
 postgres | test-simple-user | test_db       | public       | clients    | DELETE         | NO           | NO
(8 rows)
```

## Задача 3

Используя SQL синтаксис - наполните таблицы следующими тестовыми данными:

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

Используя SQL синтаксис:
- вычислите количество записей для каждой таблицы 
- приведите в ответе:
    - запросы 
    - результаты их выполнения.

```
test_db=# INSERT INTO orders VALUES (1,'Шоколад',10),
                                    (2,'Принтер',3000),
                                    (3,'Книга',500),
                                    (4,'Монитор',7000),
                                    (5,'Гитара',4000);
INSERT 0 5
test_db=# INSERT INTO clients VALUES (1,'Иванов Иван Иванович','USA'),
                                     (2,'Петров Петр Петрович','Canada'),
                                     (3,'Иоганн Себастьян Бах','Japan'),
                                     (4,'Ронни Джеймс Дио','Russia'),
                                     (5,'Richie Blackmore','Russia');
INSERT 0 5
test_db=# SELECT count (*) FROM orders;
 count 
-------
     5
(1 row)

test_db=# SELECT count (*) FROM clients;
 count 
-------
     5
(1 row)
```

## Задача 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys свяжите записи из таблиц, согласно таблице:

|ФИО|Заказ|
|------------|----|
|Иванов Иван Иванович| Книга |
|Петров Петр Петрович| Монитор |
|Иоганн Себастьян Бах| Гитара |

Приведите SQL-запросы для выполнения данных операций.

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод данного запроса.
 
Подсказк - используйте директиву `UPDATE`.
```
test_db=# UPDATE clients SET ord = 3 WHERE id = 1;
UPDATE 1
test_db=# UPDATE clients SET ord = 4 WHERE id = 2;
UPDATE 1
test_db=# UPDATE clients SET ord = 5 WHERE id = 3;
UPDATE 1

test_db=# SELECT lastname FROM clients WHERE ord IS NOT NULL;
       lastname       
----------------------
 Иванов Иван Иванович
 Петров Петр Петрович
 Иоганн Себастьян Бах
(3 rows)
```

## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).

Приведите получившийся результат и объясните что значат полученные значения.
```
test_db=# EXPLAIN SELECT lastname FROM clients WHERE ord IS NOT NULL;
                        QUERY PLAN                         
-----------------------------------------------------------
 Seq Scan on clients  (cost=0.00..18.10 rows=806 width=32)
   Filter: (ord IS NOT NULL)
(2 rows)


```

## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. Задачу 1).

Остановите контейнер с PostgreSQL (но не удаляйте volumes).

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления. 

---
