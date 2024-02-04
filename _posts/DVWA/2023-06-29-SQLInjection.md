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
| **이름**    | SQL Injection               |
| **심각도**  | 심각                                         |
| **CVSS**    | 10.0                                              |
| **CVSS String** | CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H    |
| **위치**    | [http://localhost/vulnerabilities/brute/?username=admin&password=password&Login=Login](http://localhost/vulnerabilities/brute/?username=admin&password=password&Login=Login) |

<br/>
<br/>

## 1. 취약점 설명
### SQL Injection이란.

웹 개발을 할 때 데이터베이스에 여러 정보들을 저장하고 관리하기 위해 DBMS(DataBase Management System)을 사용한다. 이런 DBMS의 데이터들을 정의, 조작 등을 하기위한 언어인 SQL(Structured Query Language)이 사용된다. 

사용자는 웹 어플리케이션의 입력 폼에 문자를 입력하면 `SELECT * FROM Users WHERE username = '' AND password = ''`와 같은 SQL쿼리문에 삽입이 된후 명령어가 실행이 된다. 이때 공격자는 삽입을 할 문자를 `' OR 1=1 -- -`와 같이 입력을 하면 실행할 쿼리문이 `SELECT * FROM Users WHERE username = '' OR 1=1-- -' AND password = ''` 이와 같이 실행이돼 usernmae과 password가 일치하지 않더라도 더 나아가 username조차 입력하지 않더라도 Users에 있는 모든 정보들을 가져올 수 있도록 SQL구문을 주입하는것이 SQL Injection이다.

### DVWA취약점.
이번 SQL Injection에서는 SQL Injection이 발생할 경우 이후의 상황들을 설명하기위해 low level만 진행을 했다.

**low**
```php
<?php

if( isset( $_POST[ 'Submit' ] ) ) {
    // Get input
    $id = $_POST[ 'id' ];

    $id = mysqli_real_escape_string($GLOBALS["___mysqli_ston"], $id);

    $query  = "SELECT first_name, last_name FROM users WHERE user_id = $id;";
    $result = mysqli_query($GLOBALS["___mysqli_ston"], $query) or die( '<pre>' . mysqli_error($GLOBALS["___mysqli_ston"]) . '</pre>' );

    // Get results
    while( $row = mysqli_fetch_assoc( $result ) ) {
        // Display values
        $first = $row["first_name"];
        $last  = $row["last_name"];

        // Feedback for end user
        echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
    }

}

// This is used later on in the index.php page
// Setting it here so we can close the database connection in here like in the rest of the source scripts
$query  = "SELECT COUNT(*) FROM users;";
$result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );
$number_of_rows = mysqli_fetch_row( $result )[0];

mysqli_close($GLOBALS["___mysqli_ston"]);
?>
```
low의 코드는 위와 같고 사실 medium도 low처럼 시큐어 코딩이 제대로 되어있지 않다.

**low**

![image](https://Jimin0605.github.io/assets/img/Write_up/SQL_Injection/2.png)

**medium**
![image](https://Jimin0605.github.io/assets/img/Write_up/SQL_Injection/1.png)

low에서는 웹 어플리케이션에서 사용자에게 직접 입력을 받아 문자가 쿼리문에 들어갔다면 medium에서는 이미 지정된 옵션들이 존재하고 그 옵션들중 하나를 선택해 요청을 보내는 방식이다. 즉 다시말하면 medium에서는 ***User ID값을 공격자 마음대로 변경해 요청을 보낼수만 있다면 간단하게 SQL Injection에 성공***할 수 있을 것이다.

그래서 다음 Write up인 Blined SQL Injection에서는 high를 진행할 것이다.

## 2. 개념 증명
### SQL Injection
low에서는 시큐어 코딩이 되어있지 않기에 `union`을 이용해서 db에서 여러 정보들을 알아낼 수 있다. 우선 컬럼 수를 알아내기 위해 `' union select '1'-- -`, `' union select '1', '2'-- -` 갯수를 하나씩 늘려 입력을 했다.

`'1'`까지 입력한 결과 다음과 같은 쿼리 error가 발생했다.

![image](https://Jimin0605.github.io/assets/img/Write_up/SQL_Injection/3.png)

이 내용에서도 알 수 있듯이 컬럼수가 다르다는 것을 확인 할 수 있고 `'1', '2'`까지 입력했을 때

![image](https://Jimin0605.github.io/assets/img/Write_up/SQL_Injection/4.png)

위와 같이 First name이 첫번째 컬럼, Surname이 두번째 컬럼인것을 확인했다. 다음으로는 db의 목록을 확인 했는데 이때 입력으로는 `' union select table_schema, '2' from information_schema.tables-- -`를 입력했다. 

![image](https://Jimin0605.github.io/assets/img/Write_up/SQL_Injection/5.png)

위와 같이 결과가 나왔고 첫번 째 컬럼에 db이름을 넣어놨으니 First name에 있는 것만 보면 dvwa와 information_schema가 있는것을 확인했다. 여기서 information_schema는 기본적으로 생성이 되어있는 DB의 메타 정보(테이블, 칼럼, 인덱스 등의 스키마 정보)들을 모아둔 DB이고 현재 사용하고있는 db는 높은확률로 dvwa라고 추측을 했다. 따라서 dvwa 테이블명 을 확인하기위해 `' union select table_name, '2' from information_schema.tables where table_schema='dvwa'-- -`를 입력했다.


![image](https://Jimin0605.github.io/assets/img/Write_up/SQL_Injection/6.png)

전과 마찬가지로 첫번 째 컬럼만 봤을 때 guestbook과 users의 테이블이 있다는 것을 알 수 있었고 현재사용하는 테이블은 계정의 대한 정보인 users를 사용하고 있다고 추측했다. 
마지막으로 이 users의 테이블에는 어떤 컬럼들이 존재하는지 확인하기위해 `' union select column_name, '2' from information_schema.columns where table_name='users'-- -`를 입력했다. 

![image](https://Jimin0605.github.io/assets/img/Write_up/SQL_Injection/7.png)
이것도 첫번째 컬럼만 봤을때 users의 테이블에는 user_id, first_name, last_name, user, password, avatar, last_login, failed_login의 컬럼들이 존재한다는 것을 확인했다. 이때 가장 눈에 띄는것이 있는데 password부분이다. SQL Injection에서 password컬럼의 값을 알아낸다면 users에 있는 계정들을 탈취할 수 있게 되고 만약 users에 관리자 계정의 정보까지 있다면 더욱 심각한 결과를 초래할 수 있게 된다.

따라서 password컬럼을 출력해 계정 탈취 시도를 하기위해 `' union select user, password from users-- -`를 입력했다.

![image](https://Jimin0605.github.io/assets/img/Write_up/SQL_Injection/8.png)

결과로는 이렇게 나왔고 관리자 계정처럼 보이는 admin부분에 password가 `5f4dcc3b5aa765d61d8327deb882cf99`라는 것을 확인했다. 출력된 여러 password의 문자들을 봤을 때 해쉬화가 되었다는 것을 짐작했고 이것이 어떤것으로 해쉬화가 되었는지 확인하기 위해 kali linux에서 hash-identifier라는 도구를 사용했다. 

![image](https://Jimin0605.github.io/assets/img/Write_up/SQL_Injection/10.png)

결과적으로 이 암호는 높은 확률로 MD5로 암호화 되있다는 것을 확인했고, 이제부터는 Brut force 즉 무차별대입을 통해 어떤 문자가 암호화되었는지 확인을 했다. 이때는 John the Ripper라는 도구를 이용해 Brut force를 진행하였다. 

user와 암호화된 password를 `:`으로 연결시켜 users.txt라는 파일에 저장을 해두었고 이후 John the Ripper를 사용해 순식간에 password를 찾아냈다.

![image](https://Jimin0605.github.io/assets/img/Write_up/SQL_Injection/9.png)

![image](https://Jimin0605.github.io/assets/img/Write_up/SQL_Injection/11.png)


## 3. 대응방안
대응방안으로는
1. 입력 유효성검사
    - TODO   
2. 파라미터화된 쿼리사용
    - TODO
3. 데이터베이스 사용자에게 최소한의 권한 부여
    - TODO
4. 보안 패치 및 업데이트
    - TODO
5. 에러 메시지 관리
    - TODO
6. 웹 어플리케이션 방화벽 사용
이 있다.
## 4. 툴 제작
```python
# TODO
```

## 레퍼런스
- Theori.DreamHack-web hacking
- 해싱된 비밀번호 추청하기 - [홈페이지 취약점 분석 이야기](https://webhack.dynu.net/?idx=20161224.001)

