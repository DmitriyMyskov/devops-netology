# Домашнее задание к занятию 3. «Введение. Экосистема. Архитектура. Жизненный цикл Docker-контейнера»

***
## Задача 1

Сценарий выполнения задачи:

- создайте свой репозиторий на https://hub.docker.com;
- выберите любой образ, который содержит веб-сервер Nginx;
- создайте свой fork образа;
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

Опубликуйте созданный fork в своём репозитории и предоставьте ответ в виде ссылки на https://hub.docker.com/username_repo.

###

Решение:

```shell
dmitriy@dmitriy-lin:~/netology/docker$ cat Dockerfile
FROM nginx
COPY ./index.html /usr/share/nginx/html
```

```shell
dmitriy@dmitriy-lin:~/netology/docker$ sudo docker build -t dmitriymyskov/netology:1.0 .
Sending build context to Docker daemon  3.072kB
Step 1/2 : FROM nginx
latest: Pulling from library/nginx
f1f26f570256: Pull complete 
84181e80d10e: Pull complete 
1ff0f94a8007: Pull complete 
d776269cad10: Pull complete 
e9427fcfa864: Pull complete 
d4ceccbfc269: Pull complete 
Digest: sha256:617661ae7846a63de069a85333bb4778a822f853df67fe46a688b3f0e4d9cb87
Status: Downloaded newer image for nginx:latest
 ---> ac232364af84
Step 2/2 : COPY ./index.html /usr/share/nginx/html
 ---> 264c0a2c8bd3
Successfully built 264c0a2c8bd3
Successfully tagged dmitriymyskov/netology:1.0

dmitriy@dmitriy-lin:~/netology/docker$ sudo docker images
REPOSITORY               TAG       IMAGE ID       CREATED              SIZE
dmitriymyskov/netology   1.0       264c0a2c8bd3   About a minute ago   142MB
nginx                    latest    ac232364af84   18 hours ago         142MB

dmitriy@dmitriy-lin:~/netology/docker$ sudo docker run --name netology -d -p 8080:80 -p 8443:443 dmitriymyskov/netology:1.0
6ab24322cd634ee73ff9e97c90fb85b35573e71670f5eda7c7f1c16c4746340c

dmitriy@dmitriy-lin:~/netology/docker$ curl http://localhost:8080
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I’m DevOps Engineer!</h1>
</body>
</html>

dmitriy@dmitriy-lin:~/netology/docker$ sudo docker push dmitriymyskov/netology:1.0
The push refers to repository [docker.io/dmitriymyskov/netology]
2c11648470af: Pushed 
a1bd4a5c5a79: Mounted from library/nginx 
597a12cbab02: Mounted from library/nginx 
8820623d95b7: Mounted from library/nginx 
338a545766ba: Mounted from library/nginx 
e65242c66bbe: Mounted from library/nginx 
3af14c9a24c9: Mounted from library/nginx 
1.0: digest: sha256:433eb7cec6c15c7411334f6f1b650cb765ad9ac14e898a2a4eb41445d2a145e1 size: 1777
```

```shell
https://hub.docker.com/r/dmitriymyskov/netology
```

***
## Задача 2

Посмотрите на сценарий ниже и ответьте на вопрос:
«Подходит ли в этом сценарии использование Docker-контейнеров или лучше подойдёт виртуальная машина, физическая машина? Может быть, возможны разные варианты?»

Детально опишите и обоснуйте свой выбор.

--

Сценарий:

- высоконагруженное монолитное Java веб-приложение;
- Nodejs веб-приложение;
- мобильное приложение c версиями для Android и iOS;
- шина данных на базе Apache Kafka;
- Elasticsearch-кластер для реализации логирования продуктивного веб-приложения — три ноды elasticsearch, два logstash и две ноды kibana;
- мониторинг-стек на базе Prometheus и Grafana;
- MongoDB как основное хранилище данных для Java-приложения;
- Gitlab-сервер для реализации CI/CD-процессов и приватный (закрытый) Docker Registry.

###

Решение:

- Высоконагруженное монолитное java веб-приложение - я бы использовал kvm, опять же java использует свою виртуальную машину и вполне вероятно сможет работать в непривелигированном lxc 
(не проверял, если да то мне кажется lxc вариант предпочтительнее).

- Nodejs веб-приложение - контейнеризация, docker или lxc - веб приложения хороший вариант для контейнеризации и микросервисной архитектуры.

- Мобильное приложение c версиями для Android и iOS - скорее всего именно docker отлично подойдет для тестирования под различные версии мобильных ОС (опять же не сталкивался с разработкой мобильных 
приложений возможно в lxc тоже не плохо будут себя чувствоать)

- Шина данных на базе Apache Kafka - я думаю что опять же docker (или lxc), причины все теже, быстро разворачивается, хорошая отказоустойчивость.

- Elasticsearch кластер для реализации логирования продуктивного веб-приложения - три ноды elasticsearch, два logstash и две ноды kibana - Думаю опять выбрал бы контейнеры.

- Мониторинг-стек на базе Prometheus и Grafana - docker или lxc :) (в контейнер добавил бы общее хранилище и красота)

- MongoDB, как основное хранилище данных для java-приложения - Скорее всего я бы использовал LXC, возможно KVM, docker тоже по идее имеет место быть, но в данном случае начал бы с LXC.

- Gitlab сервер для реализации CI/CD процессов и приватный (закрытый) Docker Registry - docker/lxc, т.к. оптимально осуществлять: конфигурирование, тестирование, сборки.

***
## Задача 3

- Запустите первый контейнер из образа ***centos*** c любым тегом в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера.
- Запустите второй контейнер из образа ***debian*** в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера.
- Подключитесь к первому контейнеру с помощью ```docker exec``` и создайте текстовый файл любого содержания в ```/data```.
- Добавьте ещё один файл в папку ```/data``` на хостовой машине.
- Подключитесь во второй контейнер и отобразите листинг и содержание файлов в ```/data``` контейнера.

###

Решение:

```shell
dmitriy@dmitriy-lin:~/netology/docker$ mkdir data

dmitriy@dmitriy-lin:~/netology/docker$ sudo docker run -dit -v /home/dmitriy/netology/docker/data:/data --name centos centos
Unable to find image 'centos:latest' locally
latest: Pulling from library/centos
a1d0c7532777: Pull complete 
Digest: sha256:a27fd8080b517143cbbbab9dfb7c8571c40d67d534bbdee55bd6c473f432b177
Status: Downloaded newer image for centos:latest
bc61c91d74f72e305eae6a00d2adb8b026ed3884c238274dd7ef1b4447f2c8ec

dmitriy@dmitriy-lin:~/netology/docker$ sudo docker run -dit -v /home/dmitriy/netology/docker/data:/data --name debian debian
Unable to find image 'debian:latest' locally
latest: Pulling from library/debian
3e440a704568: Pull complete 
Digest: sha256:7b991788987ad860810df60927e1adbaf8e156520177bd4db82409f81dd3b721
Status: Downloaded newer image for debian:latest
8d6a3a127e3da7f8d7ba6f7076f96cd7bbf32367b0520abb5663ef38083ede77

dmitriy@dmitriy-lin:~/netology/docker$ sudo docker ps
CONTAINER ID   IMAGE                        COMMAND                  CREATED          STATUS          PORTS                                                                            NAMES
8d6a3a127e3d   debian                       "bash"                   11 seconds ago   Up 9 seconds                                                                                     debian
bc61c91d74f7   centos                       "/bin/bash"              59 seconds ago   Up 57 seconds 

dmitriy@dmitriy-lin:~/netology/docker$ sudo docker exec -it centos bash
[root@bc61c91d74f7 /]# echo "my first file" > /data/myfile1.txt
[root@bc61c91d74f7 /]# exit
exit

dmitriy@dmitriy-lin:~/netology/docker$ ls -l data/
итого 4
-rw-r--r-- 1 root root 14 мар 24 17:15 myfile1.txt

dmitriy@dmitriy-lin:~/netology/docker$ echo "my second file" > data/myfile2.txt

dmitriy@dmitriy-lin:~/netology/docker$ ls -l data/
итого 8
-rw-r--r-- 1 root    root    14 мар 24 17:15 myfile1.txt
-rw-rw-r-- 1 dmitriy dmitriy 15 мар 24 17:16 myfile2.txt

dmitriy@dmitriy-lin:~/netology/docker$ sudo docker exec -it debian bash
root@8d6a3a127e3d:/# ls -la ./data/
total 16
drwxrwxr-x 2 1001 1001 4096 Mar 24 10:16 .
drwxr-xr-x 1 root root 4096 Mar 24 10:13 ..
-rw-r--r-- 1 root root   14 Mar 24 10:15 myfile1.txt
-rw-rw-r-- 1 1001 1001   15 Mar 24 10:16 myfile2.txt
root@8d6a3a127e3d:/# cat myfile1.txt
cat: myfile1.txt: No such file or directory
root@8d6a3a127e3d:/# cat ./data/myfile1.txt
my first file
root@8d6a3a127e3d:/# cat ./data/myfile2.txt
my second file
root@8d6a3a127e3d:/# exit
exit
```