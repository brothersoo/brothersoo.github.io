---
title: "[Happy House] 난중일지 1"
categories:
    - happy house
tags:
    - project
    - journal
permalink: /happyhouse/:title
---

<br>
<br>

# Happy House 프로젝트 난중일지 2022.05.19(목)

학우분과 2인 1조로 Happy House 프로젝트를 시작했습니다. 프로젝트는 5월 26일까지 약 8일간 진행될 것입니다. 두명 모두 백엔드에 뜻이 있어 프런트 엔드는 BootStrapVue Argon Dashboard 템플릿을 클론해 진행하기로 했습니다.

저는 이전부터 해보고 싶었던 기간별 그래프 작업을 담당하기로 했고, 팀원분은 카카오 지도 api 작업을 담당하기로 했습니다.

## isNew() with String type pk

기존에 사용하던 sql mapper Mybatis에서 JPA로 수정하는 작업을 우선 진행했습니다. 첫번째로 마주한 문제는 pk가 String 타입(name 컬럼)인 Upmyundong(읍면동) 테이블에서 발생했습니다. 대강 아래와 같은 테스트 코드를 작성했는데 테스트가 실패했습니다.

```java
Upmyundong umd = Upmyundong.builder().name("사당동").build();
Upmyundong persisted = repository.save(umd);
assertThat(persisted).isEqualTo(umd);
```

분명 entity Manager가 umd를 영속화 하고, save 함수는 이 영속화된 엔티티를 반환하리라 예상하여 당연히 성공할 줄 알았던 테스트입니다.

그러나 테스트는 실패했고 왜 그런가 코드를 살펴보니 save() 함수는 isNew() 메서드를 사용하여 이 엔티티가 새로운 엔티티인지 확인하고 새로운 엔티티라면 persist를, 아니라면 merge를 수행하고 있었습니다.

```java
@Transactional
@Override
public <S extends T> S save(S entity) {

  Assert.notNull(entity, "Entity must not be null.");

  if (entityInformation.isNew(entity)) {
    em.persist(entity);
    return entity;
  } else {
    return em.merge(entity);
  }
}
```

`AbstractEntityInformation` 추상 클래스의 isNew()를 확인해보았습니다.

```java
public boolean isNew(T entity) {

  ID id = getId(entity);
  Class<ID> idType = getIdType();

  if (!idType.isPrimitive()) {
    return id == null;
  }

  if (id instanceof Number) {
    return ((Number) id).longValue() == 0L;
  }

  throw new IllegalArgumentException(String.format("Unsupported primitive id type %s!", idType));
}
```

pk가 primitive type이 아니라면 pk 값이 null이라면 true, 아니라면 false를 반환하는 것을 확인했습니다.

읍면동을 추가할 일은 없을 것 같지만 만약 추가한다해도 pk인 읍면동 이름을 auto increment처리할 수는 없고 직접 지정해줄 텐데, 그렇다면 pk가 null인 상황은 없을 것이고, 항상 isNew()는 false를 반환하게 될 것입니다.

[https://jaime-note.tistory.com/65](https://jaime-note.tistory.com/65)

이런 경우에는 엔티티가 `Persistable<T>`을 상속받도록 하고 isNew() 메서드를 직접 정의해주면 된다고 합니다. Upmyundong 엔티티도 Persistable를 상속받아 isNew()를 재정의 해주면 좋겠지만 아쉽게도 현재 Upmyundong 엔티티는 name과 시구군 테이블의 외래키인 sigugun_id만 가지고 있어 마땅히 new 인지 판별할 수 있는 속성이 없었습니다.

그냥 새로운 long 타입 pk `id` 컬럼을 만들었습니다! Auto-increment로 지정하여 문제를 해결했습니다. Sido, Sigugun 테이블 모두 long type pk를 부여해주었습니다.

사실 수정하지 않아도 실행은 정상적으로 될 수 있지만, save()에서 persist가 아닌 merge가 사용될 것이고, 단순 insert문이 아닌 select-insert문이 사용될 것입니다. 읍면동을 추가할 것 같지는 않고, 만약 추가한다고 해도 매우 적은 양만 추가하므로 크게 상관은 없을 수 있지만, 대용량 batch-create 하는 다른 상황에서는 성능상 큰 문제를 일으킬 수 있습니다! 또 만약 서비스에서 save()가 반환하는 엔티티가 아닌 인자로 넘긴 엔티티를 이후 로직에 사용한다면 영속화 되지 않은 엔티티를 사용하게 되어 많은 문제를 일으킬 수 있을 것으로 보입니다.

`LOAD DATA INFILE`을 통해 지역구 데이터를 넣어주는데, 이때 필요한 파일을 생성하는 알고리즘을 다시 작성하였습니다.

지역, 아파트, 매매 정보 관련 최종 ERD는 아래와 같습니다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/happyhouse/journal1/erd.png)