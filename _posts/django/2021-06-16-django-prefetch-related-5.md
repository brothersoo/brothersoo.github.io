---
title: "Django prefetch_related(5)"
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

# prefetch_related 안전하게 사용하기

앞선 포스트에서 설명했듯이 prefetch_related는 꼼꼼히 살펴보고 사용해야 합니다.

'Prefetch 했으니 쿼리가 엄청 줄었겠지?'라고 생각했지만 실제로는 전혀 쿼리 최적화가 되지 않고 있을 수도 있습니다.\
(Test code에 쿼리 수 체크 코드도 추가해보세요!)

그렇다면 확실하게 prefetch한 데이터만 사용할 수 있는 방법은 없을까요?
<br>
<br>
<br>
<br>

# to_attr

django.db.models.query의 Prefetch의 인자로 to_attr를 넘겨줄 수 있습니다.

```
from django.db.models.query import Prefetch


queryset = Store.objects.prefetch_related(
  Prefetch(
    'menu_set',
    queryset=Menu.objects.all(),
    to_attr='menus'
  )
)
```

위 예시와 같이 to_attr='menus'를 넘겨준 것과 넘겨주지 않은 것의 차이는 무엇일까요?

to_attr parameter를 넘겨주지 않고 prefetch 된 데이터를 사용하려면 일반적인 역참조, 혹은 다대다 관계를 불러오듯이 related_name을 사용하면 됩니다.

<br>

<details>
<summary>[TMI!    parameter vs argument]</summary>
<div markdown="1">

> parameter는 함수 혹은 클래스로 넘겨주는 인자의 변수명이고 argument는 넘겨주는 데이터를 뜻합니다.
> ex)
> ```
> def some_function(*, param1, param2):
>  print(param1, param2)
>
> arg1, arg2 = 1, 2
> some_function(param1=arg1, param2=arg2)
> ```
> 여기서 param1과 param2는 parameter, arg1, arg2는 argument입니다.
  <details>
  <summary>[TMI in TMI!    '*' in parameters]</summary>
  <div markdown="1">

  > python3 부터 정의한 parameter들 중 * 이후에 쓰인 parameter들은 함수 호출시 반드시 parameter 이름을 표기해주어야 합니다.>>
  > ```
  > some_function(param1=arg1, param2=arg2)  # OK!
  > some_function(arg1, arg2)                # NG!
  > ```
  </div>
  </details>

</div>
</details>

<br>

그러나 to_attr를 사용하게 되면 넘겨준 argument를 이름으로 가지는 새로운 list type attribute가 queryset 내의 각 객체에 추가됩니다.

위 예시와 같은 경우라면 `Store.objects.first().menus`를 호출할 수 있게 되는 것입니다.

실행 결과는 'list of 첫번째 Store가 가지고 있는 메뉴들'이 됩니다.

```
queryset = Store.objects.prefetch_related(
  Prefetch(
    'menu_set',
    queryset=Menu.objects.all(),
    to_attr='menus'
  )
)

print(queryset.first().menus)        # [<Menu: Menu object> (1), <Menu: Menu object> (2), <Menu: Menu object> (3)...]
print(type(queryset.first().menus))  # <class 'list'>
```
<br>
<br>

## 주의할 점

to_attr를 사용하면 prefetch한 데이터가 list type으로 저장됩니다.

즉, django queryset 메서드를 전혀 사용할 수가 없다는 뜻입니다.

예를 들어 `queryset.first().menus.order_by('price')` 혹은 `queryset.first().menus.filter(price__gt=20000)`와 같은 ordering, filtering
등을 사용할 수 없습니다.

위와 같이 ordering 혹은 filtering, 등을 적용하기 위해서는 queryset의 Prefetch문 내에서 먼저 적용해야 합니다.

그렇기에 확실하게 prefetch한 데이터만을 사용할 수 있기도 합니다!