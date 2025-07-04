---
title:  "Apache Maven Tomcat on Centos7"
excerpt: "Maven과 Tomcat으로 웹 개발 환경 구축"
author_profile: true
date: 2019-04-18 16:10:00 +0900
categories: [api-reference]
tags: [api-reference,Centos,tomcat,maven,jdk,jenkins,subversion]
---

jenkins로 개발 배포 Workload 관리
Tomcat과 Maven을 통한 웹 서버 개발 배포 및 Subversion으로 개발 버전 관리

###### SYSTEM Info
-------------
- CentOS 7
- openjdk-1.8.0
- Tomcat-9.0.17
- Subversion-1.11.0
- Maven-3.6.1
- jenkins-2.164.2

###### Environment Configuration
-------------

1. openjdk-1.8.0 install

```
# wget http://anduin.linuxfromscratch.org/BLFS/OpenJDK/OpenJDK-1.8.0.141/OpenJDK-1.8.0.141-x86_64-bin.tar.xz
-> openjdk 다운로드

# tar -xvf jtreg-4.2-b08-891.tar.gz
-> 압축 해제

# ln -s OpenJDK-1.8.0.141-x86_64-bin java
-> 링크 설정
```

2. Tomcat install

```
# wget http://mirror.navercorp.com/apache/tomcat/tomcat-9/v9.0.17/bin/apache-tomcat-9.0.17.tar.gz
-> tomcat 다운로드

# tar xvfz apache-tomcat-9.0.17.tar.gz
-> 압축 해제

# ln -s apache-tomcat-9.0.17 tomcat
-> 링크 설정

# vi .bash_profile
------------ ※ 아래와 같이 설정 ※ --------------------------------
export JAVA_HOME=/tomcat/java
export CATALINA_HOME=/tomcat/tomcat
export PATH=$PATH:.:$JAVA_HOME/bin:$CATALINA_HOME/bin
export CLASSPATH=$JAVA_HOME/jre/lib/ext:$JAVA_HOME/lib/tools.jar:$CATALINA_HOME/lib/jsp-api.jar:$CATALINA_HOME/lib/servlet-api.jar
-----------------------------------------------------------------
-> Tomcat 환경변수 설정

# . .bash_profile
-> 변경된 환경변수 반영

# $CATALINA_HOME/bin/startup.sh
-> Tomcat 서비스 시작

# firewall-cmd --permanent --zone=public --add-port=8080/tcp
-> 방화벽 default 8080 port 해제

# firewall-cmd --reload
-> 방화벽 설정 동기화
```
<img align="left" src="/assets/images/Tomcat.png">
###### < 테스트 페이지 >

3. Subversion install

```
# vi /etc/yum.repos.d/subversion.repo
------------ ※ 아래와 같이 설정 ※ --------------------------------
[subversion]
name=subversion
baseurl=http://opensource.wandisco.com/centos/$releasever/svn-1.11/RPMS/$basearch/
#baseurl=http://opensource.wandisco.com/centos/7/svn-1.11/RPMS/$basearch/
enabled=1
gpgcheck=0
-----------------------------------------------------------------
-> subversion repository 설정

# yum install subversion
-> subversion 설치

# svnadmin create --fs-type fsfs test_project
-> subversion 저장소 생성

# vi /etc/sysconfig/svnserve
------------ ※ 아래와 같이 설정 ※ --------------------------------
OPTIONS="--threads --root /svn"
-----------------------------------------------------------------

# vi /usr/lib/systemd/system/svnserve.service
------------ ※ 아래와 같이 설정 ※ --------------------------------
[Unit]
Description=Subversion protocol daemon
After=syslog.target network.target

[Service]
Type=forking
EnvironmentFile=/etc/sysconfig/svnserve
ExecStart=/usr/bin/svnserve --daemon --pid-file=/run/svnserve/svnserve.pid $OPTIONS

[Install]
WantedBy=multi-user.target
-----------------------------------------------------------------
-> subversion 서비스 설정 ( root : /svn 임의 지정 )

# systemctl daemon-reload
-> systemctl daemon 재시작

# vi /svn/test_project/conf/svnserve.conf
------------ ※ 아래와 같이 설정 ※ --------------------------------
anon-access = none
auth-access = write
password-db = passwd
authz-db = authz
-----------------------------------------------------------------
-> subversion 설정 변경 ( 비로그인 접속자는 권한 없음, 로그인하면 쓸 수 있음, passwd와 authz 파일을 사용함 )

# vi /svn/test_project/conf/passwd
------------ ※ 아래와 같이 설정 ※ --------------------------------
admin = admin2019
-----------------------------------------------------------------
-> subversion 계정 생성

# vi /svn/test_project/conf/authz
------------ ※ 아래와 같이 설정 ※ --------------------------------
[/]
admin = rw
-----------------------------------------------------------------
-> 계정 권한 추가

# systemctl start svnserve

# svnserve -d -r /svn
-> svn 서비스시작

---※ svn 서비스 중지 --------
# kill -9 pid
# systemctl stop svnserve
-----------------------------

# ps -ef | grep svnserve

# firewall-cmd --permanent --zone=public --add-port=3690/tcp
-> 방화벽 default 8080 port 해제

# firewall-cmd --reload
-> 방화벽 설정 동기화
```

2. MiniDLNA 서버 설정

```
# vi /etc/minidlna.conf 파일 편집(27번째 줄)
friendly_name=CentOS DLNA Server
-> 서버 이름 설정
db_dir=/var/cache/minidlna
-> DB 캐시 폴더 설정
log_dir=/var/log/minidlna
-> log 폴더 설정
```

3. firewall 방화벽 설정

```
firewall-cmd --permanent --add-port=1900/udp --zone=public
firewall-cmd --permanent --add-port=8200/tcp --zone=public
firewall-cmd --reload
-> firewall 재시작
```

4. minidlna 서버 서비스 기동 및 시작 프로그램 등록

```
systemctl start minidlna.service
systemctl enable minidlna.service
```