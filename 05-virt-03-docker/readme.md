
# Домашнее задание к занятию "5.3. Введение. Экосистема. Архитектура. Жизненный цикл Docker контейнера" - Иван Дайн


---

## Задача 1

Сценарий выполения задачи:

- создайте свой репозиторий на https://hub.docker.com;

https://hub.docker.com/u/ivandayn
- выберете любой образ, который содержит веб-сервер Nginx;

`выбрал официальный nginx:latest`
- создайте свой fork образа;

`docker pull nginx`
- реализуйте функциональность:
запуск веб-сервера в фоне с индекс-страницей, содержащей HTML-код ниже:
```
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I’m DevOps Engineer!</h1>
</body>
</html>
```
```bash
$ docker run -it --name ng-test -p 80:80 nginx bash
root@9eefbfdcb7b2:/# cd /usr/share/nginx/html/
root@9eefbfdcb7b2:/usr/share/nginx/html# echo "<html><head>Hey, Netology</head><body><h1>I&rsquo;m DevOps Engineer&#33;</h1></body></html>" > index.html
root@9eefbfdcb7b2:/usr/share/nginx/html# exit
$ docker commit -m "Modified index.html" -a "Ivan Dayn" 9eefbfdcb7b2 ivandayn/nginx:vid
sha256:6264b2e43d856d00906e5e60c2d354697b448d8d00f477d0f03cf9b4021ee8f7
$ docker login -u ivandayn
$ docker push ivandayn/nginx:vid
```
![image](https://user-images.githubusercontent.com/93118042/151934903-e4da60c1-f1d6-47c4-a9a9-11b9c368d6f2.png)

Опубликуйте созданный форк в своем репозитории и предоставьте ответ в виде ссылки.

https://hub.docker.com/repository/docker/ivandayn/nginx/general

## Задача 2

Посмотрите на сценарий ниже и ответьте на вопрос:
"Подходит ли в этом сценарии использование Docker контейнеров или лучше подойдет виртуальная машина, физическая машина? Может быть возможны разные варианты?"

Детально опишите и обоснуйте свой выбор.

--

Сценарий:

- Высоконагруженное монолитное java веб-приложение;

`Для реализации монолитного приложения в контейнере потребуется изменение кода,поэтому контейнер не подойдёт, а лучше использовать виртуализацию для повышения надёжности или HAG на физических серверах`
- Nodejs веб-приложение;

`Хорошо подойдут контейнеры, можно быстро развернуть, масштабировать`
- Мобильное приложение c версиями для Android и iOS;

`Ничего быстро не смог найи про виртуализацию iOS или контейнеры с ней, для нормальной разработки и тестирования лучше использовать физические устройства`
- Шина данных на базе Apache Kafka;

`Не работал с этим продуктом, загрузок докер контейнеров с репозитория очень много, значит практика использования в контейнерах есть)`
- Elasticsearch кластер для реализации логирования продуктивного веб-приложения - три ноды elasticsearch, два logstash и две ноды kibana;

`Вспоминается анекдот, где в конце "Папа, а ты с кем сейчас разговаривал?", но если бегло погуглить, то думаю что сам elasticsearch лучше на виртуалку, клаастеризация даст отказоустойчивость, для logstash и kibana можноиспользовать докер контейнер`
- Мониторинг-стек на базе Prometheus и Grafana;

`Хорошо подойдут контейнеры, если в них будет реализован функционал, а данные хранить в отдельных томах`
- MongoDB, как основное хранилище данных для java-приложения;

`Лучше виртуализацию, целый физический сервер это слишком, а как хранилище данных контейнеры не предназначены, для сохранности данных потребуется внешний относительно контейнера ресурс`
- Gitlab сервер для реализации  CI/CD процессов и приватный (закрытый) Docker Registry.

`Не знаю как реализуется CI/CD c GitLab, но так как это функционал над внешними данными, то напрашивается контейнеризация, т.к. именно для таких задач она и задумана. Из статьи https://habr.com/ru/post/320884/ я понял, что самый простой и быстрый способ создать приватный репозиторий докер контейнеров это опять же использовать докер`

## Задача 3

- Запустите первый контейнер из образа ***centos*** c любым тэгом в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера;
- Запустите второй контейнер из образа ***debian*** в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера;
```bash
vagrant@server1:~$ docker image ls
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
debian       stable    b8d808eca4b3   6 days ago     124MB
centos       centos8   5d0da3dc9764   4 months ago   231MB
vagrant@server1:~$ docker run --name cen -d -v /home/vagrant/data:/data 5d0da3dc9764 sleep 1000
f2f203eaadc80f93f157f8a9649f02aa68d1270bfda67ce4b7c793ceac2d88c2
vagrant@server1:~$ docker run --name deb -d -v /home/vagrant/data:/data b8d808eca4b3 sleep 1000
d194e1083df88fb0c3a64246105bdb9ebd5ab0ae47ee49e7000f740cedf5d4e0
vagrant@server1:~$ docker ps
CONTAINER ID   IMAGE          COMMAND        CREATED          STATUS          PORTS     NAMES
d194e1083df8   b8d808eca4b3   "sleep 1000"   13 seconds ago   Up 12 seconds             deb
f2f203eaadc8   5d0da3dc9764   "sleep 1000"   35 seconds ago   Up 34 seconds             cen
```
- Подключитесь к первому контейнеру с помощью ```docker exec``` и создайте текстовый файл любого содержания в ```/data```;
```bash
vagrant@server1:~$ docker exec cen sh -c "echo Hi > /data/fromCentOS"
```
- Добавьте еще один файл в папку ```/data``` на хостовой машине;
```bash
vagrant@server1:~$ echo Hi > /home/vagrant/data/fromHost
```
- Подключитесь во второй контейнер и отобразите листинг и содержание файлов в ```/data``` контейнера.
```bash
vagrant@server1:~$ docker exec -it deb bash
root@d194e1083df8:/# ls -al /data
total 16
drwxrwxr-x 2 1000 1000 4096 Feb  1 07:59 .
drwxr-xr-x 1 root root 4096 Feb  1 07:58 ..
-rw-r--r-- 1 root root    3 Feb  1 07:58 fromCentOS
-rw-rw-r-- 1 1000 1000    3 Feb  1 07:59 fromHost
root@d194e1083df8:/# cat data/*
Hi
Hi
```

## Задача 4 (*)

Воспроизвести практическую часть лекции самостоятельно.

Соберите Docker образ с Ansible, загрузите на Docker Hub и пришлите ссылку вместе с остальными ответами к задачам.

https://hub.docker.com/repository/docker/ivandayn/ansible

---
