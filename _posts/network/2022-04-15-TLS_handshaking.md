---
title: "HTTPS(TLS) handshakes"
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

# TLS handshaking

지난번에는 TCP의 두가지 handshaking에 대해 알아보았습니다.

TLS을 사용한 https 통신을 하더라도 http 통신과 마찬가지로 통신의 앞 뒤에 3-way handshake와 4-way-handshake를 진행합니다. 추가로, 실제 데이터를 주고받기 전 데이터 암호화를 위한 특별한 handshaking을 진행합니다. TLS handshake를 통해 client와 server는 최종적으로 대칭키를 생성할 것이고, 그 대칭키를 사용해 앞으로 전송할 데이터들을 암호화 할 것입니다.

# 대칭키 암호화 / 비대칭키 암호화

tls에는 대칭키 암호화와 비대칭키 암호화가 모두 사용됩니다. 대칭키 암호화는 말 그대로 암호화와 복호화에 사용되는 키가 동일합니다. 비대칭키 암호화는 암호화에 사용되는 키인 공개키(public key)와 복호화에 사용되는 키인 개인키(private key)를 따로 가집니다.

# Process

우선 tls handshaking에서 어떤 일이 벌어지는지 간단하게 알아보겠습니다.

tls 1.2, Diffie Hellman 키 교환 방식을 사용한 환경입니다.

`>` 는 client에서 server로의 전송, `<`는 server에서 client로의 전송을 의미합니다.\
`[ ]` 로 싸여있는 줄은 데이터 전송이 아닌 client와 server에서의 작업입니다.

```
> CLIENT HELLO (tls version, cipher suites, random byte, session id(optional) etc 포함)
< SERVER HELLO (cipher suite, random byte, session id, 포함)
< CERTIFICATE (server public key가 담긴 certificate 포함)
< SERVER KEY EXCHANGE (parameter for diffie hellman key exchange 포함)
< SERVER HELLO DONE
> [ diffie hellman 방식으로 pre-master secret 생성]
> CLIENT KEY EXCHANGE (server public key로 암호화한 pre-master secret 포함)
> CHANGE CIPHER SPEC
> ENCRYPTED HANDSHAKE MESSAGE(FINISHED)
< CHANGE CIPHER SPEC
< ENCRYPTED HANDSHAKE MESSAGE(FINISHED)
> [ pre-master secret -> master secret -> session key ]
< [ pre-master secret -> master secret -> session key ]
```

첫번째 통신부터 차례대로 어떤 일이 일어나는지 확인해보겠습니다.

## 1. (Client -> Server) CLIENT HELLO
* 서버가 HTTPS를 지원한다는 것을 안 클라이언트는 tls 통신을 시작하자고 서버에게 알립니다. 이것이 CLIENT HELLO 입니다.
* CLIENT HELLO는 몇가지 데이터를 포함하고 있습니다.
    1. TLS version
        클라이언트가 사용할 TLS의 버전을 알립니다. TLS 버전은 1.3까지 있지만 아직 상용화되어있지는 않습니다. 보통 v1.2가 사용됩니다.
    1. Cipher suites
        - 클라리언트마다 지원하는 cipher suite가 다를 수 있습니다. 클라이언트는 자신이 지원하는 cipher suite 리스트를 서버에게 알립니다.
    2. Random byte
        - 클라이언트는 난수를 즉시 생성하여 서버에게 전송합니다. 이 난수는 추후에 대칭키를 만들 주요 재료로 사용됩니다.
    3. Session id(Optional)
        - 이전에 SSL 통신을 진행한 적이 있다면 session id를 생성했을 것이고, 이후 ssl 통신에 이 session id를 사용할 것입니다.

## 2. (Server -> Client) SERVER HELLO
* 서버는 클라이언트로부터 받은 인사에 대답을 해줄 것입니다.
* SERVER HELLO 또한 몇가지 데이터를 포함하고 있습니다.
    - Cipher suite
        - Client가 자신이 사용할 수 있는 cipher suite 리스트를 알려줬으니, 서버는 그 리스트 내에서 자신이 사용할 수 있는 cipher suite를 고를 것입니다.
    - Random byte
        - 서버 또한 난수를 즉시 생성하여 클라이언트에게 전달합니다. 이 난수와 클라이언트가 생성한 난수는 대칭키를 만들 재료로 사용될 것입니다.
    - Session id
        - 이후 tls 통신에서 사용할 session id를 클라이언트에게 보냅니다.

## 3. (Server -> Client) CERTIFICATE & SERVER KEY EXCHANGE & SERVER HELLO DONE
1. CERTIFICATE (OPTIONAL)
    - 서버는 클라이언트에게 자신의 certificate를 전달합니다. 이 certificate는 server의 안전성을 보증하고, 서버의 공개키를 가지고있습니다.
    - 여러개의 CERTIFICATE를 가지고 있을 수 있고, 이를 Certificate Chain라고 합니다. Certificate의 갯수에는 제한이 없지만 너무 많은 certificate는 과도한 연산이 필요할 것이기 때문에 최대한 적은 certificate을 가지는 것이 좋습니다(최대 네개). 자세한 내용은 [[IBM] Multiple certificate selection](https://www.ibm.com/docs/en/i/7.3?topic=ssltls-multiple-certificate-selection)에서 확인할 수 있습니다.
2. SERVER KEY EXCHANGE (OPTIONAL)
    - CERTIFICATE에서 제공하는 서버의 공개키 정보가 충분하지 않다면 SERVER KEY EXCHANGE로 추가 공개키를 클라이언트에게 전달해야 합니다.
    - 예로, SERVER HELLO에서 정한 cipher suite가 EC Diffie-Hellman 방식을 사용한다면 (Cipher suite 이름에 ECDSA가 포함), 서버는 SERVER KEY EXCHANGE에 diffie-hellman 방식에 사용될 prams를 담아야 합니다.
3. SERVER HELLO DONE
    - SERVER HELLO가 끝났다고 클라이언트에게 전달합니다.

## 4. (Client) Generate pre-master key

클라이언트는 서버가 송신한 SERVER HELLO DONE을 수신한다면, pre-master secret을 생성합니다.

서버가 고른 cipher suite에 명시된 RSA 방식, 혹은 Diffie Hellman 방식으로 pre-master secret을 생성합니다.

## 5. (Client -> Server) CLIENT KEY EXCHANGE & CHANGE CIPHER SPEC & ENCRYPTED HANDSHAKE MESSAGE

1. CLIENT KEY EXCHANGE
    - 생성한 pre-master key를 수신한 certificate에 담긴 server의 public key를 사용하여 암호화 합니다. 암호화한 key를 담아 CLIENT KEY EXCHANGE를 송신합니다.

2. CHANGE CIPHER SPEC
    - 클라이언트는 이제 안전한 암호화 방식으로 데이터를 주고받을 준비가 되었다는 것을 서버에게 알립니다.

3. ENCRYPTED HANDSHAKE MESSAGE (FINISHED)
    - 클라이언트는 이제 tls handshake를 마치겠다고 서버에게 알립니다.
    - Handshake에 대한 hash를 암호화해서 전송합니다.

## 6. (Server) Generate pre-master secret
* 서버 또한 클라이언트와 동일한 방식(cipher suite에 명시)으로 클라이언트가 생성한 pre-master와 동일한 값을 가지는 pre-master key를 생성합니다.

## 7. (Client & Server) Generate master secret key
* CLIENT HELLO와 SERVER HELLO에서 생성한 랜덤 난수를 사용할 때가 되었습니다. 두 난수들은 클라이언트와 서버 모두가 가지고 있습니다. 이 두 난수들과 `pseudorandom function`(PRF)을 사용하여 master secret key를 생성합니다.

## 8. (Server -> Client) CHANGE CIPHER SPEC & ENCRYPTED HANDSHAKE MESSAGE

1. CHANGE CIPHER SPEC
    - 서버 또한 이제 안전한 데이터 교류가 가능하다고 클라이언트에게 알립니다.

2. ENCRYPTED HANDSHAKE MESSAGE(FINISHED)
    - 서버는 이제 tls handshake를 마치겠다고 클라이언트에게 알립니다.
    - Handshake에 대한 hash를 암호화해서 전송합니다.

## 9. (Client & Server) Generate session keys

Master key는 사실 데이터를 암호화하는데 사용할 대칭키가 아닙니다. 클라이언트와 서버는 master key에서 네가지(간혹 여섯가지) session key를 추출할 수 있습니다. PRF로 생성된 master secret은 48byte로 고정되어있기 때문에 네개의 key를 추출하기 간편합니다. 클라이언트와 서버가 가질 네개의 key들의 값은 각각 동일합니다.

* Client write key
    - 클라이언트가 데이터 암호화에 사용할 대칭키 입니다. 대칭키이기 때문에 서버는 클라이언트로부터 수신한 데이터를 같은 대칭키로 복호화할 수 있습니다.
* Server write key
    - 서버가 데이터 암호화에 사용할 대칭키 입니다. 대칭키이기 때문에 클라이언트는 서버로부터 수신한 데이터를 같은 대칭키로 복호화할 수 있습니다.
* Client write MAC(Message Authentication Code) key
    - 임의의 인물이 중간에서 데이터에 조작을 가하거나 데이터가 중간에 변질되는 것을 확인하기 위해 사용됩니다. 클라이언트는 암호화되지 않은 데이터를 client write key 뿐만이 아니라 Server write 키로도 암호화 합니다. 두 암호화된 데이터를 서버에게 보낼 것이고, 서버는 두 암호화된 데이터를 각각 client write key와 client write mac key로 복호화합니다. 두 데이터가 동일한다면 데이터의 무결성이 확증된 것이고, 다르다면 중간에 문제가 생긴 데이터로 취급됩니다.
* Server write MAC(Message Authentication Code) key
    - Client write MAC key와 동일한 역할을 수행합니다.

* Client write IV
    - 일부 cipher suite에서 사용되는 key입니다. 보통 사용되지 않습니다.
* Server write IV
    - 일부 cipher suite에서 사용되는 key입니다. 보통 사용되지 않습니다.

# 정리

tls 1.2는 안전한 암호화를 위한 대칭키를 생성하기 위해 크게 두번의 round trip을 가졌습니다. 첫번째에는 HELLO 교환, 두번째에는 key와 cipher spec 교환이 이루어졌습니다. 클라이언트와 서버는 비대칭키(private key, public key)를 사용하여 안전하게 대칭키를 생성할 재료들을 교환했습니다.

비대칭키는 암호화와 복호화에 사용되는 키가 다르기 때문에 대칭키 암호화보다 안전합니다. 하지만 그만큼 비용이 상당히 많이 듭니다. 따라서 데이터를 전송할 때는 비용이 저렴한 대칭키 알고리즘을 사용하는 것이고, 이 대칭키를 안전하게 생성하는데 비대칭키 암호화를 사용하는 것입니다.

# 너무나 비싼 비용

위에서 확인한바와 같이 대칭키를 만드는데 엄청난 작업이 소요되었습니다. 이는 절대 가벼운 작업이 아니기 때문에 두가지 종류의 방식으로 프로세스를 단축시킬 수 있습니다.

## Session id

위에서 확인한 CLIENT HELLO는 첫번째 통신으로 간주했기 때문에 session id가 포함되지 않았습니다. 이후 SERVER HELLO에 session id를 담아 클라이언트에게 전달했으므로 이후 통신에서 클라이언트는 이 session id를 사용할 것입니다. 클라이언트가 저장하고 있던 session id를 CLIENT HELLO에 담아 서버에게 전송하면 서버는 해당 session id가 유효한지 확인합니다. 해당 session id가 유효하다면, session id와 함께 저장하고 있던 이전에 사용한 공개키를 그대로 사용합니다. 즉, CERTIFICATE, CLIENT/SERVER KEY EXCHANGE, cipher suite 정하기 모두 진행되지 않습니다.

하지만 Session id 방식은 몇가지 문제점이 있습니다.
1. 서버가 session id, 공개키를 캐시에 저장하고 있어야 하므로 비용이 많이 듭니다.

2. L4 스위치 로드밸런싱을 사용한다면 session id의 재사용율이 현저히 떨어집니다. 항상 같은 서버로 요청을 날리지 않기 때문에 이전에 발급받은 session id를 이번 서버는 가지고 있지 않을 가능성이 큽니다.

## Session ticket

Session ticket 또한 session id 방식과 마찬가지로 첫번째 통신에는 full handshake를 진행해야 합니다. 첫번째 통신의 handshake의 마지막에 서버는 master secret과 cipher suite를 서버만 알고 있는 session ticket key로 암호화 한 후 클라이언트에게 전달합니다. 클라이언트가 session ticket 방식을 지원한다면 캐시에 해당 ticket을 master secret과 함께 저장하고 이후 통신의 CLIENT HELLO에 포함시킬 것입니다. 서버는 CLIENT HELLO에 포함된 session ticket을 복호화하여 얻은 master key를 사용하여 데이터를 암,복호화 할 것입니다.

L4 스위치가 다른 서버로 요청을 전달했더라도, 모든 서버가 동일한 session ticket key를 가지고 있다면 session ticket을 정상적으로 복호화 할 수 있습니다.

그러나 이 session ticket은 tls의 완벽히 안전함을 위반하는 것과 마찬가지입니다. 앞서 힘들게 직접 송,수신하지 않고 클라이언트와 서버가 동일한 master secret을 가지게 만들었는데, session ticket으로 인해 우리의 master secret이 언제든 탈취당할 수 있게 되었습니다(암호화는 되어있습니다).

# tls 1.3


<br>
<br>
<br>
<br>

**참고**

[[IBM] An overview of the SSL or TLS handshake](https://www.ibm.com/docs/en/ibm-mq/7.5?topic=ssl-overview-tls-handshake)\
[[IBM] How SSL and TLS provide identification, authentication, confidentiality, and integrity](https://www.ibm.com/docs/en/ibm-mq/7.5?topic=ssl-how-tls-provide-authentication-confidentiality-integrity)\
[[IBM] Multiple certificate selection](https://www.ibm.com/docs/en/i/7.3?topic=ssltls-multiple-certificate-selection)\
[[SSL.com] What Is a Certificate Authority (CA)?](https://www.ssl.com/faqs/what-is-a-certificate-authority/)\
[[Stack Exchange] What's the purpose of Server Key Exchange when using Ephemeral Diffie-Hellman?](https://security.stackexchange.com/questions/79482/whats-the-purpose-of-server-key-exchange-when-using-ephemeral-diffie-hellman)\
[[Youtube] F5 DevCentral - Explaining the Diffie-Hellman Key Exchange](https://www.youtube.com/watch?v=pa4osob1XOk)\
[[Medium] Prof Bill BuchananOBE - Ephemeral Diffie-Hellman with RSA (DHE-RSA)](https://medium.com/asecuritysite-when-bob-met-alice/ephemeral-diffie-hellman-c07c54afabff)\
[[Stack Exchange] Where are the DH parameters in a Server Exchange Message?](https://crypto.stackexchange.com/questions/42376/where-are-the-dh-parameters-in-a-server-exchange-message)\
[[CloudFlare] What is a session key? | Session keys and TLS handshakes](https://www.cloudflare.com/ko-kr/learning/ssl/what-is-a-session-key/)\
[[Stack Exchange] What is the purpose of four different secrets shared by client and server in SSL/TLS?](https://crypto.stackexchange.com/questions/1139/what-is-the-purpose-of-four-different-secrets-shared-by-client-and-server-in-ssl)\
[[CloudFlare] TLS Session Resumption: Full-speed and Secure](https://blog.cloudflare.com/tls-session-resumption-full-speed-and-secure/)\
[[Luavis' Dev Story] 알아두면 쓸데없는 신비한 TLS 1.3](https://luavis.me/server/tls-1.3)