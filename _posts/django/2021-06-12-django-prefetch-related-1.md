---
title: "Django prefetch_related(1)"
categories:
  - Django
tags:
  - Django
  - query optimization
  - ORM
toc: true
permalink: /django/:title
---
<br>

# 앞서...

열심히 Django document를 보며 prefetch_related를 공부했습니다.

사실 장고는 [document](https://docs.djangoproject.com/en/3.2/ref/models/querysets/#prefetch-related)가 굉장히 잘 정리되어있는
편이라 document만 잘 정독해도 원하는 거의 대부분의 내용을 얻을 수 있습니다.

저는 제가 공부한 prefetch_related의 내용을 정리하고자 포스팅을 했습니다.

**정확하지 않은 내용이 있을 수 있습니다!!**

틀린 부분이 있다면 꼭 댓글로 남겨주시길 부탁드리겠습니다!

감사합니다!
<br>
<br>
<br>
<br>

# Prefetch related

prefetch_related()는 select_related()와 마찬가지로 N + 1 problem을 해결하기 위해 사용되는 django ORM 메서드입니다.

select_related와 목적은 같지만 사용되어야 하는 상황이나 컨셉은 전혀 다릅니다.

<br>
<br>
<br>

# prefetch_related() vs select_related()

select_related()는 INNER JOIN이었음을 확인할 수 있었습니다.

그렇다면 prefetch_related()는 어떻게 동작하는 것일까요?

prefetch_related 또한 select_related와 같이 eager loading을 하게 됩니다.

prefetch_related로 지정한 관계된 테이블의 데이터를 eager loading 하는 것이죠.

메인으로 가져오는 테이블의 쿼리문 한개, 그리고 역참조, 혹은 다대다 관계를 가지고 있는 테이블의 데이터를 가져오는 쿼리 한개, 총 두개의 쿼리를 사용합니다.

select_related는 일대일 관계(One to One), 정참조 관계에서 사용됩니다.

그에 반해 prefetch_related는 역참조 관계, 다대다 관계(Many to Many)에서 사용됩니다.

<br>
<br>
<br>

# Simple example - 역참조

```
class Chef(models.Model):
    name = models.CharField()
    grade = models.IntegerField()
    store = models.ForeignKey(Store)

    class Meta:
        db_table = 'chef'


class Store(models.Model):
    name = models.CharField()
    address = models.CharField()

    class Meta:
        db_table = 'store'
```

위와 같이 모델링이 되어있을때 Chef 👉 Store는 정참조 관계, Store 👉 Chef는 역참조 관계에 있는 것을 알 수가 있습니다.

이때, 성동구 혹은 마포구에 위치한 가게를 불러오되, 그 가게들에서 일하고 있는 요리사들을 earger loading 하고 싶다면 아래와 같이 작성해주시면 됩니다.

```
Store.objects.filter(address__in=['성동구', '마포구']).prefetch_related('chef_set')
```
<br>
<br>
<br>

# _set

음... 이상한 부분이 하나 있군요?

prefetch_related()에 인자로 넘겨준 string을 보면 'chef'가 아닌 'chef_set'으로 되어있습니다.

역참조 관계에 있는 데이터를 불러올 때 django는 자동으로 `{역참조_관계의_클래스명.lower()}`를 사용하도록 합니다.

만약 클래스명이 Chef가 아닌 Cook 이라면 'cook_set'으로 인자를 넘겨주면 되겠습니다.

_set 이 아닌 직접 역참조 관계를 불러올 이름을 설정하고 싶으면 ForeignKey() 필드에 인자로 `related_name = chefs`을 넘겨주면 됩니다.

이럴 경우에는 `Store.objects.prefetch_related('chefs')`로 사용하면 됩니다.

위 경우에서는 Store에 대한 데이터를 불러오는 쿼리 한개, 그리고 관계된 Chef에 대한 데이터를 불러오는 쿼리 한개, 총 두개의 쿼리를 사용하게 됩니다.

* TMI! `related_name = '+'`와 같이 +를 related_name의 인자로 넘겨줄 경우 역참조를 허용하지 않겠다는 의미가 됩니다.
  `store = models.ForeignKey(Store, related_name='+')`로 설정한 경우 `Store.objects.prefetch_related('chef')`를 사용할 시 다음과 같은
  에러 메세지가 나타납니다.
  > AttributeError: Cannot find 'chef' on Store object, 'chef' is an invalid parameter to prefetch_related()

<br>
<br>
<br>

# 어라...? 쿼리를 더 많이 쓰나요?

맞습니다. prefetch_related를 사용하게 되면 무조건 쿼리를 최소한 한개 추가로 사용하게 됩니다

한개 이상인 이유는 prefetch_related() 내에 넘겨주는 인자로 N개의 관계 테이블명을 넘겨주면 N개의 쿼리를 추가로 발생하기 때문입니다.

그렇다면 prefetch_related는 결국 실행하는 쿼리 수를 늘리는 것일까요?

상황에 따라 다릅니다.

prefetch_related를 적절히 사용하지 않으면 오히려 불필요하게 쿼리 수를 늘릴 수도 있습니다.