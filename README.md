# fred-13_microservices
fred-13 microservices repository

---------------------Homework18-----------------------

[![Build Status](https://travis-ci.com/Otus-DevOps-2018-09/fred-13_microservices.svg?branch=monitoring-1)](https://travis-ci.com/Otus-DevOps-2018-09/fred-13_microservices)

1) Создана ветка monitoring-1
2) Создал правило фаервола для Prometheus и Puma:
```
$ gcloud compute firewall-rules create prometheus-default --allow tcp:9090
$ gcloud compute firewall-rules create puma-default --allow tcp:9292
```
3) Создал Docker хост в GCE
```
docker-machine create --driver google \
    --google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
    --google-machine-type n1-standard-1 \
    --google-zone europe-west1-b \
    docker-host
```
4) Загрузил готовый образ и запустил prometheus (знакомство с веб интерфейсом):
```
docker run --rm -p 9090:9090 -d --name prometheus prom/prometheus:v2.1.0
```
5) Создал директорию monitoring/prometheus и в ней описал Dockerfile для prometheus:
```
FROM prom/prometheus:v2.1.0
ADD prometheus.yml /etc/prometheus/
```
6) Создал файл prometheus.yml с описанием сбора метрик с наших микросервисов
7) Пересоздан образ:
```
$ docker build -t $USER_NAME/prometheus .
```
8) Собрал образы при помощи скриптов docker_build.sh:
```
/src/ui      $ bash docker_build.sh
/src/post-py $ bash docker_build.sh
/src/comment $ bash docker_build.sh
```
9) Поднял сервисы и просмотрел мониторинг состояния микросервисов
10) Подключил экспортер node-exporter описанием в docker/docker-compose.yml файле как сервис
11) Запушил образы на DockerHub. [Ссылка на образы тут](https://hub.docker.com/u/fred13)

------------------------------------------------------

---------------------Homework17-----------------------

[![Build Status](https://travis-ci.com/Otus-DevOps-2018-09/fred-13_microservices.svg?branch=gitlab-ci-2)](https://travis-ci.com/Otus-DevOps-2018-09/fred-13_microservices)

1) Создана ветка gitlab-ci-2
2) Поднят проект с прошлого ДЗ
3) Создан новый проект example2 в GitLab
4) Включил runner для этого проекта
5) Создано DEV-окружение
6) Определил два новых этапа stage и production как мануал
7) Затегировал код для выкатки в staging и production
8) Определил динамическое окружение

------------------------------------------------------

---------------------Homework16-----------------------

[![Build Status](https://travis-ci.com/Otus-DevOps-2018-09/fred-13_microservices.svg?branch=gitlab-ci-1)](https://travis-ci.com/Otus-DevOps-2018-09/fred-13_microservices)

1) Создана ветка gitlab-ci-1
2) Создана ВМ в GCP по требуемым параметрам
3) Установлен docker и docker-compose на ВМ
4) Запущен Gitlab CI в docker с помощью готового образа
4) Настроена группа и проект в Gitlab
5) Определение CI/CD Pipeline с помощью описания процесса в .gitlab-ci.yml
6) Регистрация и запуск Runner
7) Добавление тестирования приложения reddit в pipeline

------------------------------------------------------

---------------------Homework15-----------------------

[![Build Status](https://api.travis-ci.com/Otus-DevOps-2018-09/fred-13_microservices.svg?branch=docker-4)](https://travis-ci.com/Otus-DevOps-2018-09/fred-13_microservices)

1) Создана ветка docker-4
2) Запуск контейнера с использованием none-драйвера из образа joffotron/docker-net-tools
3) Запуск контейнера в сетевом пространстве docker-хоста
4) Создание bridge-сети reddit в docker и запуск проекта в этой сети:
```
docker network create reddit --driver bridge
docker run -d --network=reddit mongo:latest
docker run -d --network=reddit fred13/post:1.0
docker run -d --network=reddit fred13/comment:1.0
docker run -d --network=reddit -p 9292:9292 ferd13/ui:1.0
```
5) Перезапуск проекта с новым сетевым alias (post_db, post, comment) для устранения ошибки связи контейнеров
6) Создание две docker-сети:
```
docker network create back_net --subnet=10.0.2.0/24
docker network create front_net --subnet=10.0.1.0/24
```
7) Действие из п.6 поломало общение фронта с бэком, поэтому подключим контейнеры ко второй сети:
```
docker network connect front_net post
docker network connect front_net comment
```
8) Просмотр iptables в docker-host
9) Установка docker-compose и запуск проекта с помощью этого инструмента:
```
export USERNAME=fred13
docker-compose up -d
docker-compose ps
```
10) Выполнение заданий для самостоятельной работы
11) Базвое имя проекта образуется по соотвествию директории из которой запускается docker-compose. Для того чтобы назвать по своему
необходимо в файл .env задать свой COMPOSE_PROJECT_NAME

------------------------------------------------------

---------------------Homework14-----------------------

1) Создана ветка docker-3
2) Скачинвание и распаковка архива microservices.zip ( microservices > src )
3) Создание докер файлов:

./post-py/Dockerfile
./comment/Dockerfile
./ui/Dockerfile

4) Скачивание последнего образа MongoDB и сборка образов с сервисами:

docker pull mongo:latest
docker build -t fred13/post:1.0 ./post-py
docker build -t fred13/comment:1.0 ./comment
docker build -t fred13/ui:1.0 ./ui

5) Создание изолированной сети и запуск контейнеров в ней:

docker network create reddit
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db mongo:latest
docker run -d --network=reddit --network-alias=post fred13/post:1.0
docker run -d --network=reddit --network-alias=comment fred13/comment:1.0
docker run -d --network=reddit -p 9292:9292 fred13/ui:1.0

6) Модификация содержимого ./ui/Dockerfile и пересборка контейнера ui, сравнение размеров:

REPOSITORY TAG     IMAGE ID      SIZE
fred13/ui  2.0   16a06056cc20    447MB
fred13/ui  1.0   8aa7c3bfc30e    780MB
   
7) Перезапуск приложения с новым контейнером ui (2.0)
8) В ходе выполнения пункта выше (7) потеряны данные, поэтому создадим Docker volume и подключим его к контейнеру mongo:latest:

docker volume create reddit_db
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db -v reddit_db:/data/db mongo:latest

После перезапуска контейнеров посты остаются на месте

------------------------------------------------------


---------------------Homework13-----------------------

1) Создана ветка docker-2
2) Создан новый проект docker в GCP
3) Настроена синхронизация консоли с проектом в GCP
4) Установлен инструмент docker-machine
5) Запуск контейнера в GCP:

  docker-machine create --driver google \
    --google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
    --google-machine-type n1-standard-1 \
    --google-zone europe-west1-b \
    docker-host

6) Проверка успешности запуска хоста "$ docker-machine ls"
7) Практика из лекции ( PID namespace, net namespace, user namespaces )
8) Сборка образа на базе Dockerfile с помощью файлов в корне mongod.conf, db_config, start.sh
9) Разрешаем входящий трафик:

  gcloud compute firewall-rules create reddit-app \
   --allow tcp:9292 \
   --target-tags=docker-machine \
   --description="Allow PUMA connections" \
   --direction=INGRESS

10) Регистрация на Docker Hub и загрузка своего первого образа:

  $ docker login
  $ docker tag reddit:latest fred13/otus-reddit:1.0

11) Проверка этого образа как в GCP так и локально

------------------------------------------------------


---------------------Homework12-----------------------

1) Создана ветка docker-1
2) Настроена интеграция с travis-ci
3) Создана директория docker-monolith
4) Установлен docker, docker-compose, docker-machine
5) Проработаны основные команды docker
6) Выполнено задание со *

-----------------------------------------------------
