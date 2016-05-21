---
layout: post
title: Http & Network Basic 4장 - 결과를 전달하는 HTTP 상태 코드
excerpt: "Http"
tags: [http, 그림으로 배우는 Http & Network Basic, HTTP 상태 코드, status code, status]
comments: true
---

`그림으로 배우는 Http & Network Basic 4장, 결과를 전달하는 HTTP 상태 코드`에 대해 정리한 포스트입니다.  

### 상태 코드

`상태 코드`는  Request 결과를 전달합니다.

상태코드의 첫 번째 자리는 Response의 클래스를 의미합니다.

* `1xx` Informational
	* Request를 받아들여 처리중
* `2xx` Success
	* Request를 정상적으로 처리했음
* `3xx` Redirection
	* Request를 완료하기 위해서 추가 동작이 필요
* `4xx` Client Error
	* 서버는 Request 이해 불가능
* `5xx` Server Error
	* 서버는 Request 처리 실패

### 2xx Success

`2xx`는 Request가 정상으로 처리되었음을 나타냅니다.

#### 200 OK

* Request를 서버가 정상적으로 처리했음.
* Method에 따라 포함되어 있는 정보는 달라짐
	
	> GET의 경우에는 대응되는 엔티티, HEAD의 경우에는 해더 필드만 Response에 포함됨.


#### 204 No Content

* Request를 받아 처리하는 데 성공했음.
* 되돌려 보낼 엔티티가 없음
		
	> 클라이언트에서 서버에 정보를 보내는 것으로 족하고, 클라이언트에 대해서 새로운 정보를 보낼 필요가 없는 경우에 사용

#### 206 Partial Content

* Range에 의해서 범위가 지정된 Request에 의해 서버가 부분적 `GET` Request를 받았음을 나타냄
* Content-Range로 지정된 범위의 엔티티가 포함됨

### 3xx Redirection

`3xx`는 브라우저 측에서 특별한 처리를 수행해야 함을 나타냅니다.

#### 301 Moved Permanently

* Request된 리소스에는 새로운 URI가 부여되어 있음을 나타냄(영구적으로 변경됨)
	* 이후에는 새로운 URI을 사용해야 함

	> 마지막 부분에 슬래시를 붙이지 않은 경우 자주 발생함

```
http://example.com/sample -> http://example.com/sample/
```

#### 302 Found

* Request된 리소스에는 새로운 URI가 할당되어 있어 그 URI를 참조해주라는 의미

	> 301 Moved Permanently와 비슷하지만 302는 일시적인 의미(추후에 다시 변경될 수 있음)

#### 303 See Other

* Request에 대한 리소스는 다른 URI에 있기 때문에 GET 메소드를 사용해서 얻어야 한다는 것을 나타냄
	* Redirect 장소를 GET 메소드로 얻어야 한다고 명확하게 되어 있음

> 예를 들면, POST 처리 결과를 별도의 URI에 GET 메소드로 리다이렉트 시키고 싶은 경우에 쓰일 수 있다.

#### 304 Not Modified

* 클라이언트가 조건부 Request를 했을 때 리소스에 대한 엑세스는 허락하지만 조건이 충족되지 않음
* Response 바디에 아무것도 포함하지 않음
* Redirect와는 관계가 없음
* 쿠키에 사용

#### 307 Temporary Redirect

> 301, 302, 303 이 Reponse로 오면, 대부분의 브라우저는 POST를 GET으로 바꾸어서 Request의 엔티티 바디를 삭제하고 Request를 자동적으로 재송신하도록 되어 있습니다(금지사항이지만).

302 Found와 같은 의미를 지니지만, POST에서 GET으로 치환하여 전송하지 않습니다.

### 4xx Client Error

`4xx`는 클라이언트의 원인으로 에러가 발생했음을 나타냅니다.

#### 400 Bad Request

* Request 구문이 잘못되었음을 나타냄
	* Request 내용을 재검토하고 나서 재송신할 필요가 있음

> 브라우저는 이것을 200 OK와 같이 취급

#### 401 Unauthorized

* Request에 HTTP 인증정보가 필요하는 것을 나타냄
* 유저 인증에 실패

#### 403 Forbidden

* Request된 리소스의 엑세스가 거부됨
* 엔티티 바디에 거부하는 이유를 표시

> 파일 시스템의 Permission이 부여되지 않은 경우, 엑세스 권한에 문제가 있는 등에 사용됨

#### 404 Not Found

* Request한 리소스가 서버상에 없다는 것을 나타냄

### 5xx Server Error

`5xx`는 서버 원인으로 에러가 발생하고 있음을 나타냅니다.

#### 500 Internal Server Error

* Request를 처리하는 도중에 에러가 발생하였음을 나타냄
* 웹 어플리케이션에 에러가 발생한 경우나 일시적인 경우도 있음

#### 503 Service Unavaliable

* 일시적으로 서버가 과부하 상태이거나 점검중이기 때문에 현재 Request를 처리할 수 없음을 나타냄
* 해소되기까지 시간이 걸리는 경우에는 *Retry-After* 헤더 필드로 전달하는 것이 바람직

### Reference

[그림으로 배우는 HTTP & Network Basic](http://www.interpark.com/product/MallDisplay.do?_method=Detail&ns1=list&ns2=oldList&ns3=prd&sc.shopNo=0000100000&sc.prdNo=3786671602)

