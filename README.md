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
- [HW.18 - ЛЛогирование и распределенная трассировка](#hw18)
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
