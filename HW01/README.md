### Домашнее задание - Работа с уровнями изоляции транзакции в PostgreSQL

- создать виртуальную машину в гипервизоре VMWare ESXi 
  c параметрами vCPU=2/vRAM=4G/vHDD=40G
  <details><summary><i> О системе</i></summary>
    <img src=../Images/hw-01-01.jpg alt='Параметры ВМ'>
  </details>
- установить  ubuntu-2404 (host ubutest; user boss; ip 192.168.1.244)
  <details><summary><i> Аннотация терминала </i></summary>
  
        Welcome to Ubuntu 24.04.1 LTS (GNU/Linux 6.8.0-49-generic x86_64)

         * Documentation:  https://help.ubuntu.com
         * Management:     https://landscape.canonical.com
         * Support:        https://ubuntu.com/pro

         System information as of Чт 17 ноя 2024 11:58:29 UTC

          System load:  0.0                Processes:              226
          Usage of /:   18.6% of 38.04GB   Users logged in:        0
          Memory usage: 5%                 IPv4 address for ens34: 192.168.1.244
          Swap usage:   0%
  </details>      

- настроить доступ по ssh ключу <br>
    Воспользуемся под <b>Windows</b> терминалом <b>MobaXTerm</b> <br>
    Подключимся локальной сессией (псевдо Linux) и создадим ключи ssh    

        /home/mobaxterm > ssh-keygen -t ed25519
        Generating public/private ed25519 key pair.
        Enter file in which to save the key (/home/mobaxterm/.ssh/id_ed25519):
        Enter passphrase (empty for no passphrase):
        Enter same passphrase again:
        Your identification has been saved in /home/mobaxterm/.ssh/id_ed25519
        Your public key has been saved in /home/mobaxterm/.ssh/id_ed25519.pub
        The key fingerprint is:
        SHA256:yNFYNoA/D1BihpJNy3aU6LCm0DNNLxylSizmI21pxro User@DESKTOP-TFJ0G4S
        The key's randomart image is:
        +--[ED25519 256]--+
        | +.o==+.+        |
        |+o+=*o = .       |
        |.B*+o=o .        |
        |=B=++.=o         |
        |=oOo .o+S        |
        |o=.     .        |
        |.                |
        | .               |
        |E                |
        +----[SHA256]-----+


    Копируем ключи на виртуальную машину      

        /home/mobaxterm > ssh-copy-id boss@192.168.1.244
        /bin/ssh-copy-id: INFO: Source of key(s) to be installed: "./.ssh/id_ed25519.pub"
        /bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
        /bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
        stty: standard input: Inappropriate ioctl for device

        Number of key(s) added: 1

        Now try logging into the machine, with:   "ssh 'boss@192.168.1.244'"
        and check to make sure that only the key(s) you wanted were added.

    При настройке новых подключений используем закрытый ключ (файл id_ed25519) <br>
        в папке __\_AppDataDir\_\MobaXterm\home\\.ssh__
- установить PostgreSQL 
  
    Установим версию PostgreSQL_1C из репозитория PostgresPro
      <details><summary><i>Подробнее о данной версии</i></summary>
        - Открытый репозиторий бесплатной версии PostgreSQL с расширениями для работы с 1C. <br>
        - Есть скрипт - добавляющий репозиторий для разных Linux <br>
        - Однотипная установка и настройка на разных Linux <br>
        - Возможность простого локального зеркала по версиям и Linux <br>
        - При установке работет скрипт (аналог PGTune) определяющий параметры сервера <br>
          и изменяющий конфигурацию по умолчанию для PostgreSQL 
      </details>
   
   загружаем скрипт регистрации репозитория, регистрируем репозиторий, устанавливаем СУБД

```bash
    wget https://repo.postgrespro.ru/1c/1c-16/keys/pgpro-repo-add.sh 
    chmod +x ./pgpro-repo-add.sh
    sudo ./pgpro-repo-add.sh 
    sudo apt install postgrespro-1c-16
```

```
    boss@ubutest:~$ sudo systemctl status postgrespro-1c-16.service
    ● postgrespro-1c-16.service - Postgres Pro 1c 16 database server
         Loaded: loaded (/usr/lib/systemd/system/postgrespro-1c-16.service; enabled; preset: enabled)
        Active: active (running) since Tue 2024-11-17 12:43:23 UTC; 1min 4s ago
        Process: 20776 ExecStartPre=/opt/pgpro/1c-16/bin/check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
       Main PID: 20787 (postgres)
          Tasks: 7 (limit: 6968)
         Memory: 62.1M (peak: 66.1M)
            CPU: 260ms
         CGroup: /system.slice/postgrespro-1c-16.service
                 ├─20787 /opt/pgpro/1c-16/bin/postgres -D /var/lib/pgpro/1c-16/data
                 ├─20802 "postgres: logger "
                 ├─20803 "postgres: checkpointer "
                 ├─20804 "postgres: background writer "
                 ├─20806 "postgres: walwriter "
                 ├─20807 "postgres: autovacuum launcher "
                 └─20808 "postgres: logical replication launcher "

    ноя 17 12:43:23 ubutest systemd[1]: Starting postgrespro-1c-16.service - Postgres Pro 1c 16 database server...
    ноя 17 12:43:23 ubutest postgres[20787]: 2024-11-17 12:43:23.497 UTC [20787] СООБЩЕНИЕ:  передача вывода в протокол процессу сбора прото>
    ноя 17 12:43:23 ubutest postgres[20787]: 2024-11-17 12:43:23.497 UTC [20787] ПОДСКАЗКА:  В дальнейшем протоколы будут выводиться в катал>
    ноя 17 12:43:23 ubutest systemd[1]: Started postgrespro-1c-16.service - Postgres Pro 1c 16 database server.
```


$\textsf{\color{orange}Подключимся второй сессией ssh}$
```
boss@ubutest:~$ sudo su - postgres
postgres@ubutest:~$ psql
psql (16.4)
Введите "help", чтобы получить справку.

postgres=# \c
Вы подключены к базе данных "postgres" как пользователь "postgres".
postgres=#select version();
                                         version
--------------------------------------------------------------------------------------------------
 PostgreSQL 16.4 on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 13.2.0-23ubuntu4) 13.2.0, 64-bit
(1 строка)
postgres=#
```

$\textsf{\color{blue}Первая сессия}$
```
boss@ubutest:~$ sudo su - postgres
postgres@ubutest:~$ psql
psql (16.4)
Введите "help", чтобы получить справку.

postgres=# \c
Вы подключены к базе данных "postgres" как пользователь "postgres".
postgres=#
```
$\textsf{\color{blue}Создаём тестовую базу и выключаем auto commit}$
```
postgres=# create database otus1;
CREATE DATABASE
postgres=# \c otus1
Вы подключены к базе данных "otus1" как пользователь "postgres".
otus1=# \set autocommit off
otus1=# \echo :autocommit
off
otus1=#
```
$\textsf{\color{blue}В тестовой БД создаём таблицу и заполняем её данными}$
```
otus1=# begin;
BEGIN
otus1=*# create table persons(id serial, first_name text, second_name text);
insert into persons(first_name, second_name) values('ivan', 'ivanov');
insert into persons(first_name, second_name) values('petr', 'petrov');
CREATE TABLE
INSERT 0 1
INSERT 0 1
otus1=*# commit;
COMMIT
```
$\textsf{\color{blue}Посмотрим текущий уровень изоляции}$
```
otus1=# show transaction isolation level;
 transaction_isolation
-----------------------
 read committed
(1 строка)
```
Начать новую транзакцию в обеих сессиях
```
otus1=# begin;
BEGIN
otus1=*#
```
        / * перед # указывает что это незавершённая транзакция
$\textsf{\color{blue}Добавить запись ( первая сессия )}$
```
otus1=*# insert into persons(first_name, second_name) values('sergey', 'sergeev');
INSERT 0 1
otus1=*#
```
$\textsf{\color{orange}Выбрать записи из таблицы ( вторая сессия )}$
```
otus1=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 строки)

otus1=*#
```
*Новых записей нет - транзакция в первой сессии не завершена (не видим изменённые данные в других транзакциях)* <br>

$\textsf{\color{blue}Завершить первую транзакцию}$
```
otus1=*# commit;
COMMIT
otus1=#
```
$\textsf{\color{orange}Выбрать записи из таблицы ( вторая сессия )}$
```
otus1=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 строки)

otus1=*#
```
*Новые записи есть - в первой сессии транзакция завершена (видим зафиксированные данные из других сессий)* <br>

$\textsf{\color{orange}Завершить вторую транзакцию}$
```
otus1=*# commit;
COMMIT
otus1=#
```

Начать новые транзакции с изоляцией **repeatable read**
```
otus1=# begin;
BEGIN
otus1=*# set transaction isolation level repeatable read;
SET
otus1=*# show transaction isolation level;
 transaction_isolation
-----------------------
 repeatable read
(1 строка)

otus1=*#
```
$\textsf{\color{blue}Добавить запись ( первая сессия )}$
```
otus1=*# insert into persons(first_name, second_name) values('sveta', 'svetova');
INSERT 0 1
otus1=*#
```
$\textsf{\color{orange}Выбрать записи из таблицы ( вторая сессия )}$
```
otus1=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 строки)

otus1=*#
```
*Новых записей нет - не видим изменённые данные в других транзакциях* <br>

$\textsf{\color{blue}Завершить первую транзакцию}$
```
otus1=*# commit;
COMMIT
otus1=#
```
$\textsf{\color{orange}Выбрать записи из таблицы ( вторая сессия )}$
```
otus1=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 строки)

otus1=*#
```
*Новых записей нет - не видим зафиксированные данные из других сессий* <br>

$\textsf{\color{orange}Завершить вторую транзакцию}$
```
otus1=*# commit;
COMMIT
otus1=#
```
$\textsf{\color{orange}Выбрать записи из таблицы ( вторая сессия )}$
```
otus1=# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
  4 | sveta      | svetova
(4 строки)

otus1=#
```
*Записи все есть так как транзакция во второй сессии завершена (вышли из режима repeatable read)* <br>
*-------------* <br>
*В транзакции с типом repeatable read не видно изменённых и зафиксированных данных других транзакций*


