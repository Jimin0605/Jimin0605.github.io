---
title: Brute Force Wirte up
categories: 모의해킹
tag: 
 - DVWA
 - Brut Force
 - 모의해킹
 - Grootsecurity
 - study
 - write up
---

# 개요
저의 첫 Write up인 Brute Force에서는 웹 취약점 공부를 할 수 있는 DVWA(Damn Vulnerable Web Application)을 활용하여 해킹이 진행이 됬고, write up의 구성과 일부 내용은 Groot Security의 Brute Force wirte up을 예시를 참고하여 작성이 됬습니다.
<br/>
## 목차
[1. **취약점 설명/공부** - 브루트포스 공격에 대한 연구](#1-취약점-설명)<br/>
[2. **개념증명 실습** - 브루트포스 공격에 대한 연구](#2-개념-증명)
<br/>
[3. **대응 방안 공부** - Burpsuite, Hydra 등의 툴을 이용한 브루트 포스 공격](#3-대응방안)
<br/>
[4. **툴 제작** - 자체 제작한 파이썬 툴을 이용한 브루트포스 공격](#4-툴-제작)
<br/>
<br/>
<br/>
# 1. 취약점 설명
#### Brute Force는 자료구조의 Brut Force와 해킹에서의 Brute Force 2가지의미로 사용이 된다. 그렇다고 해서 서로 완전히 다른 개념은 아니다. 자료구조에서는 문제를 해결하기위해 모든 경우의 수를 탐색하는 방식을 의미하고, 해킹에서는 비밀번호 크래킹 등에서 모든 가능한 조합을 시도하는 공격 방법을 말한다. 이 글에서는 해킹에서의 Brute Force를 다룰것이다.


# 2. 개념 증명

# 3. 대응방안

# 4. 툴 제작

|정보|설명|
|---|---|
|**이름**|사용자 인증 브루트포스(Bruteforce)|
|**심각도**|높은|
|**CVSS**|8.1|
|**CVSS String**|CVSS:3.1/AV:N/AC:H/PR:N/UI:N/S:U/C:H/I:H/A:H|
|**위치**|http://localhost/vulnerabilities/brute/?username=admin&password=password&Login=Login|



## 레퍼런스
- **grootsecurity** - DVWA 브루트포스에 대한 writeup 예시:
https://security.grootboan.com/follow-along/undefined/0-dvwa/reference-writeup