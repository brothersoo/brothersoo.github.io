---
title: "Isolation level"
categories:
    - database
tags:
    - cs
    - database
permalink: /database/:title
---

[MySQL 8.0 Reference Manual - 15.7.2.1 Transaction Isolation Levels](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html) 번역

트랜잭션 isolation은 데이터베이스 처리의 기초 중 하나입니다. Isolation은 두문자어 ACID의 I에 해당합니다; isolation level은, 동시에 여러 트랜잭션들이 변화를 일으키고 쿼리를 실행할 때, 그 결과의 신뢰성, 일관성, 그리고 재현성이 적절한 밸런스를 유지하도록 설정합니다.

InnoDB는 SQL:1992 표준에서 설명된 네개의 트랜잭션 고립 레벨들을 모두 지원합니다: `READ UNCOMMITTED`, `READ COMMITTED`, `REPEATABLE READ`, `SERIALIZABLE`. InnoDB의 기본 고립 레벨은 `REPEATABLE READ`입니다.

사용자는 단일 세션 혹은 모든 후속 connections의 고립 레벨을 `SET TRANSACTION` 명령을 통해 변경할 수 있습니다. 모든 connection들에 적용될 서버의 기본 고립 레벨을 설정하기 위해서는 `--transaction-isolation` 옵션을 command line 혹은 option file에 적용하세요.

InnoDB는 다른 locking 전략을 사용하여 설명될 각각의 트랜잭션 isolation level을 지원합니다. ACID 규정 준수를 요하는 중요한 데이터를 다루는 연산들을 위해 default `REPEATABLE READ` 레벨을 사용하여 높은 수준의 일관성을 강제할 수 있습니다. 혹은, bulk reporting과 같이 정확한 일관성과 반복 수행이 locking으로 인한 overhead 감소보다 중요하지 않은 경우에는, `READ COMMITTED`, 심지어 `READ UNCOMMITTED`를 사용하여 일관성 규칙을 완화할 수 있습니다. `SERIALIZABLE`은 `REPEATABLE READ`보다 엄격한 규율을 가지고, XA transaction, 그리고 일관성과 deadlock에 발생하는 문제 troubleshooting과 같은 특수한 경우에 주로 사용됩니다.

아래의 리스트는 MySQL이 각 transaction level을 어떻게 지원하는지 설명합니다. 리스트는 가장 빈번히 사용되는 level에서 가장 적게 사용되는 level 순으로 나열됩니다.

## REPEATABLE READ  (level 2)

REPEATABLE READ는 InnoDB의 기본 격리 수준입니다. 같은 transaction 내의 consistent reads는 첫번째 read이 발행한 snapshot을 읽습니다. 이는, 동일한 트랜잭션 내에서 여러 순수(nonlocking) `SELECT`문을 사용한다면, 이 `SELECT`문들은 서로에 대해 일관성을 가짐을 의미합니다.

Locking reads(`SELECT` `FOR UPDATE`혹은 `FOR SHARE`), `UPDATE, `DELETE`문에서 locking은 unique 검색 조건에서 unique index를 사용하는지, 아니면 범위 타입 검색 조건을 사용하는지에 따라 다릅니다.

* Unique 검색 조건에서의 unique index는 InnoDB가 탐색된 index record만 lock을 걸고 이전의 gap에는 걸지 않습니다.
* 이외의 검색 조건에서 InnoDB는, 타 세션이 스캔된 범위 내의 gap에 데이터를 삽입하지 못하도록 gap locks 혹은 next-key locks를 사용하여 스캔된 index 범위를 lock합니다.

## READ COMMITTED (level 1)

모든 consistent read는, 심지어 동일 트랜잭션 내에서도, 고유의 새로운 snapshot을 설정하고 읽습니다.

Locking reads(`SELECT` `FOR UPDATE`혹은 `FOR SHARE`), `UPDATE, `DELETE`문에서 InnoDB는 index records에만 lock을 걸고 이전의 gap들에는 걸지 않습니다. Lock된 records 옆에 새로운 삽입을 자유롭게 허용합니다. Gap locking은 외래키 제약 확인과 중복 키 확인에만 사용됩니다.

Gap locking이 사용되지 않기 때문에 다른 세션들이 gap에 새로운 row를 삽입할 수 있고, 이로 인해 phantom row가 발생할 수 있습니다.

Row-based binary logging만이 `READ COMMITTED` 격리 수준에서 지원됩니다. `binlog_format=MIXED`와 함께 `READ COMMITTED`를 사용한다면 서버는 자동으로 row-based logging을 사용합니다.

`READ COMMITTED`의 사용은 몇가지 추가 효과가 있습니다.

* `UPDATE` 혹은 `DELETE`문에서 InnoDB는 실제 수정되거나 삭제되는 row에만 lock을 겁니다. 일치하지 않는 row들의 record lock는 MySQL이 WHERE 절을 시행한 이후 해제됩니다. 이는 deadlock 발생 가능성을 크게 감소시키지만, 완전히 발생하지 않는 것은 아닙니다.
* `UPDATE`문에서 해당 row가 이미 lock이 걸려있다면, InnoDB는 "semi-consistent" read를 사용합니다. 

아래와 같은 테이블을 초기값으로 가지는 예제를 확인해보겠습니다.

```sql
CREATE TABLE t (a INT NOT NULL, b INT) ENGINE = InnoDB;
INSERT INTO t VALUES (1,2),(2,3),(3,2),(4,3),(5,2);
COMMIT;
```

이 경우에 테이블은 index를 가지고 있지 않기 때문에, 검색과 index scan은 hidden clustered index를 record locking에 사용합니다.

한 세션이 아래의 `UPDATE`문을 사용했다고 하겠습니다.
```sql
# Session A
START TRANSACTION;
UPDATE t SET b = 5 WHERE b = 3;
```
또 다른 세션이 위 세션의 실행 직후 아래의 `UPDATE`문을 실행했다고 하겠습니다.
```sql
# Session B
UPDATE t SET b = 4 WHERE b = 2;
```

InnoDB는 `UPDATE`문을 수행할 때 각 row에 exclusive lock을 걸은 후에, 수정할 지 않을지 결정합니다. 만약 수정하지 않을 경우, lock을 해제합니다. 수정할 경우에 InnoDB는 트랜잭션의 종료까지 lock을 유지합니다. 이는 아래와 같은 트랜잭션 처리를 일으킵니다.

Default `REPEATABLE READ` 격리 수준을 사용한면, 첫번째 `UPDATE`는 읽는 모든 row에 x-lock을 걸고 풀지 않습니다.

```
x-lock(1,2); x-lock 유지
x-lock(2,3); (2,3)에서 (2,5)로 수정; x-lock 유지
x-lock(3,2); retain x-lock
x-lock(4,3); update(4,3)에서 (4,5)로 수정; x-lock 유지
x-lock(5,2); x-lock 유지
```
두번째 `UPDATE`는 lock을 걸려고 하는 순간 block되고(첫번째 update가 모든 row의 lock을 유지했기 때문입니다.), 첫번째 `UPDATE`가 commit 혹은 roll back하기 전까지는 진행되지 않습니다.

만약 `READ COMMITTED` 격리 수준이 사용된다면, 첫번째 `UPDATE`는 읽은 모든 row에 x-lock을 걸고, 수정하지 않는 row라면 lock을 해제할 것입니다.

```
x-lock(1,2); (1,2) 해제
x-lock(2,3); (2,3)에서 (2,5)로 수정; x-lock 유지
x-lock(3,2); (3,2) 해제
x-lock(4,3); (4,3)에서 (4,5)로 수정; x-lock 유지
x-lock(5,2); (5,2) 해제
```

두번째 `UPDATE`에서 InnoDB는 "semi-consistent" read를 수행합니다. 각 row의 마지막 committed 버전을 읽어 MySQL이 해당 row가 `UPDATE`의 WHERE 절에 일치하는지 검사할 수 있습니다.

```
x-lock(1,2); update(1,2) to (1,4); retain x-lock
x-lock(2,3); unlock(2,3)
x-lock(3,2); update(3,2) to (3,4); retain x-lock
x-lock(4,3); unlock(4,3)
x-lock(5,2); update(5,2) to (5,4); retain x-lock
```

만약, WHERE 절에 indexed column이 포함되고 InnoDB가 해당 index를 사용한다면, indexed column만 유일하게 record lock을 걸고 유지하는데 사용될 것입니다. 아래의 예제에서, 첫번째 `UPDATE`는 b = 2인 row들에 x-lock을 걸고 유지합니다. 두번째 `UPDATE`또한 b에 걸린 index를 사용하기 때문에, 동일한 records에 x-lock을 걸려고 하는 순간 block될 것입니다.

```sql
CREATE TABLE t (a INT NOT NULL, b INT, c INT, INDEX (b)) ENGINE = InnoDB;
INSERT INTO t VALUES (1,2,3),(2,2,4);
COMMIT;

# Session A
START TRANSACTION;
UPDATE t SET b = 3 WHERE b = 2 AND c = 3;

# Session B
UPDATE t SET b = 4 WHERE b = 2 AND c = 4;
```

`READ COMMITTED` 격리 수준은 초기 시작, 혹은 런타임에 적용될 수 있습니다. 런타임에서는, 모든 세션을 대상으로 전범위 설정될 수 있고, 개별 세션당 걸릴 수도 있습니다.

## READ UNCOMMITTED

`SELECT`문은 nonlocking 방식으로 실행됩니다. 하지만 가능한 이전 버전의 row가 사용될 수도 있습니다. 게다가 이 격리 수준을 사용한다면 read의 결과가 일관성 있지 않을 수 있습니다. 이런 현상을 dirty read라 합니다. 이외 부분에서는 `READ COMMITTED` 격리 수준과 동일하게 작동합니다.

## SERIALIZABLE

이 레벨은 `REPEATABLE READ`와 동일하지만, InnoDB가 모든 일반 `SELECT`문들을 `SELECT FOR SHARE`으로 명시적으로 변환합니다(`autocommit`이 비활성화 되어있다면). 만약 `autocommit`이 활성화 되어있다면, `SELECT`이 고유한 트랜잭션이 됩니다. 이는 결국 읽기 전용이 되고, consistent(nonlocking) read로써 실행된다면 serialized 가능하고, 다른 트랜잭션들을 block할 이유가 없다는 뜻입니다. (일반 `SELECT`가, 다른 트랜잭션이 선택된 row를 수정하려고 할 때 강제로 block 시키기를 원한다면 `autocommit`을 해제하세요.)



###### snapshot: 다른 트랜잭션에 의해 수정사항이 commit 되었더라도 동일하게 유지되는 특정 시간의 데이터 표현입니다.

## 요약

### isolation level이란?
isolation level은, 동시에 여러 트랜잭션들이 변화를 일으키고 쿼리를 실행할 때, 그 결과의 신뢰성, 일관성, 그리고 재현성이 적절한 밸런스를 유지하도록 설정합니다.

### Isolation level의 종류

#### READ UNCOMMITTED (Level 0)
  - locking이 사용되지 않습니다. 타 트랜잭션이 삽입, 삭제, 수정하는 것을 막을 수 없습니다.
  - dirty read, non repeatable read, phantom read 모두 발생 가능합니다.

#### READ COMMITTED (Level 1)
  - read 시 해당 시점의 snapshot을 생성하고 읽기 때문에 commit 완료된 데이터만 읽어올 수 있습니다.
  - update시 실제 where절에 해당하는 row에 exclusive lock이 걸립니다.
  - read 시 매번 새로운 snapshot을 참조하므로 non repeatable read가 발생 가능합니다.
  - index record lock만 걸고 gap lock은 걸지 않으므로 insert가 자유롭게 가능합니다.
  - insert가 자유로우므로 phantom read도 발생 가능합니다.

#### REPEATABLE READ (Level 2)
  - 첫번째 read의 snapshot을 동일 트랜잭션 내 다른 select에서 모두 사용합니다. 다른 트랜잭션이 update를 하더라도 이전에 찍어 두었던 snapshot을 참고하므로 non repeatable read를 방지합니다.
  - unique index를 사용한 검색에서 조건에 일치하는 row에는 lock을 걸지만 gap에는 걸지 않으므로 phantom read가 발생 가능합니다.
  - 이외 경우에는 gap lock도 걸어서 phantom read를 막습니다.

#### SERIALIZABLE (Level 3)
  - 모든 select 문을 select for share로 변경하여 수행합니다. 모든 select문 자체가 하나의 트랜잭션으로써 사용됩니다.

### Isolation level에 따라 발생할 수 있는 정합성 문제들
  - dirty read
	- 어떤 트랜잭션이 한 row를 수정한 후 다른 트랜잭션이 수정된 데이터를 읽어갔습니다. 그런데 첫번째 수정하는 트랜잭션에 오류가 생겨 rollback 되면 이후 트랜잭션은 실제 적용되지 않은 데이터를 읽은 것입니다.
  - Non repeatable read
    - Non repeatable read는 첫번째 select 문과 이후 select 문의 결과 row 데이터 값이 다른 경우입니다.
  - Phantom read
    - Phantom read는 말 그대로 이전 탐색에는 없던 row가 생성되어 이후 탐색에는 나타나는 경우입니다.