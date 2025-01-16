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



