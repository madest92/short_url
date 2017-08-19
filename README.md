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

## Дефолтные переменные
### haproxy
```
proxy_stats_login: kudago          - логин для статистики haproxy
proxy_stats_password: adminadmin   - пароль для статистики haproxy
proxy_check_time: 1s               - частота проверки доступности сервиса
proxy_count_fall: 1                - количество ошибок, после которых сервер считается недоступным
proxy_count_rise: 2                - количество проверек, прежде чем сервер считается доступным
```
### mysql
```
mysql_root_password: r00tpassw0rd  - пароль для пользователя root
mysql_datadir: /var/lib/mysql      - путь к директории mysql
mysql_sst_user: sstshort           - логин для авторизации кластера
mysql_sst_password: shortshort     - пароль для авторизации кластера
mysql_memory_ratio: 0.8            - максимальное количество памяти доступная mysql(1 - 100%)
```
### webserver
```
short_script_user: webshort        - пользователь из под которого работает сервис
short_script_name: shorturl        - имя init.d скрипта
short_script_path: /usr/local/sbin/webserver-shorturl.py  - путь к скрипту
short_server_ip: "{{ inner_ip }}"  - ip адрес на котором слушает вебсервер
short_debug: True                  - включаем дебаг
short_hash_len: 6                  - длина короткой ссылки(при 6 выдает 56800235583 уникальных ссылок)
short_hash_alphabet: "Es8hFJD7oLR2QjpNdlBm3qcZtWUzaM19SubOIv4XHix5KfV0PwrCynge6TGYkA"  - перемешанный алфавит
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
