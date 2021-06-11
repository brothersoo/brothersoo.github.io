---
title: "Django select_related"
categories:
  - Django
tags:
  - Django
  - query optimization
  - ORM
toc: true
---

## Select related

select_related()는 query evaluation 수를 줄이기 위해 사용되는 ORM 메서드입니다.


## N + 1 problem

N + 1 problem은 한개의 object를 불러오는 쿼리를 사용했음에도 그 한개의 object와 관계를 가지고 있는 N개의 다른 object를 불러오기 위해
N 개의 추가 쿼리를 evaluate 해야 하는 문제입니다.

## N + 1 problem 예시

아래와 같이 Store을 foreign key로 가지고 Chef와 일대일 관계를 맺는 Menu 모델이 정의되어 있습니다.

한개의 Store은 한개 이상의 Menu를 가지고 있을 수 있습니다.

```
class Store(models.Model):
    name = models.CharField()
    brand = models.ForeignKey(Brand, on_delete=models.CASCADE)

    class Meta:
        db_table = 'store'


class Menu(models.Model):
    name = models.CharField()
    store = models.ForeignKey(Store, on_delete=models.CASCADE)
    chef = models.OneToOneField(Chef, on_delete=models.CASCADE)

    class Meta:
        db_table = 'menu'


class Chef(models.Model):
    name = models.CharField()
    grade = models.IntegerField()
    major = models.CharField()

    class Meta:
        db_table = 'chef'

class Brand(models.Model):
    name = models.CharField()
```

이때 만약 id가 1인 Menu에 대한 데이터와, 그 메뉴가 속해있는 Store에 대한 정보를 모두 알고 싶다면 두개의 ORM이 필요합니다.

1. `menu = Menu.objects.get(id=1)`
2. `menu_store = first_menu.store`

장고 shell을 켜고 위 ORM들을 실행한다면 실제 실행된 두개의 쿼리문을 콘솔에서 확인할 수 있습니다.

1. \
```
SELECT
    "menu"."id",
    "menu"."name",
    "menu"."store_id"
FROM
    "menu"
```
2. \
```
SELECT
    "store"."id",
    "store"."name"
FROM
    "store"
WHERE
    "store"."id" = 1
```

위 상황에서는 Menu와 관계된 N개(한개)의 Store에 대한 데이터를 가져오기 위해 N(한개) + 1개의 쿼리가 실행되었습니다.

만약 여기에 Menu에 Store 뿐만이 아닌 Chef와 같이 다른 Foreign Key의 데이터를 불러오기 위해서는 그 수만큼의 쿼리를 추가로 실행해야 합니다.

## select_related() 사용법

select_related()는 **"한번에 가져오기"**입니다. 어떻게 한번에 가져올까요?

위 예시에서는 menu에 대한 데이터와 그 메뉴와 관계된 store에 대한 데이터를 두개의 쿼리로 각자 가져왔습니다.

store를 select_related에 포함한 사용법, 그리고 그에 해당하는 쿼리문은 아래와 같습니다.

```
# ORM
first_menu = Menu.objects.select_related('store').get(id=1)

# SQL
SELECT
    "menu"."id",
    "menu"."name",
    "menu"."store_id",
    "store"."id",
    "store"."name"
FROM
    "menu"
INNER JOIN
    "store"
ON
    "menu"."store_id" = "store"."id"
```

음... select_related()의 정체는 INNER JOIN이었습니다!

INNER JOIN을 통해 관계된 store의 데이터를 한번에 가져오니 쿼리를 한번만 실행해도 됩니다.

`first_menu.store.name`, `first_menu.store.id`와 같은 코드를 실행해도 추가적으로 쿼리를 실행하지 않습니다.

INNER JOIN을 여러 테이블과 하고 싶다면 `Menu.objects.select_related('store', 'chef')`과 같이 여러개의 필드 이름을 넣어주면 됩니다.

## select_related()를 사용해야 하는 경우

어떤 object에 관계된 다른 object의 데이터까지 불러와야 하는 상황이라면 **반드시**, **항상** 사용하세요!

쿼리를 줄여줄 수 있는 상황에서 select_related()를 사용하지 않을 이유가 없습니다.

물론 관계된 데이터를 사용할 필요가 없는 상황에서는 select_related()를 사용하지 않는것이 좋습니다. 굳이 불필요한 INNER JOIN을 추가할 필요는 없습니다!

예를 들어 메뉴의 디테일을 불러오는 api에 요리사 정보는 함께 반환해야 하지만 매장 정보는 반환하지 않아도 되는 경우에는
`Menu.objects.select_related('chef')` 만 하시면 됩니다.

## select_related()를 사용할 수 있는 경우

select_related()는 정참조, 혹은 OneToOne 관계에서만 사용이 가능합니다.

너무나 당연한 것이 select_related는 INNER JOIN이니, 역참조 혹은 ManyToMany 관계는 사용할 수 없는 것입니다.

역참조 상황을 한번 보겠습니다.

`Store.objects.select_related('menu').get(id=1)`

위와 같이 역참조 관계에서 select_related를 사용한다면 아래와 같은 에러 메세지가 나타납니다.

`django.core.exceptions.FieldError: Invalid field name(s) given in select_related: 'menu'. Choices are: brand`

Store 모델 클래스에 정의되어있는 ForeignKey 필드는 brand 뿐이니 당연하게도 menu를 찾지 못합니다.

이러한 역참조 관계, 혹은 ManyToMany 관계에서 쿼리를 줄이기 위해 존재하는 것이 prefetch_related() 입니다.