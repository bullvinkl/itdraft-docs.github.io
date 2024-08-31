---
title: "Настройка отказоустойчивого кластера PostgreSQL в Linux"
date: "2023-08-14"
categories: 
  - Linux
  - PostgreSQL
tags: 
  - etcd
  - haproxy
  - keepalive
  - linux
  - patroni
  - postgresql
image:
  path: /commons/ai_images-min.png
  alt: "Защита от брутфорса админки Wordpress в Docker с помощью Fail2Ban"
---

> Отказоустойчивость PostgreSQL обеспечивается переключением ролей сервера с Реплики на Мастер СУБД и механизмом потоковой репликации, позволяющим передавать изменения с ведущего сервера на ведомые так, чтобы в каждый момент времени на ведомом сервере хранилась полная и консистентная копия базы данных.

Для настройки отказоустойчивости PostgreSQL будет использоваться следующее ПО:

- OS: Deiban 11 или 12
- Patroni
- Etcd
- Keepalived
- Pgbouncer
- Haproxy

Тестовый стенд развернут на системе виртуализации VirtualBox, ip-адрессация дефолтная

Кластер будет построен на 3-х нодах. Будет рассмотрена конфигурация 1 master и 1 replica сервер СУДБ PostgreSQL. Но эту конфигурацию без труда можно переделать в 1 master и 2 replicas

Таким образом, ПО на хостах будет следующим:
```sh
db1-1: etcd, keepalived, PostgreSQL, Partoni, PGbouncer, HAproxy
db1-2: etcd, keepalived, PostgreSQL, Partoni, PGbouncer, HAproxy
db1-3: etcd
```

## Подготовка хостов

Обновим ОС и установим (по желанию) базовый набор ПО
```sh
$ sudo apt update && sudo apt upgrade -y
$ sudo apt -y install nano curl bind9-utils dnsutils telnet wget net-tools traceroute git tcpdump rsync open-vm-tools mlocate htop tar zip unzip cloud-guest-utils gdisk
$ sudo apt autoremove -y
```

На всех 3-х нодах приводим файл hosts к виду:
```sh
$ sudo nano /etc/hosts
10.0.2.15 db1-1
10.0.2.16 db1-2
10.0.2.17 db1-3
```

Переименуем каждую ноду через `hostnamectl`
```sh
$ sudo hostnamectl set-hostname db1-1  # для каждой ноды свой hostname
```

## Установка Etcd (All nodes)

Скачиваем дистрибутив и распаковываем. На момент подготовки стенда, финальная версия etcd была: 3.5.5
```sh
$ cd /tmp
$ wget https://github.com/etcd-io/etcd/releases/download/v3.5.5/etcd-v3.5.5-linux-amd64.tar.gz
$ tar xzvf etcd-v3.5.5-linux-amd64.tar.gz
```

Перемещаем бинарники в каталог /usr/local/bin
```sh
$ sudo mv /tmp/etcd-v3.5.5-linux-amd64/etcd* /usr/local/bin/
```

Проверяем
```sh
$ etcd --version
etcd Version: 3.5.5
Git SHA: 19002cfc6
Go Version: go1.16.15
Go OS/Arch: linux/amd64

$ etcdctl version
etcdctl version: 3.5.5
API version: 3.5

$ etcdutl version
etcdutl version: 3.5.5
API version: 3.5
```

## Предварительная настройка etcd (All nodes)

Создаем системную группу и пользователя
```sh
$ sudo groupadd --system etcd
$ sudo useradd -s /sbin/nologin --system -g etcd etcd
```

Создаем директории, назначаем владельца, права доступа
```sh
$ sudo mkdir /opt/etcd
$ sudo mkdir /etc/etcd
$ sudo chown -R etcd:etcd /opt/etcd
$ sudo chmod -R 700 /opt/etcd/
```

Создаем конфиги для etcd (для каждой ноды разный)

db1-1:
```sh
$ sudo nano /etc/etcd/etcd.conf
ETCD_NAME="etcd1"
ETCD_LISTEN_CLIENT_URLS="http://10.0.2.15:2379,http://127.0.0.1:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://10.0.2.15:2379"
ETCD_LISTEN_PEER_URLS="http://10.0.2.15:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.0.2.15:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-postgres-cluster"
ETCD_INITIAL_CLUSTER="etcd1=http://10.0.2.15:2380,etcd2=http://10.0.2.16:2380,etcd3=http://10.0.2.17:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_DATA_DIR="/var/lib/etcd"
ETCD_ELECTION_TIMEOUT="10000"
ETCD_HEARTBEAT_INTERVAL="2000"
ETCD_INITIAL_ELECTION_TICK_ADVANCE="false"
ETCD_ENABLE_V2="true"
```

db1-2
```sh
$ sudo nano /etc/etcd/etcd.conf
ETCD_NAME="etcd2"
ETCD_LISTEN_CLIENT_URLS="http://10.0.2.16:2379,http://127.0.0.1:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://10.0.2.16:2379"
ETCD_LISTEN_PEER_URLS="http://10.0.2.16:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.0.2.16:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-postgres-cluster"
ETCD_INITIAL_CLUSTER="etcd1=http://10.0.2.15:2380,etcd2=http://10.0.2.16:2380,etcd3=http://10.0.2.17:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_DATA_DIR="/var/lib/etcd"
ETCD_ELECTION_TIMEOUT="10000"
ETCD_HEARTBEAT_INTERVAL="2000"
ETCD_INITIAL_ELECTION_TICK_ADVANCE="false"
ETCD_ENABLE_V2="true"
```

db1-3
```sh
$ sudo nano /etc/etcd/etcd.conf
ETCD_NAME="etcd3"
ETCD_LISTEN_CLIENT_URLS="http://10.0.2.17:2379,http://127.0.0.1:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://10.0.2.17:2379"
ETCD_LISTEN_PEER_URLS="http://10.0.2.17:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.0.2.17:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-postgres-cluster"
ETCD_INITIAL_CLUSTER="etcd1=http://10.0.2.15:2380,etcd2=http://10.0.2.16:2380,etcd3=http://10.0.2.17:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_DATA_DIR="/var/lib/etcd"
ETCD_ELECTION_TIMEOUT="10000"
ETCD_HEARTBEAT_INTERVAL="2000"
ETCD_INITIAL_ELECTION_TICK_ADVANCE="false"
ETCD_ENABLE_V2="true"
```

Создаем Systemd Unit на каждой из нод
```sh
$ sudo nano /etc/systemd/system/etcd.service
[Unit]
Description=Etcd Server
Documentation=https://github.com/etcd-io/etcd
After=network.target
After=network-online.target
Wants=network-online.target
  
[Service]
User=etcd
Type=notify
#WorkingDirectory=/var/lib/etcd/
WorkingDirectory=/opt/etcd/
EnvironmentFile=-/etc/etcd/etcd.conf
User=etcd
# set GOMAXPROCS to number of processors
ExecStart=/bin/bash -c "GOMAXPROCS=$(nproc) /usr/local/bin/etcd"
Restart=on-failure
LimitNOFILE=65536
IOSchedulingClass=realtime
IOSchedulingPriority=0
Nice=-20
 
[Install]
WantedBy=multi-user.target
```

Добавляем сервис в автозагрузку и запускаем
```sh
$ sudo systemctl daemon-reload
$ sudo systemctl enable etcd
$ sudo systemctl start etcd
```

## Управление Etcd

Смотрим ноды кластера
```sh
$ ETCDCTL_API=2 etcdctl member list
```

Проверяем кто лидер
```sh
$ etcdctl endpoint status --cluster -w table
```

Здоровье кластера
```sh
$ etcdctl endpoint health --cluster -w table
```

Другие команды проверки статуса:
```sh
$ ETCDCTL_API=2 etcdctl member list
$ ETCDCTL_API=3 etcdctl -w table --endpoints=db1-1:2379,db1-2:2379,db1-3:2379 endpoint status
$ ETCDCTL_API=3 etcdctl endpoint status --cluster -w table
$ etcdctl endpoint status --cluster
$ etcdctl endpoint health --cluster
```

## Установка Postgresql 15 из репозитория (db1-1, db1-2)

Добавляем репозиторий и устанавливаем PostgreSQL
```sh
$ sudo apt -y install gnupg2
$ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
$ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
$ sudo apt update
$ sudo apt -y install postgresql-15
$ sudo ln -s /usr/lib/postgresql/15/bin/* /usr/sbin/
```

Добавляем пользователя replicator
```sh
$ sudo -u postgres psql
=# create user replicator replication login encrypted password 'passwd';
```

Задаем пароль для пользователя postgres
```sh
=# \password postgres;
Enter new password for user "postgres": passwd
Enter it again: passwd
```

Подключаем расширения
```sh
=# CREATE EXTENSION pg_stat_statements;
=# LOAD 'auto_explain';
```

Создаем пользователя pgbouncer
```sh
=# create user pgbouncer password '39xYw2KcwV';
=# \q
```

Редактируем pg\_hba.conf
```sh
$ sudo nano /etc/postgresql/15/main/pg_hba.conf
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
host    all             all             10.0.2.0/24             md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     replicator      localhost               trust
host    replication     all             127.0.0.1/32            md5
host    replication     all             ::1/128                 md5
host    replication     replicator      10.0.2.0/24             md5
```

На второй и ноде удаляем (от пользователей root или postgres) содержимое каталога /var/lib/postgresql/15/main, т.к. этот каталог среплицируется после запуска Patroni
```sh
$ sudo su
# rm -rf /var/lib/postgresql/15/main/*
# exit
```

## Установка KeepAlived (db1-1, db1-2)

Установим ПО
```sh
$ sudo apt -y install keepalived
```

Правим конфиг

db1-1:
```sh
$ sudo nano /etc/keepalived/keepalived.conf
global_defs {
   router_id ocp_vrrp
   enable_script_security
   script_user root
}
 
vrrp_script haproxy_check {
   script "/usr/libexec/keepalived/haproxy_check.sh"
   interval 5 # check every 5 seconds
   weight 2 # add 2 points of prio if OK
}
 
vrrp_instance VI_1 {
   interface enp0s3
   virtual_router_id 11
   priority  101 # 101 on master, 100 on backup
   advert_int 10
   state  MASTER
   virtual_ipaddress {
       10.0.2.11
   }
   track_script {
       haproxy_check
   }
   authentication {
      auth_type PASS
      auth_pass ehr0wg1chww8
   }
}
```

db1-2:
```sh
$ sudo nano /etc/keepalived/keepalived.conf
global_defs {
   router_id ocp_vrrp
   enable_script_security
   script_user root
}
 
vrrp_script haproxy_check {
   script "/usr/libexec/keepalived/haproxy_check.sh"
   interval 5 # check every 5 seconds
   weight 2 # add 2 points of prio if OK
}
 
vrrp_instance VI_1 {
   interface enp0s3
   virtual_router_id 11
   priority  100 # 101 on master, 100 on backup
   advert_int 10
   state  BACKUP
   virtual_ipaddress {
       10.0.2.11
   }
   track_script {
       haproxy_check
   }
   authentication {
      auth_type PASS
      auth_pass ehr0wg1chww8
   }
}
```

В данном конфиге ip 10.0.2.11 - кластерный плавающий ip

Создаем каталог и скрипт проверки HAproxy (до установки и настройки HAproxy не будет отрабатывать)
```sh
$ sudo mkdir -p /usr/libexec/keepalived/
$ sudo nano /usr/libexec/keepalived/haproxy_check.sh
#!/bin/bash
/bin/kill -0 `cat /var/run/haproxy/haproxy.pid`
```

Назначаем права
```sh
$ sudo chmod 700 /usr/libexec/keepalived/haproxy_check.sh
$ sudo chmod +x /usr/libexec/keepalived/haproxy_check.sh
```

Запускаем сервис, добавляем в автозагрузку, проверяем
```sh
$ sudo systemctl start keepalived
$ sudo systemctl enable keepalived
$ sudo systemctl status keepalived
```

## Установка Patroni (db1-1, db1-2)

Устанавливаем ПО
```sh
$ sudo apt -y install python3 python3-pip python3-dev python3-psycopg2 libpq-dev
$ sudo pip3 install --upgrade pip
$ sudo pip3 install psycopg2
$ sudo pip3 install psycopg2-binary
$ sudo pip3 install wheel
$ sudo pip3 install patroni
$ sudo pip3 install python-etcd
```

Создаем каталог /etc/patroni/
```sh
$ sudo mkdir /etc/patroni/
```

Создаем конфиг patroni.yml
```sh
$ sudo nano /etc/patroni/patroni.yml
---

scope: postgres-cluster # одинаковое значение на всех узлах
name: db1-1 # разное значение на всех узлах
namespace: /service/ # одинаковое значение на всех узлах

restapi:
  listen: 10.0.2.15:8008 # разное значение на всех узлах
  connect_address: 10.0.2.15:8008 # разное значение на всех узлах
  authentication:
    username: patroni
    password: 'passwd'

etcd:
  hosts: 10.0.2.15:2379,10.0.2.16:2379,10.0.2.17:2379 # список всех узлов, на которых установлен etcd

bootstrap:
  method: initdb
  dcs:
    ttl: 60
    loop_wait: 10
    retry_timeout: 27
    maximum_lag_on_failover: 2048576
    master_start_timeout: 300
    synchronous_mode: true
    synchronous_mode_strict: false
    synchronous_node_count: 1
    # standby_cluster:
      # host: 127.0.0.1
      # port: 1111
      # primary_slot_name: patroni
    postgresql:
      use_pg_rewind: false
      use_slots: true
      parameters:
        max_connections: 800
        superuser_reserved_connections: 5
        max_locks_per_transaction: 64
        max_prepared_transactions: 0
        huge_pages: try
        shared_buffers: 512MB
        work_mem: 128MB
        maintenance_work_mem: 256MB
        effective_cache_size: 4GB
        checkpoint_timeout: 15min
        checkpoint_completion_target: 0.9
        min_wal_size: 2GB
        max_wal_size: 4GB
        wal_buffers: 32MB
        default_statistics_target: 1000
        seq_page_cost: 1
        random_page_cost: 4
        effective_io_concurrency: 2
        synchronous_commit: on
        autovacuum: on
        autovacuum_max_workers: 5
        autovacuum_vacuum_scale_factor: 0.01
        autovacuum_analyze_scale_factor: 0.02
        autovacuum_vacuum_cost_limit: 200
        autovacuum_vacuum_cost_delay: 20
        autovacuum_naptime: 1s
        max_files_per_process: 4096
        archive_mode: on
        archive_timeout: 1800s
        archive_command: cd .
        wal_level: replica
        wal_keep_segments: 130
        max_wal_senders: 10
        max_replication_slots: 10
        hot_standby: on
        hot_standby_feedback: True
        wal_log_hints: on
        shared_preload_libraries: pg_stat_statements,auto_explain
        pg_stat_statements.max: 10000
        pg_stat_statements.track: all
        pg_stat_statements.save: off
        auto_explain.log_min_duration: 10s
        auto_explain.log_analyze: true
        auto_explain.log_buffers: true
        auto_explain.log_timing: false
        auto_explain.log_triggers: true
        auto_explain.log_verbose: true
        auto_explain.log_nested_statements: true
        track_io_timing: on
        log_lock_waits: on
        log_temp_files: 0
        track_activities: on
        track_counts: on
        track_functions: all
        log_checkpoints: on
        logging_collector: on
        log_statement: mod
        log_truncate_on_rotation: on
        log_rotation_age: 1d
        log_rotation_size: 0
        log_line_prefix: '%m [%p] %q%u@%d '
        log_filename: 'postgresql-%a.log'
        log_directory: /var/log/postgresql

  initdb:  # List options to be passed on to initdb
    - encoding: UTF8
    - locale: en_US.UTF-8
    - data-checksums

  pg_hba:  # должен содержать адреса ВСЕХ машин, используемых в кластере
    - host all all 0.0.0.0/0 md5
    - host replication replicator 127.0.0.1/32 md5
    - host replication replicator 10.0.2.0/24 md5

postgresql:
  listen: 10.0.2.15,127.0.0.1:5432 # разное значение на всех узлах
  connect_address: 10.0.2.15:5432 # разное значение на всех узлах
  use_unix_socket: true
  data_dir: /var/lib/postgresql/14/main
  bin_dir: /usr/lib/postgresql/14/bin
  config_dir: /etc/postgresql/14/main
  pgpass: /var/lib/postgresql/.pgpass_patroni
  authentication:
    replication:
      username: replicator
      password: passwd
    superuser:
      username: postgres
      password: passwd
  parameters:
    unix_socket_directories: /var/run/postgresql
    stats_temp_directory: /var/lib/pgsql_stats_tmp

  remove_data_directory_on_rewind_failure: false
  remove_data_directory_on_diverged_timelines: false

#  callbacks:
#    on_start:
#    on_stop:
#    on_restart:
#    on_reload:
#    on_role_change:

  create_replica_methods:
    - basebackup
  basebackup:
    max-rate: '100M'
    checkpoint: 'fast'

watchdog:
  mode: off  # Allowed values: off, automatic, required
  device: /dev/watchdog
  safety_margin: 5

tags:
  nofailover: false
  noloadbalance: false
  clonefrom: false
  nosync: false

  # specify a node to replicate from (cascading replication)
#  replicatefrom: (node name)
```

Назначаем права
```sh
$ sudo chown postgres:postgres -R /etc/patroni
$ sudo chmod 700 /etc/patroni
```

Создаем каталог, назначаем владельца
```sh
$ sudo mkdir /var/lib/pgsql_stats_tmp
$ sudo chown postgres:postgres /var/lib/pgsql_stats_tmp
```

Тестируем старт (сразу на двух нодах)
```sh
$ sudo -u postgres patroni /etc/patroni/patroni.yml
```

Должен создастся файл .pgpass\_patroni, назначаем владельца, права
```sh
$ sudo chown postgres:postgres /var/lib/postgresql/.pgpass_patroni
$ sudo chmod 0600 /var/lib/postgresql/.pgpass_patroni
```

Создаем Sustemd Unit (на каждой ноде)
```sh
$ sudo nano /etc/systemd/system/patroni.service
[Unit]
Description=High availability PostgreSQL Cluster
After=syslog.target network.target

[Service]
Type=simple
User=postgres
Group=postgres

# Read in configuration file if it exists, otherwise proceed
EnvironmentFile=-/etc/patroni_env.conf

# Start the patroni process
ExecStart=/usr/local/bin/patroni /etc/patroni/patroni.yml

# Send HUP to reload from patroni.yml
ExecReload=/bin/kill -s HUP $MAINPID

# only kill the patroni process, not it's children, so it will gracefully stop postgres
KillMode=process

# Give a reasonable amount of time for the server to start up/shut down
TimeoutSec=60

# Do not restart the service if it crashes, we want to manually inspect database on failure
Restart=no

[Install]
WantedBy=multi-user.target
```

Добавляем сервис а автозагрузку, стартуем, смотрим статус
```sh
$ sudo systemctl daemon-reload
$ sudo systemctl start patroni
$ sudo systemctl status patroni
$ sudo systemctl enable patroni
```

Смотрим состояние кластера
```sh
$ sudo patronictl -c /etc/patroni/patroni.yml list
```

> Ошибка:  
> \== CRITICAL: system ID mismatch, node db1-2 belongs to a different cluster
> 
> Решение: Выполнить команды
> 
> ```
> $ sudo systemctl stop patroni
> $ ETCDCTL_API=2 etcdctl rm /service/postgres-cluster/initialize
> $ sudo systemctl restart patroni
> $ sudo systemctl status patroni
> ```

Создадим файла с настройками по умолчанию, что позволит не указывать настройки подключения для patronictl
```sh
$ sudo mkdir -p ~/.config/patroni/
$ sudo nano ~/.config/patroni/patronictl.yaml
dcs_api:
 etcd://localhost:2379
namespace: /service/
scope: postgres-cluster
authentication:
 username: patroni
 password: passwd
```

Проверяем
```sh
$ patronictl show-config
$ patronictl list
```

Если на других нодах PostgreSQL был запущен раньше, удалите каталог данных, чтобы заработала реплика
```sh
$ sudo systemctl stop patroni
$ sudo rm -rf /var/lib/postgresql/14/main/*
$ sudo systemctl start patroni
```

> Ошибка:
> 
> \==patroni: fatal the database system is starting up  
> Ошибка в логах PostgreSQL: fatal the database system is starting up
> 
> Решение:
> 
> ```
> $ sudo systemctl stop patroni
> $ sudo rm -rf /var/lib/postgresql/15/main/*
> $ sudo systemctl start patroni
> ```

## Установка PGbouncer (db1-1, db1-2)

Устанавливаем PGBouncer, правим конфиг
```sh
$ sudo apt -y install pgbouncer
$ sudo mv /etc/pgbouncer/pgbouncer.ini /etc/pgbouncer/pgbouncer.ini.origin
$ sudo nano /etc/pgbouncer/pgbouncer.ini
[databases]
postgres = host=127.0.0.1 port=5432 dbname=postgres
* = host=127.0.0.1 port=5432

[pgbouncer]
logfile = /var/log/postgresql/pgbouncer.log
pidfile = /var/run/postgresql/pgbouncer.pid
listen_addr = *
listen_port = 6432
unix_socket_dir = /var/run/postgresql
auth_type = md5
#auth_type = trust
auth_file = /etc/pgbouncer/userlist.txt
auth_user = postgres
auth_query = SELECT usename, passwd FROM pg_shadow WHERE usename=$1
#admin_users = pgbouncer, postgres
admin_users = postgres
ignore_startup_parameters = extra_float_digits,geqo,search_path

pool_mode = session
#pool_mode = transaction
server_reset_query = DISCARD ALL
max_client_conn = 10000
#default_pool_size = 20
reserve_pool_size = 1
reserve_pool_timeout = 1
max_db_connections = 1000
#max_client_conn = 900
default_pool_size = 500
pkt_buf = 8192
listen_backlog = 4096
log_connections = 1
log_disconnections = 1

# Documentation https://pgbouncer.github.io/config.html
```

Создадим файл userlist.txt
```sh
$ sudo nano /etc/pgbouncer/userlist.txt
"postgres" "passwd"
"pgbouncer" "passwd"
```

Перезапускаем PGbouncer на нодах
```sh
$ sudo systemctl restart pgbouncer
```

Файл /etc/pgbouncer/userlist.txt содержит имена пользователей и пароли, с которыми PGbouncer подключается к базе. Пароли не обязательно хранить в открытым виде, можно так:
```sh
...
"pgbouncer" "md576a2173be6393254e72ffa4d6df1030a"
```

Хэш 76a2173be6393254e72ffa4d6df1030a посчитан как MD5 от пароля
```sh
$ echo -n 'passwd' | md5sum
```

Тестируем
```sh
$ sudo su - postgres
$ psql -p 6432 -h 127.0.0.1 -U postgres postgres
```

## Установка и настройка HAproxy (db1-1, db1-2)

Установим ПО
```sh
$ sudo apt -y install haproxy
```

Правим конфиг

db1-1:
```sh
$ sudo nano /etc/haproxy/haproxy.cfg
global
    maxconn 100000
    log /dev/log    local0
    log /dev/log    local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    mode               tcp
    log                global
    retries            2
    timeout queue      5s
    timeout connect    5s
    timeout client     60m
    timeout server     60m
    timeout check      15s

listen stats
    mode http
    bind 10.0.2.15:7000
    stats enable
    stats uri /

listen postgres_master
    bind *:5000
    maxconn 10000
    option tcplog
    option httpchk OPTIONS /master
    http-check expect status 200
    default-server inter 3s fastinter 1s fall 3 rise 4 on-marked-down shutdown-sessions
    server db1-1 10.0.2.15:6432 check port 8008
    server db1-2 10.0.2.16:6432 check port 8008

listen postgres_replicas
    bind *:5001
    maxconn 10000
    option tcplog
    option httpchk OPTIONS /replica
    balance roundrobin
    http-check expect status 200
    default-server inter 3s fastinter 1s fall 3 rise 2 on-marked-down shutdown-sessions
    server db1-1 10.0.2.15:6432 check port 8008
    server db1-2 10.0.2.16:6432 check port 8008

listen postgres_replicas_sync
    bind *:5002
    maxconn 10000
    option tcplog
    option httpchk OPTIONS /sync
    balance roundrobin
    http-check expect status 200
    default-server inter 3s fastinter 1s fall 3 rise 2 on-marked-down shutdown-sessions
    server db1-1 10.0.2.15:6432 check port 8008
    server db1-2 10.0.2.16:6432 check port 8008

listen postgres_replicas_async
    bind *:5003
    maxconn 10000
    option tcplog
    option httpchk OPTIONS /async
    balance roundrobin
    http-check expect status 200
    default-server inter 3s fastinter 1s fall 3 rise 2 on-marked-down shutdown-sessions
    server db1-1 10.0.2.15:6432 check port 8008
    server db1-2 10.0.2.16:6432 check port 8008
```

db1-2:
```sh
$ sudo nano /etc/haproxy/haproxy.cfg
global
    maxconn 100000
    log /dev/log    local0
    log /dev/log    local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    mode               tcp
    log                global
    retries            2
    timeout queue      5s
    timeout connect    5s
    timeout client     60m
    timeout server     60m
    timeout check      15s

listen stats
    mode http
    bind 10.0.2.16:7000
    stats enable
    stats uri /

listen postgres_master
    bind *:5000
    maxconn 10000
    option tcplog
    option httpchk OPTIONS /master
    http-check expect status 200
    default-server inter 3s fastinter 1s fall 3 rise 4 on-marked-down shutdown-sessions
    server db1-1 10.0.2.15:6432 check port 8008
    server db1-2 10.0.2.16:6432 check port 8008

listen postgres_replicas
    bind *:5001
    maxconn 10000
    option tcplog
    option httpchk OPTIONS /replica
    balance roundrobin
    http-check expect status 200
    default-server inter 3s fastinter 1s fall 3 rise 2 on-marked-down shutdown-sessions
    server db1-1 10.0.2.15:6432 check port 8008
    server db1-2 10.0.2.16:6432 check port 8008

listen postgres_replicas_sync
    bind *:5002
    maxconn 10000
    option tcplog
    option httpchk OPTIONS /sync
    balance roundrobin
    http-check expect status 200
    default-server inter 3s fastinter 1s fall 3 rise 2 on-marked-down shutdown-sessions
    server db1-1 10.0.2.15:6432 check port 8008
    server db1-2 10.0.2.16:6432 check port 8008

listen postgres_replicas_async
    bind *:5003
    maxconn 10000
    option tcplog
    option httpchk OPTIONS /async
    balance roundrobin
    http-check expect status 200
    default-server inter 3s fastinter 1s fall 3 rise 2 on-marked-down shutdown-sessions
    server db1-1 10.0.2.15:6432 check port 8008
    server db1-2 10.0.2.16:6432 check port 8008
```

Проверка конфига
```sh
$ sudo haproxy -f /etc/haproxy/haproxy.cfg -c
```

Перезапускаем HAproxy, смотрим статус
```sh
$ sudo systemctl restart haproxy
$ sudo systemctl status haproxy
```

Настройка отказоустойчивого кластера завершена

## Типовые кейсы Patroni

Если master упал, сделать реплику лидером
```sh
$ sudo patronictl -c /etc/patroni/patroni.yml list
$ sudo patronictl -c /etc/patroni/patroni.yml failover
Candidate ['db1-1'] []: db1-1
Current cluster topology
+--------+-----------+---------+---------+----+-----------+
| Member | Host      | Role    | State   | TL | Lag in MB |
+ Cluster: postgres-cluster (7154050747629718791) --------+
| db1-1  | 10.0.2.15 | Replica | running |  9 |        16 |
+--------+-----------+---------+---------+----+-----------+
Are you sure you want to failover cluster postgres-cluster? [y/N]: y
2022-10-14 15:46:19.53515 Successfully failed over to "db1-1"
+--------+-----------+--------+---------+----+-----------+
| Member | Host      | Role   | State   | TL | Lag in MB |
+ Cluster: postgres-cluster (7154050747629718791) -------+
| db1-1  | 10.0.2.15 | Leader | running |  9 |           |
+--------+-----------+--------+---------+----+-----------+
```

Сменить лидера
```sh
$ sudo patronictl -c /etc/patroni/patroni.yml list
$ sudo patronictl -c /etc/patroni/patroni.yml switchover
Master [db1-1]: db1-1
Candidate ['db1-2'] []: db1-2
When should the switchover take place (e.g. 2022-10-15T02:36 )  [now]: now
Current cluster topology
+--------+-----------+--------------+---------+----+-----------+
| Member | Host      | Role         | State   | TL | Lag in MB |
+ Cluster: postgres-cluster (7154050747629718791) -+-----------+
| db1-1  | 10.0.2.15 | Leader       | running | 12 |           |
| db1-2  | 10.0.2.16 | Sync Standby | running | 12 |         0 |
+--------+-----------+--------------+---------+----+-----------+
Are you sure you want to switchover cluster postgres-cluster, demoting current master db1-1? [y/N]: y
2022-10-15 01:36:46.92463 Successfully switched over to "db1-2"
+--------+-----------+---------+---------+----+-----------+
| Member | Host      | Role    | State   | TL | Lag in MB |
+ Cluster: postgres-cluster (7154050747629718791) --------+
| db1-1  | 10.0.2.15 | Replica | stopped |    |   unknown |
| db1-2  | 10.0.2.16 | Leader  | running | 12 |           |
+--------+-----------+---------+---------+----+-----------+
cfgadmin@db1-1:~$ sudo patronictl -c /etc/patroni/patroni.yml list
+--------+-----------+--------------+---------+----+-----------+
| Member | Host      | Role         | State   | TL | Lag in MB |
+ Cluster: postgres-cluster (7154050747629718791) -+-----------+
| db1-1  | 10.0.2.15 | Sync Standby | running | 13 |         0 |
| db1-2  | 10.0.2.16 | Leader       | running | 13 |           |
+--------+-----------+--------------+---------+----+-----------+
```

Смотрим историю переключений
```sh
$ sudo patronictl -c /etc/patroni/patroni.yml history
```

Отключаем авто переключение Replica-Master
```sh
sudo patronictl -c /etc/patroni/patroni.yml pause
```

Реинициализировать кластер
```sh
sudo patronictl -c /etc/patroni/patroni.yml reinit postgres-cluster
```

postgres-cluster - задается в настройках patroni (scope: ...)

Реинициализировать ноду
```sh
$ sudo patronictl -c /etc/patroni/patroni.yml reinit postgres-cluster db1-2
```

Редактируем конфиг patroni
```sh
$ sudo patronictl -c /etc/patroni/patroni.yml edit-config
```

Перезапустить кластер (обычно после редактирования конфига patroni)
```sh
$ sudo patronictl -c /etc/patroni/patroni.yml restart postgres-cluster
#$ sudo patronictl -c /etc/patroni/patroni.yml reload postgres-cluster
```

Статусы сервисов
```sh
$ sudo patronictl -c /etc/patroni/patroni.yml list

$ etcdctl endpoint status --cluster
$ etcdctl endpoint health --cluster

$ sudo systemctl status haproxy
$ sudo systemctl status patroni
$ sudo systemctl status etcd
$ sudo systemctl status keepalived
```