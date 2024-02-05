---
title: "[RCity] Linux Quiz 1"
categories: [Redraccoon, RCity]
tag:
- wargame
- redraccoon
- ctf
- 워게임
- 레드라쿤
---

# 개요
Redraccoon에서 운영하는 RCity라는 워게임에 Linux Quiz 1 에 대한 Write up을 작성했습니다.


> **해당 공격 기법들을 허가되지 않은 실제 운영 서버에서 시도하는 것은 정보통신망법에 어긋나는 행위입니다.**
{: .prompt-warning }

## 개념정리
`strings`명령어의 사용법은 다음과같다. `strings [옵션] [파일]`

string는 파일을 ASCII로 읽었을 때 문자열로 표시가능한 부분을 출력하는 명령어다. 옵션은 다음 표와같다.

|옵션|설명|
|----|----|
|-a| 인쇄 가능한 문자열은 데이터 섹션뿐만 아니라 전체 파일을 검색합니다. 이 플래그를 생략하면, strings 명령은 오브젝트 파일의 초기화된 데이터 공간을 살펴봅니다.|
|-n [number]|디폴트 값 4자 이외의 최소 문자열 길이를 지정합니다. 문자열 길이의 최대값은 4096입니다. 이 플래그는 -Number 플래그와 동일합니다.|
|-o|파일에서 8진 오프셋을 선행으로 각 문자열을 나열합니다. 이 플래그는 -t o 플래그와 동일합니다.|
|-t format| 파일의 시작부터 오프셋을 선행으로 하는 각 문자열을 나열합니다. 형식은 Format 변수로 사용되는 문자에 따라 다릅니다. <br> d 오프셋을 10진수로 기록합니다. <br> o 오프셋을 8진수로 기록합니다. <br> x 오프셋을 16진수로 기록합니다. <br> "주: -o 및 -t Format 플래그가 명령행에 두 번 이상 정의될 때, 지정된 최종 플래그가 문자열 명령의 작동을 제어합니다."|
|-[Number]|디폴트 값 4자 이외의 최소 문자열 길이를 지정합니다. 문자열 길이의 최대값은 4096입니다. 이 플래그는 -n NUmber플래그와 동일합니다.|
|File|검색할 2진 또는 오브젝트 파일입니다.|

## 문제분석
```
🥷 시나리오: 당신은 리눅스 서버에서 성공적으로 침투한 해커입니다.
정찰 단계에서 전체 파일 시스템에서 "password"라는 단어가 파일 내용 안에 포함된 파일을 찾아야 합니다!

❓ 질문: Linux 파일 시스템에서 "password"라는 단어가 파일 내용 안에 포함된 파일을 검색하기 위해 다음 중 어떤 명령을 사용해야 할까요?
``` 
{: file='문제내용'}

```shell
 1. find / -name "password"
 2. grep -r "password" /
 3. ls -lR | grep "password"
 4. chmod -R 777 / | grep "password"
```
{: file='선택지'}

문제를 보면 지금까지와는 다르게 flag파일이 문자열기반이 아닌 다른 타입의 flag파일이라는 것을 알려주고있다. 따라서 해당 타입에 따른 명령어를 사용해 숨겨진 flag를 찾는것이 목표다.

![image](https://Jimin0605.github.io/assets/img/Redraccoon/RCity/20.png)

현재 홈 디렉토리를 보니 `flag`라는 파일이 있는 것을 확인했고 `file`명령어로 해당 파일의 타입을 확인해보니 `ELF`파일이라는 것을 확인했다. `ELF`파일은 리눅스의 실행파일이다. 팁을 주자면 `ls`를 했을 때 이런식으로 초록색(접속한 서버나 shell에 따라 달라질 수 있다.)으로 이름이 출력되면 실행파일이라는 것이다.

따라서 해당 파일을 실행시켜본 결과 처음을 제외하고 이상한 문자열들을 확인할 수 있다. 또 2번째 문단에 `Lorem ipsum`이라는 단어가 있는것을 알 수 있는데 `Lorem ipsum`은 한국어로는 로렘 입숨이라고 불리는데 출판이나 디자인분야에서 폰트 또는 레이아웃등의 예시를 보여줄 때 사용하는 아무런 뜻이 없는 글이다. 따라서 flag를 실행했을 때 나오는 문자열들은 첫번째 문단을 제외하고는 아무런 의미가없다고 추측할 수 있다.

실행파일을 실행할때는 flag를 획득하지 못했으므로 한번 실행파일을 ASCII로 읽어보겠다. `cat flag`

![image](https://Jimin0605.github.io/assets/img/Redraccoon/RCity/21.png)

확인해본결과 사람이 읽을 수 없는 이상한 문자들과 읽을 수 있는 문자들이 있는 것을 확인했다. 따라서 `strings flag`를 통해 읽을 수 있는 문자열들만 출력을 해봤다.

![image](https://Jimin0605.github.io/assets/img/Redraccoon/RCity/22.png)

결과 flag처럼보이는 문자열이 보였다.




## Reference
strings 옵션 table - [https://www.ibm.com/docs/ko/aix/7.2?topic=s-strings-command](https://www.ibm.com/docs/ko/aix/7.2?topic=s-strings-command)


