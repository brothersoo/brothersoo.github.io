---
title: "Python: Mutable and Immutable objects"
categories:
  - Python
tags:
  - Python
permalink: /python/:title
toc: true
---

<br>
<br>

> Mutable과 Immutable에 대해 다시 정리해 보는 도중, 좋은 포스트를 찾아 참고하여 정리하였습니다.\
> [Python Basics: Mutable vs Immutable Objects](https://towardsdatascience.com/https-towardsdatascience-com-python-basics-mutable-vs-immutable-objects-829a0cb1530a)

<br>
<br>

# Mutable, Immutable

Mutable과 Immutable의 개념은 거의 대부분의 언어에서 사용되고 있습니다. 간단히 설명하자면 Mutable object는 값이 변경될 수 있는 상태이고, immutable object는 값이 변경될 수 없는 상태입니다.
<br>
<br>
<br>

# Immutable object in python

파이썬에서 모든 빌트인 타입들과, 몇개의 특수한 containers들이 immutable 합니다. 특수한 컨테이너는 대표적으로 `tuple`, `frozen set` 등이 있습니다.

빌트인 타입은 `int`, `float`, `boolean`, `str` 등 여러가지가 있습니다. Built-in 타입의 자세한 내용은 아래 링크에서 확인하실 수 있습니다.\
[https://docs.python.org/3/library/stdtypes.html#built-in-types](https://docs.python.org/3/library/stdtypes.html#built-in-types)

Immutable 하다는 것은 수정이 불가한 상태라고 했는데, 생각해보면 정수형 변수들은 다양한 연산자를 통해 값을 자유롭게 수정 할 수 있습니다.

```python
a = 1
print(type(a))  # <class 'int'>
print(a)  # 1
a += 1
print(a)  # 2
```
<br>
<br>
<br>

# Python object

왜 이런가 보기 전에 python object에 대해 잠깐 이야기해보겠습니다. Python의 모든 코드는 `object` 혹은 `object`들 간의 관계로 나타내어집니다(`object`들 간의 관계로 이어져있다는 것을 잘 기억해주세요!). `int`, `str` 등의 소스를 보면 모두 `object`를 상속받고 있는 것을 확인할 수 있습니다. 이 `object`는 `identity`, `type`, 그리고 `value`를 항상 가지고 있습니다.

**Identity**\
`object`의 identity(이하 id)는 한번 생성된 이후로 절대 변경되지 않습니다. 객체의 메모리 주소와 비슷한 느낌이라고 생각하시면 됩니다. Python에서 `is` 연산자를 통한 비교는 이 id를 비교하는 것입니다. 객체의 id값을 확인하고 싶다면 `id(some_object)`를 통해 확인하시면 됩니다.

**Type**\
`object`의 type은 해당 객체가 어떠한 값을 가지고 어떠한 연산을 할 수 있는지를 알 수 있게 해줍니다. 해당 객체 type의 소스에 사용하려는 연산이 없으면 error를 나타냅니다. Integer 객체에 `len()`을 사용하면 `TypeError`를 나타내는 것과 같이요.

**Value**\
`object`의 value가 해당 객체가 mutable 한지, immutable 한지를 결정합니다. 객체의 value가 변경 가능하다면 mutable, 변경 불가하다면 immutable입니다.
<br>
<br>
<br>

# int is not mutable

다시 값을 자유롭게 변경하던 int type `a`를 확인해보겠습니다. 앞서 `a`의 value와 type은 확인해보았지만 id는 확인해보지 않았습니다.

```python
a = 1
print(a)  # 1
print(id(a))  # 140607361250004
a += 1
print(a)  # 2
print(id(a))  # 140607361250005
```

위와 같이 `a`의 값을 수정 후 id를 확인한다면 다른 id 값을 가지고 있는 것을 알 수 있습니다. 객체의 id는 생성 후 절대 변경되지 않는다고 하였으니 위에서 1을 할당한 `a`와 아래에서 2를 할당한 `a`는 다른 객체라는 것을 알 수 있습니다. 결국 int는 immutable 하다는 것을 알 수 있습니다.

처음 선언한 id 값으로 `140607361250004`를 가지는 객체는 더이상 아무런 객체와 참조 관계를 가지고 있지 않으므로 나중에 garbage collector가 수거해 갈 것입니다. int type 객체 뿐만이 아닌 모든 빌트인 타입을 가지는 객체들은 위와 동일한 결과를 보일 것입니다.
<br>
<br>
<br>

# immutable containers

Container란 다른 객체로의 참조를 가지고 있는 객체입니다. `list`, `tuple`, `dictionary` 등을 예로 들 수 있습니다. 대부분의 컨테이너는 mutable하지만, 간혹 immutable 한 컨테이너가 있습니다. `tuple`, `frozen set` 등을 예로 들 수 있습니다. Immutable한 컨테이너를 수정하면 아래와 같은 상황이 일어납니다.

```python
t = (1, 2, 3)
type(t)  # <class 'tuple'>
print(t[0])  # 1
t[0] += 1  # TypeError: 'tuple' object does not support item assignment
t[0] = 10  # TypeError: 'tuple' object does not support item assignment
```

Tuple이 포함하고 있는 다른 객체의 값을 수정하는 것은 불가능하지만, 다른 tuple과 `+` 혹은 `+=` 연산자를 통해 합치는 것은 가능합니다. 하지만 이는 위 int type 예시와 같이 기존의 tuple에 다른 tuple을 붙이는 것이 아니라, 아예 새로운 tuple을 만드는 것입니다.
<br>

## mutable container in immutable container

그런데 immutable 컨테이너가 immutable한 객체가 아닌 mutable한 객체를 포함하고 있으면 어떨까요? 물론 immutable한 컨테이너는 mutable한 객체를 가지고 있을 수 있습니다. Immutable한 tuple type 컨테이너가 mutable한 list type 객체를 가지고 있는 예를 들어보겠습니다

```python
l = [1, 2 ,3]
t = (l, 4, 5)  # list in tuple!
print(t)  # ([1, 2, 3], 4, 5)
print(id(t))  #  14060740260006
l[0] = 100
print(t)  # ([100, 2, 3], 4, 5)
print(id(t))  # 14060740260006
```

아니 이럴수가, 분명 tuple은 immutable한데 tuple의 값이 변경되었습니다. Id 값이 같은 것을 보아하니 다른 객체가 생성된 것도 아닌것으로 보입니다. 사실 위에서 `object`에 대한 설명을 잠깐 할 때 python 코드는 `object`들 간의 관계로 이루어져 있다는 말과, 위에서 컨테이너는 다른 객체로의 참조로 이루어져 있다는 말을 보면 왜 이런지 이해가 갈 것입니다.

사실 tuple `t`는 `[1, 2, 3]`, `2`, `3`의 값 자체를 가지고 있는 것이 아니라 해당 값들을 가지는 각 객체들의 주소를 가지고 있는 것입니다. 결국 `l[0] = 100`을 통해 tuple `t`의 값 (`l`의 주소: `l`의 id 또한 변경되지 않습니다.)은 전혀 변경되지 않았으니 immutable함을 잘 지켰다는 것을 알 수 있습니다.

그렇다는 것은 아무리 mutable 한 객체로의 참조를 가지고 있다 하더라도 참조하는 객체의 주소가 변경되어서는 안되겠죠? 즉, 아래 예시와 같이 아예 다른 객체를 집어넣은 것은 불가하다는 뜻입니다.

```python
t = ([1, 2, 3], 4, 5)
t[0] += [6 ,7]  # // TypeError: 'tuple' object does not support item assignment
print(t)  # ([1, 2, 3, 6, 7], 4, 5)
t[0] = [6, 7, 8]  # // TypeError: 'tuple' object does not support item assignment
print(t)  # ([1, 2, 3, 6, 7], 4, 5)
```

잠깐만요, 뭔가 이상한 부분을 눈치채셨나요? 두번째 줄에서 `t[0] += [6, 7]`을 실행했을 때 TypeError가 나타났는데 다음 줄에서 `t`를 출력해보니 `t[0]`에 `[6, 7]`이 정상적으로 합쳐져있습니다. 일단 list와 list 간 `+=` 연산자를 사용하여도 새로운 리스트가 생성되는 것이 아니라 기존의 리스트에 다른 리스트가 합쳐지는 것이므로 `t[0] += [6, 7]`은 당연히 정상적으로 실행되어야 하는 것이 맞습니다. 네번째 줄의 `t[0] = [6, 7, 8]`는 아예 새로운 리스트로 교체하는 것이므로 불가한 것이 맞습니다.

이러한 현상이 일어나는 이유는 [다음 포스트](https://brothersoo.github.io/python/iadd_list_in_tuple)에 작성해보록 하겠습니다.
<br>
<br>
<br>

# mutable objects

우리가 자주 사용하는 `list`, `dictionary`, `set` 등 많은 컨테이너들은 mutable합니다. 컨테이너가 포함하는 값들이 자유롭게 변경될 수 있습니다.

```python
l = [1, 2, 3]
print(id(l))  # 14060740040007
l[0] = 100  # 예 1
print(l)  # [100, 2, 3]
print(id(l))  # 14060740040007
l.append(3000)  # 예 2
print(l)  # [100, 2, 3, 3000]
print(id(l))  # 14060740040007
l.extend([-1])  # 예 3
print(l)  # [100, 2, 3, 3000, [-1]]
print(id(l))  # 14060740040007
l.pop()
print(l)  # [100, 2, 3, 3000]
print(id(l))  # 14060740040007
```

위 코드의 예 1과 같이 mutable한 객체의 값(list는 컨테이너이므로 다른 객체로의 참조)이 자유롭게 변경될, 수 있습니다. 예 2, 3, 4와 같이 해당 컨테이너의 타입이 가지는 함수를 사용하여 새로운 데이터를 추가 및 삭제 또한 가능합니다. list type은 `append()`, `extend()`, `pop()` 등의 구현된 메서드들을 사용할 수 있습니다.

## mutable dictionary

dictionary type을 가지는 객체들은 분명 mutable합니다. 아래 코드와 같이 자유롭게 값의 추가, 수정, 삭제가 가능합니다.

```python
d = {
    'a': 1,
    'b': 2,
    'c': 3
}
d['a'] = 4
print(d)  # {'a': 4, 'b': 5, 'c': 6}
d['d'] = 7
print(d)  # {'a': 4, 'b': 5, 'c': 6, 'd': 7}
del(d['a']) print(d)  # {'b': 5, 'c': 6, 'd': 7}
```

위 코드의 여섯번째 줄과 같이 값을 자유롭게 변경할 수 있는 것을 볼 수 있습니다. 그런데 python dictionary는 key, value를 가지는 hash table 구현체입니다. 위 예시에서 value는 변경해보았지만, key를 수정해본다면 어떨까요? 결론부터 말씀 드리면 이는 불가능합니다. Python dictionary의 key는 반드시 immutable한 type의 객체이어야 합니다. 만약 mutable 한 객체를 dictionary의 key로 설정해본다면 어떻게 될까요?

```python
l = [1, 2, 3]  # mutable list type object
d = {}  # same as d = dict()
d[l] = [1, 2, 3]  # TypeError: unhashable type: 'list'

s = {1, 2, 3}  # mutable set type object
d[s] = {1, 2, 3}  # TypeError: unhashable type: 'set'

print(d)  # {}  -> empty!
```

위 코드와 같이 dictionary의 key를 mutable한 type의 객체로 지정하는 것은 애초에 불가능 한 것을 알 수 있습니다.

위에서 설명한 immutable한 string type 객체를 사용한 예를 보겠습니다.

```python
string_key = 'a'
d = {
    string_key: 1,
    'b': 2,
    'c': 3
}
print(d)  # {'a': 1, 'b': 2, 'c': 3}
string_key = 'z'
print(d)  # {'a': 1, 'b': 2, 'c': 3}
```

당연한 결과입니다. 첫번째 줄의 string_key(이하 sk1)와 여덟번째 줄의 string_key(이하 sk2)는 전혀 다른 객체입니다. 우리가 잠시 사용한 변수명만 우연히 같았을 뿐입니다. 즉, d는 key로 항상 sk1을 사용하고 있었고, sk2는 키로 가진 적이 없었던 것입니다.

여담으로 sk1은 d가 참조하고 있고, sk2는 현재 변수 `string_key`가 참조하고 있으니 sk1과 sk2 모두 garbage collector에 의해 수거될 일은 없을 것입니다 (참조를 제거하기 전까지는).

Python dictionary의 key는 immutable한 타입의 객체라면 어떤것이든 사용이 가능합니다. `tuple` type 객체, `frozen set` type 객체 모두 dictionary의 key로 사용 가능합니다.

```python
t = (1, 2, 3)
fs = frozenset([4, 5, 6])
d = {
    t: (1, 2, 3),
    fs: [4, 5, 6]
}
print(d)  #{(1, 2, 3): (1, 2, 3), frozenset({4, 5, 6}): [4, 5, 6]}
```
<br>
<br>
<br>

읽어주셔서 감사합니다!
