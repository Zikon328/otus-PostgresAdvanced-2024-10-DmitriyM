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


- ещё некоторые комментарии

```
Для разных версий Linux - версии PostgreSQL входящие в комплект стандартного репозитория
Очень различные настройки

- Ubuntu и Debian 
    своеобразная библиотека postgresql-common
    настройки в папке /etc/postgresql - независимы от папки данных
- SUSE Linux 
    дополнительные скрипты
    параметры настройки в файле /etc/
- CentOS
    параметры
- Astra Linux 1.8
    параметры        
```

- попробуем перенести диск ВМ на другую ВМ

//  Есть ещё одна ВМ  - Astra Linux 1.8  и  также установлен PostgresPro-1C-16<br>
//  Для переноса в гипервизоре 
