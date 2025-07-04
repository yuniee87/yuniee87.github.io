---
title:  "Mariadb Replication Server on Docker"
excerpt: "Mariadb Replication Server Docker 환경에 구성하기"
author_profile: true
date: 2019-06-18 14:18:00 +0900
categories: [database]
tags: [database,mariadb,docker,replication]
---

###### SYSTEM Info
-------------
- Windows 7
- Docker
- MariaDB latest
- Centos 7

###### Mariadb Configration on Docker
-------------

###### 1. Docker Install

```
url : https://www.docker.com/
docker 다운로드 및 설치

CMD(명령프롬프트) 실행

> docker version
-> Docker 버전 확인
```

###### 2. Mariadb image pull to Docker 

```
> docker search mariadb

> docker pull mariadb:latest

> docker images
-> Docker Images 확인
```

###### 3. Mariadb Container Configration & Setting

```
--- Master 생성 ---
> docker run -d -p 3301:3301 -e MYSQL_ROOT_PASSWORD=test -e MYSQL_ROOT_USER=root --name mariadb1 centos/mariadb-102-centos7

--- Slave1 생성 ---
> docker run -d -p 3302:3302 -e MYSQL_ROOT_PASSWORD=test -e MYSQL_ROOT_USER=root --name mariadb2 centos/mariadb-102-centos7
```

( Master 설정 )
```
> docker exec -it --user=root mariadb1 bash

# vi /etc/mysql/my.cnf
------------ ※ 아래와 같이 설정 ※ --------------------------------
[client]
socket = /var/lib/mysql/mysql.sock
port = 3301

[mysqld]
socket = /var/lib/mysql/mysql.sock
port = 3301
innodb_buffer_pool_size = 512M
event_scheduler = ON
performance_schema = ON
default_storage_engine='InnoDB'
max_allowed_packet = 16M

server-id = 11
character-set-server = utf8
bind-address = 0.0.0.0
log_bin = /var/lib/mysql/log/mariadb1-bin
log-error = /var/lib/mysql/log/mariadb1_error.log
expire_logs_days = 2 # binlog 삭제 주기 설정
slow_query_log_file = /var/lib/mysql/log/mysql-slow.log
long_query_time = 5
slow-query-log = 1
sync_binlog = 0
slave_compressed_protocol = 1

##### gtid config #####
gtid_strict_mode = 1
gtid_domain_id = 1
binlog_format = mixed
-----------------------------------------------------------------

> docker restart mariadb1
```

( Slave1 설정 )
```
> docker exec -it --user=root mariadb2 bash

# vi /etc/mysql/my.cnf
------------ ※ 아래와 같이 설정 ※ --------------------------------
[client]
socket = /var/lib/mysql/mysql.sock
port = 3302

[mysqld]
socket = /var/lib/mysql/mysql.sock
port = 3302
innodb_buffer_pool_size = 512M
event_scheduler = ON
performance_schema = ON
default_storage_engine='InnoDB'
max_allowed_packet = 16M
server-id = 21
character-set-server = utf8
bind-address = 0.0.0.0
log_bin = /var/lib/mysql/mariadb2-bin
log_error = /var/log/mariadb2_error.log
expire_logs_days = 2 # binlog 삭제 주기 설정
sync_binlog = 0
slave_compressed_protocol = 1

##### Replication config #####
m1.replicate-do-db=test
m1.replicate-ignore-db=information_schema
m1.replicate-ignore-db=mysql
m1.replicate-ignore-db=performance_schema
m1.replicate-ignore-db=sys

##### gtid config #####
gtid_strict_mode = 1
gtid_domain_id = 2
binlog_format = mixed

> docker restart mariadb2
```

###### 4. Mariadb replication Configration

( Master에서 작업 )
```
# mysql -uroot

MariaDB [(none)]> show global variables like '%gtid%';
-> gtid_binlog_pos 확인 ( 예: 1-11-6 )

MariaDB [(none)]> create user 'replica'@'%' identified by 'replica';
-> replication user 생성

MariaDB [(none)]> grant replication slave on *.* to 'replica'@'%';

MariaDB [(none)]> create database test;
```

( Slave1에서 작업 )
```
# mysql -uroot

MariaDB [(none)]> set @@default_master_connection='m1';

MariaDB [(none)]> set global gtid_slave_pos='1-11-6';
-> Master gtid_binlog_pos 설정

MariaDB [(none)]> change master 'm1' to master_host='172.17.0.2', master_port=3301, master_user='gtid_rep', master_password='gtid_rep', master_use_gtid=slave_pos;
-> Replication 설정

MariaDB [(none)]> set global m1.replicate_do_db=test;

MariaDB [(none)]> start slave 'm1';
-> Replication 시작

MariaDB [(none)]> show slave 'm1' status\G;
-> Replication 확인
```

<span style="font-size:0.7em; color: red">※ 10.2 이전 버전에서는 replicate_do_db 옵션을 사용할 경우 아래와 같은 제약사항이 발생됨.</span>   
<span style="font-size:0.7em; color: red"> - create {DB Name}.{Table Name} 등 {DB Name}.xxxx 문장의 statement duplicate가 이루어지지 않음.</span>   

###### 5. Mariadb MSR(multi source replication) Configration

```
--- Slave2 추가 생성 ---
> docker run -d -p 3303:3303 -e MYSQL_ROOT_PASSWORD=test -e MYSQL_ROOT_USER=root --name mariadb3 centos/mariadb-102-centos7
```

( Slave2 설정 )
```
> docker exec -it --user=root mariadb3 bash

# vi /etc/mysql/my.cnf
------------ ※ 아래와 같이 설정 ※ --------------------------------
[client]
socket = /var/lib/mysql/mysql.sock
port = 3303

[mysqld]
socket = /var/lib/mysql/mysql.sock
port = 3303
innodb_buffer_pool_size = 512M
event_scheduler = ON
performance_schema = ON
default_storage_engine='InnoDB'
max_allowed_packet = 16M
server-id = 31
character-set-server = utf8
bind-address = 0.0.0.0
log_bin = /var/lib/mysql/mariadb3-bin
log_error = /var/log/mariadb3_error.log
expire_logs_days = 2 # binlog 삭제 주기 설정
sync_binlog = 0
slave_compressed_protocol = 1

##### Replication config #####
m1.replicate-do-db=test
m2.replicate-do-db=test1
m1.replicate-ignore-db=information_schema
m1.replicate-ignore-db=mysql
m1.replicate-ignore-db=performance_schema
m1.replicate-ignore-db=sys
m2.replicate-ignore-db=information_schema
m2.replicate-ignore-db=mysql
m2.replicate-ignore-db=performance_schema
m2.replicate-ignore-db=sys

##### gtid config #####
gtid_strict_mode = 1
gtid_domain_id = 3
binlog_format = mixed

> docker restart mariadb3
```

( Slave1에서 작업 )
```
# mysql -uroot

MariaDB [(none)]> show global variables like '%gtid%';
-> gtid_binlog_pos 확인 ( 예: 2-21-3 )

MariaDB [(none)]> create user 'replica'@'%' identified by 'replica';
-> replication user 생성

MariaDB [(none)]> grant replication slave on *.* to 'replica'@'%';

MariaDB [(none)]> create database test1;
```

( Slave2에서 작업 )
```
# mysql -uroot

MariaDB [(none)]> set @@default_master_connection='m1';

MariaDB [(none)]> set @@default_master_connection='m2';

MariaDB [(none)]> set global gtid_slave_pos='1-11-6, 2-21-3';
-> Master gtid_binlog_pos 설정

MariaDB [(none)]> change master 'm1' to master_host='172.17.0.2', master_port=3301, master_user='gtid_rep', master_password='gtid_rep', master_use_gtid=slave_pos;
-> m1 Replication 설정

MariaDB [(none)]> change master 'm2' to master_host='172.17.0.3', master_port=3302, master_user='gtid_rep', master_password='gtid_rep', master_use_gtid=slave_pos;
-> m2 Replication 설정

MariaDB [(none)]> set global m1.replicate_do_db=test;

MariaDB [(none)]> set global m1.replicate_do_db=test1;

MariaDB [(none)]> start all slaves;
-> Replication 시작

MariaDB [(none)]> show all slaves status\G;
-> Replication 확인
```