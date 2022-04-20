---
title: "Index(2) Clustered index & Non-clustered index"
categories:
    - database
tags:
    - cs
    - database
    - computer structure
permalink: /database/:title
---

<br>
<br>

# Index

> 인덱스는 데이터베이스 분야에 있어서 테이블에 대한 동작의 속도를 높여주는 자료구조를 일컫는다.

인덱스는 추가적인 쓰기 작업과 저장 공간을 활용하여 데이터베이스 테이블의 검색 속도를 향상시키기 위해 존재합니다.

## Index의 장점
  - 테이블 조회, 수정, 삭제 속도를 향상시킬 수 있습니다.

## Index의 단점
  - 인덱스를 관리하기 위해 전체 테이블 크기의 10% 정도의 추가 공간을 소모합니다.
  - 인덱스를 관리하기 위해 추가 연산이 소요됩니다.
  - 인덱스를 잘못 생성할 경우, 공간만 차지하고 사용하지 않거나, 오히려 성능이 저하될 수 있습니다.

삭제 시에는 해당 row의 인덱스를 사용하지 않음 처리하고, 수정 시에는 해당 row의 인덱스를 사용하지 않음 처리 후, 새로운 인덱스를 생성합니다. 결국 쌓이는 인덱스의 양이 커져 사용되지 않는 인덱스는 바로 제거해 주는 것이 좋습니다.

# Clustered Index

각 InnoDB 테이블은 clustered index라는 row 데이터를 저장하는 특별한 index를 가집니다. 보통, clustered index는 primary key와 동일합니다.

테이블은 Clustered index를 기준으로 물리적으로 정렬되어있습니다. 이는 테이블 당 단 하나의 clustered index만 존재할 수 있음을 의미합니다.

Clustered index는 create table문에 설정한 primary key으로 자동 생성됩니다.\
그렇다면 pk가 지정되지 않았다면 어떨까요?\
해당 경우에는 not null이면서 unique한 column이 있다면 해당 column으로 생성됩니다.\
그렇다면 not null 옵션의 unique column마저도 없다면 인덱스가 없을까요?\
이런 경우에는 hidden clustered index로 생성됩니다.

## Hidden clustered index

InnoDB는 Clustered index를 생성하기에 마땅한 column이 없다면 `GEN_CLUST_INDEX`이라는 이름의 hidden clustered index를 생섭합니다. 모든 row는 InnoDB가 할당하는 row ID를 가지고 있고 이로 정렬되어있습니다. 이 row를 사용하여 index를 생성하게 됩니다. 이 row ID는 삽입 순서에 따라 물리적으로 정렬되어 있습니다.

## Clustered index로 성능 높이기

Clustered index는 B+Tree 자료구조를 사용하여 row data를 가지고 있는 페이지로 바로 이동할 수 있기 때문에 탐색 속도가 매우 빠릅니다.

# Secondary Index (Non clustered index)

InnoDB에서 secondary index의 각 record는 관리자가 secondary index로 명시한 column 뿐만이 아니라 primary key 또한 가지고 있습니다.

Secondary index는 데이터 페이지는 그대로 둔 채 별도의 페이지에서 인덱스를 구성합니다. 보조 인덱스의 인덱스 자체는 실제 데이터가 아닌 데이터가 존재한는 주소값입니다. 검색을 할 경우 클러스터형 인덱스와는 달리 트리를 복잡하게 순회해야 하므로 속도가 비교적 느리지만, 데이터의 삽입, 수정, 삭제는 비교적 빠릅니다. 실제 데이터가 보조 인덱스를 기준으로 정렬되어있는 것이 아니므로 여러개 생성할 수 있습니다.

# Composite index

MySQL은 최대 16개의 column을 엮어 multiple-column index를 생성할 수 있습니다.

MySQL은 multiple-column index의 모든 column들을 사용하거나, 첫번째 column만 사용하거나, 첫 두개의 column만 사용하거나, 첫 세개의 column들만 사용하거나... 할 수 있습니다.

```sql
CREATE TABLE `procuct`(
    `product_id` INT,
    `name` VARCHAR(45),
    `price` INT,
    `type` VARCHAR(45),
    `vendor_id` INT,
    PRIMARY KEY (`product_id`),
)
```

위와 같은 테이블이 있고 name, price, type를 엮은 composite index를 생성할 쿼리문은 아래와 같습니다.

```sql
CREATE INDEX `idx_product_name_price_type` ON `product` (`name`, `price`, `type`);
```

Composite index에 포함된 column을 `WHERE`절에 사용한다고 무조건 index lookup을 하지 않습니다.

반드시 최 좌측에 명시한 컬럼이 포함되어야 index lookup이 진행됩니다.

index lookup이 실행되는 경우 예시입니다.
```sql
SELECT * FROM Product WHERE name = "Joseph";
SELECT * FROM Product WHERE name = "Joseph" and vendor_id = 1;
SELECT * FROM Product WHERE name = "Joseph" and (name = "John" or vendor_id = 1);
```

index lookup이 실행되지 않는 경우 예시입니다.
```sql
SELECT * FROM Product WHERE price = 10000;
SELECT * FROM Product WHERE price = 10000 or name = "Joseph" and type = "FOOD";
```