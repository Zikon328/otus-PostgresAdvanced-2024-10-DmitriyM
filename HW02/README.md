### Домашнее задание - Установка и настройка PostgteSQL в контейнере Docker

- виртуальная машина создана  в гипервизоре VMWare ESXi ( задание HW01 )
  параметры изменены на vCPU=4/vRAM=8G/vHDD=40G
  <details><summary><i> О системе</i></summary>
    <img src=../Images/hw-02-01.jpg alt='Параметры ВМ'>
  </details>
- установлена ubuntu-2404 (host ubutest; user boss; ip 192.168.1.244)
  <details><summary><i> Аннотация терминала </i></summary>
  
        Welcome to Ubuntu 24.04.1 LTS (GNU/Linux 6.8.0-49-generic x86_64)

         * Documentation:  https://help.ubuntu.com
         * Management:     https://landscape.canonical.com
         * Support:        https://ubuntu.com/pro

         System information as of Ср 08 янв 2025 08:58:29 UTC

          System load:  0.0                Processes:              228
          Usage of /:   18.6% of 38.04GB   Users logged in:        0
          Memory usage: 4%                 IPv4 address for ens34: 192.168.1.244
          Swap usage:   0%
  </details>      

- устанавливаем docker ( подключаем репозиторий для ubuntu-2404 )

```bash
wget -O- https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor | sudo tee /etc/apt/keyrings/docker.gpg > /dev/null
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu noble stable"| sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install docker-ce
```

```
sudo systemctl status docker

● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; preset: enabled)
     Active: active (running) since Wed 2025-01-08 13:10:34 UTC; 1min 7s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 3271 (dockerd)
      Tasks: 10
     Memory: 21.6M (peak: 22.5M)
        CPU: 322ms
     CGroup: /system.slice/docker.service
             └─3271 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

- загружаем образ postgresql 14

```bash
boss@ubutest:~$ docker pull postgres:14
14: Pulling from library/postgres
fd674058ff8f: Pull complete
6c60893f5921: Pull complete
6e4681af4963: Pull complete
e485590999ba: Pull complete
07b594df41d2: Pull complete
32d304bad922: Pull complete
88ac8a708577: Pull complete
d41295faaba7: Pull complete
a8bc7b60a38b: Pull complete
b69d252b29f7: Pull complete
c6fb314ec7e2: Pull complete
216ec0214ff7: Pull complete
5d9289c27dc4: Pull complete
ff6b1c025a53: Pull complete
Digest: sha256:922d38d4ca73ba5bfa8140c50b0d1f45636ca7b4c20d90506c49e2be2d7911f5
Status: Downloaded newer image for postgres:14
docker.io/library/postgres:14
```

- возможно создать папку для монтирования /var/lib/postgres  заранее, но docker сам создаёт папку если её нет.

- запускаем контейнер с postgresql

```bash
docker run -p 6432:5432 -v /var/lib/postgres:/var/lib/postgresql/data -e POSTGRES_PASSWORD=postgres -d --name otus_postgres postgres:14
```

- подключимся к СУБД в контейнере по порту 6432 с данной ВМ ( клиент postgresql стоит уже )

```bash
psql -h localhost -p 6432 -U postgres
Пароль пользователя postgres:
psql (16.6, сервер 14.15 (Debian 14.15-1.pgdg120+1))
Введите "help", чтобы получить справку.
```

- создадим БД и таблицу

```
postgres=# create database test;
CREATE DATABASE
postgres=# \l
                                                        Список баз данных
    Имя    | Владелец | Кодировка | Провайдер локали | LC_COLLATE |  LC_CTYPE  | локаль ICU | Правила ICU |     Права доступа
-----------+----------+-----------+------------------+------------+------------+------------+-------------+-----------------------
 postgres  | postgres | UTF8      | libc             | en_US.utf8 | en_US.utf8 |            |             |
 template0 | postgres | UTF8      | libc             | en_US.utf8 | en_US.utf8 |            |             | =c/postgres          +
           |          |           |                  |            |            |            |             | postgres=CTc/postgres
 template1 | postgres | UTF8      | libc             | en_US.utf8 | en_US.utf8 |            |             | =c/postgres          +
           |          |           |                  |            |            |            |             | postgres=CTc/postgres
 test      | postgres | UTF8      | libc             | en_US.utf8 | en_US.utf8 |            |             |
(4 строки)

postgres=# \c test
psql (16.6, сервер 14.15 (Debian 14.15-1.pgdg120+1))
Вы подключены к базе данных "test" как пользователь "postgres".
test=# create table test1 ( id integer );
CREATE TABLE
test=# \dt
          Список отношений
 Схема  |  Имя  |   Тип   | Владелец
--------+-------+---------+----------
 public | test1 | таблица | postgres
(1 строка)
```

- Рассмотрим вариант подключения к бд из контейнера в контейнер. 
Есть готовые образы клиентов postgresql ( jbergknoff/postgresql-client, codingpuss/postgres-client).
Но они уже устаревшие с клентом 12 версии.
Создадим свой образ с клиентом 14 версии.

- Создадим каталог pgclient и файл dockerfile

```
FROM alpine:3.15
RUN apk --no-cache add postgresql14-client
ENTRYPOINT [ "psql" ]
```

- соберём docker образ с именем pgclient14

```bash
boss@ubutest:~/pgclient$ docker build -t pgclient14 .
[+] Building 1.1s (6/6) FINISHED                                                                                                                    docker:default
 => [internal] load build definition from dockerfile                                                                                                          0.0s
 => => transferring dockerfile: 118B                                                                                                                          0.0s
 => [internal] load metadata for docker.io/library/alpine:3.15                                                                                                1.1s
 => [internal] load .dockerignore                                                                                                                             0.0s
 => => transferring context: 2B                                                                                                                               0.0s
 => [1/2] FROM docker.io/library/alpine:3.15@sha256:19b4bcc4f60e99dd5ebdca0cbce22c503bbcff197549d7e19dab4f22254dc864                                          0.0s
 => CACHED [2/2] RUN apk --no-cache add postgresql14-client                                                                                                   0.0s
 => exporting to image                                                                                                                                        0.0s
 => => exporting layers                                                                                                                                       0.0s
 => => writing image sha256:07e5848d14e98db6d0d6c486e336dd87dc11d7d11d408e16797a73f90d83ec7a                                                                  0.0s
 => => naming to docker.io/library/pgclient14  
```

- определим IP адрес docker контейнера с postgresql

```bash
boss@ubutest:~$ docker inspect otus_postgres | grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.2",
                    "IPAddress": "172.17.0.2",
```

- запускаем контейнер клиента с подключением к БД

```bash
boss@ubutest:~/pgclient$ docker run -it --rm pgclient14 postgresql://postgres:postgres@172.17.0.2:5432/test
psql (14.10, server 14.15 (Debian 14.15-1.pgdg120+1))
Type "help" for help.

test=#
```

- добавляем строки в существующую уже таблицу

```
test=# insert into test1 values (12),(23),(35),(26);
INSERT 0 4
test=# select * from test1;
 id
----
 12
 23
 35
 26
(4 rows)
```

- подключаемся с другой ВМ (astra 1.8)  к ВМ с docker контейнером postgresql

```
boss@astra8:~$ psql postgresql://postgres:postgres@192.168.1.244:6432/test
psql (17.2, сервер 14.15 (Debian 14.15-1.pgdg120+1))
Введите "help", чтобы получить справку.

test=# select * from test1;
 id
----
 12
 23
 35
 26
(4 строки)
```

- удаляем docker контейнер postgres и создаём его заново и уточняем IP адрес

```bash
boss@ubutest:~/pgclient$ docker rm -f otus_postgres
otus_postgres
boss@ubutest:~/pgclient$ docker run -p 6432:5432 -v /var/lib/postgres:/var/lib/postgresql/data -e POSTGRES_PASSWORD=postgres -d --name otus_postgres postgres:14
f8ba9aaa45a7c93a9228c8cadd5073efbd28fb34fb6e2c0293d5a416c488fcbb
boss@ubutest:~/pgclient$ docker inspect otus_postgres | grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.2",
                    "IPAddress": "172.17.0.2",
``` 

- проверяем данные в БД через клиент контейнер

```
boss@ubutest:~/pgclient$ docker run -it --rm pgclient14 postgresql://postgres:postgres@172.17.0.2:5432/test
psql (14.10, server 14.15 (Debian 14.15-1.pgdg120+1))
Type "help" for help.

test=# select * from test1;
 id
----
 12
 23
 35
 26
(4 rows)
```

Вывод: монтирование тома позволяет хранить данные вне контейнера на основной файловой системе.