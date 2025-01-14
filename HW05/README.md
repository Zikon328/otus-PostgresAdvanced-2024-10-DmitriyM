### Домашнее задание - Настройка дисков для PostgreSQL

- виртуальная машина создана  в гипервизоре VMWare ESXi ( задания HW01,HW02 )
  параметры изменены на vCPU=4/vRAM=8G/vHDD=40Gb + vHDD2=25Gb
  <details><summary><i> О системе</i></summary>
    <img src=../Images/hw-05-01.jpg alt='Параметры ВМ'>
  </details>
- установлена  ubuntu-2404 (host ubutest; user boss; ip 192.168.1.244)
- установлен PostgreSQL версии PostgresPro-1c-16 

```
    boss@ubutest:~$ sudo systemctl status postgrespro-1c-16.service
    ● postgrespro-1c-16.service - Postgres Pro 1c 16 database server
        Loaded: loaded (/usr/lib/systemd/system/postgrespro-1c-16.service; disabled; preset: enabled)
        Active: active (running) since Mon 2025-01-13 17:07:49 UTC; 8s ago
        Process: 123561 ExecStartPre=/opt/pgpro/1c-16/bin/check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
    Main PID: 123567 (postgres)
        Tasks: 7 (limit: 9387)
        Memory: 70.6M (peak: 73.1M)
            CPU: 146ms
        CGroup: /system.slice/postgrespro-1c-16.service
                ├─123567 /opt/pgpro/1c-16/bin/postgres -D /var/lib/pgpro/1c-16/data
                ├─123571 "postgres: logger "
                ├─123572 "postgres: checkpointer "
                ├─123573 "postgres: background writer "
                ├─123575 "postgres: walwriter "
                ├─123576 "postgres: autovacuum launcher "
                └─123577 "postgres: logical replication launcher "

    янв 13 17:07:49 ubutest systemd[1]: Starting postgrespro-1c-16.service - Postgres Pro 1c 16 database server...
    янв 13 17:07:49 ubutest postgres[123567]: 2025-01-13 17:07:49.365 UTC [123567] СООБЩЕНИЕ:  передача вывода в протокол процессу сбора протоколов
    янв 13 17:07:49 ubutest postgres[123567]: 2025-01-13 17:07:49.365 UTC [123567] ПОДСКАЗКА:  В дальнейшем протоколы будут выводиться в каталог "log".
    янв 13 17:07:49 ubutest systemd[1]: Started postgrespro-1c-16.service - Postgres Pro 1c 16 database server.
```

- добавим таблицу colors в текущую базу данных postgres

```
postgres=# create table colors ( id int, name varchar(30) );
CREATE TABLE
postgres=# insert into colors values (1,'red'), (2,'green'), (3,'blue'), (4,'gray');
INSERT 0 4
postgres=# select * from colors;
 id | name
----+-------
  1 | red
  2 | green
  3 | blue
  4 | gray
(4 строки)
``` 

- останавливаем PostgreSQL

```bash
boss@ubutest:~$ sudo systemctl stop postgrespro-1c-16.service
boss@ubutest:~$ sudo systemctl status postgrespro-1c-16.service
○ postgrespro-1c-16.service - Postgres Pro 1c 16 database server
     Loaded: loaded (/usr/lib/systemd/system/postgrespro-1c-16.service; disabled; preset: enabled)
     Active: inactive (dead)

янв 13 17:07:49 ubutest systemd[1]: Starting postgrespro-1c-16.service - Postgres Pro 1c 16 database server...
янв 13 17:07:49 ubutest postgres[123567]: 2025-01-13 17:07:49.365 UTC [123567] СООБЩЕНИЕ:  передача вывода в протокол процессу сбора протоколов
янв 13 17:07:49 ubutest postgres[123567]: 2025-01-13 17:07:49.365 UTC [123567] ПОДСКАЗКА:  В дальнейшем протоколы будут выводиться в каталог "log".
янв 13 17:07:49 ubutest systemd[1]: Started postgrespro-1c-16.service - Postgres Pro 1c 16 database server.
янв 13 17:18:38 ubutest systemd[1]: Stopping postgrespro-1c-16.service - Postgres Pro 1c 16 database server...
янв 13 17:18:39 ubutest systemd[1]: postgrespro-1c-16.service: Killing process 123571 (postgres) with signal SIGKILL.
янв 13 17:18:39 ubutest systemd[1]: postgrespro-1c-16.service: Deactivated successfully.
янв 13 17:18:39 ubutest systemd[1]: Stopped postgrespro-1c-16.service - Postgres Pro 1c 16 database server.
янв 13 17:18:39 ubutest systemd[1]: postgrespro-1c-16.service: Consumed 1.315s CPU time, 73.1M memory peak, 0B memory swap peak.
```

- диск sdb есть неразмеченный

```bash
boss@ubutest:~$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    0   40G  0 disk
├─sda1   8:1    0    1G  0 part /boot/efi
└─sda2   8:2    0 38,9G  0 part /
sdb      8:16   0   25G  0 disk
sr0     11:0    1  2,6G  0 rom
```

- разметим диск с использованием lvm и отформатируем в xfs

```bash
boss@ubutest:~$ sudo pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
boss@ubutest:~$ sudo vgcreate vg1 /dev/sdb
  Volume group "vg1" successfully created
boss@ubutest:~$ sudo lvcreate -l 100%FREE -n pgdata vg1
  Logical volume "pgdata" created.
boss@ubutest:~$ sudo mkfs.xfs /dev/vg1/pgdata
meta-data=/dev/vg1/pgdata        isize=512    agcount=4, agsize=1638144 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=1
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=6552576, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
Discarding blocks...Done.
```

- создаем каталог /pgdata
- в файле /etc/fstab добавляем строку

```
/dev/vg1/pgdata /pgdata  xfs  defaults   1 2
```

- проверяем настройку fstab 

```bash
boss@ubutest:~$ sudo mkdir /pgdata
boss@ubutest:~$ sudo systemctl daemon-reload
boss@ubutest:~$ sudo mount -a
boss@ubutest:~$ lsblk
NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda            8:0    0   40G  0 disk
├─sda1         8:1    0    1G  0 part /boot/efi
└─sda2         8:2    0 38,9G  0 part /
sdb            8:16   0   25G  0 disk
└─vg1-pgdata 252:0    0   25G  0 lvm  /pgdata
sr0           11:0    1  2,6G  0 rom
boss@ubutest:~$
boss@ubutest:~$ sudo chown postgres: /pgdata
boss@ubutest:~$ sudo chmod 750 /pgdata
boss@ubutest:~$ ls -la /  | grep pgdata
drwxr-x---   2 postgres postgres          6 янв 13 17:25 pgdata
```

// -- Комментарии --<br>
// ubuntu-2404  Требует daemon-reload при изменеии /etc/fstab<br>
// postgrespro-1c-16 - папка с данными должна быть с доступом 750 или 700<br>

- перенесём данные в новую папку

```bash
boss@ubutest:~$ sudo su - postgres
postgres@ubutest:~$ mv /var/lib/pgpro/1c-16/data/* /pgdata/
```

- запустим сервер PostgreSQL

```bash
sudo systemctl start postgrespro-1c-16.service
Job for postgrespro-1c-16.service failed because the control process exited with error code.
See "systemctl status postgrespro-1c-16.service" and "journalctl -xeu postgrespro-1c-16.service" for details.
boss@ubutest:~$ sudo systemctl status postgrespro-1c-16.service
× postgrespro-1c-16.service - Postgres Pro 1c 16 database server
     Loaded: loaded (/usr/lib/systemd/system/postgrespro-1c-16.service; disabled; preset: enabled)
     Active: failed (Result: exit-code) since Mon 2025-01-13 17:42:21 UTC; 8s ago
    Process: 136283 ExecStartPre=/opt/pgpro/1c-16/bin/check-db-dir ${PGDATA} (code=exited, status=1/FAILURE)
        CPU: 2ms

янв 13 17:42:21 ubutest systemd[1]: Starting postgrespro-1c-16.service - Postgres Pro 1c 16 database server...
янв 13 17:42:21 ubutest check-db-dir[136283]: "/var/lib/pgpro/1c-16/data" is missing or empty.
янв 13 17:42:21 ubutest check-db-dir[136283]: Use "/opt/pgpro/1c-16/bin/pg-setup initdb" to initialize the database cluster.
янв 13 17:42:21 ubutest systemd[1]: postgrespro-1c-16.service: Control process exited, code=exited, status=1/FAILURE
янв 13 17:42:21 ubutest systemd[1]: postgrespro-1c-16.service: Failed with result 'exit-code'.
янв 13 17:42:21 ubutest systemd[1]: Failed to start postgrespro-1c-16.service - Postgres Pro 1c 16 database server.
```

// Ошибка - нет данных в папке - надо настроить папку PGDATA

- для данной версии PostgreSQL - настройки в файле  /etc/default/postgrespro-1c-16
- меняем PGDATA на новый каталог
  
```
PGDATA=/pgdata
```

- запускаем сервер

```bash
boss@ubutest:~$ sudo systemctl start postgrespro-1c-16.service
boss@ubutest:~$ sudo systemctl status postgrespro-1c-16.service
● postgrespro-1c-16.service - Postgres Pro 1c 16 database server
     Loaded: loaded (/usr/lib/systemd/system/postgrespro-1c-16.service; disabled; preset: enabled)
     Active: active (running) since Mon 2025-01-13 17:47:06 UTC; 2s ago
    Process: 138019 ExecStartPre=/opt/pgpro/1c-16/bin/check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
   Main PID: 138021 (postgres)
      Tasks: 7 (limit: 9387)
     Memory: 59.9M (peak: 60.1M)
        CPU: 119ms
     CGroup: /system.slice/postgrespro-1c-16.service
             ├─138021 /opt/pgpro/1c-16/bin/postgres -D /pgdata
             ├─138023 "postgres: logger "
             ├─138024 "postgres: checkpointer "
             ├─138025 "postgres: background writer "
             ├─138027 "postgres: walwriter "
             ├─138028 "postgres: autovacuum launcher "
             └─138029 "postgres: logical replication launcher "

янв 13 17:47:06 ubutest systemd[1]: Starting postgrespro-1c-16.service - Postgres Pro 1c 16 database server...
янв 13 17:47:06 ubutest postgres[138021]: 2025-01-13 17:47:06.121 UTC [138021] СООБЩЕНИЕ:  передача вывода в протокол процессу сбора протоколов
янв 13 17:47:06 ubutest postgres[138021]: 2025-01-13 17:47:06.121 UTC [138021] ПОДСКАЗКА:  В дальнейшем протоколы будут выводиться в каталог "log".
янв 13 17:47:06 ubutest systemd[1]: Started postgrespro-1c-16.service - Postgres Pro 1c 16 database server.
```

// сервер запустился и видно что с параметром каталога БД  /pgdata

- посмотрим данные что вводили ранее

```
boss@ubutest:~$ sudo su - postgres
postgres@ubutest:~$ psql
psql (16.6)
Введите "help", чтобы получить справку.

postgres=# select * from colors;
 id | name
----+-------
  1 | red
  2 | green
  3 | blue
  4 | gray
(4 строки)
```

// Данные присутствуют - перенос произведён<br> 
// --<br>
// Также есть вариант настройки через символьную ссылку к папке данных<br>
// тогда в настройках не надо менять путь PGDATA<br>
// --<br>
// Через символьную ссылку перенаправляют каталог wal на другой диск<br>
// и это также делает ключ -X команды initdb<br>
// <br>


### Комментарии

```
Для разных версий Linux - версии PostgreSQL входящие в комплект стандартного репозитория
Очень различные настройки

- Ubuntu и Debian ( free )
    актуальная версия СУБД - очень быстро доступна
    своеобразная библиотека postgresql-common для управления "инстансами"
    ( минус! - данная библиотека ТОЛЬКО на Ubuntu и Debian совместимых Linux )
    настройки в папке /etc/postgresql/15/main - независимы от папки данных
    ( здесь и настраивается параметр data_directory в файле postgresql.conf )
- SUSE Linux ( 15sp4 - not free )
    актуальная версия СУБД появляется достаточно быстро в репозитории
    оригинальное размещение программы и несколько версий
    ( /usr/lib/postgresqlNN, /usr/share/postgresqlNN )
    основной скрипт запуска /usr/share/postgresql/postgresql-script
    параметры настройки в файле /etc/sysconfig/postgresql
    ( в этом файле можно установить путь к данным )
- CentOS Stream 9 ( pseudo free )
    в репозитории PostgreSQL 13
    попытка установить из глобального yum.postgresql.org  16 версию
    для CentOS 9 нет официального на yum, но взял от RedHat 9
    После установки СУБД в /usr/pgsql-16
    требуется инициализация  'sudo /usr/pgsql-16/bin/postgresql-16-setup initdb'
    каталог данных для 16 версии  /var/lib/pgsql/16/data 
    прописан в файле сервиса  /usr/lib/systemd/system/postgresql-16.service
- Astra Linux 1.7 ( not free )
    официально только PostgreSQL 11 - в родном репозитории
    можно подключить репозиторий Debian 10    
    аналогично Debian и Ubuntu
- Astra Linux 1.8 ( not free )
    PostgreSQL 15 - в родном репозитории
    можно подключить репозиторий Debian 12 (уточнить)    
    аналогично Debian и Ubuntu
```

Репозиторий - PostgresPro - свободная версия для 1С<br> 
( есть STD для тестирования )

```
- Поддерживается - AstraLinux; SUSE Linux; AlterOS; AltLinux; 
                   Debian; RedOS; RedHat; ROSA; Ubuntu; + Производные
                   GosLinux; Rocky; AlmaLinux; OSnova; OpenSUSE ...
- Единый скрипт для всех Linux для регистрации репозитория
- Программа размещается в папке  /opt/pgpro/1c-NN (NN версия - можно ставить разные)
- При установке сервер запускается в папке с данными /var/lib/pgpro/1c-NN/data
- Предварительно определяются скриптом параметры сервера (процессор/память) и 
  добавляются настройки в файле postgresql.conf
- Также если уже есть запущен сервер на портах 5432, 5433 ... - выберет свободный порт
- Настройки каталога данных в файле /etc/default/postgrespro-1c-NN  
- Можно организовать локальный репозиторий нужных версий и для определённых Linux
  ( при этом скрипт регистрации репозитория - минимальные изменения )
- МИНУСЫ - началные параметры заточены для 1С 
  - Шифрование md5 - переключить в scram256
  - дополнительные расширения - отключить
- При новой инициализации (initdb) будет как в стандартном PostgreSQL
```

### Дополнительно ++

- попробуем перенести диск ВМ на другую ВМ

//  Есть ещё одна ВМ  - Astra Linux 1.8  и  также установлен PostgresPro-1C-16<br>
//  Для переноса в гипервизоре 

- Отключим ВМ с Ubuntu и временно подключим диск с данными к Astra Linux

```
boss@astra8:~$ lsblk
NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda            8:0    0   40G  0 disk
├─sda1         8:1    0  600M  0 part /boot/efi
└─sda2         8:2    0 39,4G  0 part /
sdb            8:16   0   80G  0 disk
└─gu02-u02   252:0    0   80G  0 lvm  /u02
sdc            8:32   0   25G  0 disk
└─vg1-pgdata 252:2    0   25G  0 lvm
sdd            8:48   0   80G  0 disk
└─gu01-u01   252:1    0   80G  0 lvm  /u01
sr0           11:0    1  6,5G  0 rom
```

// диск sdc 25G автоматически просканировался на LVM

- смонтируем временно диск в папку /pgdata

```
boss@astra8:~$ sudo mkdir /pgdata
boss@astra8:~$ sudo mount /dev/vg1/pgdata /pgdata
boss@astra8:~$ lsblk
NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda            8:0    0   40G  0 disk
├─sda1         8:1    0  600M  0 part /boot/efi
└─sda2         8:2    0 39,4G  0 part /
sdb            8:16   0   80G  0 disk
└─gu02-u02   252:0    0   80G  0 lvm  /u02
sdc            8:32   0   25G  0 disk
└─vg1-pgdata 252:2    0   25G  0 lvm  /pgdata
sdd            8:48   0   80G  0 disk
└─gu01-u01   252:1    0   80G  0 lvm  /u01
sr0           11:0    1  6,5G  0 rom

boss@astra8:~$ sudo chown -R postgres: /pgdata
boss@astra8:~$ sudo chmod -R 750 /pgdata

boss@astra8:~$ ls -la / | grep pgdata
drwxr-x---  21 postgres postgres       4096 янв 14 15:36 pgdata
```

- запустим сервер СУБД с данной папкой ( исправили PGDATA в /etc/default/postgrespro-1c-16 )  

```
boss@astra8:~$ sudo systemctl start postgrespro-1c-16.service
boss@astra8:~$ sudo systemctl status postgrespro-1c-16.service
● postgrespro-1c-16.service - Postgres Pro 1c 16 database server
     Loaded: loaded (/lib/systemd/system/postgrespro-1c-16.service; enabled; preset: enabled)
     Active: active (running) since Tue 2025-01-14 16:24:53 +05; 5s ago
    Process: 19462 ExecStartPre=/opt/pgpro/1c-16/bin/check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
   Main PID: 19464 (postgres)
      Tasks: 7 (limit: 2307)
     Memory: 60.1M
        CPU: 64ms
     CGroup: /system.slice/postgrespro-1c-16.service
             ├─19464 /opt/pgpro/1c-16/bin/postgres -D /pgdata
             ├─19465 "postgres: logger "
             ├─19466 "postgres: checkpointer "
             ├─19467 "postgres: background writer "
             ├─19469 "postgres: walwriter "
             ├─19470 "postgres: autovacuum launcher "
             └─19471 "postgres: logical replication launcher "

янв 14 16:24:53 astra8 systemd[1]: Starting postgrespro-1c-16.service - Postgres Pro 1c 16 database server...
янв 14 16:24:53 astra8 postgres[19464]: 2025-01-14 11:24:53.092 UTC [19464] СООБЩЕНИЕ:  передача вывода в протокол процессу сбора протоколов
янв 14 16:24:53 astra8 postgres[19464]: 2025-01-14 11:24:53.092 UTC [19464] ПОДСКАЗКА:  В дальнейшем протоколы будут выводиться в каталог "log".
янв 14 16:24:53 astra8 systemd[1]: Started postgrespro-1c-16.service - Postgres Pro 1c 16 database server.
```

- проверим данные в базе

```
boss@astra8:~$ sudo su - postgres
postgres@astra8:~$ psql
ПРЕДУПРЕЖДЕНИЕ:  несовпадение версии для правила сортировки в базе данных "postgres"
ПОДРОБНОСТИ:  База данных была создана с версией правила сортировки 2.39, но операционная система предоставляет версию 2.36.
ПОДСКАЗКА:  Перестройте все объекты в этой базе, задействующие основное правило сортировки, и выполните ALTER DATABASE postgres REFRESH COLLATION VERSION, либо соберите PostgreSQL с правильной версией библиотеки.
psql (17.2, сервер 16.6)
Введите "help", чтобы получить справку.

postgres=# select * from colors;
 id | name
----+-------
  1 | red
  2 | green
  3 | blue
  4 | gray
(4 строки)
```

// База работает но, есть предупреждение по библиотеке libc разных версий Linux<br>
// посмотреть версию можно так

```bash
boss@astra8:~$ ldd --version
ldd (Debian GLIBC 2.36-9+deb12u7+ci202405171200+astra5+b1) 2.36
Copyright (C) 2022 Free Software Foundation, Inc.

boss@ubutest:~$ ldd --version
ldd (Ubuntu GLIBC 2.39-0ubuntu8.3) 2.39
Copyright (C) 2024 Free Software Foundation, Inc.
```

// При возвращении диска обратно в систему Ubuntu<br>
// восстановить надо права доступа

```bash
boss@ubutest:~$ sudo chown -R postgres: /pgdata
boss@ubutest:~$ sudo chmod -R 750 /pgdata
```