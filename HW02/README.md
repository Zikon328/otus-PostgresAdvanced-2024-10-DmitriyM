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

