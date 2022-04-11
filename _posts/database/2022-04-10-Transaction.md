---
title: "Transaction"
categories:
    - database
tags:
    - cs
    - database
permalink: /database/:title
---

# Transaction

> 데이터베이스 트랜잭션은 데이터베이스 관리 시스템 또는 유사한 시스템에서 상호작용의 단위이다. 여기서 유사한 시스템이란 트랜잭션이 성공과 실패가 분명하고 상호 독립적이며, 일관되고 믿을 수 있는 시스템을 의미한다.\
> [Wikipedia - 데이터베이스 트랜잭션](https://ko.wikipedia.org/wiki/%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4_%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98)

# Transaction 속성

## ACID

1. Atomicity\
한 트랜잭션은 하나의 원자 단위로써 취급되어야 합니다. 즉, 하나의 트랜잭션의 모든 작업들은, 모두 실행되거나, 모두 실행되지 않아야 합니다. 데이터베이스에 부분적으로 실행 완료된 트랜잭션이 남아 있어서는 안됩니다. 

2. Consistency\
어떤 트랜잭션이 완료된 후에, 데이터베이스는 일관된 상태를 유지해야 합니다. 어떤 트랜잭션도 데이터베이스에 있는 데이터에 부작용을 일으켜서는 안됩니다. 데이터베이스가 트랜잭션이 실행 되기 전에 일관된 상태를 유지하고 있었다면, 트랜잭션이 완료된 이후에도 일관적인 상태를 유지해야 합니다.

3. Isolation\
데이터베이스 시스템에는 하나 이상의 트랜잭션들이 동시에, 그리고 병렬적으로 실행될 수 있습니다. Isolation 특성은 모든 트랜잭션들이 자신이 시스템의 유일한 트랜잭션인 듯이 수행되고 실행됨을 나타냅니다. 어떤 트랜잭션도 다른 트랜잭션의 존재에 영향을 끼치지 않습니다.

4. Durability\
데이터베이스는 시스템에 장애가 생기거나 다시 시작되어도 마지막으로 완료된 작업을 유지할 만큼 안전해야 합니다. 트랜잭션이 데이터베이스의 데이터 덩어리를 수정하고 커밋한다면, 데이터베이스는 변경된 데이터를 유지하고 있을 것입니다. 만약 트랜잭션이 커밋된 후에 디스크에 데이터가 작성되기 직전 시스템에 오류가 생긴다면, 해당 데이터는 시스템이 복구된 후 디스크에 작성될 것입니다.

# Transaction 상태

1. Active\
Active 상태의 트랜잭션은 실행중입니다. 이는 모든 트랜잭션들의 초기 상태입니다.

2. Partially Committed\
트랜잭션이 마지막 작업을 실행한다면, 이는 partially committed 상태에 있다고 말합니다.

3. Failed\
데이터베이스 복구 시스템이 수행하는 확인이 실패한다면, 해당 트랜잭션은 failed 상태라고 말합니다. Failed 트랜잭션은 이후에도 진행되지 않습니다.

4. Aborted\
어떤 확인이 실패하여 트랜잭션이 failed 상태가 된다면, 해당 트랜잭션의 이전 트랜잭션 상태로 복구되어야 합니다. Recovery manager는 데이터베이스를 원래의 상태로 복구하기 위해 트랜잭션에서 수행된 모든 쓰기 작업들을 roll back 합니다. 데이터베이스 recovery module은 트랜잭션이 abort 된다면 아래 두가지 작업 중 하나를 선택할 수 있습니다.
    - transaction 재시작
    - transaction 죽이기

5. Committed\
트랜잭션이 모든 작업을 모두 성공적으로 수행한다면, 그 트랜잭션은 committed 상태라 불립니다. 해당 트랜잭션의 모든 결과는 이제 데이터베이스 시스템에 영구적으로 반영됩니다.

<br>
<br>
<br>

[[tutorialspoint] DBMS - Transaction](https://www.tutorialspoint.com/dbms/dbms_transaction.htm)