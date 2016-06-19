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

일반적인 몇 가지 웹 서버 문제의 증상과 해결방안에 대해 설명합니다.

#### 설정 문제

*설정 문제* 는 웹 서버에서 확인할 수 있는 일반적이고 간단한 문제입니다.

설정에 구문오류가 있을 경우에는 서버를 리로드하지 않고 아래 명령어로 구문오류를 확인할 수 있습니다.

```sh
$ apache2ctl configtest
Syntax OK
```

만약 구문 오류가 있다면 해당 부분을 찾을 수 있도록 오류에 대한 파일과 줄 번호를 알려줄 것입니다.

```sh
$ apache2ctl configtest
apache2: syntax error on line 233 of /etc/apache2/apache2.conf; Could not open configuration file /etc/apache2/conf/: No such file or directory
```

#### 권한 문제

웹 서버의 권한 문제도 일반적인 골칫거리 중 하나입니다. 

* 아파치와 엔진엑스의 초기 프로세스를 모두 root로 실행했더라도 모든 서브 프로세스가 실제로는 더욱 제한된 권한을 가진 사용자(보통 www-data 혹은 아파치 같은 사용자로 서비스를 실행하고 있음
* 만약 해당 파일에 권한이 없다면 권한 문제를 겪을 수 있음

1. 서비스 중인 특정 파일에 권한이 없다고 가정합니다.
	* 여기서는 테스트를 위해 apache의 index파일의 권한을 *chmod 000 index.html* 로 축소시킵니다.
2. curl로 페이지 읽기를 시도합니다.

	```sh
	$ sudo chmod 000 index.html
	$ curl localhost
	<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
	<html><head>
	<title>403 Forbidden</title>
	</head><body>
	<h1>Forbidden</h1>
	<p>You don't have permission to access /index.html
	on this server.<br />
	</p>
	<hr>
	<address>Apache/2.4.18 (Ubuntu) Server at localhost Port 80</address>
	</body></html>
	```
3. 예상대로 실패하지만 permission 문제인지는 확실하지 않습니다.
4. */var/log/apache2/error.log* 를 확인해보면 아래와 같은 에러 메시지를 볼 수 있습니다.
	*/var/www/html/index.html* 에 접근이 거부당한 것을 알수 있습니다.

	```
	[Sun Jun 19 16:27:58.948110 2016] [core:error] [pid 1190:tid 140592575883008] \
	(13)Permission denied: [client 127.0.0.1:50388] AH00132: file permissions deny server access: /var/www/html/index.html
	```
5. 해당 파일에 권한을 줘 문제를 해결할 수 있습니다.
	* chmod o+x 명령어로 해당 파일을 누구나 읽을 수 있도록 할 수도 있음
	* 혹은, 파일의 group 소유권을 www-data 그룹이 소유하도록 변경

#### 느려지거나 사용할 수 없는 웹 서버 문제

이 단락에서는 느려진 웹 서버에 대한 일반적인 원인과 그러한 원인의 증상에 대해 살펴봅니다.

서버가 느려지거나 일시적으로 사용할 수 없을 때 맨 먼저 확인해봐야 하는 것은 부하입니다.

* 여기서는 웹 서버 프로세스로 인한 부하의 경우를 가정합니다.
* CPU나 RAM, 혹은 IO에서 발생하는지 판단하는 방법을 설명합니다.

1. CPU에 의한 것
	* 웹서버가 실행되 동적으로 내용을 생성하는 CGI나 PHP 코드 등과 관련된 문제를 해결해야 할 수도 있음
	* 웹 서버 로그를 통해 높은 부하가 발생하는 동안 어떤 페이지에 접근하는지 확인
	* 다양한 동적 페이지가 얼마나 많은 CPU를 소모했는지 측정하기 위해 해당 페이지들을 로드
		* 주 서버가 과부하 상태라면 테스트 서버에서 이 작업을 수행
2. RAM에 집중돼 있는 경우
	* 점점 더 많은 스왑 저장소를 사용해서 심지어 RAM이 완전히 고갈될지도 모르는 상황
	* 일반적으로 아파치 prefork 서버에서 일반적으로 발생하지만 worker 에서도 발생할 수 있음
	* 웹 서버를 설정할 때 응답을 위해 서버에서 생성할 수 있는 웹 서버 인스턴스의 최대 개수를 설정할 수 있음
		* apache prefork에서는 MaxClient
	* 시나리오
		* 서버가 너무 많은 요청을 받아 RAM이 처리할 수 있는 것보다 많은 프로세스를 생성하면 결국 스왑 영역을 사용하게 됨
		* 이로 인해 응답이 훨씬 느려짐
		* 궁극적으로 RAM과 스왑을 모두 소진할 때까지 현재 부하를 처리하게 됨
		* 그래서 쌓여있는 부하를 처리하기 위해 프로세스가 더 필요한 상황이 됨.
	* 해결방안
		* RAM 용량에 적당한 웹 서버 프로세스 개수를 계산
			1. 개별 웹 서버 프로세스가 얼마나 많은 RAM을 사용하는지 계산
			2. 운영체제에서 사용하는 부분을 제외한 전체 RAM을 확인
			3. 현재 남아있는 여유 RAM에 몇개의 아파치 프로세스가 적당한지 계산(스왑으로 넘어가지 않는)
		* RAM 용량에 비해 더 많은 프로세스를 시작하지 않도록 웹 서버를 설정
			* 정확한 측정이 되지 않을 경우 평균 혹은 최대로 사용하는 경우를 가정하고 설정
3. I/O에서 부하가 비롯되는 경우
	* 데이터베이스 요청에 의해 디스크 I/O가 포화 상태가 될 수 있음
		* 별도의 데이터베이스 서버를 구성
			* 네트워크를 통해 데이터베이스에서 응답을 기다리는 각 웹 서버 프로세스에서 부하가 걸릴 수 있다(원래보다는 적겠지만 말이다)
		* 저장 장치의 속도를 올림
		* 9장에서 데이터베이스 문제를 해결하는 방벙법을 더 참고한다.
	* 로그에서 IP 주소를 호스트명으로 변환하기 위해 역방향 DNS 해석을 사용하도록 설정했을 수도 있다.
		* 웹 서버 프로세스들은 요청을 만료하기 전에 단순히 이 요청이 끝날 때까지 기다릴 수도 있음


### Reference

[책정보, 데브옵스 : 네이버 책](http://book.naver.com/bookdb/book_detail.nhn?bid=7204813)
