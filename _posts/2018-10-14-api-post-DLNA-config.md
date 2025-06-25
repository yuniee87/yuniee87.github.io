---
title:  "DLNA Server ON Centos7"
author_profile: true
date: 2018-10-14 19:50:00 +0900
categories: [api-reference]
tags: [api-reference,Linux,Centos]
---

DLNA(Digital Living Network Alliance) 서버는 디지털 미디어(사진, 음악, 비디오 등)를 네트워크를 통해 다른 DLNA 지원 기기에 공유하고 스트리밍할 수 있도록 해주는 기술
--- 

> ###### SYSTEM Info
---
- CentOS 7
- DLNA latest

> ###### DLNA Server Install

1. DLNA server를 위한 minidlna 설치

````
# yum install epel-release
-> 필요 패키지 설치

# rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-5.el7.nux.noarch.rpm

# rpm --import http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro
-> minidlna 설치를 위한 yum repo 설정

# yum install minidlna
-> minidlna 설치
````

> ###### minidlna 설정
   
1. minidlna 디렉토리 설정
````
# vi /etc/minidlna.conf 파일 편집(18번째 줄) 

media_dir=V,/var/share/movie
-> 동영상 설정
   
media_dir=A,/var/share/music
-> 음악 설정
   
media_dir=P,/var/share/photo
-> 사진 설정
````
   
2. MiniDLNA 서버 설정
````
# vi /etc/minidlna.conf 파일 편집(27번째 줄)
   
friendly_name=CentOS DLNA Server
-> 서버 이름 설정
   
db_dir=/var/cache/minidlna
-> DB 캐시 폴더 설정
   
log_dir=/var/log/minidlna
-> log 폴더 설정
````

3. firewall 방화벽 설정
   
````
firewall-cmd --permanent --add-port=1900/udp --zone=public
firewall-cmd --permanent --add-port=8200/tcp --zone=public
firewall-cmd --reload
-> firewall 재시작
````

4. minidlna 서버 서비스 기동 및 시작 프로그램 등록
   
````
systemctl start minidlna.service
systemctl enable minidlna.service
````