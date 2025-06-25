---
title:  "Centos7 samba server 구축"
author_profile: true
date: 2018-10-14 18:48:00 +0900
categories: [api-reference]
tags: [api-reference,Linux,Centos]
---

> #### SYSTEM Info
---
- Samba 4.10
- CentOS 7

> #### Samba Install

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