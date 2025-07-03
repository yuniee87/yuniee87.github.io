---
title:  "Oracle DBMS_OUTPUT on Mariadb"
excerpt: "Mariadb에서 Oracle DBMS_OUTPUT 기능 사용하기"
author_profile: true
date: 2019-04-13 12:54:00 +0900
categories: [database]
tags: [database,mariadb,oracle]
---

DBMS_OUTPUT 패키지는 PL/SQL 블록(BEGIN ~ END)이나 프로시저, 함수 등 SUB PROGRAM 및 패키지, 트리거등에서 메시지를 출력할 수 있는 기능을 제공하는데 메시지를 버퍼에 저장하고 버퍼에서 읽어오기 위한 기능을 제공하는 오라클 패키지 입니다.

###### SYSTEM Info
-------------
- Oracle 11g
- MariaDB latest

###### Create Procedure
-------------

1. 작성 예제

```
DROP PROCEDURE IF EXISTS DBMS_OUTPUT;
CREATE PROCEDURE IF NOT EXISTS DBMS_OUTPUT(
    IN i_ITEM varchar(2000), 
    OUT o_RESULT varchar(2000)
)
    COMMENT 'Oracle DBMS_OUTPUT Procedure'
BEGIN
    DECLARE EXiT HANDLER FOR SQLEXCEPTION
    BEGIN
        GET DIAGNOSTICS CONDITION 1 @sqlstate = RETURNED_SQLSTATE, @errno = MYSQL_ERRNO, @text = MESSAGE_TEXT;
        SET o_RESULT = CONCAT("ERROR ", @errno, " (", @sqlstate, "): ", @text);
    END;
    SELECT i_ITEM as 'OUTPUT';
END
```

2. 실행 예제

```
CALL DBMS_OUTPUT('Test');
```

