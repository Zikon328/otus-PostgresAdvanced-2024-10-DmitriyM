
### Сборка WAL-G из исходников на Astra Linux 1.8

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
-- скопируем в основной bin каталог
boss@astra8:~/go/src/github.com/wal-g/wal-g$ sudo cp main/pg/wal-g /usr/bin/wal-g
```


