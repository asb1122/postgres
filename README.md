# postgres

sudo apt update
sudo apt install -y postgresql postgresql-contrib
sudo systemctl status postgresql

<img width="678" height="168" alt="Снимок экрана 2026-11-01 в 11 13 54" src="https://github.com/user-attachments/assets/62890019-8afb-4dbf-ab81-b1e0f72fcca2" />

провекра процесса
ps aufx | grep [p]ostgres
<img width="943" height="258" alt="Снимок экрана 2026-11-01 в 11 15 50" src="https://github.com/user-attachments/assets/1573636e-ddb6-41af-8112-fce06617dada" />

вход в postgres
sudo su - postgres

воход в среду
psql

\l  - покажет все талблицы которые есть

\du  - пользователи которые есть в базе

создание базы univer
create database univer;

\c univer - переклбчение на созданную базу

создание таблицы группы
create table groups (group_id serial primary key, group_name varchar(50) not null unique, start_date date not null);

\dt - выводит список всех таблиц в текущей базе данных

создание таблицы стеденты с референсом на таблицу groups(group_id)
create table students (student_id serial primary key, name varchar(200) not null, email varchar(255) not null unique, group_id int not null references groups(group_id));

create table cources (cource_id serial primary key, title varchar(255) not null, description text);

создание связущей таблицы
create table temp (cource_id int not null references cources(cource_id), student_id int not null references students(student_id));

\dt - выводит список всех таблиц в текущей базе данных

````univer=# \dt
            List of tables
 Schema |   Name   | Type  |  Owner   
--------+----------+-------+----------
 public | cources  | table | postgres
 public | groups   | table | postgres
 public | students | table | postgres
 public | temp     | table | postgres
(4 rows)

````
добавление данных в таблицу 
````
insert into groups (group_name, start_date) values ('dos-36', '2026-10-14'), ('dos-37', '2026-10-15');
````
````
insert into students (name, email, group_id) values ('I.Ivanov', 'iivanov@edu.com', 1), ('P.Petrov', 'ppetrov@edu.com', 2);
````
````
insert into cources (title, description) values ('DevOps Full', 'Full devops course'), ('K8S', 'only kubernetes');
````
````
insert into temp (cource_id, student_id) values (2, 1), (1, 2);
````
Select примеры:
````
select * from students;
````
````
 student_id |   name   |      email      | group_id 
------------+----------+-----------------+----------
          1 | I.Ivanov | iivanov@edu.com |        1
          2 | P.Petrov | ppetrov@edu.com |        2
(2 rows)
````

пример запроса где s выступает как аллиас для сокращения длины запроса 
````
select s.name from students s;
````
````
   name   
----------
 I.Ivanov
 P.Petrov
(2 rows)
````

селекст на выборку из нескольких таблиц (студенты и группы) где условие совпедение id групп
````
select s.name, g.group_name from students s join groups g  on g.group_id = s.group_id;
````
````
   name   | group_name 
----------+------------
 I.Ivanov | dos-36
 P.Petrov | dos-37
(2 rows) 
````

````
select s.name, c.title from temp t join students s on s.student_id = t.student_id join cources c on c.cource_id = t.cource_id;
````
````
   name   |    title    
----------+-------------
 I.Ivanov | K8S
 P.Petrov | DevOps Full
(2 rows)
````

транзакции - либо все запросы выполнились, если один запрос упал, то ничего не применяется
````begin;````
````
insert into groups (group_name, start_date) values ('dos-34', '20206-10-13');
````
возврат в случае падения
````
rollback;
````
````exit````

Создание пользователя:
````
create role devops with login password 'devopspass';
````
проверка
````
postgres=# \du
                              List of roles
 Role name  |                         Attributes                         
------------+------------------------------------------------------------
 carss_user | 
 devops     | 
 postgres   | Superuser, Create role, Create DB, Replication, Bypass RLS
````
создание базы данных с владельцем пользователем devops
````
create database infra owner devops;
````
проверка
````
postgres=# \l
                                                        List of databases
   Name    |   Owner    | Encoding | Locale Provider |   Collate   |    Ctype    | Locale | ICU Rules |     Access privileges     
-----------+------------+----------+-----------------+-------------+-------------+--------+-----------+---------------------------
 autos     | carss_user | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | 
 carss     | carss_user | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | =Tc/carss_user           +
           |            |          |                 |             |             |        |           | carss_user=CTc/carss_user
 infra     | devops     | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | 
 postgres  | postgres   | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | 
 template0 | postgres   | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | =c/postgres              +
           |            |          |                 |             |             |        |           | postgres=CTc/postgres
 template1 | postgres   | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | =c/postgres              +
           |            |          |                 |             |             |        |           | postgres=CTc/postgres
 univer    | postgres   | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | 
(7 rows)
````

для выдачи прав на доспут в базу вновь созданному пользаку нужно внести настроки в этот файл:
````sudo nano /etc/postgresql/18/main/pg_hba.conf ````

# Database administrative login by Unix domain socket
# TYPE  DATABASE        USER            ADDRESS                 METHOD
# IPv4 local connections:
...
host    infra           devops          0.0.0.0/0               scram-sha-256
...

и в ````sudo nano /etc/postgresql/18/main/postgresql.conf````
#------------------------------------------------------------------------------
# CONNECTIONS AND AUTHENTICATION
#------------------------------------------------------------------------------

# - Connection Settings -

listen_addresses = '*'                  # what IP address(es) to listen on;

после - рестартануть сервис
````sudo systemctl restart postgresql````
и првоерить порт 5432
````
sudo ss -nlpt | grep 5432
LISTEN 0      200          0.0.0.0:5432      0.0.0.0:*    users:(("postgres",pid=534764,fd=6))                       
LISTEN 0      200             [::]:5432         [::]:*    users:(("postgres",pid=534764,fd=7)) 
````

и выполнить вход в среду postgres
````
psql -U devops -h 127.0.0.1 -d infra -W

````

!!!!!!!!!!!!!!!!РАЗОБРАТЬСЯ КАК ЗАПУСТИТЬ PG АДМИН!!!!!!!!!!!!!!!



Бэкапы:








