---
title: "브라우저에 `google.com`을?"
categories:
    - Network
tags:
    - network
    - tcp
permalink: /network/:title
toc: true
---

<br>
<br>
<br>

# Chrome 검색창에 `google.com`을 입력 후 enter를 누른다면?

## 1. URL 분석

Chrome은 정확한 데이터 요청을 위해 URL을 분석할 것입니다.

### scheme
Scheme은 `http://`, `https://`, `file://`, `ftp://` 등의 전송 계획입니다. 일반적으로 웹페이지 정보는 http 혹은 https 통신으로 받아올 수 있습니다. 이전 버전의 Chrome은 이 scheme이 생략되어있다면 자동으로 http://를 사용했습니다. 하지만 현재 80% 이상의 웹 서비스는 https를 지원하고, 이런 서비스들에 http 요청을 날리면 https로 요청을 한번 더 날려야 합니다. 요청을 두번 날리는 비효율성을 극복하기 위해, 최신 chrome은 http:// 대신 https://를 사용합니다.

### domain
ip 주소를 알아낼 고정 이름입니다. google.com이 도메인에 해당합니다.

브라우저는 scheme, domain, path, resource를 분석하여 알맞은 GET request를 제작할 준비가 되었습니다.

### 검색어
많은 브라우저들은 주소창에 주소를 입력 기능 뿐만이 아니라 검색 기능도 포함하고 있으므로, url인지 검색어인지도 판독해야 합니다.

## 2. IP 주소 찾기

우리가 요청한 정보를 원활히 얻어오기 위해서는 인터넷 상의 어떤 서버에 연결해야 하는지 알아야 합니다. 우리가 입력한 url의 domain을 사용하여 웹사이트를 호스팅하는 서버의 ip 주소를 알아내야 합니다. 이는 DNS lookup을 통해 이루어집니다.

IP 주소를 빠르게 얻어와야 하기 때문에 DNS 데이터는 우리의 브라우저와 인터넷 전역의 여러곳에 캐시되어있습니다. 브라우저는 가장 첫번째로 자신의 캐시를 확인하고, 없다면 os 캐시를, 또 없다면 라우터의 로컬 네트워크 캐시를, 거기에도 없다면 사용중인 네트워크 혹은 ISP의 DNS 서버를 확인합니다. 이 캐시 단계중 어느곳에서도 찾지 못한다면 사용중인 네트워크 혹은 ISP의 DNS 서버는 재귀적 DNS lookup을 수행합니다. 재귀적 DNS lookup을 통해 원하는 정보를 얻을때까지 인터넷 전역의 여러 DNS 서버들에게 해당 정보를 요청하게 됩니다.

원하는 ip 주소를 얻어냈다면, 인터넷상의 서버를 찾고 연결을 시작할 준비가 되었습니다.

## 경로 찾기

TCP 통신중 클라이언트에서 전송한 패킷들이 서버에 도착하기 위한 최적의 경로를 찾아야 합니다. 저희 집 인터넷 케이블이 곧바로 구글 서버에 연결되어 있는 것이 아니므로 여러 라우터와 스위치를 거쳐 구글 서버로 이동할 수 있습니다. 터미널에 `traceroute google.com` 명령을 입력하여 어떤 경로를 거치는지 확인할 수 있습니다.

![traceroute]({{ site.url }}{{ site.baseurl }}/assets/images/network/enter_google_on_browser/1_traceroute.png)

집에 설치된 공유기부터(1) 구글 서버까지(13) 여러 기기들을 거친 것을 확인할 수 있습니다.

Handshake, http request, http response, 데이터 전송 등 모두 이러한 방식으로 경로를 찾게 됩니다.

## 3. 서버와 TCP 연결 맺기

네트워크상에서 통신을 하기 위해서는 internet protocol을 사용해야 합니다. TCP/IP가 가장 많이 사용되는 프로토콜입니다. 이 프로토콜을 사용하기 위해서 클라이언트와 서버는 3-way handshake를 통해서 서로의 정보를 알아가는 시간을 가져야 합니다. 3-way handshake에 대한 내용은 [TCP handshakes](https://brothersoo.github.io/network/TCP_handshaking) 포스트에 작성하였습니다.

## 4. 서버에 HTTP 요청 전송

악수를 했으니 데이터를 주고받을 준비가 끝났습니다. 브라우저는 서버에게 HTTPS (혹은 HTTP) 요청을 전송하여 페이지의 컨텐트를 받아올 것입니다. HTTP 요청은 request line, headers, 그리고 body를 포함합니다. Request line은 서버에게 클라이언트(해당 경우에는 브라우저)가 어떤 작업을 원하는지 알 수 있는 정보가 있습니다.

### Request line
Request line이 포함하고 있는 정보들
* Request method (GET, POST, PUT, PATCH, DELETE 등)
* url 분석단계에서 알아낸 path
* HTTP 버전

Request line 예시: `GET /hello_world HTTP/1.1`

### Headers
Headers는 요청을 라우팅하는데 도움을 주는 데이터, 그리고 어떤 타입의 클라이언트가 요청을 만들었는지를 담고 있습니다. 또한 A/B 테스팅, blue/green 배포, canary 배포에도 사용될 수 있습니다.

Headers 예시:
```
authority: www.google.com
method: GET
path: /complete/search?q&cp=0&client=gws-wiz&xssi=t&hl=ko&authuser=0&psi=El9eYqfUL5XR-QaKjYLwDQ.1650351890870&dpr=1
scheme: https
accept: */*
accept-encoding: gzip, deflate, br
accept-language: ko,en-US;q=0.9,en;q=0.8,ko-KR;q=0.7
```

### Body
우리가 전송한 요청과 같은 GET 요청에서 보통 body는 비어있습니다. POST, PUT, PATCH와 같이 자원을 조작하는 요청들은 클라이언트가 작성한 생성, 혹은 수정에 대한 데이터를 body에 가지고 있습니다.

Body의 포맷은 여러가지가 있고, 이는 request header의 `Content-Type`에 명시되어있습니다.

## 5. 서버의 response 전송

클라리언트의 요청을 받은 서버는 요청에 담긴 request line, header, body를 사용하여 요청에 대한 응답을 준비합니다.

Response에는 다음과 같은 정보들이 담겨있습니다.
* Status line
* Response Header
* Resource (HTML, CSS, Javascript, Image, data ...)

## 6. 브라우저 화면에 자료 render

브라우저가 서버로부터의 response를 수신한다면, 브라우저는 response header를 분석하여 자원을 어떻게 render할지를 정합니다.

<br>
<br>
<br>

**참고**
[[aws] What happens when you type a URL into your browser?](https://aws.amazon.com/ko/blogs/mobile/what-happens-when-you-type-a-url-into-your-browser/)