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

### Восстановление "кластера" на другой ВМ

- настроим доступ к папке /backups с других ВМ по сети как NFS

```
```

- есть вторая ВМ с установленным Astra Linux 1.8  и также PostgresPro-std-17<br>
  настроим доступ к диску /backups первой ВМ по сети

```
```

- настраивать pg_probackup и базу на второй ВМ - не надо ( работаем от пользователя postgres )<br>
  проверим что видим бэкапы по сети
  
- восстановим "кластер" из бэкапа на второй ВМ 

```
```

- прорверим наличие базы

```
```

// база в наличии - востановление успешно


### Работа с репликой




