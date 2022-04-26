---
title: "Load balancing"
categories:
    - architecture
tags:
    - infra structure
permalink: /architecture/:title
---

> Load balancing은 네트워크 트래픽을 여러대의 서버에 분산하는 작업입니다. 이는 하나의 서버가 너무 많은 요청을 받지 않도록 해줍니다. 작업을 고르게 퍼트림으로써, 로드 밸런싱은 애플리케이션 응답 속도를 향상시킵니다.\
> [[AVI Networks] What is Load Balancing?](https://avinetworks.com/what-is-load-balancing/)

서비스의 성능을 향상시키기 위해 scale-out하여 서버 인스턴스의 수를 늘릴 수 있습니다. 서버의 수를 늘렸다면 트래픽을 적절히 서버들에 분산시켜 하나의 서버에 트래픽이 몰리지 않고 최적의 환경을 만들어야 합니다. 이를 위해 로드밸런서가 사용됩니다.

로드밸런서는 들어오는 요청을 무작위, 혹은 요청을 더 잘 수행할 수 있는 서버를 판별 후 전달합니다. 이를 통해 과부하가 생기는 서버는 줄어들 것이고, 더 나은 ux를 제공할 것입니다.

로드밸런서는 요청을 처리하는 서버의 상태가 정상적이지 않다면, 해당 서버가 복구될 때까지 제외시키는 작업도 할 수 있습니다. 새로운 서버가 추가된다면 로드밸런서는 해당 서버에게도 들어오는 요청을 전달해줄 것입니다.

<br>

# Types of load balancing

## 기능에 따른 분류

OSI 모델에서 우리의 네트워크 통신은 일곱개로 나뉜 계층을 순차적으로 통과하며 특수한 작업을 수행합니다. 어떤 계층에서 로드밸런싱이 수행되냐에 따라 로드밸런서의 종류가 나뉠 수 있습니다. 하지만 3 이하의 계층까지만 사용해서는 서버의 health check(서버가 정상 운영하는지 상태 확인)이 불가하기에 보통 4 계층 이상의 장비에서 로드밸런싱이 수행됩니다. 현재는 OSI 모델 대신 TCP/IP 5 계층이 사용되기 때문에 transport 계층, 그리고 application 계층에서 로드 밸런싱이 수행됩니다.

### Network load balancer(NLB) / Layer 4(L4) load balancer

Network 로드밸런싱은 IP 주소 혹은 목적지 포트번호와 같은 네트워크 변수를 사용하는 transport 계층의 트래픽 분산 처리입니다. 이 로드밸런싱은 4 이하의 계층(1, 2, 3, 4 계층) 정보를 사용하므로 쿠키 데이터, HTTP 헤더, 위치, 애플리케이션 행위 등과 같은 application 레벨의 변수들을 전혀 사용하지 않고 port 번호와 IP 데이터를 사용하여 로드밸런싱을 진행합니다. 보통 L4 로드밸런서라고 하기에 4 계층정보만 사용하는 로드밸런서라고 생각할 수 있지만, ip를 사용하는 것을 보면 알 수 있듯이 4 계층 이하의 데이터를 사용합니다. (Network load balancer)

L4 로드밸런서는 높은 수준의 데이터(application level)은 사용하지 않고 IP와 port만 사용하기 때문에 패킷을 모조리 분석할 필요가 없습니다. 이는 상대적으로 짧은 수행 시간을 가진다는 뜻입니다. 하지만 적은 데이터를 사용하여 간단한 방식으로 로드밸런싱을 수행하기 때문에 수준높은 정교한 로드밸런싱은 불가합니다. Round robin과 같이 단순한 알고리즘을 사용하여 부하를 분산할 수 있습니다.

### Application load balancer(ALB) / Layer7(L7) load balancer

OSI 모델의 가상 상층부인 application layer에서 로드밸런싱이 수행됩니다. HTTP 헤더, SSL 세션, URL, 데이터 타입, 쿠키와 같은 더욱 풍부한 데이터를 사용하여 로드밸런싱이 수행되기 때문에 더욱 정교한 로드밸런싱이 가능합니다.

L7 로드밸런싱은 application 계층까지의 데이터를 모두 사용하므로 비교적 많은 자원을 요합니다.

### Global server load balancer(GSLB) / Multi-site load balancer

Global server load balancer(이하 GSLB)는 사실 로드밸런서라기보다는 DNS 서비스 입니다. 일반적인 DNS 서버는 저장된 ip 주소 중 하나를 반환할 뿐, 서버에 이상이 있는지, 어떤 서버가 가장 적합한지를 확인하지는 않습니다(전세계에 걸쳐 여러대의 서버가 있는 경우). GSLB는 서버들의 health check를 하여 이상이 있는 서버의 ip는 유저에게 제공하지 않습니다. GSLB가 로드밸런싱을 하지 않는 것은 아닙니다. 일반적인 round robin 방식을 사용하는 일반적인 DNS 서버와는 달리 각 서버의 처리중인 로드를 파악하고 부하가 적은 서버의 ip를 우선적으로 반환하는 섬세한 로드밸런싱이 가능합니다. 또, latency를 확인해 최적의 서버의 ip를 제공하며, 사용자의 위치에서 가장 물리적으로 가까운 서버를 제공할 수 있습니다.

## 구성에 따른 분류

### Hardware 로드 밸런서

이름에서 알 수 있듯이, 여러대의 서버에 트래픽을 분산하는 물리적, on-premise 하드웨어 장비입니다. 높은 트래픽을 수용할 수 있지만, 유연성에 한계가 있고 비용이 상대적으로 높습니다. 로드 밸런서 내부의 코드를 지원하기 위해 펌웨어에 의존적입니다. 버그 수정, 새로운 버전의 배포를 위해 펌웨의 업데이트를 지원합니다.

### Software 로드 밸러서

소프트웨어 로드 밸런서는 서비스에 설치되어야 하는 애플리케이션입니다. 상업적으로 판매되는 상품일 수도 있고 오픈소스로 개방되어있을 수 있습니다. 대개 OS에 종속적이기 때문에 OS 업데이나 교체에 유의하여야 합니다.

<br>

# Types of load balancing methods

* **Round Robin**: 들어오는 요청들은 존재하는 서버들에 순서대로 배분될 것입니다.

* **Weighted round robin**: 트래픽 수용 능력과 같은 서버 환경에 따라 순차적으로 배분합니다.

* **Least connection**: 현재 연결이 가장 적은 서버에게 전달합니다.

* **Least response time**: 활성화 된 연결이 가장 적고, 평균 응답 시간이 가장 짧은 서버에게 전달합니다.

# Load Balancing products

## Hardware load balancer

### [F5 networks load balancer](http://www.openbase.co.kr/solution/network)

## Cloud load balancer

* AWS Elastic Load Balancing

* Azure Application Gateway

## Software load balancer

* *F5 NginX Plus

* API Gateway - AWS api gateway / spring cloud gateway

**참고**

[[AVI Networks] What is Load Balancing?](https://avinetworks.com/what-is-load-balancing/)\
[[appviewx] Load Balancer and Types](https://www.appviewx.com/education-center/load-balancer-and-types/#the-problem-statement)\
[[NGINX] What is layer 4 load balancing?](https://www.nginx.com/resources/glossary/layer-4-load-balancing/)\
[[Level up coding] L4 vs L7 Load Balancing](https://levelup.gitconnected.com/l4-vs-l7-load-balancing-d2012e271f56)
[[joinc.co.kr] GSLB - Global server Load Balancing](https://www.joinc.co.kr/w/man/12/GSLB)