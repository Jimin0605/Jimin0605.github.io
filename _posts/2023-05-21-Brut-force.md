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
저의 첫 Write up인 Brute Force에서는 웹 취약점 공부를 할 수 있는 DVWA(Damn Vulnerable Web Application)을 활용하여 해킹이 진행이 됬으며, write up의 구성과 일부 내용은 Groot Security의 Brute Force wirte up을 예시를 참고하여 작성이 됐습니다.
<br/>
## 목차
[1. **취약점 설명/공부** - 브루트포스 공격에 대한 연구](#1-취약점-설명)<br/>
[2. **개념증명 실습** - 브루트포스 공격에 대한 연구](#2-개념-증명)
<br/>
[3. **대응 방안 공부** - Burpsuite, Hydra 등의 툴을 이용한 브루트 포스 공격](#3-대응방안)
<br/>
[4. **툴 제작** - 자체 제작한 파이썬 툴을 이용한 브루트포스 공격](#4-툴-제작)
<br/>
[5. 레퍼런스](#레퍼런스)
<br/>
<br/>
<br/>
# 1. 취약점 설명
 Brute Force는 자료구조의 Brut Force와 해킹에서의 Brute Force 2가지의미로 사용이 된다. 그렇다고 해서 서로 완전히 다른 개념은 아니다. 자료구조에서는 문제를 해결하기위해 모든 경우의 수를 탐색하는 방식을 의미하고, 해킹에서는 비밀번호 크래킹 등에서 모든 가능한 조합을 시도하는 공격 방법을 말한다. 이 글에서는 해킹에서의 Brute Force를 다룰것이다.

앞서서 말했듯이 Brute Force는 암호 해독이나 인증 시스템을 침입하는 과정에서 사용되는 공격기법으로, 비밀번호 크래킹 등을 위해 가능한 조합들을 대입하여 접근을 시도하는 것을 말한다. 하지만 가능한 조합들이 문자열의 길이, 대소문자의 여부, 특수문자의 여부, 자신이 가지고 있는 정보에 따라 수도 없이 많아지기 때문에 시간복잡도면에서 정말 오랜 시간이 걸릴 수 있다는 단점과, 가능한 모든 경우의 수를 탐색하기위 해 많은 메모리가 필요될 수 있어 공간복잡도 측면에서도 이를 처리할 수 없는 상황이 발생할 수 있다.

이러한 단점들을 보완하기 위해 1. 오프라인 사전 공격

글자크기 test



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