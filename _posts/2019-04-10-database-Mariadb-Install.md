---
title:  "Mariadb Server ON Centos7"
author_profile: true
date: 2019-04-10 20:58:00 +0900
categories: [database]
tags: [database,mariadb,Centos]
---

###### SYSTEM Info
-------------
- CentOS 7
- MariaDB latest

###### MariaDB Server Install
-------------

1. Install MariaDB Server of YUM

```
# vi /etc/yum.repos.d/MariaDB.repo
------------ ※ 아래와 같이 설정 ※ --------------------------------
# MariaDB 10.3 CentOS repository list - created 2019-04-04 08:20 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = mariadb
baseurl = http://yum.mariadb.org/10.3/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
------------------------------------------------------------------
-> yum repository 설정
# yum install mariadb-server*
-> mariadb 설치
# systemctl start mariadb
-> 서비스 시작
```

2. Mariadb Data Directory Modify

```
# systemctl stop mariadb
-> 서비스 종료
# mv /var/lib/mysql <변경될경로>/mysql
-> 경로 변경
# vi /etc/my.cnf
------------ ※ 아래와 같이 변경 ※ --------------------------------
[client-server]
socket=/data/mariadb/mysql.sock
[mysqld]
datadir=/data/mariadb
------------------------------------------------------------------
-> 경로 설정 변경
# systemctl start mysql
-> 서비스 시작
# mysql -uroot -p
MariaDB [(none)]> select @@datadir
-> data 경로 확인
```

3. Mariadb Lowercase Parameter Modify

```
$ vi /etc/my.cnf
------------ ※ 아래와 같이 변경 ※ --------------------------------
[mysqld]
lower_case_table_names = 1
------------------------------------------------------------------
-> 대소문자 구분 0 / 대소문자 구분안함 1
```

4. Mariadb firewall Disabled

```
# firewall-cmd --permanent --zone= public --add-port= 3306/tcp
-> 방화벽 default 3306 port 해제
# firewall-cmd --reload
-> 방화벽 설정 동기화
```

5. UTF-8 Character Set Change

```
$ vi /etc/my.cnf
------------ ※ 아래와 같이 변경 ※ --------------------------------
[mysqld]
init_connect ="SET collation_connection = utf8_general_ci"
init_connect ="SET NAMES utf8"
character-set-server = utf8
collation-server = utf8_general_ci
[client]
default-character-set = utf8
[mysqldump]
default-character-set = utf8
[mysql]
default-character-set = utf8
------------------------------------------------------------------
-> 캐릭터셋 변경 설정
```
