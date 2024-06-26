# Домашнее задание к занятию 5. «Elasticsearch» 

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

## Решение 1

```dockerfile
FROM centos:7

LABEL maintainer="reocoker1985@yndex.ru"
LABEL org.label-schema.docker.cmd="docker run -p 9200:9200 -p 9300:9300 reocoker/centoselastic"

ENV VER elasticsearch-7.17.17

RUN <<EOT
yum -y update 
yum install -y wget perl-Digest-SHA java-1.8.0-openjdk.x86_64
yum -y clean all 
rm -rf /var/cache
EOT

RUN <<EOT
wget https://artifacts.elastic.co/downloads/elasticsearch/${VER}-linux-x86_64.tar.gz
wget https://artifacts.elastic.co/downloads/elasticsearch/${VER}-linux-x86_64.tar.gz.sha512
shasum -a 512 -c ${VER}-linux-x86_64.tar.gz.sha512
tar -zxf ${VER}-linux-x86_64.tar.gz
rm ${VER}-linux-x86_64*
EOT

RUN <<EOT
adduser elasticsearch
chown -R elasticsearch:elasticsearch /${VER}
chown -R elasticsearch:elasticsearch /var/lib
EOT

USER elasticsearch

WORKDIR /${VER}

RUN cat <<EOT >> ./config/elasticsearch.yml
cluster.name: netology_cluster
node.name: netology_test 
path.data: /var/lib
network.host: 0.0.0.0
discovery.type: single-node
EOT

EXPOSE 9200
EXPOSE 9300

CMD ["./bin/elasticsearch"]

```
Cсылкa на образ в репозитории dockerhub :  https://hub.docker.com/r/reocoker/centoselastic

![1.png](./img/1.png)

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

## Решение 2

![2.png](./img/2.png)

В системе из одного сервера  Elasticsearch хранит на нём все “primary shards”, но создавать “replica shards” такой системе будет негде.
Поэтому статус в приведённом примере является жёлтым из-за ненулевого значения “unassigned_shards”, которое примерно равно “active_shards”.

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

## Решение 3

![3.png](./img/3.png)
![4.png](./img/4.png)
![5.png](./img/5.png)
