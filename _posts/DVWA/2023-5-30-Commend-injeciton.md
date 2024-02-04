---
title: Command injeciton Wirte up
categories: 모의해킹
tag: 
 - DVWA
 - command inejction
 - 커맨드 인젝션
 - 모의해킹
 - Grootsecurity
 - study
 - write up
---

# 개요
이번 Write up에서는 DVWA웹 어플리케이션을 이용해 Command injection공격을 수행하였습니다.

## 목차
[1. **취약점 설명/공부**](#1-취약점-설명)<br/>
[2. **개념증명 실습**](#2-개념-증명)<br/>
[3. **대응 방안**](#3-대응방안)<br/>
[4. 레퍼런스](#레퍼런스)<br/><br/>
## 취약점 정보
<br>

| 정보        | 설명                                             |
| ----------- | ------------------------------------------------ |
| **이름**    | 커맨드 인젝션(command injection)                |
| **심각도**  | 높은                                            |
| **CVSS**    | 10.0                                            |
| **CVSS String** | CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H    |
| **위치**    | [http://localhost/vulnerabilities/exec/](http://localhost/vulnerabilities/exec/) |

<br/>
<br/>

## 1. 취약점 설명
### Command injection이란.
웹 애플리케이션을 개발을 할때 기능을 직접 코딩을 해 개발을 하는 것 보다 설치되어 잇는 소프트웨어의 명령어를 통해 기능을 구현하는 것이 더 편리해 그것을 사용할 때가 있다. 이때 파라미터에 사용자가 입력한 인자를 전달할 때 입력값을 제대로 필터링하지 않아 의도한 기능이 아닌 수행을 하는 기능을 실행시키는 Command를 입력하는 것이 Command injection이라고 한다.

다음은 command injection에서 자주사용되는 기본적인 문법인 메타 문자이다.
<br>
<br>

|메타 문자|설명|Example|
|--------|----|-------|
|**\`\`**|**명령어 치환** <code>``</code> 안에 들어있는 명령어를 실행한 결과로 치환됩니다.|  <code>$ echo \`echo theori` theori</code>|
|**$()**|**명령어 치환** `$()`안에 들어있는 명령어를 실행한 결과로 치환 됩니다. 이 문자는 위와 다르게 중복 사용이 가능합니다. (`echo $(echo $(echo theori))`)|<code>$ echo $(echo theori)theori</code>|
|**&&**|**명령어 연속 실행** 한 줄에 여러 명령어를 사용하고 싶을 때 사용합니다. 앞 명령어에서 에러가 발생하지 않아야 뒷 명령어를 실행합니다. (Logical And)|<code>$ echo hello && echo theori hello theori</code>
|**\|\|**|**명령어 연속 실행** 한 줄에 여러 명령어를 사용하고 싶을 때 사용합니다. 앞 명령어에서 에러가 발생해야 뒷 명령어를 실행합니다. (Logical Or)|<code>$ cat / || echo theori cat: /: Is a directory theori</code>|
|**;**|**명령어 구분자** 한 줄에 여러 명령어를 사용하고 싶을 때 사용합니다. `;`은 단순히 명령어를 구분하기 위해 사용하며, 앞 명령어의 에러 유무와 관계없이 뒷 명령어를 실행합니다.|<code>$ echo hello ; echo theori hello theori</code>
|**\|**|**파이프** 앞 명령어의 결과가 뒷 명령어의 입력으로 들어갑니다.|<code>echo id | /bin/sh uid=1001(theori) gid=1001(theori) groups=1001(theori)</code>|


###### by. Dreamhack
<br>
<br>
## DVWA취약점.

```php
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = $_REQUEST[ 'ip' ];

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}

?>
```
이 php코드는 low level의 코드이다. 사용자에게 command를 입력받는 부분을 보면 필터링을 하지 않고있고, 그것을 그대로 불러와 shell_exec를 통해 명령어를 실행시키는 것을 확인할 수 있다. 따라서 BrutForce때와 마찬가지로 low level에서는 해당 취약점에 대한 별다른 시큐어코딩이 되어있지 않다는 것을 알 수 있다. 즉 메타문자를 사용하면 간단하게 커맨드를 injection할 수 있다. 
<br>
<br>
 
```php
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = $_REQUEST[ 'ip' ];

    // Set blacklist
    $substitutions = array(
        '&&' => '',
        ';'  => '',
    );

    // Remove any of the charactars in the array (blacklist).
    $target = str_replace( array_keys( $substitutions ), $substitutions, $target );

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}

?>
```
이 코드의 level은 medium이다. low와 달리 medium에서는 메타문자인 `&&`와 `;`를 필터링해 없애주는것을 확인할 수 있다. 그러나 모든 메타문자를 막지는 않았으므로 다른 문자를 사용하면 커맨드를 injection할 수 있을 것이다.
<br>
<br>

```php
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = trim($_REQUEST[ 'ip' ]);

    // Set blacklist
    $substitutions = array(
        '&'  => '',
        ';'  => '',
        '| ' => '',
        '-'  => '',
        '$'  => '',
        '('  => '',
        ')'  => '',
        '`'  => '',
        '||' => '',
    );

    // Remove any of the charactars in the array (blacklist).
    $target = str_replace( array_keys( $substitutions ), $substitutions, $target );

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}

?>
```
이 코드의 level은 high다. medium에서는 메타문자 `&&, ;`를 필터링을 해주었다면, 지금은 <code>&, ;, |, -, $, (, ), `, ||</code>로 모든 메타문자를 필터링 해주는것을 확인 할 수 있다. 하지만, 코드를 다시 확인해보면 파이프(|)뒤에 한칸이 띄어져 있는것을 알 수 있다. 개발자가 코딩을 할 때의 실수로인해 command injection에대한 취약점이 발생된것다.
<br>
<br>

```php
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Check Anti-CSRF token
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

    // Get input
    $target = $_REQUEST[ 'ip' ];
    $target = stripslashes( $target );

    // Split the IP into 4 octects
    $octet = explode( ".", $target );

    // Check IF each octet is an integer
    if( ( is_numeric( $octet[0] ) ) && ( is_numeric( $octet[1] ) ) && ( is_numeric( $octet[2] ) ) && ( is_numeric( $octet[3] ) ) && ( sizeof( $octet ) == 4 ) ) {
        // If all 4 octets are int's put the IP back together.
        $target = $octet[0] . '.' . $octet[1] . '.' . $octet[2] . '.' . $octet[3];

        // Determine OS and execute the ping command.
        if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
            // Windows
            $cmd = shell_exec( 'ping  ' . $target );
        }
        else {
            // *nix
            $cmd = shell_exec( 'ping  -c 4 ' . $target );
        }

        // Feedback for the end user
        echo "<pre>{$cmd}</pre>";
    }
    else {
        // Ops. Let the user name theres a mistake
        echo '<pre>ERROR: You have entered an invalid IP.</pre>';
    }
}

// Generate Anti-CSRF token
generateSessionToken();

?>
```
이 코드의 level은 impossible이다. 이 level의 코드는 다른 코드들과 달리 특정한 문자에 대한 필터링보다는 구조적인 필터를 하고있다. 먼저 이 command injection에 있는 이 입력은 `ping`명령어를 실행시키는 목적의 기능을 가지고있다. ping 다음 인자로는 ip인 `000.000.000.000`와 같이 3개의 점을 기준으로 숫자열들이 이루어져야 한다. 그래서 이 코드에는 ip를입력받고 `.`을 기준으로 4개의 문자열을 octet이라는 배열에 넣은뒤, 각 배열의 문자열이 숫자인지 그리고 `.`을 기준으로 나눈 영역의 수가 4개인지를 체크한뒤 명령어를 실행한다. 따라서 `0.0.0.0`과 같은 ip를 앞에 써 주고 메타문자를 사용해 다른 명령어를 실행하려해도 문자를 사용하는 순간 필터링이 되어 command injection을 효과적으로 막을 수 있다.


## 2. 개념 증명
개념증명은 medium과 high만 진행했다. 
[image](https://Jimin0605.github.io/assets/img/Write_up/Command_Injection/1.png)

## 3. 대응방안


## 레퍼런스
- Theori.DreamHack-command injection에 대한 설명: [https://learn.dreamhack.io/187#4](https://learn.dreamhack.io/187#4)


