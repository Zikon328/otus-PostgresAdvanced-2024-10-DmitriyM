$\textsf{\color{blue}Домашнее задание - SQL и реляционные СУБД}$

- создал виртуальную машину в гипервизоре VMWare ESXi 
c параметрами vCPU=2/vRAM=4G/vHDD=40G
- установил ubuntu-2404 (host ubutest; user boss; ip 192.168.1.244)
- настроил доступ по ssh ключу
  <details>
    <summary><i>Подробное подключение по ssh</i></summary>
    Воспользуемся под <b>Windows</b> терминалом <b>MobaXTerm</b> <br>
    Подключимся локальной сессией (псевдо Linux) и создадим ключи ssh    

        ssh-keygen -t ed25519

    Копируем ключи на виртуальную машину      

        ssh-copy-id -i id_ed25519.pub boss@192.168.1.244

    Принастройке новых подключений используем закрытый ключ (файл id_ed25519)
    
  </details>
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
   sudo ./pgpro-repo-add.sh 
   sudo apt install postgrespro-1c-16
```



$\textsf{\color{orange}Подключимся второй сессией ssh}$


New string

The background color is `#ffffff` for light mode and `#000000` for dark mode.


${\color{blue}Create \tiny{\texttt{in a different 
font style}} \color{darkgreen}my home}$ <br>
