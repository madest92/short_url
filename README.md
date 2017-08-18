# URL Shortener

Отказоустойчивый и легкомаштабируемый вебсервер коротких ссылок.

Написан на python3, в качестве системы управления конфигурации используется ansible(*Требование задания*)

Для хранения ключей используется БД Percona XtraDB Cluster. Для балансировки трафика - haproxy


## Пример ansible/hosts
```
[webservers]
host1 inner_ip=192.168.0.101
host2 inner_ip=192.168.0.102
host3 inner_ip=192.168.0.103

[proxys]
proxy1 inner_ip=192.168.0.201 external_ip=192.168.1.201
proxy2 inner_ip=192.168.0.202 external_ip=192.168.1.202

[dbs]
db1 inner_ip=192.168.0.11
db2 inner_ip=192.168.0.12
db3 inner_ip=192.168.0.13
```


## Как добавить новый сервер
1. Обновляем inventory hosts, добавив сервер в одну из групп(webservers/proxys/dbs)

2. *Если вебсервер*: запускаем плейбук **ansible-playbook playbook.yml -l HOST**, где *HOST* - имя нового сервера

*Если БД*: раскатываем роль **dbs**, чтобы обновить конфигурацию всех мастер-баз

3. Обновляем конфиги проксей, чтобы они увидели новый сервер **ansible-playbook playbook.yml -l proxys -t configuration**


## Удаление сервера из роли
1. Обновляем inventory hosts, убрав сервер из группы

2. Обновляем конфиги проксей **ansible-playbook playbook.yml -l proxys -t configuration**


## Обновление конфигурации сервиса
Достаточно запустить **ansible-playbook playbook.yml -l ROLE -t configuration**, где *ROLE* - имя роли


## Что хочется(можно) доделать:
1. Балансировка haproxy через keepalived. Локально потестировать не удалось, поднимать виртуаки лень

2. В текущей схеме идет двойной трифик через прокси(http/mysql). В идеале нужен отдельный haproxy для mysql

3. Добавить кластер редиса, который будет хранить топовые обращения
