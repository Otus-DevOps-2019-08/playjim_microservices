# playjim_microservices
Репозиторий для работы над домашними заданиями в рамках курса **"DevOps практики и инструменты"**

**Содрежание:**
<a name="top"></a>
- [HW.12 - Технология контейнеризации. Введение в Docker](#hw12)
    - [Доп. задание](#jiji1)  
---

<a name="hw12"></a>
# Домашнее задание  12
## Технология контейнеризации. Введение в Docker

- Установлены:
```sh
$ docker version
Client: Docker Engine - Community
 Version:           19.03.1
 API version:       1.40
 Go version:        go1.12.5
 ...
Server: Docker Engine - Community
 Engine:
  Version:          19.03.1
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.5

$ docker-compose -v
docker-compose version 1.24.1, build 4667896b

$ docker-machine -v
docker-machine version 0.16.1, build cce350d7
```
- Запуск конта:
```sh
$ docker run hello-world
...
Hello from Docker!
```
- Список контов:
```sh
$ docker ps -a
```
- Список имиджей:
```sh
$ docker images
```
- Запуск и вход внутрь конта:
```sh
$ docker run -it ubuntu:16.04 /bin/bash
```
- **docker run** запускает новый конт каждый раз
- Если флаг **--rm**, то после остановки конта, содержимое будет удалено с диска
- Работа с **docker ps**
```sh
$ docker ps -a --format "table {{.ID}}\t{{.Image}}\t{{.CreatedAt}}\t{{.Names}}"
CONTAINER ID        IMAGE               CREATED AT                      NAMES
c6fcb5db6c5b        ubuntu:16.04        2019-11-19 00:27:04 +0300 MSK   musing_davinci
```
- Запуск созданного конта
```sh
$ docker start c6fcb5db6c5b
```
- Присоединить терминал к созданному конту
```sh
$ docker attach c6fcb5db6c5b
```
- 
```sh
docker run = docker create + docker start + docker attach
```
- **docker create** юзается, если не надо стартовать конт сразу после создания

### Docker run
- Через параметры передаются лимиты(cpu/mem/disk), ip, volumes  
*-i* – запускает контейнер в foreground режиме (docker attach)  
*-d* – запускает контейнер в background режиме  
*-t* - создает TTY  
*docker run -it ubuntu:16.04 bash*  
*docker run -dt nginx:latest*
### Docker exec
- Запускает новый процесс внутри контейнера
```sh
docker exec -it <u_container_id> bash
```
### Docker commit
- Создает image из контейнера
- Контейнер при этом остается запущенным
```sh
$ docker commit c6fcb5db6c5b jull/ubuntu-tmp-file
$ docker images
REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
jull/ubuntu-tmp-file   latest              cd948fb78b1e        9 seconds ago       123MB
```
<a name="jiji1"></a>
## Доп. задание
- Информация о контейнере
```sh
$ docker inspect c6fcb5db6c5b
```
- Информация об имидже
```sh
$ docker inspect 5f2bf26e3524
```
- Описание отличий контейнера и образа в файле docker-monolith/docker-1.log

### Docker kill & stop
- **kill** сразу посылает SIGKILL
- **stop** посылает SIGTERM, и через 10 секунд посылает SIGKILL
- **SIGTERM** - сигнал остановки приложения 
- **SIGKILL** - безусловное завершение процесса
```sh
$ docker ps -q
c6fcb5db6c5b
$ docker kill $(docker ps -q)
c6fcb5db6c5b
```
### Docker system df
- Отображает сколько дискового пространства занято образами, контейнерами и volume’ами
- Отображает сколько из них не используется и возможно удалить
```sh
$ docker system df
```
### Docker rm & rmi
- **rm** - удаляет контейнер, можно добавить флаг **-f**, чтобы удалялся работающий container
- **rmi** удаляет image, если от него не зависят запущенные контейнеры
```sh
$ docker rm $(docker ps -a -q)
fb9e81c240cf
c6fcb5db6c5b
```
```sh
$ docker rmi $(docker images -q)
Untagged: jull/ubuntu-tmp-file:latest
Deleted: sha256:cd948fb78b1e621df8fed21133b738d404b6c554962f72891e5a06f1824ca59a
...
```

## Docker контейнеры. GCE.
- Создал новый проект
- На своей машине:
```sh
gcloud init
gcloud auth application-default login
```
### Docker machine
- **docker-machine** - встроенный в докер инструмент для создания хостов и установки на них docker engine. Имеет поддержку облаков и систем виртуализации (Virtualbox, GCP и др.)  

**Команда создания:** 
```sh
docker-machine create <имя>
```
**Переключение между именами:**
```sh
eval $(docker-machine env <имя>)
```
**Переключение на локальный докер:**
```sh
eval $(docker-machine env --unset)
```
**Удаление**
```sh
docker-machine rm <имя>
```
- **docker-machine** создает хост для докер демона с указываемым образом в *--google-machine-image*. Образы, которые используются для построения докер контейнеров, к этому никак не относятся.
- Все докер команды, которые запускаются в той же консоли после *eval $(docker-machine env <имя>)* работают с удаленным докер демоном в GCP.  

```sh
$ export GOOGLE_PROJECT=docker-666666

$ docker-machine create --driver google \
--google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
--google-machine-type n1-standard-1 \
--google-zone europe-west1-b \
docker-host

...
Docker is up and running!
```
- Проверяем, что наш Docker-хост успешно создан:
```sh
$ docker-machine ls
NAME          ACTIVE   DRIVER   STATE     URL                       SWARM   DOCKER     ERRORS
docker-host   -        google   Running   tcp://35.240.64.47:2376           v19.03.5

$ eval $(docker-machine env docker-host)
```

Отличия вывода команд:
```sh
docker run --rm -ti tehbilly/htop
```
Выводит только PID 1, с которым запущен htop (т.е. процессы в неймспейсе конта)
```sh
docker run --rm --pid host -ti tehbilly/htop
```
Выводит процессы хостовой машины, на которой запущен докер (т.е. процессы неймспейса хостовой машины)  

- Создал в каталоге docker-monolith файлы:
  - [mongod.conf](https://gist.githubusercontent.com/Lisskha/790957b5e3b103fcae09a02626f6de25/raw/98e76c1d9084f37e05d9f37d17f8b0a6b0124d65/mongod.conf) - конфиг для mongodb
  - [start.sh](https://gist.githubusercontent.com/Lisskha/790957b5e3b103fcae09a02626f6de25/raw/98e76c1d9084f37e05d9f37d17f8b0a6b0124d65/start.sh) - скрипт запуска приложения
  - [db_config](https://gist.githubusercontent.com/Lisskha/790957b5e3b103fcae09a02626f6de25/raw/98e76c1d9084f37e05d9f37d17f8b0a6b0124d65/db_config) - переменная окружения со ссылкой на mongodb
  - [Dockerfile](https://gist.githubusercontent.com/Lisskha/790957b5e3b103fcae09a02626f6de25/raw/98e76c1d9084f37e05d9f37d17f8b0a6b0124d65/Dockerfile) - текстовое описание образа

- Сборка образа (запускается в каталоге docker-monolith)
```sh
docker build -t reddit:latest .
```
*-t* - тег для собранного образа
- Смотреть все образы:
```sh
$ docker images -a
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
reddit              latest              4e8932235b85        23 minutes ago      692MB
<none>              <none>              b19e58d2fdb5        23 minutes ago      692MB
<none>              <none>              f92db4a5c445        23 minutes ago      692MB
...
```
- Запуск контейнера и проверка результата:
```sh
$ docker run --name reddit -d --network=host reddit:latest

5a8623f5c010f0adaa3b64dac44646ebd10452ec20c4a1d33cd7a77ded49e2d4

$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
5a8623f5c010        reddit:latest       "/start.sh"              10 seconds ago      Up 8 seconds                            reddit

$ docker-machine ls
NAME          ACTIVE   DRIVER   STATE     URL                       SWARM   DOCKER     ERRORS
docker-host   *        google   Running   tcp://35.240.64.47:2376           v19.03.5
```
- Добавил правило для фаерволла:
```sh
$ gcloud compute firewall-rules create reddit-app \
--allow tcp:9292 \
--target-tags=docker-machine \
--description="Allow PUMA connections" \
--direction=INGRESS
```
- Проверка  
http://35.240.64.47:9292/

### Docker Hub
**[Docker hub](https://hub.docker.com/)** - облачный registry сервис от компании Docker  
- Аутентифицировался на докер хаб:
```sh
$ docker login
...
Login Succeeded
```
- Протегировал и запушил свой образ в докер хаб:
```sh
$ docker tag reddit:latest <my-login>/otus-reddit:1.0
$ docker push <my-login>/otus-reddit:1.0
```
- Скачала и запустил образ на локальной тачке:
```sh
$ docker run --name reddit -d -p 9292:9292 <my-login>/otus-reddit:1.0

$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<my-login>/otus-reddit   1.0                 4e8932235b85        54 minutes ago      692MB

$ docker ps -a
CONTAINER ID        IMAGE                   COMMAND             CREATED              STATUS              PORTS                    NAMES
40f026f1ba90        <my-login>/otus-reddit:1.0   "/start.sh"         About a minute ago   Up About a minute   0.0.0.0:9292->9292/tcp   reddit
```
- Проверка  
http://localhost:9292/

- **Команды:**
```sh
# Смотреть логи конта
$ docker logs 40f026f1ba90 -f

# Зайти в конт
$ docker exec -it 40f026f1ba90 bash
    ps aux
    # Убить контейнер
    killall5 1

# Проверить что конт стопнулся
$ docker ps -a
    STATUS - Exited

# Стартануть конт и проверить что он запустился
$ docker start 40f026f1ba90 && docker ps -a

# Остановить и удалить конт
$ docker stop 40f026f1ba90; docker rm 40f026f1ba90

# Запуск конта без запуска приложения
$ docker run --name reddit --rm -it <my-login>/otus-reddit:1.0 bash

# Смотреть инфу об образе
$ docker inspect 4e8932235b85

# Смотреть куски информации
$ docker inspect 4e8932235b85 -f '{{.ContainerConfig.Cmd}}'
[/bin/sh -c #(nop)  CMD ["/start.sh"]]

# Запуск конта с приложением из образа (по image id)
$ docker run --name reddit -d -p 9292:9292 4e8932235b85

29eed99a817e4b2a77492cf71dd762d373e4f90eea28efaaa458c155970997ee

# Зайти в конт, добавить/удалить файлы/каталоги
$ docker exec -it reddit bash
    mkdir /test111 && touch test111/test.txt
    rmdir opt/

# Посмотреть внесенные изменения
$ docker diff 29eed99a817e

# Пересоздать конт и проверить что изменения не сохранились
$ docker stop reddit && docker rm reddit && docker run --name reddit --rm -it 4e8932235b85 bash -c ls

```
[Содержание](#top)
