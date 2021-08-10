---
title: "Memory allocation(1) Reference counting"
categories:
  - Python
  - Memory allocation
tags:
  - Python
permalink: /python/:title
---
\
Python은 C, C++과 같이 프로그래머가 직접 malloc(), calloc()과 같은 함수를 사용하여 메모리를 할당해주거나 해제해줄 필요가 없습니다.
파이썬은 reference counting과 garbage collection을 사용하여 자동으로 메모리 할당/해제를 처리합니다.

## Reference counting
Reference counting은 말 그대로 어떤 객체가 다른 객체로부터 참조된 횟수를 세는 방식을 사용합니다. sys 모듈의 getrefcount()를 사용하면 객체의 참조된
횟수를 구할 수 있습니다.
```python
import sys


class Python():
  def __init__(self):
    ...

if __name__ == "__main__":
  obj = Python()
  print(sys.getrefcount(obj))  # 2
  obj2 = obj
  print(sys.getrefcount(obj))  # 3
  obj3 = obj
  print(sys.getrefcount(obj))  # 4
  obj2 = 'obj2'
  print(sys.getrefcount(obj))  # 3
  obj3 = 'obj3'
  print(sys.getrefcount(obj))  # 2
```
예상한 결과에 +1이 된 값이 출력됨을 확인할 수 있습니다. 이는 getrefcount()로 obj가 인자로 넘겨질 때 임시로 참조되기 때문입니다.

위 예제와 같이 reference count는 obj가 참조될 때마다 1 증가하고 참조가 해제될 때마다 1 감소하는 것을 확인 할 수 있습니다.
Reference count가 0이 되면 obj의 메모리 할당이 해제됩니다.

Reference count는 매우 효과적이지만, 몇가지 경우는 정상적으로 메모리 해제를 하지 못합니다. 대표적인 경우로는 reference cycle이 일어나는 경우입니다.
Reference cycle은 한개, 혹은 그 이상의 객체들이 서로를 참조하고 있는 상태를 말합니다.
```python
# 예시 1
a.attr = b
b.attr = a

# 예시 2
def make_cycle():
  L = []
  L.append(L)

make_cycle()
```
첫번째 예시와 같이 두 객체의 속성이 각각 상대 객체를 참조하고 있는 경우가 대표적인 reference cycle의 예시라고 볼 수 있습니다.

두번째 예시를 보면 make_cycle() 함수가 종료되면 L의 메모리는 해제되어야 하지만 L은 내부적으로 reference cycle이 일어나고 있으므로 메모리 해제가 되지
않습니다. L에 할당된 메모리는 garbage collection이 호출되어야 해제될 수 있습니다.

