---
title: "Microservice Architecture"
categories:
    - architecture
tags:
    - msa
permalink: /architecture/:title
---

<br>
<br>
<br>

# Microservice Architecture(MSA)

> Microservice는 - microservice architecture이라고도 알려진 - 다음과 같은 성질들을 가지고 있는 서비스들의 집합으로 구성된 설계 방식입니다.
> - 유지보수와 테스트가 용이함
> - 느슨하게 연관됨
> - 독립적으로 배포 가능함
> - 비즈니스 단위로 구성됨
> - 작은 팀 단위로 운영됨
>
> [[microservices.io] What are microservices?](https://microservices.io/)

넷플릭스, 우버, 아마존, 이베이와 같이 서비스의 규모가 어마무시하게 커진 기업들은 꽤 옛날(?)에 microservice architecture(이하 MSA)를 채택하였습니다. 국내에서도 우아한 형제들, SK 플래닛과 같은 큰 기업들이 MSA를 채택하였고, 많은 스타트업들도 이미 MSA를 적용하였거나, 모노리식 구조에서 MSA로 전환하는 계획을 짜고 수행중에 있습니다. 왜 이렇게나 많은 기업들이 MSA에 열광하는걸까요?

<br>

# Monolithic Architecture

> Monolithic architecture는 소프트웨어 프로그램의 전통적인 통합적 모델 구조입니다. 여기서 monolithic은 한 곳에 모두 구성되어있는 것을 뜻합니다.\
> [[TechTarget] monolithic architecture](https://www.techtarget.com/whatis/definition/monolithic-architecture)

MSA에서 느슨히 엮인 구조로 여러개의 서비스들이 운영되는 것과 달리, 모노리식 아키텍처에서는 하나의 서비스 안에서 로직들이 상호 작용하고, 상호 연결되어있습니다. 우리가 통상적으로 만들어본 서비스들은 대개 모노리식 구조를 가지고 있었을 것입니다. Django 프레임워크를 사용해보셨다면 많은 애플리케이션들이 하나의 프로젝트 내에서 상호작용하며 수행되는 것을 경험해보셨을 것입니다. Django는 대표적인 monolithic 구조를 지니는 프레임워크입니다.

<br>

# Why MSA?

많은 모노리식 구조를 가지는 서비스들이 정삭적으로 기능을 수행하고, 상용되고 있음에도 불구하고, 왜 MSA를 사용해야 하는 것일까요? MSA를 채택해야 하는지, 채택하지 않아야 하는지는 서비스에 따라 다릅니다. 다음과 같은 특징들을 가진 프로텍트들은 MSA를 굳이 사용하지 않아도 됩니다.

1. 서비스의 규모가 크지 않은 경우
2. 비즈니스 도메인간 밀접한 상호작용을 요구하는 경우
3. 팀의 규모가 크지 않은 경우
4. MSA를 충분히 학습할 시간이 없는 경우

그렇다면 MSA를 사용하면 어떤 이점을 가져올 수 있을까요?

1. 확장성 용이
2. 오류 격리와 탄력적 애플리케이션
3. 특정 프로그래밍 언어 및 기술에 종속적이지 않음
4. 향상된 데이터 보안

<br>

# Patterns in MSA

MSA에 포함될 여러 서비스들 이외에도 여러 기술들이 적용되어야 할 수 있습니다.

## Gateway

MSA는 비즈니스 도메인 단위로 서비스들이 분리되어있습니다. 서비스의 양이 많아지면서 클라이언트가 모든 서비스들의 정보를 알고있기가 힘들어질 수도 있습니다. 한번에 여러 서비스를 호출해야할 수도 있고, 클라이언트의 종류에 따라 다른 요청을 할 수도 있습니다. 또 서비스들의 주소와 포트 번호가 유동적으로 변경될 수 있으며, 서비스들이 클라이언트에게 노출되지 않기를 원할 수 있습니다.

이런 문제들을 해결해 주는 패턴이 API gateway 패턴입니다. API Gateway를 서비스들 앞단에 둠으로써 클라이언트는 하나의 entry point를 얻을 수 있습니다. 클라이언트의 요청에 따라 적절한 서비스에 요청을 전달합니다.

API gateway에서 사용자 인증 처리를 하여 보안적 기능을 구현할 수도 있습니다.

Netflix zuul, spring cloud gateway를 사용할 수 있습니다. Zuul은 blocking API를 사용하여 websocket과 같은 long lived connection을 지원하지 않습니다. Spring cloud gateway는 non-blocking API를 사용하여 websocket을 지원합니다.

## Circuit breaker

MSA 내에서는 서비스간 데이터 요청이 빈번히 일어납니다. 그런데 네트워크 연결 불량, 타임아웃, 일시적인 장애 등, 데이터를 요청할 서비스에 문제가 생길 수 있습니다. 재요청을 통해 데이터를 얻을 수도 있지만, 심각한 문제로 인해 장애가 발생하는 것이라면 데이터를 정상적으로 받아오는데 오랜 시간이 걸릴 것입니다. 사용자는 영문도 모른채 최악의 경험을 할 수 있고, 하나의 서비스 장애로 연쇄적으로 다른 서비스들 또한 문제가 발생할 수 있습니다.

Circuit breaker 패턴을 통해 이런 문제를 극복할 수 있습니다. 사용자는 서비스를 프록시를 통해 호출할 것이고, 이 프록시는 circuit breaker(회로 차단기) 역할을 수행합니다. 임계치 이상의 장애가 발생한다면, circuit breaker는 일정 시간동안 차단되어있을 것이고, 그 시간동안 들어오는 요청들은 모두 실패됩니다. 차단 시간이 끝나면 서비스가 복구되었는지 확인하는 테스트가 수행되고, 테스트가 성공한다면 circuit breaker는 다시 복구되어 해당 서비스는 다시 정상 작동할 것입니다. 테스트에 실패한다면 다시 일정 시간동안 차단됩니다.

Spring Cloud Circuit Breaker는 실제 구현체들의 facade 라이브러리이고, 실제 구현체는 Netflix Hystrix, Resilience4J 등이 있습니다. Hystrix는 더이상 업그레이드 지원 계획이 없으므로 Resilience4J 사용을 권장합니다.

## Service discovery

시대가 발전하며 microservice들이 가상 / 컨테이너화 환경에서 실행되었고, 그로인해 서비스들의 수와 주소가 쉽게 변할 수 있어졌습니다. 동적으로 변하는 서비스들의 정보를 관리하기 위한 패턴이 service discovery 패턴 입니다.

Spring cloud eureka, aws elastic load balancer, 쿠버네티스를 사용한다면 k8s service를 사용할 수 있습니다.

## Saga

MSA 환경에서 각 서비스 당 하나씩 데이터베이스를 가질 수 있습니다. 하나의 요청으로 여러개의 서비스를 호출하고 데이터베이스에 연결한다면, 모든 데이터베이스 사용이 하나의 트랜잭션에서 실행되듯이 수행되어야 합니다. 2pc는 MSA에 적합하지 않습니다. 그대신 saga 패턴을 사용하여 트랜잭션 문제를 해결할 수 있습니다.

Saga는 순차적인 로컬 트랜잭션입니다. 각 로컬 트랜잭션은 데이터베이스 작업을 한 후 saga의 다음 로컬 트랜잭션을 작동시기키 위해 message 혹은 event를 발생시킵니다. 만약 트랜잭션이 모종의 이유로 실패한다면, saga는 이전에 수행되었던 로컬 트랜잭션에서 발생한 모든 변화들을 취소할 보상 트랜잭션을 수행합니다.

Saga는 두가지 방식으로 만들 수 있습니다.

### 1. Choreography

Choreography-based saga의 각 로컬 트랜잭션들은 다른 서비스들의 로컬 트랜잭션을 실행시키는 도메인 이벤트를 발햅합니다.

### 2. Orchestration

Orchestration-based saga에서는 orchestrator가 어떤 로컬 트랜잭션을 실행할지 참여 서비스에게 알립니다.

## Transactional Outbox




<br>
<br>
<br>

**참고**

[[microservice.io] Pattern: API Gateway / Backends for Frontends](https://microservices.io/patterns/apigateway.html)\
[[Medium] Design Patterns for Microservices - Circuit Breaker Pattern](https://medium.com/nerd-for-tech/design-patterns-for-microservices-circuit-breaker-pattern-ba402a45aac2)\
[[microservice.io] Pattern: Server-side service discovery](https://microservices.io/patterns/server-side-discovery.html)\
[[microservice.io] Pattern: Saga](https://microservices.io/patterns/data/saga.html)\
[[microservice.io] Pattern: Transactional outbox](https://microservices.io/patterns/data/transactional-outbox.html)
