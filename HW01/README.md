$\textsf{\color{blue}Домашнее задание - SQL и реляционные СУБД}$

- создал виртуальную машину в гипервизоре VMWare ESXi 
c параметрами vCPU=2/vRAM=4G/vHDD=40G
- установил ubuntu-2404 (host ubutest; user boss; ip 192.168.1.244)
- настроил доступ по ssh ключу <br>
    Воспользуемся под <b>Windows</b> терминалом <b>MobaXTerm</b> <br>
    Подключимся локальной сессией (псевдо Linux) и создадим ключи ssh    

        ssh-keygen -t ed25519

    Копируем ключи на виртуальную машину      

        ssh-copy-id -i id_ed25519.pub boss@192.168.1.244

    При настройке новых подключений используем закрытый ключ (файл id_ed25519)
- установка PostgreSQL 
  
    Установим версию PostgreSQL_1C из репозитория PostgresPro
      <details><summary><i>Подробнее о данной версии</i></summary>
        Открытый репозиторий бесплатной версии PostgreSQL с расширениями для работы с 1C. <br>
        Есть скрипт - настраивающий репозиторий для разных Linux <br>
        Однотипная установка и настройка на разных Linux <br>
        Возможность простого локального зеркала по версиям и Linux <br>
        При установке работет скрипт аналог PGTune определяющий параметры сервера <br>
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
        Active: active (running) since Tue 2024-11-19 10:43:23 UTC; 1min 4s ago
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

    ноя 19 10:43:23 ubutest systemd[1]: Starting postgrespro-1c-16.service - Postgres Pro 1c 16 database server...
    ноя 19 10:43:23 ubutest postgres[20787]: 2024-11-19 10:43:23.497 UTC [20787] СООБЩЕНИЕ:  передача вывода в протокол процессу сбора прото>
    ноя 19 10:43:23 ubutest postgres[20787]: 2024-11-19 10:43:23.497 UTC [20787] ПОДСКАЗКА:  В дальнейшем протоколы будут выводиться в катал>
    ноя 19 10:43:23 ubutest systemd[1]: Started postgrespro-1c-16.service - Postgres Pro 1c 16 database server.
```


$\textsf{\color{orange}Подключимся второй сессией ssh}$
```
    boss@ubutest:~$ sudo su - postgres
    postgres@ubutest:~$ psql
    psql (16.4)
    Введите "help", чтобы получить справку.

    postgres=# \c
    Вы подключены к базе данных "postgres" как пользователь "postgres".
    postgres=#
```


New string

The background color is `#ffffff` for light mode and `#000000` for dark mode.


${\color{blue}Create \tiny{\texttt{in a different 
font style}} \color{darkgreen}my home}$ <br>
