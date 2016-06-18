---
layout: post
title: 데브옵스 8장 - 웹 사이트가 다운됐는가? 웹 서버 문제 추적하기
excerpt: "데브옵스"
tags: [devops, 데브옵스, 웹서버, web server]
comments: true
---

위키북스에서 출판한 데브옵스 8장 `웹 사이트가 다운됐는가? 웹 서버 문제 추적하기` 를 읽고 정리한 포스트입니다.

### Overview

### 서버가 동작 중인가?

첫 번째 웹 서버 문제 중 하나는 웹 서버가 완전히 사용할 수 없는 경우입니다.

> 여기서는 10.1.2.5:80이라 가정합니다.

#### 원격 포트가 열려 있는가?

클라이언트에서 아래 명령어로 포트가 열려있는지 확인합니다.

1. *텔넷* 으로 먼저 시도해봅니다.

	``` sh
	$ telnet 10.1.2.5 80
	Trying 10.1.2.5...
	telnet: Unable to connect to remote host: Connection refuse
	```
	* *Connection refused* 가 나온다면:
		1. 포트가 다운됨
			* 아파치가 원격 호스트에서 실행되고 있지 않음
			* 해당 포트가 리스닝 상태에 있지 않을 가능성
		2. 방화벽이 접근 차단

2. 방화벽을 감지하기도 하기 때문에 *nmap* 을 좀더 권장한다.

	``` sh
	$ nmap -p 80 10.1.2.5
	Stating Nmap 4.62 ( http://nmap.org ) at 2009-02-05 18.49 PSTInteresting ports on web1 (10.1.2.5):
	PORT STATE SERVICE
	80/tcp filtered http
	```
	* closed : 서버가 닫혀있음
	* opened : 서버가 열려있음
	* filtered : 방화벽에 막힘
		* 어떤 방화벽이 라우팅 구간에 존재해서 패킷을 바닥에 떨어뜨리고 있음
		* 해당 포트가 차단되었는지 게이트웨이(10.1.1.1)와 web1에 적용한 방화벽 규칙을 확인해야 한다.
	
#### 로컬에서 원격 호스트 테스트하기

호스트의 80번 포트를 사용할 수 있는지 여부를 테스트 하는 몇가지 방법에 대해 설명한다.

> 웹서버에 ssh로 접속해서 작업을 진행합니다.

**리스닝 포트 테스트하기**

*netstat -nlp* 명령어는 리스닝 상태에 있는 모든 포트와 해당 포트를 연 프로세스의 목록을 보여줍니다.

```sh
$ sudo netstat -lnp | grep :80
tcp 0 0 0.0.0.0:80 0.0.0.0:* LISTEN 919/Apache
```

순서대로 의미하는 바는 아래와 같습니다.

1. tcp : 포트가 사용하는 프로토콜
2. 0 0 : 수신 큐와 송신 큐(여기서는 둘 다 0으로 돼 있다.)
3. 0.0.0.0:80 : 호스트가 모든 IP주소에 대해 80번 포트를 리스닝하고 있음
4. 919/Apache : 포트를 열은 프로세스 이름

여기서 Apache가 리스닝 상태라는 것을 확인할 수 있습니다. 없을 경우에는 실행해야 한다는 사실도 알 수 있습니다.

**방화벽 규칙**

프로세스가 실행 중이며 80 포트에서 리스닝 중인 상태라면 방화벽이 접속 불가의 원인이 될 수 있다.

**iptables** 명령어로 방화벽 규칙을 볼 수 있습니다.

1. 방화벽이 비활성화

	```sh
	$ sudo /sbin/iptables -L
	Chain INPUT (policy ACCEPT)
	target     prot opt source               destination         

	Chain FORWARD (policy ACCEPT)
	target     prot opt source               destination         

	Chain OUTPUT (policy ACCEPT)
	target     prot opt source               destination   
	```

2. 방화벽이 기본적으로 모든 패킷을 버리도록 되어 있음

	```sh
	$ sudo /sbin/iptables -L
	Chain INPUT (policy DROP)
	target     prot opt source               destination         

	Chain FORWARD (policy DROP)
	target     prot opt source               destination         

	Chain OUTPUT (policy DROP)
	target     prot opt source               destination   
	```

3. 80번 포트를 차단하는 방화벽 규칙이 있는 경우

	이런 경우 서버의 방화벽 규칙을 수정해 문제를 해결할 수 있을 수도 있다.

	```sh
	$ sudo /sbin/iptables -L
	Chain INPUT (policy ACCEPT)
	target     prot opt source               destination         
	REJECT tcp -- 0.0.0.0/0 0.0.0.0/0 tcp dpt:80
	reject-with icmp-port-unreachable

	Chain FORWARD (policy ACCEPT)
	target     prot opt source               destination         

	Chain OUTPUT (policy ACCEPT)
	target     prot opt source               destination   
	```

### 명령줄에서 웹 서버 테스트하기

웹 서버가 실제로 올바른 포트에서 리스닝하고 있다는 것을 확신한다면 그 다음 문제 해결 단계는 웹 서버가 실제로 요청에 응답하는지 확인하는 것입니다.  
여기서는 *curl* 이나 *telnet*과 같은 명령줄 도구를 이용합니다.

* 대부분의 서버에 GUI 브라우저가 설치되어 있지 않으며 ssh 접속으로도 테스트가 가능하기 때문에

#### Curl로 웹 서버 테스트하기

Curl은 여러 도구 중에서도 HTTP와 HTTPS 프로토콜로 통신할 수 있는 명령줄 도구입니다.

* curl은 HTTP 프로토콜을 처리함으로써 인증 테스트, 데이터 게시, SSL 사용 등을 지원

curl로 웹 서버를 테스트하는 것은 아주 간단합니다.

```sh
$ curl http://www.example.net
<html><body><h1>It works!</h1>
<p>....
```

만약 접근할 수 없다면 아래 메시지가 출력될 것입니다.

```sh
$ curl http://www.example.net
curl: (7) couldn't connect to host
```

또한 추가 정보를 원할 경우 *-w* 옵션으로 요청할 수 있습니다. 아래는 상태 코드, 요청에 걸린 시간, 다운로드된 데이터의 크기, 콘텐츠 타입을 출력해줍니다.

```sh
$ curl -w "%{http_code} %{time_total} %{size_download} %{content_type}\n" http://www.example.net
<html><body><h1>It works!</h1>
<p>....
200 0.004 177 text/html
```

#### 텔넷으로 웹 서버 테스트하기

텔넷은 모든 리눅스 시스템에 설치되어 있고, HTTP에 대해 몇 가지 기본적인 사항만 알고 있다면 귀중한 각종 진단 데이터를 웹 서버에서 가져올 수 있습니다.

* curl이 설치되 있지 않은 서버에서 웹 서버를 테스트해야 할 경우
* 저수준 HTTP 호출을 표시해야 할 때

텔넷은 *telnet [HOST] [PORT]* 로 사용할 수 있습니다. 

```sh
$ telnet www.example.net 80
Trying 10.1.2.5...
Connected to www.example.net.
Escape character is '^]'.
```

접속 이후 아래와 같이 요청을 보낼 수 있습니다

```sh
GET / HTTP/1.1
host: www.example.net
```

1. GET / : 기본 인덱스 페이지를 요청
2. HTTP/1.1 : 프로토콜
3. host: www.example.net : 연결하고자 하는 호스트의 이름

위 명령어를 치고 엔터를 누르면 서버로부터 완전한 응답을 받을 수 있습니다.

```sh
$ telnet localhost 80
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
GET / HTTP/1.1
host: localhost

HTTP/1.1 200 OK
Date: Sat, 18 Jun 2016 06:12:51 GMT
Server: Apache/2.4.18 (Ubuntu)
Last-Modified: Sat, 18 Jun 2016 06:12:08 GMT
ETag: "11c-5358759e9ae13"
Accept-Ranges: bytes
Content-Length: 284
Vary: Accept-Encoding
Content-Type: text/html


<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
  <head>
    <title>Apache2 Ubuntu Default Page: It works</title>
  </head>
  <body>
        It works!
  </body>
</html>
```

### HTTP 상태 코드

웹 서버 문제를 해결할 때 웹 서버가 각 요청에 응답하는 상태 코드는 매우 유용합니다.

> 하 이부분만 몇번을 하는거지.. 넘나 많이 나오는 것. 자세한 설명은 다 생략하겠음

#### 1xx 정보 전달 코드

1로 시작하는 상태 코드는 정보를 제공하는 응답 코드입니다.

#### 2xx 성공 코드

2로 시작하는 상태 코드는 성공적인 요청을 나타냅니다.

#### 3xx 리다이렉션 코드

3으로 시작하는 상태코드는 서버로부터의 일종의 클라이언트 리다이렉션 메시지를 나타냅니다.

#### 4xx 클라이언트 오류 코드

4로 시작하는 상태코드는 서버가 클라이언트 측에 오류가 있는 것으로 판단하고 다루게 합니다.(404 Not Found)

#### 5xx 서버 오류 코드

5로 시작하는 상태코드는 서버측의 에러를 나타냅니다.(500 Internal Server Error)

### 웹 서버 로그 분석하기

웹 서버 문제를 해결하는 기본적인 방법 중 하나는 *로그를 이용하는 것*입니다.

* 웹 서버가 받아들이는 각 요청은 표준 형식으로 로그에 기록됨
	* 이 로그는 문제 해결을 위한 유용한 정보가 됨
	* apache : /var/log/apache2
	* nginx : /var/log/nginx
	* access.log(요청 로그), error.log(오류 로그)

curl로 GET 요청을 보낸 이후 access.log를 확인해보면 아래와 같은 접속 기록이 추가됩니다.

```sh
$ curl -w "%{http_code} %{time_total} %{size_download} %{content_type}\n" http://www.example.net
<html><body><h1>It works!</h1>
<p>....
200 0.004 177 text/html
```

```
10.1.2.3 - - [18/Jun/2016:15:12:10 +0900] "GET / HTTP/1.1" 200 303 "-" \
"curl/7.19.7(x86_64-pc-linux-gnu) libcurl/7.19.7 OpenSSL/0.9.8k zlib/1.2.3.3\
libidn/1.15
```

로그 항목은 여러 다른 항목으로 나눠져 있으며, 해당 영역이 데이터를 가지고 있지 않거나 적용할 수 없는 경우에는 '-'로 대체됩니다.

이 로그를 통해 아래 정보들을 알 수 있습니다.

1. 요청을 한 IP 주소가 10.1.2.3라는 것
2. *GET / HTTP/1.1* 요청에 대해 HTTP코드 200을 반환
3. User-Agent는 curl임

### 웹 서버 통계 얻기

Apache와 Nginx 모두 서버 상태 페이지를 제공합니다. 그를 통해 아래와 같은 것들을 알 수 있습니다.

* 얼마나 많은 웹 서버 프로세스가 현재 요청을 서비스하고 있는지
* idel상태의 프로세스는 얼마나 되는지
* 바쁜 프로세스는 무었을 하고 있는지

Ubuntu 서버 Apache의 경우에는 *a2enmod status* 를 통해 서버 상태 페이지를 활성화합니다.

서버가 www.example.net에 위치한다고 가정했을 때 http://www.example.net/server-status 로 접근이 가능합니다.
 

### 일반적인 웹 서버 문제 해결하기

#### 설정 문제

#### 권한 문제

#### 느려지거나 사용할 수 없는 웹 서버 문제

### Reference


