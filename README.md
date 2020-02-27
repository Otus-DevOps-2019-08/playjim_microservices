# playjim_microservices
Репозиторий для работы над домашними заданиями в рамках курса **"DevOps практики и инструменты"**

**Содрежание:**
<a name="top"></a>
- [HW.12 - Технология контейнеризации. Введение в Docker](#hw12)
    - [Доп. задание](#jiji1)  
- [HW.13 - Docker-образы. Микросервисы](#hw13)
- [HW.14 - Docker: сети, docker-compose](#hw14)
- [HW.15 - Устройство Gitlab CI. Построение процесса непрерывной поставки](#hw15)
- [HW.16 - Введение в мониторинг. Системы мониторинга](#hw16)
- [HW.17 - Мониторинг приложения и инфраструктуры](#hw17)
- [HW.18 - Логирование и распределенная трассировка](#hw18)
- [HW.19 - Kubernetes - The Hard Way](#hw19)
- [HW.20 - Kubernetes. Запуск кластера и приложения. Модель безопасности.](#hw20)
- [HW.21 - Kubernetes. Networks ,Storages.](#hw21)
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
$ export GOOGLE_PROJECT=_ваш-проект_
$ docker-machine create --driver google --google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts --google-machine-type n1-standard-1 --google-zone europe-west1-b docker-host
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


---

<a name="hw13"></a>
# Домашнее задание  13
## Docker-образы. Микросервисы

[Написание Dockerfile](https://docs.docker.com/engine/reference/builder/)

При сборке `image` из одинакового базового образа первая команда `build` кеширует его, а последующие уже будут собираться не с первого шага.

## Задание со *
Для запуска контейнеров с определенной переменной окружения применяется ключ `--env`. Для указания нескольких переменных, указывается либо файл со значениями, либо несколько таких ключей.

Пример:
```
$ docker run -d --network=reddit1 --network-alias=post1_db --network-alias=comment1_db mongo:latest
$ docker run -d --network=reddit1 --network-alias=post1 --env POST_DATABASE_HOST=post1_db playjim/post:1.0
$ docker run -d --network=reddit1 --network-alias=comment1 --env COMMENT_DATABASE_HOST=comment1_db playjim/comment:1.0
$ docker run -d --network=reddit1 -p 9292:9292 --env POST_SERVICE_HOST=post1 --env COMMENT_SERVICE_HOST=comment1 playjim/ui:2.0
```
### Создание Docker volume:
```
$ docker volume create reddit_db
reddit_db
$ docker run -d --network=reddit1 --network-alias=post1_db --network-alias=comment1_db -v reddit_db:/data/db mongo:latest
```
[Содержание](#top)

<a name="hw14"></a>
# Домашнее задание  14
## Docker: сети, docker-compose

Типы сетей в Docker:
- none - только loopback, сеть изолирована
- host - использует сеть хоста
- bridge - отдельные namespaces сети ("виртуальная" сеть)
- MacVlan - на основе сабинтерфейсов Linux, поддерка 802.1Q
- Overlay - несколько Docker хостов в одну сеть, работает поверх VXLAN

При запуске контейнера можно указать только одну сеть параметром `--network=<name>`. Для подключения дополнительных сетей к контейнерам применить команду: `docker network connect`. Также несколько сетей могут быть подключены к контейнеру при запуске, если используется `docker-compose`.

[Документация Docker-compose](https://docs.docker.com/compose/)

Docker-compose позволяет запускать сразу несколько контейнеров по заданному сценарию.

Пример:
```
# docker-compose.yml

version: '3.3'
services:
  post_db:
    image: mongo:${TAGDB}
    volumes:
      - post_db:/data/db
    networks:
      - back_net
  ui:
    build: ./ui
    image: ${USERNAME}/ui:${TAG}
    ports:
      - ${PORT}/tcp
    networks:
      - front_net
  post:
    build: ./post-py
    image: ${USERNAME}/post:${TAG}
    networks:
      - back_net
      - front_net
  comment:
    build: ./comment
    image: ${USERNAME}/comment:${TAG}
    networks:
      - back_net
      - front_net
    
volumes:
  post_db:

networks:
  back_net:
  front_net:
```
Значения переменных подставляется или из переменных окружения или из файла `.env`. Имя проекта `docker-compose` присваивается от имени директории, в котором он располагается. Для переопределения этого имени используется параметр `-p` или переменная `COMPOSE_PROJECT_NAME`.
```
# .env

PORT=9292:9292
TAG=1.0
TAGDB=3.2
USERNAME=playjim
```
[Содержание](#top)

<a name="hw15"></a>
# Домашнее задание  15
## Устройство Gitlab CI. Построение процесса непрерывной поставки

### Инсталяция Gitlab CI
- Необходимые параметры для машины, на которой будем разворачивать Gitlab CI
  ```sh
  1               CPU
  3.75GB          RAM 
  50-100GB        HDD 
  Ubuntu 16.04    IMAGE
  ```
  - В [официальной документации](https://docs.gitlab.com/ce/install/requirements.html) описаны рекомендуемые характеристики сервера 

С помощью docker-machine создаем ВМ и не забываем разрешить подключение по http/https:
```
  $ docker-machine create --driver google \
  --google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
  --google-zone europe-west1-b \
  --google-machine-type n1-standard-1 \
  --google-disk-size 100 \
  gitlab-ci
    ...
  Docker is up and running!
  To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: 
  docker-machine env gitlab-ci

```
### Подготовка окружения
- На ВМ создаем каталоги и создаем файл [docker-compose.yml](https://gist.githubusercontent.com/Lisskha/a26e4c3f6ff7d94656fd1eb6a624131b/raw/b782c399a746bc7017b97f86a0a21971bf66313e/srv%2520gitlab%2520docker-compose.yml)
  ```sh
  $ eval $(docker-machine env gitlab-ci)
  $ docker-machine ssh gitlab-ci
  
  sudo su
  mkdir -p /srv/gitlab/config /srv/gitlab/data /srv/gitlab/logs
  cd /srv/gitlab/
  vim docker-compose.yml
  ```
- Запускаем сборку образа и запускаем Gitlab CI (надо запускать в каталоге, где лежит docker-compose.yml)
  ```sh
  /srv/gitlab# docker-compose up -d
  ```
- Откуда взяли содержимое файла docker-compose.yml - https://docs.gitlab.com/omnibus/docker/README.html#install-gitlab-using-docker-compose  
- Проверка  
  http://ip-vm/

## Работа с Gitlab CI
### Настройка
- Выкл регу новых юезров
  - В Settings/General/Sign-up restrictions сняла галку с Sign-up enabled 
### Создание проекта
***Из лекции:***
- Каждый проект в Gitlab CI принадлежит к группе проектов
- В проекте может быть определен CI/CD пайплайн
- Задачи (jobs), входящие в пайплайн, должны исполняться GitLab runner

### Запуск раннера в контейнере
```
docker run -d --name gitlab-runner --restart always \
-v /srv/gitlab-runner/config:/etc/gitlab-runner \
-v /var/run/docker.sock:/var/run/docker.sock \
gitlab/gitlab-runner:latest
```

### Регистрация раннера
```
docker exec -it gitlab-runner gitlab-runner register --run-untagged --locked=falseна runners
```
[Содержание](#top)

<a name="hw16"></a>
# Домашнее задание  16
## Введение в мониторинг. Системы мониторинга

### Подготовка окружения
Правим правила фаервола на gcp. Открываем порты для Prometheus и Puma:
```
$ gcloud compute firewall-rules create prometheus-default --allow tcp:9090
$ gcloud compute firewall-rules create puma-default --allow tcp:9292 
```
Создадим Docker хост в GCE и настроим локальное окружение на работу с ним:
```
$ export GOOGLE_PROJECT=_ваш-проект_

# create docker host
docker-machine create --driver google \
    --google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
    --google-machine-type n1-standard-1 \
    --google-zone europe-west1-b \
    docker-host

# configure local env
eval $(docker-machine env docker-host)
```
Систему мониторинга Prometheus будем запускать внутри
Docker контейнера. Для начального знакомства воспользуемся
готовым образом с DockerHub.
```
$ docker run --rm -p 9090:9090 -d --name prometheus prom/prometheus:v2.1.0 
$ docker ps 
```
Открываем веб интерфейс. По умолчанию сервер слушает на порту 9090. Чтобы узнать ip созданной ВМ, используем команду:
```
$ docker-machine ip docker-host
```
Соберем на основе готового образа с DockerHub свой Docker образ с конфигурацией для мониторинга наших микросервисов. 
Создайте директорию monitoring/prometheus. Затем в этой директории создайте простой Dockerfile, который будет копировать файл конфигурации с нашей машины внутрь контейнера:
```
#monitoring/prometheus/Dockerfile

FROM prom/prometheus:v2.1.0
ADD prometheus.yml /etc/prometheus/
```
Мы определим простой конфигурационный файл для сбора метрик с наших микросервисов. В директории *monitoring/prometheus* создайте файл **prometheus.yml** со следующим cодержимым:
```
---
global:
  scrape_interval: '5s'

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets:
        - 'localhost:9090'

  - job_name: 'ui'
    static_configs:
      - targets:
        - 'ui:9292'

  - job_name: 'comment'
    static_configs:
      - targets:
        - 'comment:9292'
```
В директории *prometheus* собираем Docker образ:
```
$ export USER_NAME=playjim
$ docker build -t $USER_NAME/prometheus .
```
Где USER_NAME - ВАШ логин от DockerHub. 

В коде микросервисов есть healthcheck-и для проверки работоспособности приложения. Сборку образов теперь необходимо производить при помощи скриптов **docker_build.sh**, которые есть в директории каждого сервиса. С его помощью мы добавим информацию из Git в наш healthcheck. 

Выполните сборку образов при помощи скриптов docker_build.sh в директории каждого сервиса.
```
/src/ui $ bash docker_build.sh
/src/post-py $ bash docker_build.sh
/src/comment $ bash docker_build.sh
```
Или сразу все из корня репозитория:
```
for i in ui post-py comment; do cd src/$i; bash docker_build.sh; cd -; done
```
Будем поднимать наш Prometheus совместно с микросервисами. Определите в вашем *docker/docker-compose.yml* файле новый сервис.
```
services:
...
  prometheus:
    image: ${USERNAME}/prometheus
    networks:
      - back_net
      - front_net
    ports:
      - '9090:9090'
    volumes:
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention=1d'

volumes:
  prometheus_data:
``` 
Не забываем добавить секцию networks для сервиса **prometheus**, чтобы он смог общаться со всеми серверами.

### Node exporter
Воспользуемся [Node_exporter](https://github.com/prometheus/node_exporter) для сбора информации о работе Docker хоста (виртуалки, где у нас запущены контейнеры)и предоставлению этой информации в Prometheus. 

### Пуш образов на DockerHub
Запушим собранные вами образы на DockerHub:
```
$ docker login
Login Succeeded
$ docker push $USER_NAME/ui
$ docker push $USER_NAME/comment
$ docker push $USER_NAME/post
$ docker push $USER_NAME/prometheus 
``` 
Ссылка на мой профиль в DockerHub: https://hub.docker.com/u/playjim

[Содержание](#top)

<a name="hw17"></a>
# Домашнее задание 17
## Мониторинг приложения и инфраструктуры

Сначала создаем на GCP ВМ с докером.
```
$ export GOOGLE_PROJECT=_ваш-проект_

# Создать докер хост
docker-machine create --driver google \
    --google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
    --google-machine-type n1-standard-1 \
    --google-zone europe-west1-b \
    docker-host

# Настроить докер клиент на удаленный докер демон
eval $(docker-machine env docker-host)

# Переключение на локальный докер
eval $(docker-machine env --unset)

$ docker-machine ip docker-host

$ docker-machine rm docker-host
```
Оставим описание приложений в **docker-compose.yml**, а мониторинг выделим в отдельный файл **docker-compose-monitoring.yml**.
Для запуска приложений будем как и ранее использовать`docker-compose up -d`, а для мониторинга - ` docker-compose -f docker-compose-monitoring.yml up -d`

Мы будем использовать [cAdvisor](https://github.com/google/cadvisor) для наблюдения за состоянием наших Docker контейнеров.
cAdvisor собирает информацию о ресурсах потребляемых контейнерами и характеристиках их работы.

Добавим информацию о новом сервисе в конфигурацию Prometheus, чтобы он начал собирать метрики:
```
scrape_configs:
...
  - job_name: 'cadvisor'
    static_configs:
    - targets:
      - 'cadvisor:8080'
```

Пересоберем образ Prometheus с обновленной конфигурацией:
```
$ export USER_NAME=username # где username - ваш логин на Docker Hub
$ docker build -t $USER_NAME/prometheus .
```

Не забываем открыть порты:
```
$ gcloud compute firewall-rules create prometheus-default --allow tcp:9090
$ gcloud compute firewall-rules create puma-default --allow tcp:9292
$ gcloud compute firewall-rules create cadvisor-default --allow tcp:8080

```
Запустим сервисы:
```
$ docker-compose up -d
$ docker-compose -f docker-compose-monitoring.yml up -d
```
cAdvisor имеет UI, в котором отображается собираемая о контейнерах информация.
Откроем страницу Web UI по адресу http://<docker-machinehost-ip>:8080

### Визуализация метрик: Grafana
Используем инструмент Grafana для визуализации данных из Prometheus.
Добавим новый сервис в **docker-compose-monitoring.yml**:
```
  grafana:
    image: grafana/grafana:5.0.0
    volumes:
      - grafana_data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=secret
    depends_on:
      - prometheus
    ports:
      - 3000:3000
    networks:
      - front_net
      - back_net

volumes:
  grafana_data:
```
Внесем правки в правила firewall gcp:
```
$ gcloud compute firewall-rules create grafana-default --allow tcp:3000
```
Alertmanager - дополнительный компонент для системы мониторинга Prometheus, который отвечает за первичную обработку алертов и дальнейшую отправку оповещений по заданному назначению. Создайте новую директорию monitoring/alertmanager. В этой директории создайте **Dockerfile** со следующим содержимым:
```
FROM prom/alertmanager:v0.14.0
ADD config.yml /etc/alertmanager/
```
Настройки Alertmanager-а как и Prometheus задаются через YAML файл или опции командой строки. В директории monitoring/alertmanager создайте файл config.yml, в котором определите отправку нотификаций в ВАШ тестовый слак канал. Для отправки нотификаций в слак канал потребуется создать СВОЙ Incoming Webhook monitoring/alertmanager/config.yml:
```
global:
  slack_api_url: 'https://hooks.slack.com/services/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'

route:
  receiver: 'slack-notifications'

receivers:
- name: 'slack-notifications'
  slack_configs:
  - channel: '#dmitry_borisov'

```
1. Соберем образ alertmanager:
```
monitoring/alertmanager $ docker build -t $USER_NAME/alertmanager .
```
2. Добавим новый сервис в компоуз файл мониторинга. Не забудьте
добавить его в одну сеть с сервисом Prometheus:
```
services:
...
alertmanager:
  image: ${USER_NAME}/alertmanager
  command:
    - '--config.file=/etc/alertmanager/config.yml'
  ports:
    - 9093:9093
  networks:
    - front_net
    - back_net
```

Создадим файл **alerts.yml** в директории prometheus, в котором определим условия при которых должен срабатывать алерт и посылаться Alertmanager-у. Мы создадим простой алерт, который будет срабатывать в ситуации, когда одна из наблюдаемых систем (endpoint) недоступна для сбора метрик (в этом случае метрика up с лейблом instance равным имени данного эндпоинта будет равна нулю).

**monitoring/prometheus/alerts.yml**:
```
groups:
  - name: alert.rules
    rules:
    - alert: InstanceDown
      expr: up == 0
      for: 1m
      labels:
        severity: page
      annotations:
        description: '{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute'
        summary: 'Instance {{ $labels.instance }} down'
```
Добавим операцию копирования данного файла в Dockerfile:
**monitoring/prometheus/Dockerfile**:
```
FROM prom/prometheus:v2.1.0
ADD prometheus.yml /etc/prometheus/
ADD alerts.yml /etc/prometheus/
```
**prometheus.yml**:
```
rule_files:
  - "alerts.yml"

alerting:
  alertmanagers:
  - scheme: http
    static_configs:
    - targets:
      - "alertmanager:9093"
```
Пересоберем образ Prometheus.

Ссылка на мой профиль DockerHub:
https://hub.docker.com/u/playjim

[Содержание](#top)

<a name="hw18"></a>
# Домашнее задание 18
## Логирование и распределенная трассировка

Создаем Docker хост в GCE:
```
$ export GOOGLE_PROJECT=docker-xxxxxx

$ docker-machine create --driver google --google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts --google-machine-type n1-standard-1 --google-open-port 5601/tcp --google-open-port 9292/tcp --google-open-port 9411/tcp logging

$ eval $(docker-machine env logging)

$ docker-machine ip logging
```
Обновив код в директории /src выполняем сборку образов ui, post-py, comment:
```
export USER_NAME=playjim

/src/ui $ bash docker_build.sh && docker push $USER_NAME/ui
/src/post-py $ bash docker_build.sh && docker push $USER_NAME/post
/src/comment $ bash docker_build.sh && docker push $USER_NAME/comment
```
Или сразу все из корня репозитория:
```
for i in ui post-py comment; do cd src/$i; bash docker_build.sh; cd -; done
```
Как упоминалось на лекции хранить все логи стоит централизованно: на одном (нескольких) серверах. В этом ДЗ мы рассмотрим пример системы централизованного логирования на примере Elastic стека (ранее известного как ELK): который включает в себя 3 осовных компонента:
 - ElasticSearch (TSDB и поисковый движок для хранения данных)
 - Logstash (для агрегации и трансформации данных)
 - Kibana (для визуализации)
Однако для агрегации логов вместо Logstash мы будем использовать Fluentd, таким образом получая еще одно популярное сочетание этих инструментов, получившее название EFK

Для нашей системы логирования создаем отдельный compose-файл **docker/docker-compose-logging.yml**:
```
version: '3.3'
services:
  zipkin:
    image: openzipkin/zipkin
    ports:
      - "9411:9411"
    networks:
      - front_net
      - back_net

  fluentd:
    image: ${USERNAME}/fluentd
    ports:
      - "24224:24224"
      - "24224:24224/udp"

  elasticsearch:
    image: elasticsearch:7.5.0
    container_name: elasticsearch
    environment:
      - xpack.security.enabled=false
      - discovery.type=single-node
      - bootstrap.memory_lock=true
    ulimits:
      memlock:
        soft: -1
        hard: -1
    expose:
      - 9200
    ports:
      - "9200:9200"
      - "9300:9300"

  kibana:
    image: kibana:7.5.0
    container_name: kibana
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch

networks:
  front_net:
  back_net:
```

[Содержание](#top)

<a name="hw19"></a>
# Домашнее задание 19
## Kubernetes - The Hard Way

Опишем приложение в контексте Kubernetes с помощью manifest-ов в YAML-формате. Основным примитивом будет Deployment. Основные задачи сущности Deployment:      
 - Создание Replication Controller-а (следит, чтобы число запущенных Pod-ов соответствовало описанному);
 - Ведение истории версий запущенных Pod-ов (для различных стратегий деплоя, для возможностей отката);
 - Описание процесса деплоя (стратегия, параметры стратегий).

**post-deployment.yml**:
```
---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: post-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: post
  template:
    metadata:
      name: post
      labels:
        app: post
    spec:
      containers:
      - image: chromko/post
        name: post
```
Также я опишу **comment-deployment.yml**, **mongo-deployment.yml**, **ui-deployment.yml**;

# [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)
Для практики и в качестве обучения я пройду ["Kubernetes The Hard Way"](https://github.com/kelseyhightower/kubernetes-the-hard-way).

## Запуск команд параллельно с tmux

[tmux] (https://github.com/tmux/tmux/wiki) можно использовать для одновременного запуска команд на нескольких экземплярах вычислений. Лабораторные работы в этом руководстве могут потребовать выполнения одних и тех же команд для нескольких вычислительных экземпляров, в этих случаях рассмотрите возможность использования tmux и разбиения окна на несколько панелей с включенными синхронизирующими панелями, чтобы ускорить процесс подготовки.

> Разделить окно вертикально `ctrl + b "` 
> Разделить окно горизонтально `ctrl + b %`
> Включите синхронизацию, нажав `ctrl + b`, а затем` shift +: `. Далее введите `setw synchronize-panes on` в командной строке. Чтобы отключить синхронизацию: `setw synchronize-panes off`.
Для управления мышью нужно создать конфиг ~/.tmux.conf:
```
set-option -g -q mouse on
bind-key -T root WheelUpPane if-shell -F -t = "#{alternate_on}" "send-keys -M" "select-pane -t =; copy-mode -e; send-keys -M"
bind-key -T root WheelDownPane if-shell -F -t = "#{alternate_on}" "send-keys -M" "select-pane -t =; send-keys -M"
```
## Установка необходимых клиентских интсрументов

Утилиты командной строки `cfssl` и` cfssljson` будут использоваться для обеспечения [инфраструктуры PKI](https://en.wikipedia.org/wiki/Public_key_infrastructure) и создания сертификатов TLS.

The `kubectl` command line utility is used to interact with the Kubernetes API Server.

## Настройка gcp и выделение ресурсов под кластер
Создаю `kubernetes-the-hard-way` пользовательскую сеть VPC:
```
gcloud compute networks create kubernetes-the-hard-way --subnet-mode custom
```
A [subnet](https://cloud.google.com/compute/docs/vpc/#vpc_networks_and_subnets) must be provisioned with an IP address range large enough to assign a private IP address to each node in the Kubernetes cluster.
```
gcloud compute networks subnets create kubernetes \
  --network kubernetes-the-hard-way \
  --range 10.240.0.0/24
```
## Правила брандмауэра
Создаю правило брандмауэра, которое разрешает внутреннюю связь по всем протоколам:
```
gcloud compute firewall-rules create kubernetes-the-hard-way-allow-internal \
  --allow tcp,udp,icmp \
  --network kubernetes-the-hard-way \
  --source-ranges 10.240.0.0/24,10.200.0.0/16
```
Создаю правило брандмауэра, которое разрешает внешний SSH, ICMP и HTTPS:
```
gcloud compute firewall-rules create kubernetes-the-hard-way-allow-external \
  --allow tcp:22,tcp:6443,icmp \
  --network kubernetes-the-hard-way \
  --source-ranges 0.0.0.0/0
```
## Публичный ip-адрес k8s
Выделяю статический IP-адрес, который будет подключен к внешнему балансировщику нагрузки на серверах API Kubernetes:
```
gcloud compute addresses create kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region)
```
## Compute instances
### Kubernetes Controllers
Создаю три вычислительных экземпляра, в которых будет размещена плоскость управления Kubernetes:
```
for i in 0 1 2; do
  gcloud compute instances create controller-${i} \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-1804-lts \
    --image-project ubuntu-os-cloud \
    --machine-type n1-standard-1 \
    --private-network-ip 10.240.0.1${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubernetes-the-hard-way,controller
done
```
### Kubernetes Workers
Каждому рабочему экземпляру требуется выделение подсети pod из диапазона CIDR кластера Kubernetes. Выделение подсети под будет использоваться для настройки сети контейнера в дальнейшем упражнении.`pod-cidr` Метаданные экземпляра будет использоваться , чтобы выставить стручок подсеть распределения в случаи вычислительных во время выполнения.

Создаю три вычислительных экземпляра, в которых будут размещаться workerы Kubernetes:
```
for i in 0 1 2; do
  gcloud compute instances create worker-${i} \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-1804-lts \
    --image-project ubuntu-os-cloud \
    --machine-type n1-standard-1 \
    --metadata pod-cidr=10.200.${i}.0/24 \
    --private-network-ip 10.240.0.2${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubernetes-the-hard-way,worker
done
```
### Настройка доступа по SSH
Для подключения к экземплярам я буду использовать ssh.
```
gcloud compute ssh controller-0
```
## Provisioning a CA and Generating TLS Certificates

### Certificate Authority  
<details>
<summary> Генерим CA конф файл, серт и приватный ключ</summary>
<p>
 
```sh
$ cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF

$ cat > ca-csr.json <<EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "Oregon"
    }
  ]
}
EOF

# Генерим серт и ключ
$ cfssl gencert -initca ca-csr.json | cfssljson -bare ca

# Результат
$ ll
-rw-------    ca-key.pem
-rw-r--r--    ca.pem
```
</p>
</details> 

### Client and Server Certificates
Генерим клиентские и серверные сертификаты для каждого компонента Kubernetes и клиентский сертификат для админа Kubernetes
  
<details>
<summary> The Admin Client Certificate </summary>
<p>

```sh
cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:masters",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

# Генерим серт и ключ
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin

# Результат
-rw-------   admin-key.pem
-rw-r--r--   admin.pem
```

</p>
</details>  

<details>
<summary> The Kubelet Client Certificates </summary>
<p>

```sh
for instance in worker-0 worker-1 worker-2; do
cat > ${instance}-csr.json <<EOF
{
  "CN": "system:node:${instance}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:nodes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

EXTERNAL_IP=$(gcloud compute instances describe ${instance} \
  --format 'value(networkInterfaces[0].accessConfigs[0].natIP)')

INTERNAL_IP=$(gcloud compute instances describe ${instance} \
  --format 'value(networkInterfaces[0].networkIP)')

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${instance},${EXTERNAL_IP},${INTERNAL_IP} \
  -profile=kubernetes \
  ${instance}-csr.json | cfssljson -bare ${instance}
done

# Результат
-rw-------   worker-0-key.pem
-rw-r--r--   worker-0.pem
-rw-------   worker-1-key.pem
-rw-r--r--   worker-1.pem
-rw-------   worker-2-key.pem
-rw-r--r--   worker-2.pem
```

</p>
</details>  

<details>
<summary> The Controller Manager Client Certificate </summary>
<p>

```sh
cat > kube-controller-manager-csr.json <<EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-controller-manager",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

# Генерим серт и ключ
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager

# Результат
-rw-------   kube-controller-manager-key.pem
-rw-r--r--   kube-controller-manager.pem
```

</p>
</details>  

<details>
<summary> The Kube Proxy Client Certificate </summary>
<p>

```sh
cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:node-proxier",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

# Генерим серт и ключ
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy

# Результат
-rw-------   kube-proxy-key.pem
-rw-r--r--   kube-proxy.pem
```

</p>
</details>  

<details>
<summary> The Scheduler Client Certificate </summary>
<p>

```sh
cat > kube-scheduler-csr.json <<EOF
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-scheduler",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

# Генерим серт и ключ
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-scheduler-csr.json | cfssljson -bare kube-scheduler

# Результат
-rw-------   kube-scheduler-key.pem
-rw-r--r--   kube-scheduler.pem
```

</p>
</details>  

<details>
<summary> The Kubernetes API Server Certificate </summary>
<p>

```sh
# Вытащить статик ип из GCE
KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region) \
  --format 'value(address)')

KUBERNETES_HOSTNAMES=kubernetes,kubernetes.default,kubernetes.default.svc,kubernetes.default.svc.cluster,kubernetes.svc.cluster.local

cat > kubernetes-csr.json <<EOF
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

# Генерим серт и ключ
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=10.32.0.1,10.240.0.10,10.240.0.11,10.240.0.12,${KUBERNETES_PUBLIC_ADDRESS},127.0.0.1,${KUBERNETES_HOSTNAMES} \
  -profile=kubernetes \
  kubernetes-csr.json | cfssljson -bare kubernetes

# Результат
-rw-------   kubernetes-key.pem
-rw-r--r--   kubernetes.pem
```

</p>
</details> 

> The kubernetes-the-hard-way static IP address will be included in the list of subject alternative names for the Kubernetes API Server certificate. This will ensure the certificate can be validated by remote clients.  

> The Kubernetes API server is automatically assigned the kubernetes internal dns name, which will be linked to the first IP address (10.32.0.1) from the address range (10.32.0.0/24) reserved for internal cluster services during the [control plane bootstrapping](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/08-bootstrapping-kubernetes-controllers.md#configure-the-kubernetes-api-server) lab.

<details>
<summary> The Service Account Key Pair </summary>
<p>

```sh
cat > service-account-csr.json <<EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

# Генерим серт и ключ
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  service-account-csr.json | cfssljson -bare service-account

# Результат
-rw-------   service-account-key.pem
-rw-r--r--   service-account.pem
```

</p>
</details>  

### Distribute the Client and Server Certificates
- Копируем серты и ключи на воркеры
```sh
for instance in worker-0 worker-1 worker-2; do
  gcloud compute scp ca.pem ${instance}-key.pem ${instance}.pem ${instance}:~/
done
```
- Копируем серты и ключи на контроллеры
```sh
for instance in controller-0 controller-1 controller-2; do
  gcloud compute scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
    service-account-key.pem service-account.pem ${instance}:~/
done
```
## Генерация конфигурационных файлов Kubernetes для аутентификации
### Client Authentication Configs
Будем генерить kubeconfig файлы для `controller manager`, `kubelet`, `kube-proxy`, планировщика и админа.  
- **`Kubernetes Public IP Address`**  
  Каждому kubeconfig нужно канекаться к Kubernetes API Server. Для доступности заасайним статик ip на внешний LB за которым будет Kubernetes API Servers
    - Извлечь статик IP
      ```sh
      KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
      --region $(gcloud config get-value compute/region) \
      --format 'value(address)')
      ```
- **`The kubelet Kubernetes Configuration File`**  
  При создании файлов kubeconfig для Kubelets должен использоваться сертификат клиента, соответствующий имени узла Kubelet. Тогда будет норм работать Kubernetes [Node Authorizer](https://kubernetes.io/docs/reference/access-authn-authz/node/)  
  > Команды нужно запускать в той же дире, где генерили серты в разделе [Provisioning a CA and Generating TLS Certificates](#Provisioning-a-CA-and-Generating-TLS-Certificates) 
    - Генерация kubeconfig файлов для каждого воркера
      ```sh
      for instance in worker-0 worker-1 worker-2; do kubectl config set-cluster kubernetes-the-hard-way \
      --certificate-authority=ca.pem \
      --embed-certs=true \
      --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443 \
      --kubeconfig=${instance}.kubeconfig

      kubectl config set-credentials system:node:${instance} \
      --client-certificate=${instance}.pem \
      --client-key=${instance}-key.pem \
      --embed-certs=true \
      --kubeconfig=${instance}.kubeconfig

      kubectl config set-context default \
      --cluster=kubernetes-the-hard-way \
      --user=system:node:${instance} \
      --kubeconfig=${instance}.kubeconfig

      kubectl config use-context default --kubeconfig=${instance}.kubeconfig
      done
      ```
    - Создались файлы
      ```
      -rw-------   worker-0.kubeconfig
      -rw-------   worker-1.kubeconfig
      -rw-------   worker-2.kubeconfig
      ```
- **`The kube-proxy Kubernetes Configuration File`**
  - Генерация kubeconfig файла для kube-proxy
    ```sh
    kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443 \
    --kubeconfig=kube-proxy.kubeconfig

    kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.pem \
    --client-key=kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

    kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

    kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
    ``` 
  - Создался файл
    ```
    -rw-------   kube-proxy.kubeconfig
    ```
- **`The kube-controller-manager Kubernetes Configuration File`**
  - Генерим kubeconfig для kube-controller-manager
    ```sh
    kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

    kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.pem \
    --client-key=kube-controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

    kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

    kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
    ```
  - Файл готов
    ```
    -rw-------   kube-controller-manager.kubeconfig
    ```
- **`The kube-scheduler Kubernetes Configuration File`**
  - Генерим kubeconfig для kube-scheduler
    ```sh
    kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

    kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.pem \
    --client-key=kube-scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

    kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

    kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
    ```
  - Файл готов
    ```
    -rw-------   kube-scheduler.kubeconfig
    ```
- **`The admin Kubernetes Configuration File`**
  - Генерим kubeconfig для админа
    ```sh
    kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=admin.kubeconfig

    kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

    kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=admin \
    --kubeconfig=admin.kubeconfig

    kubectl config use-context default --kubeconfig=admin.kubeconfig
    ```
  - Файл готов
    ```
    -rw-------   admin.kubeconfig
    ```

### Distribute the Kubernetes Configuration Files
- Копируем кубеконфиги kubelet и kube-proxy на воркеры
```sh
for instance in worker-0 worker-1 worker-2; do
  gcloud compute scp ${instance}.kubeconfig kube-proxy.kubeconfig ${instance}:~/
done
```
- Копируем кубеконфиги admin, kube-controller-manager и kube-scheduler на контроллеры
```sh
for instance in controller-0 controller-1 controller-2; do
  gcloud compute scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig ${instance}:~/
done
```
## Generating the Data Encryption Config and Key
Kubernetes хранит различные данные, включая состояние кластера, конфиги приложенией и секреты. Kubernetes поддерживает возможность [шифрования данных](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/) кластера в состоянии покоя.  
Создадим ключ шифрования и [encryption config](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#understanding-the-encryption-at-rest-configuration), подходящие для шифрования Kubernetes Secrets 

### The Encryption Key
```sh
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```
### The Encryption Config File
- Создали файл `encryption-config.yaml`
```sh
cat > encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```
- И скопировали его на контроллеры
```sh
for instance in controller-0 controller-1 controller-2; do
  gcloud compute scp encryption-config.yaml ${instance}:~/
done
```
## Bootstrapping the etcd Cluster
Kubernetes components are stateless and store cluster state in [etcd](https://github.com/etcd-io/etcd). We will bootstrap a three node etcd cluster and configure it for high availability and secure remote access.

### Prerequisites
> Все дальнейшие команды нужно запускать на каждом инстансе: `controller-0`, `controller-1`, and `controller-2`
- Логинимся на контроллеры
```sh
$ gcloud compute ssh controller-0
$ gcloud compute ssh controller-1
$ gcloud compute ssh controller-2
```
### Bootstrapping an etcd Cluster Member
- Качаем официальные [etcd release binaries](https://github.com/etcd-io/etcd/releases) с GitHub
  ```sh
  wget -q --show-progress --https-only --timestamping \
  "https://github.com/etcd-io/etcd/releases/download/v3.4.0/etcd-v3.4.0-linux-amd64.tar.gz"
  ```
- Разархивируем и устанавливаем `etcd server` и `etcdctl`
  ```sh
  tar -xvf etcd-v3.4.0-linux-amd64.tar.gz
  sudo mv etcd-v3.4.0-linux-amd64/etcd* /usr/local/bin/
  ```
- Конфигурим etcd server
  ```sh
  sudo mkdir -p /etc/etcd /var/lib/etcd
  sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
  ```
  - The instance internal IP address will be used to serve client requests and communicate with etcd cluster peers. Retrieve the internal IP address for the current compute instance
  ```sh
  INTERNAL_IP=$(curl -s -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/network-interfaces/0/ip)
  ```
  - Each etcd member must have a unique name within an etcd cluster. Set the etcd name to match the hostname of the current compute instance
  ```sh
  ETCD_NAME=$(hostname -s)
  ```
  - Создаем systemd unit `etcd.service` и стартуем etcd server

<details>
<summary> etcd.service</summary>
<p>

```sh
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/kubernetes.pem \\
  --peer-key-file=/etc/etcd/kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster controller-0=https://10.240.0.10:2380,controller-1=https://10.240.0.11:2380,controller-2=https://10.240.0.12:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

</p>
</details>  

```sh
sudo systemctl daemon-reload
sudo systemctl enable etcd
sudo systemctl start etcd
```

### Verification
- List the etcd cluster members
```sh
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
```
> output
```sh
3a57933972cb5131, started, controller-2, https://10.240.0.12:2380, https://10.240.0.12:2379, false
f98dc20bce6225a0, started, controller-0, https://10.240.0.10:2380, https://10.240.0.10:2379, false
ffed16798470cab5, started, controller-1, https://10.240.0.11:2380, https://10.240.0.11:2379, false
```
## Bootstrapping the Kubernetes Control Plane
In this lab you will bootstrap the Kubernetes control plane across three compute instances and configure it for high availability. You will also create an external load balancer that exposes the Kubernetes API Servers to remote clients. The following components will be installed on each node: Kubernetes API Server, Scheduler, and Controller Manager.

### Prerequisites
> Все дальнейшие команды нужно запускать на каждом инстансе: `controller-0`, `controller-1`, and `controller-2`
- Логинимся на контроллеры
```sh
$ gcloud compute ssh controller-0
$ gcloud compute ssh controller-1
$ gcloud compute ssh controller-2
```

### Provision the Kubernetes Control Plane
- Create the Kubernetes configuration directory  
  `sudo mkdir -p /etc/kubernetes/config`
- Download the official Kubernetes release binaries
```sh
wget -q --show-progress --https-only --timestamping \
  "https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-apiserver" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-controller-manager" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-scheduler" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kubectl"
```
- Install the Kubernetes binaries
```sh
chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl
sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
```

**`Configure the Kubernetes API Server`**
```sh
sudo mkdir -p /var/lib/kubernetes/
sudo mv ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
service-account-key.pem service-account.pem \
encryption-config.yaml /var/lib/kubernetes/
```
- The instance internal IP address will be used to advertise the API Server to members of the cluster. Retrieve the internal IP address for the current compute instance
```sh
INTERNAL_IP=$(curl -s -H "Metadata-Flavor: Google" \
http://metadata.google.internal/computeMetadata/v1/instance/network-interfaces/0/ip)
```
- Create the kube-apiserver.service systemd unit file

<details>
<summary> kube-apiserver.service </summary>
<p>

```sh
cat <<EOF | sudo tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\
  --enable-admission-plugins=NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
  --etcd-cafile=/var/lib/kubernetes/ca.pem \\
  --etcd-certfile=/var/lib/kubernetes/kubernetes.pem \\
  --etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \\
  --etcd-servers=https://10.240.0.10:2379,https://10.240.0.11:2379,https://10.240.0.12:2379 \\
  --event-ttl=1h \\
  --encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
  --kubelet-client-certificate=/var/lib/kubernetes/kubernetes.pem \\
  --kubelet-client-key=/var/lib/kubernetes/kubernetes-key.pem \\
  --kubelet-https=true \\
  --runtime-config=api/all \\
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/kubernetes.pem \\
  --tls-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

</p>
</details>  

**`Configure the Kubernetes Controller Manager`**  
- Move the kube-controller-manager kubeconfig into place and create the `kube-controller-manager.service` systemd unit file
```sh
sudo mv kube-controller-manager.kubeconfig /var/lib/kubernetes/
```
<details>
<summary> kube-controller-manager.service </summary>
<p>

```sh
cat <<EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --address=0.0.0.0 \\
  --cluster-cidr=10.200.0.0/16 \\
  --cluster-name=kubernetes \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \\
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.pem \\
  --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

</p>
</details>  

**`Configure the Kubernetes Scheduler`**  
- Move the kube-scheduler kubeconfig into place and create the `kube-scheduler.yaml` configuration file
```sh
sudo mv kube-scheduler.kubeconfig /var/lib/kubernetes/
```
```sh
cat <<EOF | sudo tee /etc/kubernetes/config/kube-scheduler.yaml
apiVersion: kubescheduler.config.k8s.io/v1alpha1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
leaderElection:
  leaderElect: true
EOF
```
- Create the `kube-scheduler.service` systemd unit file
```sh
cat <<EOF | sudo tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --config=/etc/kubernetes/config/kube-scheduler.yaml \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

**`Start the Controller Services`**  
```sh
sudo systemctl daemon-reload
sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
```

### Enable HTTP Health Checks

A [Google Network Load Balancer](https://cloud.google.com/load-balancing/docs/network/) will be used to distribute traffic across the three API servers and allow each API server to terminate TLS connections and validate client certificates. The network load balancer only supports HTTP health checks which means the HTTPS endpoint exposed by the API server cannot be used. As a workaround the nginx webserver can be used to proxy HTTP health checks. In this section nginx will be installed and configured to accept HTTP health checks on port 80 and proxy the connections to the API server on https://127.0.0.1:6443/healthz  

> The /healthz API server endpoint does not require authentication by default  

Установим базовый веб-сервер для хелсчеков
```sh
sudo apt-get update
sudo apt-get install -y nginx
```
```sh
cat > kubernetes.default.svc.cluster.local <<EOF
server {
  listen      80;
  server_name kubernetes.default.svc.cluster.local;

  location /healthz {
     proxy_pass                    https://127.0.0.1:6443/healthz;
     proxy_ssl_trusted_certificate /var/lib/kubernetes/ca.pem;
  }
}
EOF
```
```sh
sudo mv kubernetes.default.svc.cluster.local \
    /etc/nginx/sites-available/kubernetes.default.svc.cluster.local

sudo ln -s /etc/nginx/sites-available/kubernetes.default.svc.cluster.local /etc/nginx/sites-enabled/
```
```sh
sudo systemctl restart nginx
sudo systemctl enable nginx
```
**`Verification`**
На каждом контроллере проверяем статус
```sh
kubectl get cs --kubeconfig admin.kubeconfig

NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok
etcd-1               Healthy   {"health":"true"}
controller-manager   Healthy   ok
etcd-0               Healthy   {"health":"true"}
etcd-2               Healthy   {"health":"true"}
```
Проверяем хелсчеки (тоже на контроллерах)
```sh
curl -H "Host: kubernetes.default.svc.cluster.local" -i http://127.0.0.1/healthz

HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Sun, 05 Jan 2020 01:44:07 GMT
Content-Type: text/plain; charset=utf-8
Content-Length: 2
Connection: keep-alive
X-Content-Type-Options: nosniff
ok
```

## RBAC for Kubelet Authorization

In this section you will configure RBAC permissions to allow the Kubernetes API Server to access the Kubelet API on each worker node. Access to the Kubelet API is required for retrieving metrics, logs, and executing commands in pods.

> This tutorial sets the Kubelet --authorization-mode flag to Webhook. Webhook mode uses the [Subject Access Review](https://kubernetes.io/docs/reference/access-authn-authz/authorization/#checking-api-access) API to determine authorization

Дальнейшие команды влияют на весь кластер, и их нужно запускать только один раз с одного из узлов контроллера
```sh
gcloud compute ssh controller-0
```
Создаем `system:kube-apiserver-to-kubelet` [Cluster Role](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole) с разрешениями для доступа к API Kubelet и выполнения наиболее распространенных задач, связанных с управлением подами 
```sh
cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
EOF
```
Kubernetes API Server аутентифицируется в Kubelet как пользователь `kubernetes`, используя сертификат клиента, как определено флагом `--kubelet-client-certificate`  
Bind the `system:kube-apiserver-to-kubelet` ClusterRole to the `kubernetes` user
```sh
cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF
```

### The Kubernetes Frontend Load Balancer
In this section you will provision an external load balancer to front the Kubernetes API Servers. The `kubernetes-the-hard-way` static IP address will be attached to the resulting load balancer.

> The compute instances created in this tutorial will not have permission to complete this section. **Run the following commands from the same machine used to create the compute instances**

**`Provision a Network Load Balancer`**
- Create the external load balancer network resources
```sh
KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region) \
  --format 'value(address)')

gcloud compute http-health-checks create kubernetes \
  --description "Kubernetes Health Check" \
  --host "kubernetes.default.svc.cluster.local" \
  --request-path "/healthz"

gcloud compute firewall-rules create kubernetes-the-hard-way-allow-health-check \
  --network kubernetes-the-hard-way \
  --source-ranges 209.85.152.0/22,209.85.204.0/22,35.191.0.0/16 \
  --allow tcp

gcloud compute target-pools create kubernetes-target-pool \
  --http-health-check kubernetes

gcloud compute target-pools add-instances kubernetes-target-pool \
  --instances controller-0,controller-1,controller-2

gcloud compute forwarding-rules create kubernetes-forwarding-rule \
  --address ${KUBERNETES_PUBLIC_ADDRESS} \
  --ports 6443 \
  --region $(gcloud config get-value compute/region) \
  --target-pool kubernetes-target-pool
```

**`Verification`**  
> The compute instances created in this tutorial will not have permission to complete this section. Run the following commands from the same machine used to create the compute instances.

- Получаем статик IP `kubernetes-the-hard-way`
```sh
KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region) \
  --format 'value(address)')
```
- Курлом смотрим инфо о версии кубика в кластере
```sh
curl --cacert ca.pem https://${KUBERNETES_PUBLIC_ADDRESS}:6443/version

{
  "major": "1",
  "minor": "15",
  "gitVersion": "v1.15.3",
  "gitCommit": "2d3c76f9091b6bec110a5e63777c332469e0cba2",
  "gitTreeState": "clean",
  "buildDate": "2019-08-19T11:05:50Z",
  "goVersion": "go1.12.9",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```
## Bootstrapping the Kubernetes Worker Nodes

Будем бутстрапить воркеры. На каждой ноде будут установлены: 
- [runc](https://github.com/opencontainers/runc)
- [container networking plugins](https://github.com/containernetworking/cni)
- [containerd](https://github.com/containerd/containerd)
- [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/)
- [kube-proxy](https://kubernetes.io/docs/concepts/cluster-administration/proxies/)

> Команды будут выполняться на каждом воркере
- Заходим на каждый воркер
```sh
gcloud compute ssh worker-0
```

### Provisioning a Kubernetes Worker Node
- Install the OS dependencies
  ```sh
  sudo apt-get update
  sudo apt-get -y install socat conntrack ipset
  ```
  > The socat binary enables support for the `kubectl port-forward` command
- Disable Swap
  > By default the kubelet will fail to start if swap is enabled. It is recommended that swap be disabled to ensure Kubernetes can provide proper resource allocation and quality of service.
  - Verify if swap is enabled
    ```sh
    sudo swapon --show
    ```
    - Если вывод пустой, то свап отклюен
  - Disable swap
    ```sh
    sudo swapoff -a
    ```
  > To ensure swap remains off after reboot consult your Linux distro documentation

### Download and Install Worker Binaries
```sh
wget -q --show-progress --https-only --timestamping \
https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.15.0/crictl-v1.15.0-linux-amd64.tar.gz \
https://github.com/opencontainers/runc/releases/download/v1.0.0-rc8/runc.amd64 \
https://github.com/containernetworking/plugins/releases/download/v0.8.2/cni-plugins-linux-amd64-v0.8.2.tgz \
https://github.com/containerd/containerd/releases/download/v1.2.9/containerd-1.2.9.linux-amd64.tar.gz \
https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kubectl \
https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-proxy \
https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kubelet
```
- Create the installation directories
```sh
sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```
- Install the worker binaries
```sh
mkdir containerd
tar -xvf crictl-v1.15.0-linux-amd64.tar.gz
tar -xvf containerd-1.2.9.linux-amd64.tar.gz -C containerd
sudo tar -xvf cni-plugins-linux-amd64-v0.8.2.tgz -C /opt/cni/bin/
sudo mv runc.amd64 runc
chmod +x crictl kubectl kube-proxy kubelet runc 
sudo mv crictl kubectl kube-proxy kubelet runc /usr/local/bin/
sudo mv containerd/bin/* /bin/
```

### Configure CNI Networking
- Retrieve the Pod CIDR range for the current compute instance
```sh
POD_CIDR=$(curl -s -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/attributes/pod-cidr)
```
- Create the bridge network configuration file
```sh
cat <<EOF | sudo tee /etc/cni/net.d/10-bridge.conf
{
    "cniVersion": "0.3.1",
    "name": "bridge",
    "type": "bridge",
    "bridge": "cnio0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "ranges": [
          [{"subnet": "${POD_CIDR}"}]
        ],
        "routes": [{"dst": "0.0.0.0/0"}]
    }
}
EOF
```
- Create the loopback network configuration file
```sh
cat <<EOF | sudo tee /etc/cni/net.d/99-loopback.conf
{
    "cniVersion": "0.3.1",
    "name": "lo",
    "type": "loopback"
}
EOF
```

### Configure containerd
- Create the containerd configuration file
```sh
sudo mkdir -p /etc/containerd/
```
```sh
cat << EOF | sudo tee /etc/containerd/config.toml
[plugins]
  [plugins.cri.containerd]
    snapshotter = "overlayfs"
    [plugins.cri.containerd.default_runtime]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runc"
      runtime_root = ""
EOF
```
- Create the containerd.service systemd unit file
<details>
<summary> containerd.service </summary>
<p>

```sh
cat <<EOF | sudo tee /etc/systemd/system/containerd.service
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target

[Service]
ExecStartPre=/sbin/modprobe overlay
ExecStart=/bin/containerd
Restart=always
RestartSec=5
Delegate=yes
KillMode=process
OOMScoreAdjust=-999
LimitNOFILE=1048576
LimitNPROC=infinity
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF
```

</p>
</details>

### Configure the Kubelet
```sh
  sudo mv ${HOSTNAME}-key.pem ${HOSTNAME}.pem /var/lib/kubelet/
  sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
  sudo mv ca.pem /var/lib/kubernetes/
```
- Create the kubelet-config.yaml configuration file
<details>
<summary> kubelet-config.yaml </summary>
<p>

```sh
cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.32.0.10"
podCIDR: "${POD_CIDR}"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/${HOSTNAME}.pem"
tlsPrivateKeyFile: "/var/lib/kubelet/${HOSTNAME}-key.pem"
EOF
```

</p>
</details>

  > The resolvConf configuration is used to avoid loops when using CoreDNS for service discovery on systems running `systemd-resolved`

- Create the kubelet.service systemd unit file:
<details>
<summary> kubelet.service </summary>
<p>

```sh
cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --container-runtime=remote \\
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

</p>
</details>

### Configure the Kubernetes Proxy
```sh
sudo mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```
- Create the kube-proxy-config.yaml configuration file
```sh
cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "10.200.0.0/16"
EOF
```
- Create the kube-proxy.service systemd unit file
<details>
<summary> kube-proxy.service </summary>
<p>

```sh
cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

</p>
</details>

### Start the Worker Services
```sh
  sudo systemctl daemon-reload
  sudo systemctl enable containerd kubelet kube-proxy
  sudo systemctl start containerd kubelet kube-proxy
```

### Verification
> The compute instances created in this tutorial will not have permission to complete this section. Run the following commands from the same machine used to create the compute instances.

List the registered Kubernetes nodes
```sh
gcloud compute ssh controller-0 \
--command "kubectl get nodes --kubeconfig admin.kubeconfig"

NAME       STATUS   ROLES    AGE   VERSION
worker-0   Ready    <none>   80s   v1.15.3
worker-1   Ready    <none>   77s   v1.15.3
worker-2   Ready    <none>   75s   v1.15.3
```
## Configuring kubectl for Remote Access
In this lab you will generate a kubeconfig file for the `kubectl` command line utility based on the `admin` user credentials.

> Run the commands in this lab from the same directory used to generate the admin client certificates.

### The Admin Kubernetes Configuration File
Each kubeconfig requires a Kubernetes API Server to connect to. To support high availability the IP address assigned to the external load balancer fronting the Kubernetes API Servers will be used.

Generate a kubeconfig file suitable for authenticating as the `admin` user
```sh
  KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
    --region $(gcloud config get-value compute/region) \
    --format 'value(address)')

  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem

  kubectl config set-context kubernetes-the-hard-way \
    --cluster=kubernetes-the-hard-way \
    --user=admin

  kubectl config use-context kubernetes-the-hard-way
```

### Verification
- Check the health of the remote Kubernetes cluster
```sh
$ kubectl get componentstatuses

NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health":"true"}
etcd-1               Healthy   {"health":"true"}
etcd-2               Healthy   {"health":"true"}
```
- List the nodes in the remote Kubernetes cluster
```sh
$ kubectl get nodes

NAME       STATUS   ROLES    AGE     VERSION
worker-0   Ready    <none>   7m30s   v1.15.3
worker-1   Ready    <none>   7m27s   v1.15.3
worker-2   Ready    <none>   7m25s   v1.15.3
```
## Provisioning Pod Network Routes

Pods scheduled to a node receive an IP address from the node's Pod CIDR range. At this point pods can not communicate with other pods running on different nodes due to missing network routes.

In this lab you will create a route for each worker node that maps the node's Pod CIDR range to the node's internal IP address.

> There are other ways to implement the Kubernetes networking model.

### The Routing Table
In this section you will gather the information required to create routes in the `kubernetes-the-hard-way` VPC network.
- Смотреть внутренний IP-адрес и диапазон Pod CIDR для каждого воркера
```sh
$ for instance in worker-0 worker-1 worker-2; do
  gcloud compute instances describe ${instance} \
  --format 'value[separator=" "](networkInterfaces[0].networkIP,metadata.items[0].value)'
  done

10.240.0.20 10.200.0.0/24
10.240.0.21 10.200.1.0/24
10.240.0.22 10.200.2.0/24
```

### Routes
- Create network routes for each worker instance
```sh
$ for i in 0 1 2; do
  gcloud compute routes create kubernetes-route-10-200-${i}-0-24 \
    --network kubernetes-the-hard-way \
    --next-hop-address 10.240.0.2${i} \
    --destination-range 10.200.${i}.0/24
  done
```
- List the routes in the kubernetes-the-hard-way VPC network
```sh
$ gcloud compute routes list --filter "network: kubernetes-the-hard-way"

NAME                            NETWORK                  DEST_RANGE     NEXT_HOP                  PRIORITY
default-route-68e820c79db00527  kubernetes-the-hard-way  10.240.0.0/24  kubernetes-the-hard-way   1000
default-route-e0148b1790f684a4  kubernetes-the-hard-way  0.0.0.0/0      default-internet-gateway  1000
kubernetes-route-10-200-0-0-24  kubernetes-the-hard-way  10.200.0.0/24  10.240.0.20               1000
kubernetes-route-10-200-1-0-24  kubernetes-the-hard-way  10.200.1.0/24  10.240.0.21               1000
kubernetes-route-10-200-2-0-24  kubernetes-the-hard-way  10.200.2.0/24  10.240.0.22               1000
```
## Deploying the DNS Cluster Add-on

In this lab you will deploy the [DNS add-on](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) which provides DNS based service discovery, backed by [CoreDNS](https://coredns.io/), to applications running inside the Kubernetes cluster.

### The DNS Cluster Add-on
- Deploy the `coredns` cluster add-on
```sh
$ kubectl apply -f https://storage.googleapis.com/kubernetes-the-hard-way/coredns.yaml

serviceaccount/coredns created
clusterrole.rbac.authorization.k8s.io/system:coredns created
clusterrolebinding.rbac.authorization.k8s.io/system:coredns created
configmap/coredns created
deployment.apps/coredns created
service/kube-dns created
```
- List the pods created by the `kube-dns` deployment
```sh
$ kubectl get pods -l k8s-app=kube-dns -n kube-system

NAME                     READY   STATUS    RESTARTS   AGE
coredns-5fb99965-8h4qp   1/1     Running   0          76s
coredns-5fb99965-h9cfc   1/1     Running   0          76s
```
### Verification
- Create a `busybox` deployment
```sh
$ kubectl run --generator=run-pod/v1 busybox --image=busybox:1.28 --command -- sleep 3600
```
- List the pod created by the `busybox` deployment
```sh
$ kubectl get pods -l run=busybox

NAME      READY   STATUS    RESTARTS   AGE
busybox   1/1     Running   0          45s
```
- Retrieve the full name of the busybox pod
```sh
POD_NAME=$(kubectl get pods -l run=busybox -o jsonpath="{.items[0].metadata.name}")
```
- Execute a DNS lookup for the kubernetes service inside the busybox pod
```sh
$ kubectl exec -ti $POD_NAME -- nslookup kubernetes

Server:    10.32.0.10
Address 1: 10.32.0.10 kube-dns.kube-system.svc.cluster.local
Name:      kubernetes
Address 1: 10.32.0.1 kubernetes.default.svc.cluster.local
```
## Smoke Test

In this lab you will complete a series of tasks to ensure your Kubernetes cluster is functioning correctly.

### Data Encryption
In this section you will verify the ability to [encrypt secret data at rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#verifying-that-data-is-encrypted)

- Create a generic secret
```sh
kubectl create secret generic kubernetes-the-hard-way \
  --from-literal="mykey=mydata"
```
- Print a hexdump of the `kubernetes-the-hard-way` secret stored in etcd
```sh
gcloud compute ssh controller-0 \
  --command "sudo ETCDCTL_API=3 etcdctl get \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem\
  /registry/secrets/default/kubernetes-the-hard-way | hexdump -C"
```
> The etcd key should be prefixed with `k8s:enc:aescbc:v1:key1`, which indicates the `aescbc` provider was used to encrypt the data with the `key1` encryption key.

### Deployments
In this section you will verify the ability to create and manage [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

- Create a deployment for the [nginx](https://nginx.org/en/) web server
```sh
$ kubectl create deployment nginx --image=nginx
```
- List the pod created by the nginx deployment
```sh
$ kubectl get pods -l app=nginx

NAME                     READY   STATUS    RESTARTS   AGE
nginx-554b9c67f9-djhrq   1/1     Running   0          39s
```

### Port Forwarding
In this section you will verify the ability to access applications remotely using [port forwarding](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)

- Retrieve the full name of the nginx pod
```sh
POD_NAME=$(kubectl get pods -l app=nginx -o jsonpath="{.items[0].metadata.name}")
```
- Forward port 8080 on your local machine to port 80 of the nginx pod
```sh
$ kubectl port-forward $POD_NAME 8080:80

Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```
- In a new terminal make an HTTP request using the forwarding address
```sh
$ curl --head http://127.0.0.1:8080

HTTP/1.1 200 OK
Server: nginx/1.17.6
Date: Mon, 06 Jan 2020 01:29:00 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 19 Nov 2019 12:50:08 GMT
Connection: keep-alive
ETag: "5dd3e500-264"
Accept-Ranges: bytes
```

### Logs
In this section you will verify the ability to retrieve container logs.

- Print the nginx pod logs
```sh
$ kubectl logs $POD_NAME

127.0.0.1 - - [06/Jan/2020:01:29:00 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.64.1" "-"
```
### Exec
In this section you will verify the ability to [execute commands in a container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/#running-individual-commands-in-a-container)

- Print the nginx version by executing the nginx -v command in the nginx container
```sh
$ kubectl exec -ti $POD_NAME -- nginx -v

nginx version: nginx/1.17.6
```

## Services
In this section you will verify the ability to expose applications using a [Service](https://kubernetes.io/docs/concepts/services-networking/service/)

- Expose the nginx deployment using a [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport) service
```sh
$ kubectl expose deployment nginx --port 80 --type NodePort

service/nginx exposed
```
> The LoadBalancer service type can not be used because your cluster is not configured with [cloud provider integration](https://kubernetes.io/docs/setup/#cloud-provider). Setting up cloud provider integration is out of scope for this tutorial.

- Retrieve the node port assigned to the nginx service
```sh
NODE_PORT=$(kubectl get svc nginx \
  --output=jsonpath='{range .spec.ports[0]}{.nodePort}')
```
- Create a firewall rule that allows remote access to the nginx node port
```sh
gcloud compute firewall-rules create kubernetes-the-hard-way-allow-nginx-service \
  --allow=tcp:${NODE_PORT} \
  --network kubernetes-the-hard-way
```
- Retrieve the external IP address of a worker instance
```sh
EXTERNAL_IP=$(gcloud compute instances describe worker-0 \
  --format 'value(networkInterfaces[0].accessConfigs[0].natIP)')
```
- Make an HTTP request using the external IP address and the nginx node port
```sh
$ curl -I http://${EXTERNAL_IP}:${NODE_PORT}

HTTP/1.1 200 OK
Server: nginx/1.17.6
Date: Mon, 06 Jan 2020 01:37:04 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 19 Nov 2019 12:50:08 GMT
Connection: keep-alive
ETag: "5dd3e500-264"
Accept-Ranges: bytes
```
## Cleaning Up

In this lab you will delete the compute resources created during this tutorial.

## Compute Instances
- Delete the controller and worker compute instances
```sh
gcloud -q compute instances delete \
  controller-0 controller-1 controller-2 \
  worker-0 worker-1 worker-2 \
  --zone $(gcloud config get-value compute/zone)
```
## Networking
- Delete the external load balancer network resources
```sh
gcloud -q compute forwarding-rules delete kubernetes-forwarding-rule \
  --region $(gcloud config get-value compute/region)

gcloud -q compute target-pools delete kubernetes-target-pool

gcloud -q compute http-health-checks delete kubernetes

gcloud -q compute addresses delete kubernetes-the-hard-way
```
- Delete the kubernetes-the-hard-way firewall rules
```sh
gcloud -q compute firewall-rules delete \
  kubernetes-the-hard-way-allow-nginx-service \
  kubernetes-the-hard-way-allow-internal \
  kubernetes-the-hard-way-allow-external \
  kubernetes-the-hard-way-allow-health-check
```
- Delete the kubernetes-the-hard-way network VPC
```sh
gcloud -q compute routes delete \
  kubernetes-route-10-200-0-0-24 \
  kubernetes-route-10-200-1-0-24 \
  kubernetes-route-10-200-2-0-24

gcloud -q compute networks subnets delete kubernetes

gcloud -q compute networks delete kubernetes-the-hard-way
```

[Содержание](#top)

<a name="hw20"></a>
# Домашнее задание 20
## Kubernetes. Запуск кластера и приложения. Модель безопасности.
### План
 - Развернуть локальное окружение для работы с Kubernetes
 - Развернуть Kubernetes в GKE
 - Запустить reddit в Kubernetes

### Разварачиваю локальное окружение Kubernetes
1) **kubectl** - фактически, главная утилиты для работы
c Kubernetes API (все, что делает kubectl, можно
сделать с помощью HTTP-запросов к API k8s)
2) Директории **~/.kube** - содержит служебную инфу
для kubectl (конфиги, кеши, схемы API)
3) **minikube** - утилиты для разворачивания локальной
инсталляции Kubernetes. 

## Kubectl
Устанавливаю [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

## Minikube
Для работы Minukube требуется локальный гипервизор. Я буду использовать VirtualBox

Инструкция по установке Minikube для разных ОС:
https://kubernetes.io/docs/tasks/tools/install-minikube/

После установки запускаю Minikube:
```
$ minikube start
😄  minikube v1.7.2 on Ubuntu 18.04
✨  Automatically selected the virtualbox driver
💿  Downloading VM boot image ...
    > minikube-v1.7.0.iso.sha256: 65 B / 65 B [--------------] 100.00% ? p/s 0s
    > minikube-v1.7.0.iso: 166.68 MiB / 166.68 MiB [-] 100.00% 8.69 MiB p/s 20s
🔥  Creating virtualbox VM (CPUs=2, Memory=2000MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.17.2 on Docker 19.03.5 ...
💾  Downloading kubectl v1.17.2
💾  Downloading kubelet v1.17.2
💾  Downloading kubeadm v1.17.2
🚀  Launching Kubernetes ... 
🌟  Enabling addons: default-storageclass, storage-provisioner
⌛  Waiting for cluster to come online ...
🏄  Done! kubectl is now configured to use "minikube"
```
P.S. Если нужна конкретная версия kubernetes, указывайте флаг
```--kubernetes-version <version> (v1.8.0)```

P.P.S.По-умолчанию используется VirtualBox. Если у вас другой гипервизор, то ставьте флаг
```--vm-driver=<hypervisor> ```

Наш Minikube-кластер развернут. При этом автоматически был настроен конфиг kubectl.
Проверим, что это так:
```
$ kubectl get nodes
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   18m   v1.17.2
```
Конфигурация kubectl - это **контекст**.

Контекст - это комбинация:
1) cluster - API-сервер
2) user - пользователь для подключения к кластеру
3) namespace - область видимости (не обязательно, поумолчанию default) 
Информацию о контекстах kubectl сохраняет в файле ```~/.kube/config```:
```
$ cat ~/.kube/config 
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/playjim/.minikube/ca.crt
    server: https://192.168.99.100:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /home/playjim/.minikube/client.crt
    client-key: /home/playjim/.minikube/client.key
```
Кластер (cluster) - это:
1) server - адрес kubernetes API-сервера
2) certificate-authority - корневой сертификат (которым
подписан SSL-сертификат самого сервера), чтобы
убедиться, что нас не обманывают и перед нами тот
самый сервер
+ name (Имя) для идентификации в конфиге

Пользователь (user) - это:
1) Данные для аутентификации (зависит от того, как настроен
сервер). Это могут быть:
- username + password (Basic Auth
- client key + client certificate
- token
- auth-provider config (например GCP)
+ name (Имя) для идентификации в конфиге

Контекст (контекст) - это:
1) cluster - имя кластера из списка clusters
2) user - имя пользователя из списка users
3) namespace - область видимости по-умолчанию (не
обязательно)
+ name (Имя) для идентификации в конфиге


Обычно порядок конфигурирования kubectl следующий:
1) Создать cluster:
$ kubectl config set-cluster … cluster_name
2) Создать данные пользователя (credentials)
$ kubectl config set-credentials … user_name
3) Создать контекст
$ kubectl config set-context context_name \
--cluster=cluster_name \
--user=user_name
4) Использовать контекст
$ kubectl config use-context context_name

Таким образом kubectl конфигурируется для подключения к
разным кластерам, под разными пользователями.
Текущий контекст можно увидеть так: 
```
$ kubectl config current-context
minikube
```
Список всех контекстов можно увидеть так: 
```
$ kubectl config get-contexts
CURRENT   NAME       CLUSTER    AUTHINFO   NAMESPACE
*         minikube   minikube   minikube   
```

## Запуск приложения reddit
Каталог с yaml манифестами приложения находится в **./kubernetes/reddit**

Основные объекты - это ресурсы Deployment.
Как помним из предыдущего занятия, основные его задачи:
+ Создание ReplicationSet (следит, чтобы число запущенных
Pod-ов соответствовало описанному)
+ Ведение истории версий запущенных Pod-ов (для
различных стратегий деплоя, для возможностей отката)
+ Описание процесса деплоя (стратегия, параметры
стратегий)

Для запуска приложения сначало создаю namespace **dev**, а потом применяю конфигурацию:
```
$ kubectl apply -f ./kubernetes/reddit/dev-namespace.yml
namespace/dev created
$ kubectl apply -f ./kubernetes/reddit/ -n dev
deployment.apps/comment created
service/comment-db created
service/comment created
namespace/dev unchanged
deployment.apps/mongo created
service/mongodb created
deployment.apps/post created
service/post-db created
service/post created
deployment.apps/ui created
service/ui created
```
Minikube может выдавать web-странцы с сервисами
которые были помечены типом **NodePort**
Попробуйте:
```
$ minikube service list
```
Получить список расширений:
```
$ minikube addons list
```

<a name="hw21"></a>
# Домашнее задание 21
## Kubernetes. Networks ,Storages.
### План
- Ingress Controller
- Ingress
- Secret
- TLS
- LoadBalancer Service
- Network Policies
- PersistentVolumes
- PersistentVolumeClaims

- [HW.21 - Kubernetes. Networks ,Storages.](#hw21)

### Service - определяет конечные узлы доступа (Endpoint’ы):
- селекторные сервисы (k8s сам находит POD-ы по label’ам)
- безселекторные сервисы (мы вручную описываем
конкретные endpoint’ы)

и способ коммуникации с ними (тип (type) сервиса):
- ClusterIP - дойти до сервиса можно только изнутри
кластера
- nodePort - клиент снаружи кластера приходит на
опубликованный порт
- LoadBalancer - клиент приходит на облачный (aws elb,
Google gclb) ресурс балансировки
- ExternalName - внешний ресурс по отношению к кластеру

**ClusterIP** - это виртуальный (в реальности нет интерфейса,
pod’а или машины с таким адресом) IP-адрес из диапазона
адресов для работы внутри, скрывающий за собой IP-адреса
реальных POD-ов. Сервису любого типа (кроме
ExternalName) назначается этот IP-адрес. 

Kubernetes не имеет своего собственного DNSсервера для разрешения имен. Поэтому используется плагин
**kube-dns** (это тоже Pod).
Его задачи:
- ходить в API Kubernetes’a и отслеживать Service-объекты
- заносить DNS-записи о Service’ах в собственную базу
- предоставлять DNS-сервис для разрешения имен в IP-адреса
(как внутренних, так и внешних)

Service с типом **NodePort** - похож на сервис типа
ClusterIP, только к нему прибавляется прослушивание
портов нод (всех нод) для доступа к сервисам снаружи.
При этом ClusterIP также назначается этому сервису для
доступа к нему изнутри кластера.
**kube-proxy** прослушивается либо заданный порт
(nodePort: 32092), либо порт из диапазона 30000-32670.
Дальше IPTables решает, на какой Pod попадет трафик.

Тип **LoadBalancer** позволяет нам использовать **внешний
облачный** балансировщик нагрузки как единую точку
входа в наши сервисы, а не полагаться на IPTables и не
открывать наружу весь кластер.

Для более удобного управления входящим
снаружи трафиком и решения недостатков
LoadBalancer можно использовать другой объект
Kubernetes - **Ingress**.

**Ingress** – это набор правил внутри кластера Kubernetes,
предназначенных для того, чтобы входящие подключения
могли достичь сервисов (Services)

Сами по себе Ingress’ы это просто правила. Для их
применения нужен **Ingress Controller**
[Содержание](#top)