---
title: "TCP handshakes (feat. wireshark)"
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

# TCP

> TCP(Transmission Control Protocol)은 인터넷 프로토콜 suite의 핵심 프로토콜 중 하나로, IP와 함께 TCP/IP라는 명칭으로도 널리 불린다. TCP는 근거리 통신망이나 인트라넷, 인터넷에 연결된 컴퓨터에서 실행되는 프로그램 간에 일련의 옥텟을 안정적으로, 순서대로, 에러없이 교환할 수 있게 한다.\
> [[위키백과] TCP](https://ko.wikipedia.org/wiki/%EC%A0%84%EC%86%A1_%EC%A0%9C%EC%96%B4_%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C)

TCP는 일련의 옥텟을 안정적으로, 순서대로, 에러없이 교환할 수 있게 합니다. 이 특성들의 시작과 종료를 위해 TCP에서는 두가지 handshake를 진행합니다.

`www.google.com` 요청을 할 때의 패킷 정보들을 분석하기 위해 wireshark를 사용했습니다.

## Wireshark

네트워크 수업에서 wireshark를 사용하여 패킷 분석을 했던 기억이 어렴풋이 있어 이번에도 wireshark를 사용하여 tcp header의 상태가 어떻게 변화되는지 확인해보고자 합니다. Wireshark는 가장 널리 쓰이는 네트워크 프로토콜 분석기 중 하나입니다. [Download wireshark](https://www.wireshark.org/#download)를 클릭하시면 wireshark 공식 다운로드 페이지로 이동합니다.

<br>
<br>

# TCP header

주로 확인할 데이터들은 TCP header에 있는 데이터들입니다. TCP header는 end-to-end TCP 소켓의 파라미터와 상태를 담고있는 첫번째 24바이트 TCP 세그멘트 입니다. 이 header는 두 TCP 엔드포인트간 소통의 상태를 추적하는데 사용됩니다. TCP는 어떠한 시스템들이 소통하고 있는지는 추적하지 않고, 어떠한 end to end 소켓들이 열려있는지만 추적합니다.

<br>
<br>

# TCP header 형식

![tcp header]({{ site.url }}{{ site.baseurl }}/assets/images/network/tcp_headers_while_handshaking_http/1_tcp_header.png)

<br>
<br>

# 예제

`www.google.com`에 데이터를 요청했을 때 어떤 패킷들이 오고가는지 확인해보겠습니다. 다양한 프로세스들이 알게모르게 우리의 컴퓨터에서 인터넷 통신을 하고 있습니다. 그 중에서 저는 구글과의 통신 관련된 패킷만 확인하고 싶습니다. `$ curl http://google.com -v` 커맨드를 터미널에서 실행하여 google과 데이터를 주고받아보겠습니다.

![curl http://www.google.com -v]({{ site.url }}{{ site.baseurl }}/assets/images/network/tcp_headers_while_handshaking_http/2_curl_google.png)

`172.217.31.132` 주소에 80번 포트를 사용하여 데이터를 요청한 것을 확인할 수 있습니다. wireshark 상단 필터 입력 칸에 `ip.addr == 172.217.31.132`를 입력하면 해당 ip 주소와의 통신 패킷만 볼 수 있습니다.

![wireshark http://google.com]({{ site.url }}{{ site.baseurl }}/assets/images/network/tcp_headers_while_handshaking_http/3_wireshark_google.png)

몇개의 패킷 정보가 나타났습니다. 가장 상단의 패킷을 확인하면 SYN, ACK, ACK로 이루어진 패킷들을 확인할 수 있습니다.

10715, 11095, 11096번 패킷들이 3-way handshaking에 사용된 패킷들입니다. 제 컴퓨터가 구글 서버로 전송한 SYN이 담긴 패킷을 먼저 확인해보겠습니다.

<br>
<br>

## 1. Opening handshaking (3-way handshake)

### SYN packet by client

![client to server syn packet]({{ site.url }}{{ site.baseurl }}/assets/images/network/tcp_headers_while_handshaking_http/4_c2s_syn_packet.png)

위 항목부터 차례로 확인해보겠습니다.

**Source Port: 58512**\
제가 요청에 사용한 포트 번호입니다. 제가 사용하는 포트 번호는 매 요청당 랜덤값으로 변경됩니다. 이번 요청에서 사용한 포트 번호는 58512이고, 구글도 해당 요청에 대한 응답을 전송할때 이 포트 번호를 사용할 것입니다.

**Destination Port: 80**\
구글에서 사용한 포트 번호입니다. 80은 옛날옛적 HTTP 0.9에서 소개된 기본 포트 번호입니다. http 통신을 위한 포트 번호를 입력하지 않는다면 주소 뒤에 자동으로 포트번호 80을 붙입니다. `http://www.google.com`을 사용하는것과 `http://www.google.com:80`을 사용하는 것은 동일한 작업입니다.

서비스의 포트 번호를 80으로 설정한다면 사용자가 포트 번호를 입력하지 않아도 자동으로 80으로 설정될 수 있습니다. HTTPS 통신의 기본 포트는 443입니다. 대부분의 큰 서비스들은 포트를 80과 443으로 두어 사용자가 포트번호를 입력하지 않아도 되게끔 하는 것 같습니다! (TODO: incorrect)

**[Stream index: 13]**\
TCP stream은 어떤 conversation 안에서 주고받은 application message라고 합니다. Stream index가 동일하다면 같은 conversation 내에서 이루어졌다고 보면 될 것 같습니다.[참고](https://superuser.com/questions/1009951/relation-between-tcp-conversations-and-tcp-streams-in-wireshark)

**[Conversation completeness: Complete, WITH_DATA (31)]**\
Conversation은 말 그대로 대화입니다. TCP 통신은 client와 server 간 양방향 대화입니다. 요청마다 "연결 가능하세요?", "네 연결 가능합니다. 준비 되셨나요?" "네 준비됐습니다. 시작하시죠" 와 같은 대화를 나누게 됩니다. Conversation이 complete 상태라는 것은 opening handshake와 closing handshake가 모두 이루어진 conversation이라는 뜻입니다. WITH_DATA는 데이터를 주고받은 conversation이라는 뜻입니다. 31은 해당 conversation에서 어떠한 일들이 일어났는지를 말해줍니다. 각 숫자는 아래와 같이 정보를 가지고 있습니다.
- 1 : SYN
- 2 : SYN-ACK
- 4 : ACK
- 8 : DATA
- 16 : FIN
- 32 : RST

여섯가지 종류의 정보가 있으므로 총 6비트가 필요하고 000000(2) ~ 111111(2) (0 ~ 63) 범위의 숫자 내에서 정보 조합을 이뤄낼 수 있습니다. 3-way handshaking의 경우 SYN -> SYN-ACK -> ACK로 이루어지므로 3-way handshake만 진행된 conversation이라면 completeness는 7(1 + 2 + 4)이 될 것입니다. 해당 conversation은 31의 completeness를 가지고 있습니다. SYN, SYN-ACK, ACK, DATA, FIN 패킷을 주고받았다는 뜻입니다. Opening handshaking은 SYN, SYN-ACK, ACK 모두 계산해줬는데 왜 closing handshaking은 FIN 하나만 계산했는지는 잘 모르겠습니다...

**[TCP Segment Len: 0]**\
TCP segment length는 header를 제외한 담긴 데이터의 크기입니다. 0인것을 보아하니 handshaking에 사용하는 패킷들은 헤더 정보만 사용하는 것 같습니다!

**Sequence Number: 0 (relative sequence number)**\
상대적인 sequence number입니다. TCP는 순서를 보장해주는 통신이기 때문에 해당 sequence number로 패킷 순서를 맞출 수 있습니다. 이번 통신에서 나는 이 숫자로 sequence number를 시작할 것이니 동기화(SYNchronize) 해달라는 요청입니다. 첫번째 패킷이기 때문에 0입니다. 실제로 0은 아니고 비교하며 보기 쉬우라고 raw sequence number - ISN을 한 것입니다.

**Sequence Number (raw): 2160456513**\
실제 sequence number입니다. Sequence number는 랜덤으로 지정되는데 첫 sequence number(ISN: Initial Sequence Number)가 0으로 시작될 시, 많은 요청이 이루어질 때 중복될 수 있기 때문이라고 합니다. 또, IP address spoofing 혹은 session hijacking 같은 공격을 예방해준다고도 합니다.

**[Next Sequence Number: 1 (relative sequence number)]**\
다음 패킷 전송에서 사용될 sequence number입니다. 다음 sequence number는 이번 sequence number + segment length 입니다. 이번 패킷의 segment length는 0인데 왜 1 증가 되었을까요? 이는 SYN 요청이기 때문입니다. Payload 데이터가 없더라도 SYN 혹은 FIN flag가 설정되어있다면 1 증가된다고 합니다.

**Acknowledgment Number: 0**\
아직 SYNchronize 요청을 받은 것이 없으니 신경쓰지 않아도 됩니다. 그냥 0입니다.

**Acknowledgment Number (raw): 0**\
마찬가지로 신경쓸 필요 없이 0입니다.

**Flags: 0x002(SYN)**\
TCP header의 몇가지 큰 데이터를 확인했습니다. 여섯개의 비트 플래그들은 어떻게 찍혔을지 확인해보겠습니다.

![client syn bit flags]({{ site.url }}{{ site.baseurl }}/assets/images/network/tcp_headers_while_handshaking_http/5_bit_flags.png)

이번 패킷은 SYNchronize 요청이니 SYN flag만 설정되어있는 것을 확인할 수 있습니다. 해당 패킷을 수신한 서버는 SYN flag가 1로 설정된 것을 확인하고 Sequence number에 대한 ACKnowledge를 준비할 것입니다.

<br>

### SYN-ACK packet by server

![server to client syn ack packet]({{ site.url }}{{ site.baseurl }}/assets/images/network/tcp_headers_while_handshaking_http/6_s2c_syn_ack_packet.png)

방금 client가 송신한 sequence number 동기화 요청에 알겠다는 대답인 SYN과, 내가 사용할 sequence number 또한 동기화해달라는 SYN 요청을 함께 전송하는 것입니다. 포함된 데이터의 개념은 동일하니 sequence number, acknowledgement number, 그리고 플래그 비트만 간단히 확인해보겠습니다.

**Sequence Number (raw): 3210038829**\
이번 conversation에서 server가 사용할 sequence number입니다. 역시 랜덤으로 생성된 ISN입니다.

**Acknowledgement number (raw): 3509583631**\
수신한 SYN 패킷을 인정(ACKnowledge)한다는 의미입니다. 수신한 SYN 패킷의 마지막 sequence number(3509583630) + 1을 다음에 수신할 패킷의 시작 sequence number로 알고있겠다는 뜻입니다.

**Flags: 0x012 (SYN, ACK)**\
![server syn ack bit flags]({{ site.url }}{{ site.baseurl }}/assets/images/network/tcp_headers_while_handshaking_http/7_syn_ack_bit_flags.png)

SYN 요청과 ACK 대답 모두 포함된 패킷이니 ACK 비트와 SYN 비트 모두 1로 표시되었음을 알 수 있습니다.

<br>

### ACK packet by client

서버의 SYN 요청에 대한 ACK 응답입니다.

![client to server ack packet]({{ site.url }}{{ site.baseurl }}/assets/images/network/tcp_headers_while_handshaking_http/8_c2s_ack_packet.png)

**Sequence Number: 1 (relative sequence number)**\
앞선 SYN 요청에서 next sequence number를 1로 지정했으니 이번 sequence number는 1로 설정했습니다.

**[Next Sequence Number: 1 (relative sequence number)]**\
현재 sequence number + segment length 입니다. SYN flag도, FIN flag도 설정되어있지 않으니 이전처럼 1을 더해주지 않습니다.

**Acknowledgment Number: 1 (relative ack number)**\
마찬가지로 server 측에서 송신한 ACK-SYN 패킷의 마지막 segment number(2193251506) + 1을 다음 패킷의 시작 sequence number로 알고있겠다 하는 것입니다. (마찬가지로 relative 이니 raw - server's ISN)

<br>

이렇게 3-way handshaking에서 각 way의 tcp header가 어떤 데이터를 담고있는지 확인해보았습니다. 간단히 어떤 대화를 하는지 써보면 다음과 같습니다.

```
client ---> server  : "내 sequence number 동기화해줘"
client <--- server  : "네 sequence number 인정. 내 sequence number도 동기화해줘"
client <--- server  : "네 sequence number 인정."
```

<br>
<br>

## 2. Data request

앞서 지금 분석하고 있는 conversation의 completeness가 31임을 확인했고, 이는 데이터 전송이 포함되어있는 conversation임을 뜻하는 것을 확인했습니다. Opening handshaking이 정상적으로 수행되었으니 이젠 데이터를 안전하게 주고받을 준비가 되었습니다. 아래 패킷들이 데이터를 주고받는데 사용된 패킷들입니다.

![data request packets]({{ site.url }}{{ site.baseurl }}/assets/images/network/tcp_headers_while_handshaking_http/9_data_request_packets.png)


### HTTP GET request & ACK

![http get request]({{ site.url }}{{ site.baseurl }}/assets/images/network/tcp_headers_while_handshaking_http/10_http_get_request.png)

악수를 했으니 용무를 봐야합니다. 클라이언트가 원하는 어떤 정보를 원하는지를 HTTP에 담아 서버에게 전송했습니다.

<br>

### Send data

![response data]({{ site.url }}{{ site.baseurl }}/assets/images/network/tcp_headers_while_handshaking_http/11_response_data.png)

서버가 요청을 받았으니 요청을 잘 받았다는 말과 요청한 데이터를 전송 해주어야 합니다.

HTTP GET Request를 받은 서버는 요청한 데이터를 클라이언트에게 전송해줍니다. 요청을 확인했다는 ack 이후 열개의 패킷들은(11847 ~ 11856) 규격에 맞는 크기로 단편화된 패킷들입니다. 하나의 데이터였지만 그 크기가 MTU(Maximum Transmission Unit)을 넘어 중간 장치가 크기를 줄여 여러개의 패킷들로 분리했습니다.

**Sequence Number: 1 + (1418 * (i - 1)) (relative sequence number)**\
한 패킷당 tcp segment length가 1418로 지정되었으므로 sequence number는 1418씩 증가하게 됩니다.

**Acknowledgment number: 79 (relative ack number)**\
원래 하나의 전송이므로 클라이언트로부터 수신받을 다음 패킷의 seq number는 모두 79로 알고 있을 것입니다.

<br>

### HTTP 200 OK & ACK

![response data]({{ site.url }}{{ site.baseurl }}/assets/images/network/tcp_headers_while_handshaking_http/12_200_ok.png)

데이터 전송이 완료되고 status 200과 함께 text/html 데이터를 클라이언트에게 송신합니다.

클라이언트는 잘 수신했다고 ack를 송신합니다.

<br>
<br>

## 3. Closing handshaking (4-way handshake)

원하는 통신이 완료되었으니 FIN을 날려 이 conversation을 마무리 해야 합니다. 보통 closing handshaking을 배울 때 client가 먼저 FIN을 날리는 4-way handshaking을 배우지만, 사실 꼭 client가 먼저 FIN을 날릴 필요는 없다고 합니다. 원하는 데이터를 모두 전송했다면, server가 먼저 FIN을 날릴 수 있습니다.

(closing handshake img)

### FIN, ACK

### FIN, ACK

<br>
<br>

## 참고
[[inetdaemon] TCP Header](https://www.inetdaemon.com/tutorials/internet/tcp/tcp_header.shtml)

[[stack overflow] Why does a SYN or FIN bit in a TCP segment consume a byte in the sequence number space?](https://stackoverflow.com/questions/2352524/why-does-a-syn-or-fin-bit-in-a-tcp-segment-consume-a-byte-in-the-sequence-number)



tcp.port == 443 and tcp.completeness == 31

www.google.com      ip.addr == 172.217.31.132
google.com          ip.addr == 172.217.175.110