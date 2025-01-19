### Исходные данные

```
- Astra Linux 1.8.1.12
- PostgresPro-Std-17  (port 5433)  
- WAL-G 3.0.3
```

### Сборка WAL-G из исходников 

```
boss@astra8:~$ sudo apt install git
...
boss@astra8:~$ git clone https://github.com/wal-g/wal-g $(go env GOPATH)/src/github.com/wal-g/wal-g
...
boss@astra8:~/wal-g-3.0.3$ sudo apt update
boss@astra8:~/wal-g-3.0.3$ sudo apt install golang-go g++
...
-- оказался golang 1.21
-- но нужен golang версии 1.22 - перезапишем поверх
boss@astra8:~$ wget https://go.dev/dl/go1.22.5.linux-amd64.tar.gz
boss@astra8:~$ sudo tar -xvf go1.22.5.linux-amd64.tar.gz -C /usr/local
...
boss@astra8:~$ cd $(go env GOPATH)/src/github.com/wal-g/wal-g
boss@astra8:~/go/src/github.com/wal-g/wal-g$ export USE_BROTLI=1
boss@astra8:~/go/src/github.com/wal-g/wal-g$ export USE_LIBSODIUM=1
boss@astra8:~/go/src/github.com/wal-g/wal-g$ export USE_LZO=1
boss@astra8:~/go/src/github.com/wal-g/wal-g$ make deps
boss@astra8:~/go/src/github.com/wal-g/wal-g$ make pg_build
-- собрали для postgresql - и проверим версию
boss@astra8:~/go/src/github.com/wal-g/wal-g$ main/pg/wal-g --version
wal-g version   e42ffe43        2024.12.04_09:31:40     PostgreSQL
-- скопируем в локальный bin каталог
boss@astra8:~/go/src/github.com/wal-g/wal-g$ sudo cp main/pg/wal-g /usr/local/bin/wal-g
```

### Настройка программы WAL-G

```
-- создадим файл ~/.walg.json  у пользователя postgres с содержимым

{
    "WALG_COMPRESSION_METHOD": "brotli",
    "WALG_DELTA_MAX_STEPS": "5",
    "WALG_FILE_PREFIX": "/var/lib/postgresql/backups/walg",
    "WALG_UPLOAD_DISK_CONCURRENCY": "4",
    "PGDATA": "/var/lib/pgpro/std-17/data",
    "PGPORT": "5433",
    "PGHOST": "/tmp"
}
```

### Создание бэкапов

```
-- создадим первый бэкап - полный
postgres@astra8:~$ wal-g backup-push /var/lib/pgpro/std-17/data/
INFO: 2024/12/04 14:55:26.265892 Backup will be pushed to storage: default
INFO: 2024/12/04 14:55:26.280904 Couldn't find previous backup. Doing full backup.
INFO: 2024/12/04 14:55:26.288795 Calling pg_start_backup()
INFO: 2024/12/04 14:55:26.338945 Initializing the PG alive checker (interval=1m0s)...
INFO: 2024/12/04 14:55:26.339064 Starting a new tar bundle
INFO: 2024/12/04 14:55:26.339101 Walking ...
INFO: 2024/12/04 14:55:26.339264 Starting part 1 ...
INFO: 2024/12/04 14:55:33.966961 Finished writing part 1.
INFO: 2024/12/04 14:55:33.966994 Starting part 2 ...
INFO: 2024/12/04 14:55:42.683633 Finished writing part 2.
INFO: 2024/12/04 14:55:42.683665 Starting part 3 ...
INFO: 2024/12/04 14:55:51.059654 Finished writing part 3.
INFO: 2024/12/04 14:55:51.059688 Starting part 4 ...
INFO: 2024/12/04 14:56:06.839979 Finished writing part 4.
INFO: 2024/12/04 14:56:06.840012 Starting part 5 ...
INFO: 2024/12/04 14:56:25.392119 Finished writing part 5.
INFO: 2024/12/04 14:56:25.392155 Starting part 6 ...
INFO: 2024/12/04 14:56:37.186382 Finished writing part 6.
INFO: 2024/12/04 14:56:37.186403 Starting part 7 ...
INFO: 2024/12/04 14:56:44.095861 Packing ...
INFO: 2024/12/04 14:56:44.096671 Finished writing part 7.
INFO: 2024/12/04 14:56:44.096814 Starting part 8 ...
INFO: 2024/12/04 14:56:44.096915 /global/pg_control
INFO: 2024/12/04 14:56:44.097277 Finished writing part 8.
INFO: 2024/12/04 14:56:44.097295 Calling pg_stop_backup()
INFO: 2024/12/04 14:56:44.120367 Starting part 9 ...
INFO: 2024/12/04 14:56:44.120401 backup_label
INFO: 2024/12/04 14:56:44.120407 tablespace_map
INFO: 2024/12/04 14:56:44.120560 Finished writing part 9.
INFO: 2024/12/04 14:56:44.122096 Querying pg_database
INFO: 2024/12/04 14:56:44.212762 Wrote backup with name base_000000010000000200000095 to storage default

postgres@astra8:~$ wal-g backup-list
INFO: 2024/12/04 14:57:48.387876 List backups from storages: [default]
backup_name                   modified                  wal_file_name            storage_name
base_000000010000000200000095 2024-12-04T14:56:44+05:00 000000010000000200000095 default

-- создадим ещё один бэкап - будет дельта
postgres@astra8:~$ wal-g backup-push /var/lib/pgpro/std-17/data/
INFO: 2024/12/04 15:06:10.793924 Backup will be pushed to storage: default
INFO: 2024/12/04 15:06:10.805001 LATEST backup is: 'base_000000010000000200000095'
INFO: 2024/12/04 15:06:10.805289 Delta backup from base_000000010000000200000095 with LSN 2/95000060.
INFO: 2024/12/04 15:06:10.818644 Calling pg_start_backup()
INFO: 2024/12/04 15:06:10.867333 Initializing the PG alive checker (interval=1m0s)...
INFO: 2024/12/04 15:06:10.867430 Delta backup enabled
INFO: 2024/12/04 15:06:10.867601 Starting a new tar bundle
INFO: 2024/12/04 15:06:10.867776 Walking ...
INFO: 2024/12/04 15:06:10.867990 Starting part 1 ...
INFO: 2024/12/04 15:06:10.868980 Starting part 2 ...
INFO: 2024/12/04 15:06:10.869357 Starting part 3 ...
INFO: 2024/12/04 15:06:10.874025 Starting part 4 ...
INFO: 2024/12/04 15:06:10.883802 Packing ...
INFO: 2024/12/04 15:06:10.884376 Finished writing part 4.
INFO: 2024/12/04 15:06:10.884617 Finished writing part 1.
INFO: 2024/12/04 15:06:10.884870 Finished writing part 2.
INFO: 2024/12/04 15:06:10.885249 Finished writing part 3.
INFO: 2024/12/04 15:06:10.885398 Starting part 5 ...
INFO: 2024/12/04 15:06:10.885431 /global/pg_control
INFO: 2024/12/04 15:06:10.885702 Finished writing part 5.
INFO: 2024/12/04 15:06:10.885708 Calling pg_stop_backup()
INFO: 2024/12/04 15:06:10.913331 Starting part 6 ...
INFO: 2024/12/04 15:06:10.913580 backup_label
INFO: 2024/12/04 15:06:10.913679 tablespace_map
INFO: 2024/12/04 15:06:10.914474 Finished writing part 6.
INFO: 2024/12/04 15:06:10.915944 Querying pg_database
INFO: 2024/12/04 15:06:10.976208 Wrote backup with name base_00000001000000020000009A_D_000000010000000200000095 to storage default

-- список бэкапов
postgres@astra8:~$ wal-g backup-list
INFO: 2024/12/04 15:06:18.245364 List backups from storages: [default]
backup_name                                              modified                  wal_file_name            storage_name
base_000000010000000200000095                            2024-12-04T14:56:44+05:00 000000010000000200000095 default
base_00000001000000020000009A_D_000000010000000200000095 2024-12-04T15:06:10+05:00 00000001000000020000009A default

-- удаление всех бэкапов
postgres@astra8:~$ wal-g delete everything FORCE --confirm
```

- создадим ещё полный бэкап и посмотрим данные по архиву

```
postgres@astra8:~$ wal-g backup-push /var/lib/pgpro/std-17/data/
...
postgres@astra8:~$ wal-g backup-list
INFO: 2024/12/04 15:12:23.742956 List backups from storages: [default]
backup_name                   modified                  wal_file_name            storage_name
base_00000001000000020000009C 2024-12-04T15:10:26+05:00 00000001000000020000009C default

-- подробный список 
postgres@astra8:~$ wal-g backup-list --detail
INFO: 2024/12/04 15:12:49.793022 List backups from storages: [default]
backup_name                   modified                  wal_file_name            storage_name start_time           finish_time          hostnamedata_dir                   pg_version start_lsn  finish_lsn is_permanent
base_00000001000000020000009C 2024-12-04T15:10:26+05:00 00000001000000020000009C default      2024-12-04T10:09:39Z 2024-12-04T10:10:26Z astra8   /var/lib/pgpro/std-17/data 170002     2/9C000028 2/9C000160 false
postgres@astra8:~$ wal-g backup-list --detail --pretty
INFO: 2024/12/04 15:13:17.050292 List backups from storages: [default]
+---+-------------------------------+-----------------------------------+--------------------------+--------------+-----------------------------------+-----------------------------------+----------+----------------------------+------------+------------+------------+-----------+
| # | BACKUP NAME                   | MODIFIED                          | WAL FILE NAME            | STORAGE NAME | START TIME                        | FINISH TIME                       | HOSTNAME | DATADIR                    | PG VERSION | START LSN  | FINISH LSN | PERMANENT |
+---+-------------------------------+-----------------------------------+--------------------------+--------------+-----------------------------------+-----------------------------------+----------+----------------------------+------------+------------+------------+-----------+
| 0 | base_00000001000000020000009C | Wednesday, 04-Dec-24 15:10:26 +05 | 00000001000000020000009C | default      | Wednesday, 04-Dec-24 10:09:39 UTC | Wednesday, 04-Dec-24 10:10:26 UTC | astra8   | /var/lib/pgpro/std-17/data | 170002     | 2/9C000028 | 2/9C000160 | false     |
+---+-------------------------------+-----------------------------------+--------------------------+--------------+-----------------------------------+-----------------------------------+----------+----------------------------+------------+------------+------------+-----------+
```

- так более читаемая информация

```
postgres@astra8:~$ wal-g backup-list --detail --json | jq .
INFO: 2024/12/04 15:13:51.941365 List backups from storages: [default]
[
  {
    "backup_name": "base_00000001000000020000009C",
    "time": "2024-12-04T15:10:26.755881407+05:00",
    "wal_file_name": "00000001000000020000009C",
    "storage_name": "default",
    "start_time": "2024-12-04T10:09:39.262821Z",
    "finish_time": "2024-12-04T10:10:26.758489Z",
    "date_fmt": "%Y-%m-%dT%H:%M:%S.%fZ",
    "hostname": "astra8",
    "data_dir": "/var/lib/pgpro/std-17/data",
    "pg_version": 170002,
    "start_lsn": 11207180328,
    "finish_lsn": 11207180640,
    "is_permanent": false,
    "system_identifier": 7444446147733877000,
    "uncompressed_size": 8403386439,
    "compressed_size": 1684893052
  }
]
```

- Настраиваем архивацию WAL с помощью WAL-G

```
boss@astra8:~$ sudo mkdir /var/log/postgresql
boss@astra8:~$ sudo chown postgres: /var/log/postgresql

-- добавим в файл postgresql.conf следующие строки
wal_level=replica
archive_mode=on
archive_command='/usr/local/bin/wal-g wal-push \"%p\" >> /var/log/postgresql/archive_command.log 2>&1' 
archive_timeout=60
restore_command='/usr/local/bin/wal-g wal-fetch \"%f\" \"%p\" >> /var/log/postgresql/restore_command.log 2>&1'

-- так как изменили archive_mode - перезагружаем службу 
```

- Посмотреть состояние WAL архива

```
postgres@astra8:~$ wal-g wal-show
+-----+------------+-----------------+--------------------------+--------------------------+---------------+----------------+--------+---------------+
| TLI | PARENT TLI | SWITCHPOINT LSN | START SEGMENT            | END SEGMENT              | SEGMENT RANGE | SEGMENTS COUNT | STATUS | BACKUPS COUNT |
+-----+------------+-----------------+--------------------------+--------------------------+---------------+----------------+--------+---------------+
|   1 |          0 |             0/0 | 00000001000000020000009D | 0000000100000002000000AB |            15 |             15 | OK     |             0 |
+-----+------------+-----------------+--------------------------+--------------------------+---------------+----------------+--------+---------------+

postgres@astra8:~$ wal-g wal-show --detailed-json | jq .
[
  {
    "id": 1,
    "parent_id": 0,
    "switch_point_lsn": 0,
    "start_segment": "00000001000000020000009D",
    "end_segment": "0000000100000002000000AB",
    "segments_count": 15,
    "missing_segments": [],
    "segment_range_size": 15,
    "status": "OK"
  }
]

```

- Сделаем следующий бэкап и посмотрим состояние

```
postgres@astra8:~$ wal-g backup-push /var/lib/pgpro/std-17/data/
INFO: 2025/01/19 20:57:43.705164 Backup will be pushed to storage: default
INFO: 2025/01/19 20:57:43.717780 LATEST backup is: 'base_00000001000000020000009C'
INFO: 2025/01/19 20:57:43.718119 Delta backup from base_00000001000000020000009C with LSN 2/9C000060.
INFO: 2025/01/19 20:57:43.741441 Calling pg_start_backup()
INFO: 2025/01/19 20:57:43.864979 Initializing the PG alive checker (interval=1m0s)...
INFO: 2025/01/19 20:57:43.865357 Delta backup enabled
INFO: 2025/01/19 20:57:43.865428 Starting a new tar bundle
INFO: 2025/01/19 20:57:43.865494 Walking ...
INFO: 2025/01/19 20:57:43.866335 Starting part 1 ...
INFO: 2025/01/19 20:57:43.867193 Starting part 2 ...
INFO: 2025/01/19 20:57:43.867545 Starting part 3 ...
INFO: 2025/01/19 20:57:43.883128 Starting part 4 ...
INFO: 2025/01/19 20:57:43.945377 Packing ...
INFO: 2025/01/19 20:57:43.946436 Finished writing part 4.
INFO: 2025/01/19 20:57:43.946835 Finished writing part 2.
INFO: 2025/01/19 20:57:43.947148 Finished writing part 3.
INFO: 2025/01/19 20:57:43.950404 Finished writing part 1.
INFO: 2025/01/19 20:57:43.950444 Starting part 5 ...
INFO: 2025/01/19 20:57:43.950563 /global/pg_control
INFO: 2025/01/19 20:57:43.950786 Finished writing part 5.
INFO: 2025/01/19 20:57:43.950792 Calling pg_stop_backup()
INFO: 2025/01/19 20:57:43.962980 Starting part 6 ...
INFO: 2025/01/19 20:57:43.963588 backup_label
INFO: 2025/01/19 20:57:43.963841 tablespace_map
INFO: 2025/01/19 20:57:43.964529 Finished writing part 6.
INFO: 2025/01/19 20:57:43.965429 Querying pg_database
INFO: 2025/01/19 20:57:44.028552 Wrote backup with name base_00000001000000020000009F_D_00000001000000020000009C to storage default

postgres@astra8:~$ wal-g backup-list --detail --json | jq .
INFO: 2025/01/19 20:58:16.648264 List backups from storages: [default]
[
  {
    "backup_name": "base_00000001000000020000009C",
    "time": "2025-01-19T20:09:10.337690234+05:00",
    "wal_file_name": "00000001000000020000009C",
    "storage_name": "default",
    "start_time": "2025-01-19T15:08:20.485748Z",
    "finish_time": "2025-01-19T15:09:10.336261Z",
    "date_fmt": "%Y-%m-%dT%H:%M:%S.%fZ",
    "hostname": "astra8",
    "data_dir": "/var/lib/pgpro/std-17/data",
    "pg_version": 170002,
    "start_lsn": 11207180384,
    "finish_lsn": 11207180696,
    "is_permanent": false,
    "system_identifier": 7460572854766239000,
    "uncompressed_size": 8403090737,
    "compressed_size": 1684640561
  },
  {
    "backup_name": "base_00000001000000020000009F_D_00000001000000020000009C",
    "time": "2025-01-19T20:57:44.025663558+05:00",
    "wal_file_name": "00000001000000020000009F",
    "storage_name": "default",
    "start_time": "2025-01-19T15:57:43.717514Z",
    "finish_time": "2025-01-19T15:57:44.025462Z",
    "date_fmt": "%Y-%m-%dT%H:%M:%S.%fZ",
    "hostname": "astra8",
    "data_dir": "/var/lib/pgpro/std-17/data",
    "pg_version": 170002,
    "start_lsn": 11257511976,
    "finish_lsn": 11257512288,
    "is_permanent": false,
    "system_identifier": 7460572854766239000,
    "uncompressed_size": 603543,
    "compressed_size": 66496
  }
]

postgres@astra8:~$ wal-g wal-show
+-----+------------+-----------------+--------------------------+--------------------------+---------------+----------------+--------+---------------+
| TLI | PARENT TLI | SWITCHPOINT LSN | START SEGMENT            | END SEGMENT              | SEGMENT RANGE | SEGMENTS COUNT | STATUS | BACKUPS COUNT |
+-----+------------+-----------------+--------------------------+--------------------------+---------------+----------------+--------+---------------+
|   1 |          0 |             0/0 | 00000001000000020000009D | 0000000100000002000000AB |            15 |             15 | OK     |             1 |
+-----+------------+-----------------+--------------------------+--------------------------+---------------+----------------+--------+---------------+

postgres@astra8:~$ wal-g wal-show --detailed-json | jq .
[
  {
    "id": 1,
    "parent_id": 0,
    "switch_point_lsn": 0,
    "start_segment": "00000001000000020000009D",
    "end_segment": "0000000100000002000000AB",
    "segments_count": 15,
    "missing_segments": [],
    "backups": [
      {
        "backup_name": "base_00000001000000020000009F_D_00000001000000020000009C",
        "time": "2025-01-19T20:57:44.025663558+05:00",
        "wal_file_name": "00000001000000020000009F",
        "storage_name": "default",
        "start_time": "2025-01-19T15:57:43.717514Z",
        "finish_time": "2025-01-19T15:57:44.025462Z",
        "date_fmt": "%Y-%m-%dT%H:%M:%S.%fZ",
        "hostname": "astra8",
        "data_dir": "/var/lib/pgpro/std-17/data",
        "pg_version": 170002,
        "start_lsn": 11257511976,
        "finish_lsn": 11257512288,
        "is_permanent": false,
        "system_identifier": 7460572854766239000,
        "uncompressed_size": 603543,
        "compressed_size": 66496
      }
    ],
    "segment_range_size": 15,
    "status": "OK"
  }
]
``` 