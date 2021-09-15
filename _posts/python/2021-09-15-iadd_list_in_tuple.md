---
title: "Python: Inplace add(+=) list in tuple"
categories:
  - Python
tags:
  - Python
permalink: /python/:title
---

<br>

[Mutable과 immutable에 대한 지난 포스팅](https://brothersoo.github.io/python/mutable_and_immutable_objects)에서 이해가 잘 가지 않는 현상을 발견해 궁금증을 해소해보고자 찾아보았습니다. 이해가 가지 않았던 현상은 tuple이 list type 객체를 원소로 가지고 있을 때, 해당 list type 원소에 inplace add(+= 연산)을 할 경우 error가 발생하지만, 결과적으로는 해당 원소에 붙이고자 한 값이 정상적으로 들어가는 현상입니다.

```python
l = [1, 2, 3]
t = (l, 4, 5)
l[0] += [100, 101]  # TypeError: 'tuple' object does not support item assignment
print(t)  # ([1, 2, 3, 100, 101], 4, 5)
```

위 코드와 같이 `+=` 연산을 하는 시점에서 `TypeError`가 발생하지만, 다음 라인에서 튜플을 출력해 볼 시 `[100, 101]`이 바라던 대로 합쳐져있음을 확인할 수 있습니다.

Tuple의 immutable한 속성때문에 그런가 싶을 수 있지만, 사실 list의 inplace add는 `self`를 return하는 mutating method이기 때문에 tuple 안 원소의 id값이 변하는 일은 일어나지 않아, 해당 에러가 이해가 가지 않는 상황이었습니다. 서칭을 열심히 한 결과 아주 좋은 포스팅 및 글들을 몇가지 확인할 수 있었습니다.

> [[lerner] `__add__` vs `__iadd__` (`+` vs `+=`)](https://lerner.co.il/2019/06/06/why-do-python-lists-let-you-a-tuple-when-you-cant-a-tuple/)
>
> [[Stack Overflow] Tuples: += operator throws exception, but succeeds?](https://stackoverflow.com/questions/38344244/tuples-operator-throws-exception-but-succeeds)
>
> [[Python doc] dis — Disassembler for Python bytecode](https://docs.python.org/3/library/dis.html#module-dis)

<br>
<br>
<br>

# 의문의 현상에 대한 이유

의문의 현상에 대한 이유는 list type의 `l[0] += [100, 101]` 연산의 절차를 확인하면 알 수 있습니다. 코드의 실행 절차를 확인하는데 사용하기 좋은 모듈로 `dis`가 있습니다. Dis는 disassembler의 약자로, CPython(혹은 Jyton, PyPy와 같은 다른 구현체)이 컴파일한 바이트 코드를 disassemble 하여 분석할 수 있게 해줍니다. CPython 및 다른 구현체의 compiling과 interpreting은 다른 포스팅에서 다루어 보도록 하겠습니다.

`dis` 모듈의 dis 함수를 사용하여 `l[0] += [100, 101]`가 어떠한 절차로 실행되는지 확인해 보겠습니다.

```python
from dis import dis


l = [1, 2, 3]
t = (l, 4, 5)

dis('t[0] += [100, 101]')
'''
  1           0 LOAD_NAME                0 (t)
              2 LOAD_CONST               0 (0)
              4 DUP_TOP_TWO
              6 BINARY_SUBSCR
              8 LOAD_CONST               1 (100)
             10 LOAD_CONST               2 (101)
             12 BUILD_LIST               2
             14 INPLACE_ADD
             16 ROT_THREE
             18 STORE_SUBSCR
             20 LOAD_CONST               3 (None)
             22 RETURN_VALUE
'''
```

첫번째 행의 1은 몇번째 줄에 대한 절차인가를 알려주는 수입니다(위 예는 한줄의 연산에 대한 분석이므로 1만 존재합니다). 두번째 행의 숫자는(0, 2, 4, ...) 해당 연산의 byte offset입니다. 두번째 행의 문자열들은 사람이 읽기 힘든 byte code를 친절하게도 사람이 읽을 수 있는 instruction으로 변환하여 나타낸 문자열입니다 (실제 바이트코드는 "\b0x\b01\a12..." 와 같이 생겼습니다). 해당 instruction들이 어떠한 작업을 하는지는 [python 공식 문서](https://docs.python.org/3/library/dis.html#python-bytecode-instructions)에 잘 작성되어있습니다.

CPython은 stack 기반 virtual machine을 사용합니다. 다양한 데이터들을 Instructions이 의미하는 대로 stack에 넣었다 뺐다 하며 연산을 진행할 것입니다. 위 instructions를 한줄한줄 확인해보겠습니다.

확인하기 전 알아두어야 할 것들
* `TOS`: Top Of Stack. Stack의 가장 윗 값을 나타냅니다.
* `TOSn`: Stack의 위에서 n + 1번째 값을 나타냅니다. ex) TOS1 -> 위에서 두번째 값
* `BINARY_` operation: 앞에 `BINARY_`가 붙은 instruction은 TOS와 TOS1을 stack에서 pop 한 후 결과를 다시 stack에 push 합니다.
* `INPLACE_` operation: binary operation과 동일하나, in-place을 지원한다면 in-place 연산을 수행합니다.

<br>

Instructions sequence

1. **LOAD_NAME &nbsp;&nbsp;&nbsp;&nbsp; 0 (t)**\
t라는 이름을 가지는 요소의 값을 stack에 push 합니다. Stack에 `([1, 2, 3], 4, 5)`를 push 합니다.
2. **LOAD_CONST &nbsp;&nbsp;&nbsp;&nbsp; 0 (0)**\
0을 stack에 push 합니다.
3. **DUP_TOP_TWO**\
stack의 TOS와 TOS1을 복사하여 stack에 push 합니다. 이때, TOS와 TOS1의 순서가 지켜진 상태로 push됩니다.
4. **BINARY_SUBSCR**\
`TOS = TOS1[TOS]`를 수행합니다(TOS는 top of stack를 뜻하고 TOSn은 맨 위에서 n+1번째 값을 뜻합니다.). 즉 stack에서 TOS(0)과 TOS1(`([1, 2, 3], 4, 5)`)을 pop 한 후 TOS1[TOS] (`[1, 2, 3]`)을 stack에 push 합니다.
5. **LOAD_CONST &nbsp;&nbsp;&nbsp;&nbsp; 1 (100)**\
stack에 100을 push 합니다.
6. **LOAD_CONST &nbsp;&nbsp;&nbsp;&nbsp; 2 (101)**\
stack에 101을 push 합니다.
7. **BUILD_LIST &nbsp;&nbsp;&nbsp;&nbsp; 2**\
stack의 위 두개의 값을 리스트로 만들어 stack에 push 합니다. 즉, 100과 101을 이용해 `[100, 101]`을 stack에 push 합니다. (순서가 변하지 않는 것으로 보아 pop을 하지 않고 list를 생성하는 메서드의 인자로 순서대로 전달하는 것 같습니다(뇌피셜))
8. **INPLACE_ADD**\
TOS = (TOS1 += TOS)를 연산합니다. 즉, TOS1(`[1, 2, 3]`) += TOS(`[100, 101]`)의 값인 `[1, 2, 3, 100, 101]`을 stack에 push 합니다.
9. **ROT_THREE**\
TOS를 TOS2가 되도록 합니다. 기존의 TOS1과 TOS2는 한칸씩 올라와 각각 TOS와 TOS1이 됩니다. `(index + 1) % 3` 같은 느낌으로 생각하시면 될 것 같습니다.
10. **STORE_SUBSCR**\
TOS1[TOS] = TOS2를 수행합니다. 즉, 문제의 `t[0] = [1, 2, 3, 100, 101]`을 수행합니다. 인터프리팅 과정에서 이 instruction을 수행할 때 에러가 나타나는것으로 보입니다.
11. **LOAD_CONST &nbsp;&nbsp;&nbsp;&nbsp; 3 (None)**\
stack에 `None`을 push 합니다.
12. **RETURN_VALUE**\
TOS와 함께, 요청한 연산의 결과인 컨테이너를 반환합니다.

위에 작성한 것과 같이, 여덟번째 instruction에서 `[1, 2, 3] += [100, 101]`이 이미 실행 된 이후에, 열번째 instruction이 tuple의 값을 수정하는 것과 같이 취급되어 `TypeError`를 발생시키는 것 같습니다.
<br>
<br>

## What about `t[0].append()`?

반면에, 에러 없이 실행된 `t[0].append([100, 101])`은 어떤 instructions를 가지는지 확인해보겠습니다.

```python
dis(t[0].append([100, 101]))
'''
  1           0 LOAD_NAME                0 (t)
              2 LOAD_CONST               0 (0)
              4 BINARY_SUBSCR
              6 LOAD_METHOD              1 (append)
              8 LOAD_CONST               1 (100)
             10 LOAD_CONST               2 (101)
             12 BUILD_LIST               2
             14 CALL_METHOD              1
             16 RETURN_VALUE
'''
```

Instructions를 확인해보면 조금 다른 방식으로 수행됨을 볼 수 있습니다. **`LOAD_METHOD`** instruction을 통해 `t[0]`\(list type)의 `append`라는 이름을 가지는 메서드를 가져 온 후 나중에 해당 메서드를 호출하는 방식입니다. `list object`의 `append` 메서드는 return 값을 가지지 않습니다. self 조차 반환하지 않는데, 이는 간단히 아래 코드를 통해 확인해 볼 수 있습니다.

```python
l = [1]
l.append(2).append(3)  # AttributeError: 'NoneType' object has no attribute 'append'
```

리스트 타입인 `l`에 append 메서드를 chain 형식으로 두번 사용할 경우 위와 같이 NonType에서 append 메서드를 찾을 수 없다고 나타납니다. 이는 `l.append(2)`의 return 값이 없으므로(`None`), 연속적으로 append 메서드를 사용할 수 없다는 것을 나타냅니다.

결국 `t[0].append([100, 101])`은 `t[0] += [100, 101]`과 다르게 인덱스를 사용한 튜플의 수정(STORE_SUBSCR)이 없다는 것을 볼 수 있었습니다.
<br>
<br>
<br>

# 결론

`t[0] += [100, 101]`은 결국 아래와 같은 단계가 bytecode 단위로 실행되는 것입니다.

```python
t = ([1, 2, 3], 4, 5)
l = t[0]
l += [100, 101]
t[0] = l
```

아무리 기존의 값과 동일한 값(`self`: return value of `list.__iadd__()`) 이더라도 튜플의 0번 인덱스에 값을 대입하는 것이므로 에러가 출력되는 것이었습니다. 하지만 이전에 실행된 `l += [100, 101]`과 같은 단계가 먼저 실행되기 때문에 리스트의 데이터는 변경된 것입니다.
<br>
<br>
<br>

# 추가된 의문점

그렇다면 파이썬은 왜 중간에 에러가 남에도 불구하고 이전에 실행했던 instruction을 취소하지 않는걸까요? Database transaction과 비슷한 느낌으로 취소할 수는 없는 걸까요? 효율성 혹은 성능 때문에 그런 걸까요? 잘 모르겠습니다... 알아내게 되면 다음 포스팅에 작성해보도록 하겠습니다.
<br>
<br>
<br>

틀린 내용이 있다면 말씀 부탁드리겠습니다. 감사합니다!