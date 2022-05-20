---
title: "[Happy House] 난중일지 2"
categories:
    - happy house
tags:
    - project
    - journal
permalink: /happyhouse/:title
---

<br>
<br>

# Happy House 프로젝트 난중일지 2022.05.20(금)

이제 제가 구현해보고 싶었던 매매 정보 변동 그래프를 위한 api 작업을 시작했습니다.

프런트에서 넘겨줄 인자들은 아래와 같습니다.

```
code : 지역 코드,
houseId: 아파트 id,
type: month 혹은 year,
fromYear: 범위 시작 년도,
toYear: 범위 끝 년도,
fromMonth: 범위 시작 월,
toMonth: 범위 끝 월
```

년도 혹은 월을 기준으로 통계를 낼 수 있고, type에 명시해주었습니다. code가 나타내는 지역구에 속해있는 house의 정보를 반환할 것입니다. (이 글을 작성하면서 보니 houseId를 기준으로 where를 거니 굳이 code가 필요 없어보입니다. Join 한번을 뺄 수 있을 것 같군요!)

반환해줄 데이터 예시는 아래와 같습니다.

```
{
  "dateRange": {
    "type": "month",
    "fromYear": 2022,
    "toYear": 2022,
    "fromMonth": 1,
    "toMonth": 5
  },
  "deals": [
    {
      "name": "인왕산아이파크",
      "date": 1,
      "avgPrice": 139900,
      "dealNumber": 1
    },
    {
      "name": "인왕산아이파크",
      "date": 2,
      "avgPrice": 135000,
      "dealNumber": 1
    },
    {
      "name": "인왕산아이파크",
      "date": 3,
      "avgPrice": 134500,
      "dealNumber": 2
    }
  ]
}
```

범위 내 평균 거래가, 거래 수를 db 레벨에서 뽑아내기 위해 aggregate를 사용했습니다. 그러나 jpql에서 group by를 사용하는 것은 너무나도 불편했습니다. jpql과 projection을 사용하거나, native 쿼리를 사용하여 처리해주는 방식이 있었습니다.

jpql과 직접 dto를 지정해주는 projection을 사용하여 repository 메서드를 만들어보았습니다. 어이없게도 평균을 구할 price 컬럼에 alias를 안걸어서 발생하는 NullPointerException을 한참이나 발견하지 못해 시간을 많이 썼습니다(이xx군의 도움으로 해결할 수 있었습니다. sthx). 또, 작동에는 문제가 없었지만 테스트 코드에서 자꾸 에러가 발생했는데 이는 ㅇ넘겨주는 인자들에 @Param() 어노테이션을 붙이지 않아서 발생하는 문제였습니다. @Query 어노테이션을 사용하여 만든 쿼리가 아니라 findByCodeStartingWith()와 같이 JpaRepository 메서드에서 사용할 인자들 모두 빈 value라도 가지는 @Param("")을 붙여주어야 테스트 코드가 정상적으로 작동했습니다. 별로라고 생각했습니다.

[https://github.com/spring-projects/spring-data-jpa/issues/2476](https://github.com/spring-projects/spring-data-jpa/issues/2476)

두 방법 모두 시도해보았지만 동적으로 group by 쿼리를 생성해야 하는 저의 조건에는 적합하지 않았고, 최종적으로 queryDsl을 사용하기로 했습니다.

pom.xml에 dependency와 plugin을 작성하고 mvn install을 진행했습니다. 정상적으로 target/generated-sources/java/...에 Q-Entity들이 생성되었지만, intellij는 이 Q-Entity들을 찾지 못했습니다.

Project Structure를 확인해보니 source folder에도 정상적으로 target/.../java가 등록되어있었습니다. 어이없게도 해당 디렉토리를 해제하고 다시 등록하니 정상 탐지했습니다;

querydsl을 사용하여 동적 쿼리를 즐겁게 생성했습니다(django가 갑자기 그리워졌습니다). 그런데 service에서 기존의 findAll, findByCodeStartingWith()과 같은 JpaRepository 메서드들을 사용하기 위해 JpaRepository를, querydsl을 사용하기 위한 repository 총 두개를 의존해야 했습니다. 너무나 못마땅했고 방법이 없나 확인해보니 다행이도 좋은 방법을 제시하고 있었습니다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/happyhouse/journal2/custom_repository.png)

위 이미지와 같이 JpaRepository interface와 queryDsl을 사용할 repositoryCustom interface를 모두 상속받는 repository interface를 만들고 이 repository만 의존하도록 하였습니다.