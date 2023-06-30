---
title: SQL Injection Write up
categories: 
- 모의해킹
tag:
- DVWA
- SQL Injection
- 모의해킹
- Grootsecurity
- study
- write up
---
# 개요
이번 Write up에서는 DVWA웹 어플리케이션을 이용해 SQL Injection공격을 수행하였습니다.

## 목차
[1. **취약점 설명/공부**](#1-취약점-설명)
<br/>
[2. **개념증명 실습** ](#2-개념-증명)
<br/>
[3. **대응 방안 공부** ](#3-대응방안)
<br/>
[4. **툴 제작** ](#4-툴-제작)
<br/>
[5. 레퍼런스](#레퍼런스)
<br/>
<br/>
## 취약점 정보

| 정보        | 설명                                             |
| ----------- | ------------------------------------------------ |
| **이름**    | 사용자 인증 브루트포스(Bruteforce)                |
| **심각도**  | 심각                                         |
| **CVSS**    | 10.0                                              |
| **CVSS String** | CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H    |
| **위치**    | [http://localhost/vulnerabilities/brute/?username=admin&password=password&Login=Login](http://localhost/vulnerabilities/brute/?username=admin&password=password&Login=Login) |

<br/>
<br/>

# 1. 취약점 설명
## SQL Injection이란.
웹 개발을 할 때 데이터베이스에 여러 정보들을 저장하고 관리하기 위해 DBMS(DataBase Management System)을 사용한다. 이런 DBMS의 데이터들을 정의, 조작 등을 하기위한 언어인 SQL(Structured Query Language)이 사용된다. 

사용자는 웹 어플리케이션의 입력 폼에 문자를 입력하면 `SELECT * FROM Users WHERE username = '' AND password = ''`와 같은 SQL쿼리문에 삽입이 된후 명령어가 실행이 된다. 이때 공격자는 삽입을 할 문자를 `' OR 1=1 -- -`와 같이 입력을 하면 실행할 쿼리문이 `SELECT * FROM Users WHERE username = '' OR 1=1-- -' AND password = ''` 이와 같이 실행이돼 usernmae과 password가 일치하지 않더라도 더 나아가 username조차 입력하지 않더라도 Users에 있는 모든 정보들을 가져올 수 있는 공격이 되도

## DVWA취약점.
```php
```

# 2. 개념 증명
## SQL Injection

# 3. 대응방안

# 4. 툴 제작
```python
```

## 레퍼런스
- Theori.DreamHack-web hacking

