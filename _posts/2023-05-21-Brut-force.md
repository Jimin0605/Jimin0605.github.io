---
title: Brute Force Wirte up
categories: 모의해킹
tag: 
 - DVWA
 - Brut Force
 - SQL Injection
 - burp suite
 - 브루트포스
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
## Brute Force란.
 Brute Force는 **자료구조의 Brut Force**와 **해킹에서의 Brute Force** 2가지의미로 사용이 된다. 그렇다고 해서 서로 완전히 다른 개념은 아니다. 자료구조에서는 문제를 해결하기위해 모든 경우의 수를 탐색하는 방식을 의미하고, 해킹에서는 비밀번호 크래킹 등에서 모든 가능한 조합을 시도하는 공격 방법을 말한다. 이 글에서는 해킹에서의 Brute Force를 다룰것이다.

앞서서 말했듯이 Brute Force는 암호 해독이나 인증 시스템을 침입하는 과정에서 사용되는 공격기법으로, 비밀번호 크래킹 등을 위해 가능한 조합들을 대입하여 접근을 시도하는 것을 말한다. 하지만 가능한 조합들이 **문자열의 길이, 대소문자의 여부, 특수문자의 여부, 자신이 가지고 있는 정보**에 따라 수도 없이 많아지기 때문에 시간복잡도면에서 정말 오랜 시간이 걸릴 수 있다는 단점과, 가능한 모든 경우의 수를 탐색하기위 해 많은 메모리가 필요될 수 있어 공간복잡도 측면에서도 이를 처리할 수 없는 상황이 발생할 수 있다.

이러한 단점들을 보완하기 위해 **사전 대입 공격, 스마트 브루트 포스, 병렬 브루트포스, 가속화된 Brut Force**가 있다. 여기서 사전 대입 공격은 aaaa, aaab, aaac, ··· 이런식으로 무차별적으로 대입하는것이 아닌 비밀번호로 예를 들면 해당 사용자의 이름, 자주쓰는 닉네임, 생년월일, 전화번호 등 특정한 정보들을 조합해 여러 문자열들을 만들어 대입하는 방식이다. 병렬 Brut Force는 여러 개의 처리 장치나 컴퓨터를 동시에 사용하여 Brut Force알고리즘을 병렬화하는 방식이다.

## DVWA취약점.
```php
<?php

if( isset( $_GET[ 'Login' ] ) ) {   // Login 파라미터에 인자가 들어가면 True반환
    // Get username
    $user = $_GET[ 'username' ];    //

    // Get password
    $pass = $_GET[ 'password' ];
    $pass = md5( $pass );           // pass를 md5로 해싱

    // Check the database
    $query  = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";     // 쿼리 생성
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );    // db 쿼리 실행

    if( $result && mysqli_num_rows( $result ) == 1 ) {      // 명령어 실행 후 행의 수
        // Get users details
        $row    = mysqli_fetch_assoc( $result );    // 연관배열
        $avatar = $row["avatar"];

        // Login successful
        echo "<p>Welcome to the password protected area {$user}</p>";
        echo "<img src=\"{$avatar}\" />";
    }
    else {
        // Login failed
        echo "<pre><br/>Username and/or password incorrect.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?>
```
위 코드는 DVWA brut force low level의 php소스이다.
<br/>
소스를 보면 login을 하는데에 Brut Force에 대한 시큐어 코딩이 안되있는것을 확인 할 수 있다. 따라서 Brut Force를 하기위해 Burp Suite에 Intuder기능을 사용해 공격을 진행해본 결과 1분 내에 관리자의 이름과 비밀번호를 알아냈다.

그리고 Brut Force이외에도 쿼리를 넣을 때 password에는 md5로 해쉬화 하지만 username에는 별 다른 필터링이 없이 그대로 넣어지는 것을 봐서 SQL Injection 또한 공격이 가능하다는 것을 확인 할 수 있다. 

SQL Injection payload
admin' or '1'='1

# 2. 개념 증명
## Brut Force
개념 증명을 위해 해당 url에 Burpsuite의 내장되어 있는 Intruder기능을 사용해 공격을 시도 했고, 사용자의 username을 알고 있다고 가정을 하고 진행을 하겠다.
![image](/assets/images/Write_up/Brut_Force/1.png)


우선 HTTP요청을 잡기위해 입력란에 아무값이나 넣은 후 요청을 보낸 결과
![image](/assets/images/Write_up/Brut_Force/2.png)
위와같은 요청이 잡혔고, 로그인을 할 때 GET요청으로 username과 password, Login의 파라미터들이 있다는 것을 확인 했다. username이 admin이라는 것을 알고있으므로 username에는 admin을 넣어주었고, 위에 소스코드에서도 봤다시피 Login파라미터에 값이 없으면 동작을 하지 않으므로 아무런 값이나 집어 넣어줬다. 그리고 password부분에 Brut Force를 진행할 것이므로 아래와 같이 작업을 해주었다.

![image](/assets/images/Write_up/Brut_Force/3.png)
<br/>
<br/>
![image](/assets/images/Write_up/Brut_Force/4.png)
Payload의 type은 Simple list를 사용했고 사전 대입 문자열들로 12개 정도를 입력해주었고, 진짜 password인 password또한 추가해 주었다.

![image](/assets/images/Write_up/Brut_Force/5.png)
Attack결과로 위와같은 response의 결과값들이 나왔다. 여기서 Payload가 password이 외에 것들은 데이터 길이가 모두 4666으로 같은 값들이 나온것에 반해 password는 4704의 상대적으로 조금 큰 값이 나온것을 확인 할 수 있었고, password의 reponse를 보면 "Welcome to the password protected area admin"라는 문자열이 보이는 것으로 보아 로그인이 성공한 것을 알 수 있다.
<br/>
<br/>
## SQL Injection
해당 소스코드에서 Brut Force말고도 SQL Injection이 가능하다는 것도 확인 했었다. 

```php
 if( $result && mysqli_num_rows( $result ) == 1 ) {      // 명령어 실행 후 행의 수
        // Get users details
        $row    = mysqli_fetch_assoc( $result );    // 연관배열
        $avatar = $row["avatar"];

        // Login successful
        echo "<p>Welcome to the password protected area {$user}</p>";
        echo "<img src=\"{$avatar}\" />";
    }
```
전체 소스코드에서 if문 부분을 보면 $result 결과값이 있어야 하고 해당 결과값이 2가지 이상이 나온다면 거짓이되므로 username까지 알아내기에는 까다롭기에 username까지는 알고있다는 가정하에 진행을 하겠다.

```php
 $user = $_GET[ 'username' ];    //

    // Get password
    $pass = $_GET[ 'password' ];
    $pass = md5( $pass );           // pass를 md5로 해싱

    // Check the database
    $query  = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";     // 쿼리 생성
```
쿼리 를 생성하는 소스코드를 가져왔다. 해당 코드를 보면 처음 말했던 것처럼 password는 md5로 해쉬화하지만 username은 별다른 필터링을 하지 않고 바로 쿼리문에 집어 넣는 것을 확인할 수 있고, $user는 싱글쿼터(')안에 넣어 사용되고 있는것도 확인 할 수 있었다. 따라서 이 쿼리는 AND 로인해 user와 그 user의 해당하는 해쉬화된 pass값이 같아야지만 모든 행을 보여주는 소스이다. 결과적으로 username에 payload는 password와의 and로 묶여있는것을 풀어주기만 하면 로그인이 될 것이다.

![image](/assets/images/Write_up/Brut_Force/6.png)
```
username: admin'-- -
```
위와같은 방식으로 username은 admin으로 넣어주고 뒤에 password부분은 주석으로 날려보내면 로그인이 성공한 것을 알 수 있다.
<br/>
<br/>

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