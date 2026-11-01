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

<img width="920" height="255" alt="Снимок экрана 2026-11-01 в 17 59 17" src="https://github.com/user-attachments/assets/31368ad7-fc4f-4821-88f0-19c91373a39c" />


=========================================================
Репликация БД
1. физическая
WAL - файлы


2. логичкская
сами изменения

==========================================================

Репликация в PostgreSQL — это механизм копирования данных с одного сервера (Мастера или Primary) на один или несколько других серверов (Реплик или Standby).
Основой современной репликации в Postgres являются WAL-файлы (Write-Ahead Logs — журналы предзаписи).

Что такое WAL-файлы и их роль в репликации?

Когда в базе данных происходит любое изменение (например, INSERT или UPDATE), PostgreSQL работает по принципу «сначала запиши в журнал»:
1. Изменение сначала записывается в бинарный лог на диске — WAL-файл.
2. Только после этого изменения вносятся в файлы самих таблиц.
Каждый WAL-файл имеет фиксированный размер (по умолчанию 16 МБ).
В репликации WAL — это источник правды. Вместо того чтобы пересылать тяжелые таблицы, Мастер просто отправляет Реплике поток своих WAL-файлов. Реплика «проигрывает» эти логи у себя и получает точную копию Мастера.

Виды репликации в PostgreSQL

Репликацию разделяют по двум главным критериям: по способу передачи данных (Физическая vs Логическая) и по моменту подтверждения транзакции (Асинхронная vs Синхронная).

1. Разница по способу передачи данных


Критерий	Физическая (Потоковая) репликация	Логическая репликация
Что передается	WAL-файлы целиком (байт-в-байт копия диска).	Логические команды (стриминг изменений вроде «в таблицу Х добавлена строка Y»).
Масштаб	Копируется весь сервер целиком (все базы данных и настройки).	Можно выбрать отдельные таблицы или конкретную базу данных.
Режим Реплики	Только для чтения (Read-Only). Запись запрещена.	На Реплике можно писать свои данные в другие таблицы или создавать индексы.
Версии Postgres	Версии Мастера и Реплики должны строго совпадать (например, обе 16).	Можно реплицировать данные между разными версиями (например, с 14 на 17).
Основной кейс	Отказоустойчивость (HA), бэкапы, снятие нагрузки по чтению.	Перенос данных без простоя, сбор данных из разных БД в одно хранилище.


2. Разница по моменту подтверждения (Синхронность)

• Асинхронная репликация (по умолчанию):
	• Как работает: Мастер записывает WAL у себя, подтверждает клиенту успех транзакции и только потом отправляет WAL на реплику.
	• Плюсы: Максимальная скорость работы. Мастер не ждет реплику.
	• Минусы: Риск потери данных. Если Мастер «умрет» до того, как WAL долетит до реплики, последние транзакции пропадут.
• Синхронная репликация:
	• Как работает: Мастер принял транзакцию, отправил WAL на реплику, дождался от неё подтверждения (что она получила или применила WAL) и только после этого говорит клиенту «готово».
	• Плюсы: Нулевая потеря данных (RPO=0). При поломке Мастера реплика гарантированно актуальна.
	• Минусы: Запись становится медленнее (зависит от сетевой задержки до реплики). Если реплика «зависнет», Мастер тоже заблокирует операции записи.

Как именно WAL передаются на реплику?

В физической репликации есть два метода доставки WAL-файлов:
1. Файловый (WAL-shipping): Мастер полностью заполняет файл 16 МБ, закрывает его и через команду archive_command (например, утилитой rsync) перекидывает файл на реплику. Реплика его подхватывает. Минус: задержка данных может составлять до 16 МБ изменений.
2. Потоковый (Streaming): Специальный процесс на реплике (walreceiver) подключается к мастеру (walsender) и скачивает WAL-данные непрерывным потоком, буквально побайтово. Это стандарт для современных систем.
Вы планируете настроить отказоустойчивый кластер (тогда нужна физическая потоковая репликация) или хотите выборочно переносить таблицы между разными базами (нужна логическая)?




Архитектура:

1. master-slave

2. master-master

sudo apt update
sudo apt install postgresql repmgr 


epmgr - (Replication Manager) — это самая популярная и проверенная временем утилита с открытым исходным кодом для управления репликацией и обеспечения высокой доступности (High Availability) в кластерах PostgreSQL
это ставить и на master и на slave

раскидать ключи пользователя постргрес между вмками
на master
sudo -u postgres ssh-keygen -t ed 25519 -N "" -f /var/lib/postgresql/.ssh/id_ed25519
sudo cat /var/lib/posgresql/.ssh/id_ed25519.pub 
sudo -u postgres nano /var/lib/postgresql/.ssh/authorized_keys - вставить ключ с slave

на slave 
sudo -u postgres ssh-keygen -t ed 25519 -N "" -f /var/lib/postgresql/.ssh/id_ed25519
sudo cat /var/lib/posgresql/.ssh/id_ed25519.pub 
sudo -u postgres nano /var/lib/postgresql/.ssh/authorized_keys - вставить ключ с master


входим по ip с каждой машины по очереди
sudo -u postgres ssh <ip>


=============================
НА МАСТЕР
настраивваем конфиги
sudo nano /etc/postgresql/18/main/postgresql.conf

listen_addresses = '*' - или приватный ip

wal_level = replica

max_wal_senders = 10
max_replication_slots = 10
max_keep_size = 256 или 262144

hot_standby = on - настрока позволяющая посылаь на него read запросы


настрйока replication connetcion DB между master и slave

sudo nano /etc/postgresql/18/main/pg_hba.conf

в Allow replication connection from localhost, by a user with the replication privilege.

host      replication       repmgr         ip master хоста с масокой        scram-sha-256
host      replication       repmgr         ip slave хоста с масокой         scram-sha-256
host      repmgr            repmgr         ip master хоста с масокой        scram-sha-256
host      repmgr            repmgr         ip slave хоста с масокой         scram-sha-256


sudo systemctl restart postgresql

sudo ss -nlpt - проверка что postgres на порту 5432 слушает
=============================
НА SLAVE - повторить все тоже самое

=============================

мастер и slave - настройка пользователей postgres и repmgr

sudo -u postgres psql
CREATE ROLE repmgr WITH LOGIN SUPERUSER PASSWORD 'repmgrpass';
CREATE DATABASE repmgr OWNER repmgr;
\q


Создать файлик для хранения пароля (сдедать для мастера и слэйв)
````
sudo su - postgres
nano /var/lib/postgresql/.pgpass
<ip>:5432:repmgr:repmgr:repmgrpass - мастер
<ip>:5432:replication:replication:repmgrpass  - мастер
<ip>:5432:repmgr:repmgr:repmgrpass - мастер   - slave
<ip>:5432:replication:replication:repmgrpass  - slave

exit
````

Создать конфиг реп менеджер - мастер
````sudo nano /etc/repmgr.conf````
````
node_id=1
node_name=pg-1
conninfo='host=<ip-1> user=repmgr dbname=repmgr connect_timeout=5'
data_directory='/var/lib/postgresql/18/main'
pg-bindir='/var/lib/postgresql/16/bin'
passfile='/var/lib/postgresql/.pgpass'

failover=automatic
promote_command='/usr/bin/repmgr standby promote -f /etc/repmgr.conf'
follow_command='/usr/bin/repmgr standby promote -f /etc/repmgr.conf --upstream-node-id=%n'

monitoring_history=yes
monito_interval_sec=5
reconnect_attemps=5
reconnect_interval=3
````
задать владельца созданым когфигам
````sudo chown postgres:postgres /etc/repmgt.conf````

для slave
````sudo nano /etc/repmgr.conf````
````
node_id=2
node_name=pg-2
conninfo='host=<ip-2> user=repmgr dbname=repmgr connect_timeout=5'
data_directory='/var/lib/postgresql/18/main'
pg-bindir='/var/lib/postgresql/16/bin'
passfile='/var/lib/postgresql/.pgpass'

failover=automatic
promote_command='/usr/bin/repmgr standby promote -f /etc/repmgr.conf'
follow_command='/usr/bin/repmgr standby promote -f /etc/repmgr.conf --upstream-node-id=%n'

monitoring_history=yes
monito_interval_sec=5
reconnect_attemps=5
reconnect_interval=3
````
задать владельца созданым когфигам
````sudo chown postgres:postgres /etc/repmgt.conf````


Регистрация мастера как мастера:
````sudo -u postgres repmgr -f /etc/repmgr.conf primary register````




















