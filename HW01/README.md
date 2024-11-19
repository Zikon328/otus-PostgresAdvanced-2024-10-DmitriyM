$\textsf{\color{blue}Домашнее задание - SQL и реляционные СУБД}$

- создал виртуальную машину в гипервизоре VMWare ESXi 
c параметрами vCPU=2/vRAM=4G/vHDD=40G
- установил ubuntu-404 (host ubutest; user boss; ip 192.168.1.244)
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
        Возможность простого локального зеркала по версиям и Linux 
      </details>

$\textsf{\color{orange}Подключимся второй сессией ssh}$


New string

The background color is `#ffffff` for light mode and `#000000` for dark mode.


${\color{blue}Create \tiny{\texttt{in a different 
font style}} \color{darkgreen}my home}$ <br>
