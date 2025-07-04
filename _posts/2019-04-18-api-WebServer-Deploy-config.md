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
<small>< 테스트 페이지 ></small>

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

4. Maven install

```
​# wget http://www-us.apache.org/dist/maven/maven-3/3.6.1/binaries/apache-maven-3.6.1-bin.tar.gz
-> maven 다운로드

# tar xvfz apache-maven-3.6.1-bin.tar.gz
-> maven 압축 풀기

# vi .bash_profile
------------ ※ 아래와 같이 설정 ※ --------------------------------
export M2_HOME=/tomcat/apache-maven-3.6.1
export PATH=$M2_HOME/bin
-----------------------------------------------------------------
-> maven 설정

# . .bash_profile
-> 변경된 환경변수 반영

#  mvn --version
-> mvn 버전확인
```

5. Jenkins install & Deploy Environment Config

```
# wget http://mirrors.jenkins.io/war/2.173/jenkins.war
-> jenkins 다운로드

# mv jenkins.war /tomcat/tomcat/webapps/
-> jenkins 소스 배포

# yum install fontconfig
-> library 설치

# vi /tomcat/tomcat/bin/catalina.sh
------------ ※ 아래와 같이 설정 ※ --------------------------------
JAVA_OPTS="-Djava.awt.headless=true"
-----------------------------------------------------------------
-> tomcat 설정 추가 ( JFreeChart를 unix상에서 사용할 경우 )

# $CATALINA_HOME/bin/shutdown.sh
# $CATALINA_HOME/bin/startup.sh
-> Tomcat 재기동

# vi /root/.jenkins/secrets/initialAdminPassword
```
<img align="left" src="/assets/images/jenkins_first_password.png">
<small>-> 초기 패스워드 확인 및 입력</small>

<img align="left" src="/assets/images/jenkins_first_Proxyconfig.png">
<small>-> Conigure proxy ( proxy 구성하여 plug-in 설치 / plug-in 설치 스킵하고 진행 )</small>

<img align="left" src="/assets/images/jenkins_first_adminUser_Create.png">
<small>-> first admin 계정 설정</small>

<img align="left" src="/assets/images/jenkins_first_plugin_setting.png">
<small><strong>-> 플로그인 설정</strong></small>  
<small> 1. Jenkins 관리 탭 선택</small>  
<small> 2. 플로그인 관리 탭 선택</small>  
<small> 3. 고급 탭 선택</small>  
<small> 4. 업데이트 사이트 > 사이트경로 변경 ( http://updates.jenkins.io/update-center.json )</small>  
<small> 5. Submit 선택</small>  
<small> 6. 지금 확인 클릭</small>  