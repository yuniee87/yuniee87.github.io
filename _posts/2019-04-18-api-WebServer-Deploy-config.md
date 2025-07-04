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
< 테스트 페이지 >
<img align="left" src="/assets/images/Tomcat.png">


###### minidlna 설정
-------------

1. minidlna 디렉토리 설정

```
# vi /etc/minidlna.conf 파일 편집(18번째 줄) 
media_dir=V,/var/share/movie
-> 동영상 설정
media_dir=A,/var/share/music
-> 음악 설정
media_dir=P,/var/share/photo
-> 사진 설정
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