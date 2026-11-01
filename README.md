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




