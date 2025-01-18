### Домашнее задание - Бэкапы БД PostgeSQL

- виртуальная машина создана  в гипервизоре VMWare ESXi ( задания HW01,HW02,HW05 )
  параметры изменены на vCPU=4/vRAM=8G/vHDD=40Gb + vHDD2=25Gb + vHDD3=50Gb
- установлена  ubuntu-2404 (host ubutest; user boss; ip 192.168.1.244)
- установлен PostgreSQL версии PostgresPro-1c-16 

- Но - установим PostgresPro-Std-17 (not free) на порт 5433

```
wget https://repo.postgrespro.ru/std/std-17/keys/pgpro-repo-add.sh 
chmod +x ./pgpro-repo-add.sh
sudo ./pgpro-repo-add.sh 
sudo apt install postgrespro-std-17
```

- Загрузим демо базу данных ( предварительно создав пустую базу air ) 

```
postgres@ubutest:~$ pg_restore -p 5433 --format=custom --no-owner -d air /tmp/postgres_air_2024.backup
postgres@ubutest:~$ psql -p 5433
psql (17.2)
Введите "help", чтобы получить справку.

postgres=# \l+
                                                                                             Список баз данных
    Имя    | Владелец | Кодировка | Провайдер локали | LC_COLLATE  |  LC_CTYPE   | Локаль | Правила ICU |     Права доступа     | Размер  | Табл. пространство |                  Описание
-----------+----------+-----------+------------------+-------------+-------------+--------+-------------+-----------------------+---------+----------------------------------------------------------------
 air       | postgres | UTF8      | icu              | ru_RU.UTF-8 | ru_RU.UTF-8 | ru-RU  |             |                       | 7992 MB | pg_default         |
 postgres  | postgres | UTF8      | icu              | ru_RU.UTF-8 | ru_RU.UTF-8 | ru-RU  |             |                       | 7477 kB | pg_default         | default administrative connection database
 template0 | postgres | UTF8      | icu              | ru_RU.UTF-8 | ru_RU.UTF-8 | ru-RU  |             | =c/postgres          +| 7321 kB | pg_default         | unmodifiable empty database
           |          |           |                  |             |             |        |             | postgres=CTc/postgres |         |                    |
 template1 | postgres | UTF8      | icu              | ru_RU.UTF-8 | ru_RU.UTF-8 | ru-RU  |             | =c/postgres          +| 7549 kB | pg_default         | default template for new databases
           |          |           |                  |             |             |        |             | postgres=CTc/postgres |         |                    |
(4 строки)

```

- Данная версия PostgreSQL содержит в репозитории pg_probackup 2.8.5 (not free) - установим

```
sudo apt install pg-probackup-std-17
-- добавим ссылку на программу
sudo ln -s /opt/pgpro/std-17/bin/pg_probackup /usr/bin/pg_probackup
```

- Настройка pg_probackup

```
-- создаём каталог для бэкапов
boss@ubutest:~$ sudo mkdir /backups/probackup
boss@ubutest:~$ sudo chown postgres: /backups/probackup
-- инициализируем каталог
sudo -i -u postgres
postgres@ubutest:~$ pg_probackup init -B /backups/probackup
INFO: Backup catalog '/backups/probackup' successfully initialized
-- установим переменную на данный каталог и можно использовать без ключа -B
postgres@ubutest:~$ export BACKUP_PATH=/backups/probackup
-- добавляем инстанс ( учитываем порт 5433 )
postgres@ubutest:~$ pg_probackup add-instance -D /pgdata/air -p 5433 --instance=air
INFO: Instance 'air' successfully initialized
-- посмотрим какие параметры по умолчанию назначены
postgres@ubutest:~$ pg_probackup show-config --instance=air
# Backup instance information
pgdata = /pgdata/air
system-identifier = 7460572854766238660
xlog-seg-size = 16777216
# Connection parameters
pgdatabase = postgres
pgport = 5433
# Archive parameters
archive-timeout = 5min
# Logging parameters
log-level-console = INFO
log-level-file = OFF
log-format-console = PLAIN
log-format-file = PLAIN
log-filename = pg_probackup.log
log-rotation-size = 0TB
log-rotation-age = 0d
# Retention parameters
retention-redundancy = 0
retention-window = 0
wal-depth = 0
# Compression parameters
compress-algorithm = none
compress-level = 1
# Remote access parameters
remote-proto = ssh
# Backup instance information
write-rate-limit = 0GBps
```

- сдедаем полный архив с разной упаковкой и потоками<br>
  // здесь на тестах работаем от пользователя postgres - что нежелательно - WARNING предупреждение<br>
  // в реальной работе надо создавать пользователя отдельного и по настройке следовать инструкции PostgresPro 

```
postgres@ubutest:~$ pg_probackup backup --instance=air -b FULL --stream
INFO: Backup start, pg_probackup version: 2.8.5, instance: air, backup ID: SQ73B8, backup mode: FULL, wal mode: STREAM, remote: false, compress-algorithm: none, compress-level: 1
INFO: This PostgreSQL instance was initialized with data block checksums. Data block corruption will be detected
WARNING: Current PostgreSQL role is superuser. It is not recommended to run pg_probackup under superuser.
INFO: Database backup start
INFO: wait for pg_backup_start()
INFO: PGDATA size: 8014MB
INFO: Current Start LSN: 2/90000028, TLI: 1
INFO: Start transferring data files
INFO: Data files are transferred, time elapsed: 1m:25s
INFO: wait for pg_stop_backup()
INFO: pg_stop_backup() successfully executed
INFO: stop_stream_lsn 2/91000000 currentpos 2/91000000
INFO: backup->stop_lsn 2/900001C8
INFO: Getting the Recovery Time from WAL
INFO: Syncing backup files to disk
INFO: Backup files are synced, time elapsed: 2s
INFO: Validating backup SQ73B8
INFO: Backup SQ73B8 data files are valid
INFO: Backup SQ73B8 resident size: 8038MB
INFO: Backup SQ73B8 completed

postgres@ubutest:~$ pg_probackup backup --instance=air -b FULL --stream --compress-algorithm=lz4 --compress-level=9
...
postgres@ubutest:~$ pg_probackup backup --instance=air -b FULL --stream --compress-algorithm=lz4 --compress-level=9 -j 3
...
postgres@ubutest:~$ pg_probackup backup --instance=air -b FULL --stream --compress-algorithm=zstd --compress-level=3 -j 3
...
postgres@ubutest:~$ pg_probackup backup --instance=air -b FULL --stream --compress-algorithm=zlib --compress-level=3 -j 3
...
postgres@ubutest:~$ pg_probackup backup --instance=air -b FULL --stream --compress-algorithm=zlib --compress-level=6 -j 3
...
```

- статистика по алгоритмам и сжатию ( быстрее и лучше сжимает - zstd , но вместе с lz4 его нет в бесплатной версии )

```
postgres@ubutest:~$ pg_probackup show --instance=air
===================================================================================================================================================
 Instance  Version  ID      Recovery Time                  Mode  WAL Mode  TLI    Time    Data   WAL  Zalg  Zratio  Start LSN   Stop LSN    Status
===================================================================================================================================================
 air       17       SQ73YG  2025-01-16 19:12:29.943367+00  FULL  STREAM    1/0  1m:28s  2311MB  16MB  zlib    3.47  2/9A000028  2/9A0001C8  OK      
 air       17       SQ73W0  2025-01-16 19:10:29.710545+00  FULL  STREAM    1/0     56s  2370MB  16MB  zlib    3.38  2/98000028  2/980001C8  OK
 air       17       SQ73UJ  2025-01-16 19:09:16.479732+00  FULL  STREAM    1/0     36s  2286MB  16MB  zstd    3.51  2/96000028  2/960001C8  OK
 air       17       SQ73RO  2025-01-16 19:07:56.480770+00  FULL  STREAM    1/0     59s  3476MB  16MB   lz4    2.31  2/94000028  2/940001C8  OK
 air       17       SQ73N7  2025-01-16 19:06:38.375030+00  FULL  STREAM    1/0  2m:20s  3476MB  16MB   lz4    2.31  2/92000028  2/920001C8  OK
 air       17       SQ73B8  2025-01-16 18:58:33.749503+00  FULL  STREAM    1/0  1m:27s  8022MB  16MB  none    1.00  2/90000028  2/900001C8  OK
```

- удалим большинство бэкапов и оставим последний

```
postgres@ubutest:~$ pg_probackup delete --instance=air -i SQ73B8
INFO: Resident data size to free by delete of backup SQ73B8 : 8038MB
INFO: Delete: SQ73B8 2025-01-16 18:58:33+00
postgres@ubutest:~$ pg_probackup delete --instance=air -i SQ73N7
INFO: Resident data size to free by delete of backup SQ73N7 : 3492MB
INFO: Delete: SQ73N7 2025-01-16 19:06:38+00
postgres@ubutest:~$ pg_probackup delete --instance=air -i SQ73RO
INFO: Resident data size to free by delete of backup SQ73RO : 3492MB
INFO: Delete: SQ73RO 2025-01-16 19:07:56+00
postgres@ubutest:~$ pg_probackup delete --instance=air -i SQ73UJ
INFO: Resident data size to free by delete of backup SQ73UJ : 2302MB
INFO: Delete: SQ73UJ 2025-01-16 19:09:16+00
postgres@ubutest:~$ pg_probackup delete --instance=air -i SQ73W0
INFO: Resident data size to free by delete of backup SQ73W0 : 2386MB
INFO: Delete: SQ73W0 2025-01-16 19:10:29+00
```

### Восстановление "кластера" на другой ВМ

- настроим доступ к папке /backups с других ВМ по сети как NFS

```
boss@ubutest:~$ sudo apt update
...
boss@ubutest:~$ sudo apt install nfs-kernel-server
...
boss@ubutest:~$ sudo -- sh -c "echo '/backups  192.168.1.0/24(rw,no_root_squash,no_subtree_check)' >> /etc/exports"
boss@ubutest:~$ sudo exportfs -a
-- выдадим полный доступ для всех (other) 
boss@ubutest:~$ sudo chmod -R o+rwX /backups/
```

- есть вторая ВМ с установленным Astra Linux 1.8  и также PostgresPro-std-17<br>
  настроим доступ к диску /backups первой ВМ по сети

```
boss@astra8:~$ sudo apt update
...
boss@astra8:~$ sudo apt install nfs-common
...
-- временно подключим NFS ресурс
boss@astra8:~$ sudo mkdir /backups
boss@astra8:~$ sudo mount -t nfs 192.168.1.244:/backups /backups 
```

- настраивать pg_probackup и базу на второй ВМ - не надо ( работаем от пользователя postgres )<br>
  проверим что видим бэкапы по сети

```
postgres@astra8:~$ pg_probackup show -B /backups/probackup/

BACKUP INSTANCE 'air'
===================================================================================================================================================
 Instance  Version  ID      Recovery Time                  Mode  WAL Mode  TLI    Time    Data   WAL  Zalg  Zratio  Start LSN   Stop LSN    Status
===================================================================================================================================================
 air       17       SQ73YG  2025-01-17 00:12:29.943367+05  FULL  STREAM    1/0  1m:28s  2311MB  16MB  zlib    3.47  2/9A000028  2/9A0001C8  OK
 ``

- восстановим "кластер" из бэкапа на второй ВМ ( /var/lib/pgpro/std-17/data )

```
boss@astra8:~$ sudo systemctl stop postgrespro-std-17.service
boss@astra8:~$ sudo rm -r /var/lib/pgpro/std-17/data
boss@astra8:~$ sudo su - postgres
postgres@astra8:~$ pg_probackup restore -B /backups/probackup/ --instance=air -D /var/lib/pgpro/std-17/data
INFO: Validating backup SQ73YG
INFO: Backup SQ73YG data files are valid
INFO: Backup SQ73YG WAL segments are valid
INFO: Backup SQ73YG is valid.
INFO: Restoring the database from backup SQ73YG
INFO: Start restoring backup files. PGDATA size: 8030MB
INFO: Backup files are restored. Transferred bytes: 8030MB, time elapsed: 1m:16s
INFO: Restore incremental ratio (less is better): 100% (8030MB/8030MB)
INFO: Syncing restored files to disk
INFO: Restored backup files are synced, time elapsed: 1s
INFO: Restore of backup SQ73YG completed.
```

- запускаем СУБД

```
boss@astra8:~$ sudo systemctl start postgrespro-std-17.service
boss@astra8:~$ sudo systemctl status postgrespro-std-17.service
● postgrespro-std-17.service - Postgres Pro std 17 database server
     Loaded: loaded (/lib/systemd/system/postgrespro-std-17.service; enabled; preset: enabled)
     Active: active (running) since Sat 2025-01-18 11:16:34 +05; 7s ago
    Process: 82934 ExecStartPre=/opt/pgpro/std-17/bin/check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
   Main PID: 82936 (postgres)
      Tasks: 7 (limit: 2307)
     Memory: 95.7M
        CPU: 118ms
     CGroup: /system.slice/postgrespro-std-17.service
             ├─82936 /opt/pgpro/std-17/bin/postgres -D /var/lib/pgpro/std-17/data
             ├─82943 "postgres: logger "
             ├─82944 "postgres: checkpointer "
             ├─82945 "postgres: background writer "
             ├─82953 "postgres: walwriter "
             ├─82954 "postgres: autovacuum launcher "
             └─82955 "postgres: logical replication launcher "

янв 18 11:16:31 astra8 systemd[1]: Starting postgrespro-std-17.service - Postgres Pro std 17 database server...
янв 18 11:16:32 astra8 postgres[82936]: 2025-01-18 06:16:32.174 UTC [82936] LOG:  redirecting log output to logging collector process
янв 18 11:16:32 astra8 postgres[82936]: 2025-01-18 06:16:32.174 UTC [82936] HINT:  Future log output will appear in directory "log".
янв 18 11:16:34 astra8 systemd[1]: Started postgrespro-std-17.service - Postgres Pro std 17 database server.
```

- прорверим наличие базы

```
boss@astra8:~$ sudo su - postgres
postgres@astra8:~$ psql -p 5433
WARNING:  database "postgres" has a collation version mismatch
ПОДРОБНОСТИ:  The database was created using collation version 153.121.44.8, but the operating system provides version 153.120.42.
ПОДСКАЗКА:  Rebuild all objects in this database that use the default collation and run ALTER DATABASE postgres REFRESH COLLATION VERSION, or build PostgreSQL with the right library version.
psql (17.2)
Введите "help", чтобы получить справку.

postgres=# \l+
                                                                                             Список баз данных
    Имя    | Владелец | Кодировка | Провайдер локали | LC_COLLATE  |  LC_CTYPE   | Локаль | Правила ICU |     Права доступа     | Размер  | Табл. пространство |       Описание
-----------+----------+-----------+------------------+-------------+-------------+--------+-------------+-----------------------+---------+--------------------+--------------------------------------------
 air       | postgres | UTF8      | icu              | ru_RU.UTF-8 | ru_RU.UTF-8 | ru-RU  |             |                       | 7991 MB | pg_default         |
 postgres  | postgres | UTF8      | icu              | ru_RU.UTF-8 | ru_RU.UTF-8 | ru-RU  |             |                       | 7477 kB | pg_default         | default administrative connection database
 template0 | postgres | UTF8      | icu              | ru_RU.UTF-8 | ru_RU.UTF-8 | ru-RU  |             | =c/postgres          +| 7321 kB | pg_default         | unmodifiable empty database
           |          |           |                  |             |             |        |             | postgres=CTc/postgres |         |                    |
 template1 | postgres | UTF8      | icu              | ru_RU.UTF-8 | ru_RU.UTF-8 | ru-RU  |             | =c/postgres          +| 7393 kB | pg_default         | default template for new databases
           |          |           |                  |             |             |        |             | postgres=CTc/postgres |         |                    |
(4 строки)
```

// база в наличии - востановление успешно, если не считать проблемы с сортировкой на разных версиях Linux<br>
// подобная проблема была HW05 - там провайдер был libc здесь icu - разные версии на Astra Linux и Ubuntu<br>
// Надо следить за этим и при реализации реплики и кластеров Patroni ( Linux должен быть СТРОГО одной версии ) 

### Под нагрузкой снимаем бэкап

- запускаем виртуальную нагрузку со второй ВМ 

```
-- установим sysbench
boss@astra8:~$ sudo apt -y install sysbench
-- создадим пользователя и базу данных на первой ВМ
boss@astra8:~$ sudo su - postgres
postgres@astra8:~$ psql -p 5433 -h 192.168.1.244
psql (17.2)
Введите "help", чтобы получить справку.

postgres=# create role sbtest with login password 'sbtest';
CREATE ROLE
postgres=# create database sbtest owner sbtest;
CREATE DATABASE
-- инициализируем sysbench
sysbench \
--db-driver=pgsql \
--oltp-table-size=100000 \
--oltp-tables-count=24 \
--threads=1 \
--pgsql-host=192.168.1.244 \
--pgsql-port=5433 \
--pgsql-user=sbtest \
--pgsql-password=sbtest \
--pgsql-db=sbtest \
/usr/share/sysbench/tests/include/oltp_legacy/parallel_prepare.lua \
run
sysbench 1.0.20 (using system LuaJIT 2.1.0-beta3)

Running the test with following options:
Number of threads: 1
Initializing random number generator from current time


Initializing worker threads...

Threads started!

thread prepare0
Creating table 'sbtest1'...
Inserting 100000 records into 'sbtest1'
Creating secondary indexes on 'sbtest1'...
Creating table 'sbtest2'...
Inserting 100000 records into 'sbtest2'
Creating secondary indexes on 'sbtest2'...
...
Creating table 'sbtest24'...
Inserting 100000 records into 'sbtest24'
Creating secondary indexes on 'sbtest24'...
SQL statistics:
    queries performed:
        read:                            0
        write:                           912
        other:                           48
        total:                           960
    transactions:                        1      (0.03 per sec.)
    queries:                             960    (27.22 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          35.2653s
    total number of events:              1

Latency (ms):
         min:                                35264.72
         avg:                                35264.72
         max:                                35264.72
         95th percentile:                    35191.04
         sum:                                35264.72

Threads fairness:
    events (avg/stddev):           1.0000/0.00
    execution time (avg/stddev):   35.2647/0.00
```



### Работа с репликой

- создаем реплику на второй ВМ с Astra Linux переводя наш восстановленый бэкап в режим standby

```
```





