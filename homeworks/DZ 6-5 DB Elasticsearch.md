# Домашнее задание к занятию 5. «Elasticsearch»

***
## Задача 1

В этом задании вы потренируетесь в:

- установке Elasticsearch,
- первоначальном конфигурировании Elasticsearch,
- запуске Elasticsearch в Docker.

Используя Docker-образ [centos:7](https://hub.docker.com/_/centos) как базовый и 
[документацию по установке и запуску Elastcisearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html):

- составьте Dockerfile-манифест для Elasticsearch,
- соберите Docker-образ и сделайте `push` в ваш docker.io-репозиторий,
- запустите контейнер из получившегося образа и выполните запрос пути `/` c хост-машины.

Требования к `elasticsearch.yml`:

- данные `path` должны сохраняться в `/var/lib`,
- имя ноды должно быть `netology_test`.

В ответе приведите:

- текст Dockerfile-манифеста,
- ссылку на образ в репозитории dockerhub,
- ответ `Elasticsearch` на запрос пути `/` в json-виде.

Подсказки:

- возможно, вам понадобится установка пакета perl-Digest-SHA для корректной работы пакета shasum,
- при сетевых проблемах внимательно изучите кластерные и сетевые настройки в elasticsearch.yml,
- при некоторых проблемах вам поможет Docker-директива ulimit,
- Elasticsearch в логах обычно описывает проблему и пути её решения.

Далее мы будем работать с этим экземпляром Elasticsearch.

### Решение:

```shell
dmitriy@dmitriy-lin:~/netology/docker/hw6-5$ cat Dockerfile 
FROM centos:7
RUN cd /opt && \
    groupadd elasticsearch && \
    useradd -c "elasticsearch" -g elasticsearch elasticsearch &&\
    yum update -y && yum -y install wget nano mc perl-Digest-SHA && \
    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.7.1-linux-x86_64.tar.gz && \
    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.7.1-linux-x86_64.tar.gz.sha512 && \
    shasum -a 512 -c elasticsearch-8.7.1-linux-x86_64.tar.gz.sha512 && \
    tar -xzf elasticsearch-8.7.1-linux-x86_64.tar.gz && \
        rm elasticsearch-8.7.1-linux-x86_64.tar.gz elasticsearch-8.7.1-linux-x86_64.tar.gz.sha512 && \
        mkdir /var/lib/data && chmod -R 777 /var/lib/data && \
        chown -R elasticsearch:elasticsearch /opt/elasticsearch-8.7.1 && \
        yum -y remove wget perl-Digest-SHA && \
        yum clean all
USER elasticsearch
WORKDIR /opt/elasticsearch-8.7.1/
COPY elasticsearch.yml  config/
EXPOSE 9200 9300
ENTRYPOINT ["bin/elasticsearch"]

dmitriy@dmitriy-lin:~/netology/docker/hw6-5$ cat elasticsearch.yml 
node:
  name: netology_test
path:
  data: /var/lib/data
xpack.ml.enabled: false
```

Собираем, запускаем образ и пушим (проблему с доступом к artifacts.elastic.co - решил впн)

```shell
dmitriy@dmitriy-lin:~/netology/docker/hw6-5$ sudo docker build -t dmitriymyskov/elasticsearch:1.0 .

dmitriy@dmitriy-lin:~/netology/docker/hw6-5$ sudo docker images
[sudo] пароль для dmitriy: 
REPOSITORY                    TAG       IMAGE ID       CREATED         SIZE
dmitriymyskov/elasticsearch   1.0       24bdfd0202ef   3 hours ago     1.65GB

dmitriy@dmitriy-lin:~/netology/docker/hw6-5$ sudo docker ps
CONTAINER ID   IMAGE                             COMMAND               CREATED          STATUS          PORTS                                                                                  NAMES
bd61594c1f25   dmitriymyskov/elasticsearch:1.0   "bin/elasticsearch"   37 seconds ago   Up 36 seconds   0.0.0.0:9200->9200/tcp, :::9200->9200/tcp, 0.0.0.0:9300->9300/tcp, :::9300->9300/tcp   elasticsearch

dmitriy@dmitriy-lin:~/netology/docker/hw6-5$ curl --insecure -u elastic https://localhost:9200
Enter host password for user 'elastic':
{
  "name" : "netology_test",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "hKUzybzvTnu_S2mxivVYlw",
  "version" : {
    "number" : "8.7.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "f229ed3f893a515d590d0f39b05f68913e2d9b53",
    "build_date" : "2023-04-27T04:33:42.127815583Z",
    "build_snapshot" : false,
    "lucene_version" : "9.5.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}

dmitriy@dmitriy-lin:~/netology/docker/hw6-5$ sudo docker push dmitriymyskov/elasticsearch:1.0
The push refers to repository [docker.io/dmitriymyskov/elasticsearch]
19f66a12db51: Pushed 
d621c8d53ab7: Pushed 
174f56854903: Mounted from library/centos 
1.0: digest: sha256:5fe69fc743efbb3981214a5ce0dc3818455969eedd421681d8974c956ad49996 size: 949
```

Ссылка на образ в docker hub: https://hub.docker.com/r/dmitriymyskov/elasticsearch/tags

***

## Задача 2

В этом задании вы научитесь:

- создавать и удалять индексы,
- изучать состояние кластера,
- обосновывать причину деградации доступности данных.

Ознакомьтесь с [документацией](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html) 
и добавьте в `Elasticsearch` 3 индекса в соответствии с таблицей:

| Имя | Количество реплик | Количество шард |
|-----|-------------------|-----------------|
| ind-1| 0 | 1 |
| ind-2 | 1 | 2 |
| ind-3 | 2 | 4 |

Получите список индексов и их статусов, используя API, и **приведите в ответе** на задание.

Получите состояние кластера `Elasticsearch`, используя API.

Как вы думаете, почему часть индексов и кластер находятся в состоянии yellow?

Удалите все индексы.

**Важно**

При проектировании кластера Elasticsearch нужно корректно рассчитывать количество реплик и шард,
иначе возможна потеря данных индексов, вплоть до полной, при деградации системы.

### Решение:

```shell
dmitriy@dmitriy-lin:~/netology/docker/hw6-5$ curl -X PUT --insecure -u elastic "https://localhost:9200/ind-1?pretty" -H 'Content-Type: application/json' -d'
> {
>    "settings": {
>      "index": {
>        "number_of_shards": 1,
>        "number_of_replicas": 0
>      }
>    }
> }
> '
Enter host password for user 'elastic':
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "ind-1"
}
dmitriy@dmitriy-lin:~/netology/docker/hw6-5$ curl -X PUT --insecure -u elastic "https://localhost:9200/ind-2?pretty" -H 'Content-Type: application/json' -d'
> {
>    "settings": {
>      "index": {
>        "number_of_shards": 2,
>        "number_of_replicas": 1
>      }
>    }
> }
> '
Enter host password for user 'elastic':
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "ind-2"
}
dmitriy@dmitriy-lin:~/netology/docker/hw6-5$ curl -X PUT --insecure -u elastic "https://localhost:9200/ind-3?pretty" -H 'Content-Type: application/json' -d'
> 
> {
>    "settings": {
>      "index": {
>        "number_of_shards": 4,
>        "number_of_replicas": 2
>      }
>    }
> }
> '
Enter host password for user 'elastic':
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "ind-3"
}

dmitriy@dmitriy-lin:~/netology/docker/hw6-5$ curl -X GET --insecure -u elastic https://localhost:9200/_cat/indices?v=true
Enter host password for user 'elastic':
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   ind-1 YAOcQLi2RLSlYCWQxuxJuQ   1   0          0            0       225b           225b
yellow open   ind-3 tthOreoaR4SLdNqibvqqgQ   4   2          0            0       900b           900b
yellow open   ind-2 sr0RjyVETj2jDAmBfkedMA   2   1          0            0       450b           450b

dmitriy@dmitriy-lin:~/netology/docker/hw6-5$ curl -X GET --insecure -u elastic "https://localhost:9200/_cluster/health?pretty"
Enter host password for user 'elastic':
{
  "cluster_name" : "elasticsearch",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 8,
  "active_shards" : 8,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 10,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 44.44444444444444
}

dmitriy@dmitriy-lin:~/netology/docker/hw6-5$ curl -X DELETE --insecure -u elastic "https://localhost:9200/ind-1?pretty"
Enter host password for user 'elastic':
{
  "acknowledged" : true
}

dmitriy@dmitriy-lin:~/netology/docker/hw6-5$ curl -X DELETE --insecure -u elastic "https://localhost:9200/ind-2?pretty"
Enter host password for user 'elastic':
{
  "acknowledged" : true
}

dmitriy@dmitriy-lin:~/netology/docker/hw6-5$ curl -X DELETE --insecure -u elastic "https://localhost:9200/ind-3?pretty"
Enter host password for user 'elastic':
{
  "acknowledged" : true
}

dmitriy@dmitriy-lin:~/netology/docker/hw6-5$ curl -X GET --insecure -u elastic https://localhost:9200/_cat/indices?v=true
Enter host password for user 'elastic':
health status index uuid pri rep docs.count docs.deleted store.size pri.store.size
```

- Как вы думаете, почему часть индексов и кластер находится в состоянии yellow?
  Думаю потому что кластера нету, имеем только одну ноду которая является мастером, соответсвенно в соответсвии с настройкой нет реплик у двух желтых.

***
## Задача 3

В этом задании вы научитесь:

- создавать бэкапы данных,
- восстанавливать индексы из бэкапов.

Создайте директорию `{путь до корневой директории с Elasticsearch в образе}/snapshots`.

Используя API, [зарегистрируйте](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-register-repository.html#snapshots-register-repository) 
эту директорию как `snapshot repository` c именем `netology_backup`.

**Приведите в ответе** запрос API и результат вызова API для создания репозитория.

Создайте индекс `test` с 0 реплик и 1 шардом и **приведите в ответе** список индексов.

[Создайте `snapshot`](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html) 
состояния кластера `Elasticsearch`.

**Приведите в ответе** список файлов в директории со `snapshot`.

Удалите индекс `test` и создайте индекс `test-2`. **Приведите в ответе** список индексов.

[Восстановите](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-restore-snapshot.html) состояние
кластера `Elasticsearch` из `snapshot`, созданного ранее. 

**Приведите в ответе** запрос к API восстановления и итоговый список индексов.

Подсказки:

- возможно, вам понадобится доработать `elasticsearch.yml` в части директивы `path.repo` и перезапустить `Elasticsearch`.

### Решение:
```shell
dmitriy@dmitriy-lin:~/netology/docker/hw6-5$ sudo docker exec -it elasticsearch bash
[elasticsearch@4b1c757156b6 elasticsearch-8.7.1]$ cat config/elasticsearch.yml
node:
  name: netology_test
path:
  data: /var/lib/data
  repo: /opt/elasticsearch-8.7.1/data/snapshot
xpack.ml.enabled: false

dmitriy@dmitriy-lin:~/netology/docker/hw6-5$ curl -X PUT --insecure -u elastic "https://localhost:9200/_snapshot/netology_backup?pretty" -H 'Content-Type: application/json' -d'
{
   "type": "fs",
   "settings": {
     "location": "/opt/elasticsearch-8.7.1/data/snapshot"
   }
 }
 '
Enter host password for user 'elastic':
{
  "acknowledged" : true
}
dmitriy@dmitriy-lin:~/netology/docker/hw6-5$ curl -X PUT --insecure -u elastic "https://localhost:9200/test?pretty" -H 'Content-Type: application/json' -d'
> {
>    "settings": {
>      "index": {
>        "number_of_shards": 1,
>        "number_of_replicas": 0
>      }
>    }
>  }
>  '
Enter host password for user 'elastic':
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test"
}
dmitriy@dmitriy-lin:~/netology/docker/hw6-5$ curl -X GET --insecure -u elastic "https://localhost:9200/_cat/indices/*?v=true&s=index&pretty"
Enter host password for user 'elastic':
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test  kmnsdYSfQky-0obZBcwhCw   1   0          0            0       225b           225b

dmitriy@dmitriy-lin:~/netology/docker/hw6-5/data/snapshot$ curl -X PUT --insecure -u elastic "https://localhost:9200/_snapshot/netology_backup/my_snapshot?pretty"
Enter host password for user 'elastic':
{
  "accepted" : true
}

dmitriy@dmitriy-lin:~/netology/docker/hw6-5/data/snapshot$ ls -la
итого 44
drwxrwxrwx 3 dmitriy dmitriy  4096 мая 12 11:08 .
drwxrwxr-x 3 dmitriy dmitriy  4096 мая 12 09:21 ..
-rw-r--r-- 1 denis   denis     839 мая 12 11:08 index-0
-rw-r--r-- 1 denis   denis       8 мая 12 11:08 index.latest
drwxr-xr-x 4 denis   denis    4096 мая 12 11:08 indices
-rw-r--r-- 1 denis   denis   16737 мая 12 11:08 meta-8-qnEB9KSdaUeUOM-c9yog.dat
-rw-r--r-- 1 denis   denis     347 мая 12 11:08 snap-8-qnEB9KSdaUeUOM-c9yog.dat

dmitriy@dmitriy-lin:~/netology/docker/hw6-5/data/snapshot$ curl -X DELETE --insecure -u elastic "https://localhost:9200/test?pretty"
Enter host password for user 'elastic':
{
  "acknowledged" : true
}
dmitriy@dmitriy-lin:~/netology/docker/hw6-5/data/snapshot$ curl -X PUT --insecure -u elastic "https://localhost:9200/test-2?pretty" -H 'Content-Type: application/json' -d'
> {
>    "settings": {
>      "index": {
>        "number_of_shards": 1,
>        "number_of_replicas": 0
>      }
>    }
>  }
>  '
Enter host password for user 'elastic':
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test-2"
}
dmitriy@dmitriy-lin:~/netology/docker/hw6-5/data/snapshot$ curl -X GET --insecure -u elastic "https://localhost:9200/_cat/indices/*?v=true&s=index&pretty"
Enter host password for user 'elastic':
health status index  uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test-2 TyVZG8SCQkCmRMhHJshwjQ   1   0          0            0       225b           225b
dmitriy@dmitriy-lin:~/netology/docker/hw6-5/data/snapshot$ curl -X POST --insecure -u elastic "https://localhost:9200/_snapshot/netology_backup/my_snapshot/_restore?pretty"
Enter host password for user 'elastic':
{
  "accepted" : true
}
dmitriy@dmitriy-lin:~/netology/docker/hw6-5/data/snapshot$ curl -X GET --insecure -u elastic "https://localhost:9200/_cat/indices/*?v=true&s=index&pretty"
Enter host password for user 'elastic':
health status index  uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test   z1eQ19VFTN6vT8xFEVhyVw   1   0          0            0       225b           225b
green  open   test-2 TyVZG8SCQkCmRMhHJshwjQ   1   0          0            0       225b           225b

```