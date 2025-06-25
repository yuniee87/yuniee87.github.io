---
title:  "Centos7 samba server 구축"
author_profile: true
date: 2018-10-14 18:48:00 +0900
categories: [api-reference]
tags: [api-reference,Linux,Centos]
---

SAMBA란, 마이크로소프트와 인텔에서 개발한 SMB(Server Message Block) 네트워크 프로토콜을 이용해 윈도우와 유닉스 계열의 운영체제나 다른 시스템 간의 자원을 공유할 수 있도록 만든 프로그램

> ###### SYSTEM Info
- Samba 4.10
- CentOS 7

> ###### Samba Install

1. Install samba server through the yum 

```
# yum install samba
```

2. Firewall exception to samba

```
# firewall-cmd --permanent --zone=public --add-service=samba
# firewall-cmd --reload
```

3. Create samba user

```
# adduser <user name>
# smbpasswd -a <user name>
..
```

4. Selinux exception to samba directory

```
# chcon -t samba_share_t <directory path>
# setsebool -P samba_export_all_rw on
```

5. samba config

```
# vi /etc/samba/smb.conf

[data]
        comment = data
        path = /data1
        valid users = admin
        guest ok = yes
        writable = yes
        browsable = yes
comment -> alias

path -> 공유할 디렉토리 설정

valid users -> samba 공유계정 설정

guest ok -> 외부사용자 설정

writable -> 쓰기권한 설정

browsable -> 자동검색 기능 설정
```

6. systemd service config to samba

```
# systemctl restart smb

# systemctl enable smb

# systemctl restart nmb

# systemctl enable nmb
```