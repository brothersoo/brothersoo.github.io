---
title: "[Java] System.arraycopy()"
categories:
  - java
tags:
  - java
  - array
permalink: /java/:title
---

Python을 공부하다 Java를 공부하게 되면서 두려웠던 것이 참 많았습니다.

대표적으로 꼼꼼한 type 명시, OOP, String/array 처리하기 등...

그 중에서도 알고리즘 문제를 java로 푸는것에 가장 두려움을 주었던 것 중 하나는 list slicing이었습니다.

Python은 `some_list[n:m]`와 같이 list를 범위로 쪼개는 것이 매우 간편하지만 java는 비교적 array, list를 쪼개는 것이 까다롭습니다.\
~~Python은 list slicing, string slicing, iteration, zip, enumerate 등 인덱스 및 숫자를 처리하는데 도움을 주는 간편한 것들이 너무나도 많죠^^*~~
<br>
<br>
<br>
<br>

# Copying arrays

지금까지는 array를 복사할 때 for문을 돌며 index 하나하나 복사를 해주고 있었습니다.

그러나 너무나 감사하게도 sonar lint가 System.arraycopy()를 알려주었습니다.

> Arrays should not be copied using loops (java:S3012)
>> Using a loop to copy an array or a subset of an array\
>> is simply wasted code when there are built-in functions to do it for you.\
>> Instead, use Arrays.copyOf to copy an entire array into another array,\
>> use System.arraycopy to copy only a subset of an array into another array,\
>> and use Arrays.asList to feed the constructor of a new list with an array.
>>
>> Note that Arrays.asList simply puts a Collections wrapper around the original array,\
>> so further steps are required if a non-fixed-size List is desired.

살짝 번역하자면

> 이미 동일한 기능을 하는 함수가 있음에도 불구하고 loop을 사용하여 배열을,\
> 혹은 배열의 부분을 복사하는 것은 아주 비효율적입니다.\
> 대신, 한 배열을 통째로 복사하는데에는 `Arrays.copyOf`를 사용하시고,\
> 배열의 일부분을 복사하는데에는 `System.arraycopy`를 사용하시고,\
> 그리고 새로운 `List`의 생성자에 배열을 넘기기 위해서는 `Arrays.asList`를 사용하세요.
>
> 참고로 `Array.asList`는 단순히 원본 배열을 `Collections wrapper`로 감싸는 것이므로,\
> 고정된 길이의 배열이 아닌 유연한 길이의 `List`가 필요하다면 추가적인 단계가 필요합니다.

~~사실 마지막 문장이 잘 이해가 안가네요^^;
Arrays.asList가 가변 길이의 list를 생성해주는데 어떤 추가 단계들이 필요하다는건지?^^;
영어 능력이 부족합니다...~~

저는 array의 부분을 복사해야 했기 때문에 System.arraycopy()를 사용했습니다.

~~여담으로 해당 메서드를 사용한 알고리즘 문제는 그 유명한 knapsack 문제였습니다.~~
<br>
<br>
<br>
<br>

# System.arraycopy()

`System.arraycopy()`는 총 디섯가지 필수 parameter를 가집니다.

순서대로 `Object src`, `int srcPos`, `Object dest`, `int destPos`, `int length`가 되겠습니다.

1. `Object src`
`src`로 넘겨받은 객체는 복사의 대상입니다. 즉, `src`의 값들이 복사됩니다.

2. `int srcPos`
`src`로 넘겨받은 arg의 복사 시작 위치입니다. `srcPos`로 3을 넘겨받으면 `src[3]`부터 복사의 대상이 됩니다.

3. `Object dest`
`dest`로 넘겨받은 객체는 복사가 이루어질 대상입니다. 즉, `src`의 원소들이 복사되어 `dest`에 삽입됩니다.

4. `int destPos`
`dest`에서 대입이 시작될 위치입니다. 즉, `destPos`가 2라면 `dest[2]`에서부터 대입이 이루어집니다.

5. `length`
복사가 될 데이터의 수 입니다.
`srcPos`부터 `length`개의 데이터가 복사될 것입니다.

짧은 예를 만들어보면, 아래와 같습니다.
```java
public static void main(String[] args) {
  int[] arrayOne = new int[]{1, 2, 3, 4, 5};
  int[] arrayTwo = new int[arrayOne.length];

  System.arraycopy(arrayOne, 2, arrayTwo, 0, 2);
  System.out.println(Arrays.toString(arrayTwo));
}

// out: [3, 4, 0, 0, 0]
```

arrayOne의 2번 인덱스부터 2개의 데이터가 arrayTwo의 0번 인덱스부터 복사된 것을 확인할 수 있습니다.
<br>
<br>
<br>
<br>

# How does it work?

작동하는 로직을 대강 확인하고 싶어 소스를 확인해보고자 하였으나, native keyword를 가지고 있었습니다.

native keyword가 무엇인지 잘 몰라 검색을 해보니 [Stack overflow - What is the native keyword in Java for?](https://stackoverflow.com/questions/6101311/what-is-the-native-keyword-in-java-for)를 찾을 수 있었습니다.

사실 읽어보아도 잘 모르겠습니다...

c, 혹은 다른 언어로 작성된 라이브러리를 사용할 수 있게 해주는 것 같은데 이 라이브러리는 c++로 작성된 것 같아보입니다.

[System.arraycopy source code](https://programmersought.com/article/51181117081/)\
결국 위 링크에서 System.arraycopy의 전체 소스 코드를 확인할 수 있는데 너무 복잡합니다...

대강 확인해보니 가장 중요한 부분은 _Copy_conjoint_jlongs_atomic() 메서드라고 생각이 듭니다.

```java
void _Copy_conjoint_jlongs_atomic(jlong* from, jlong* to, size_t count) {
  if (from > to) {
    jlong *end = from + count;
    while (from < end)
      os::atomic_copy64(from++, to++);
  }
  else if (from < to) {
    jlong *end = from;
    from += count - 1;
    to   += count - 1;
    while (from >= end)
      os::atomic_copy64(from--, to--);
  }
}
```

뭐 이런 느낌으로 작동하는 것 같네요^^*~~사실 잘 모릅니다.~~

document를 확인하니 src와 dest가 동일한 배열 객체를 참조하고 있는 경우에는 마치 다음과 같이 작동한다고 합니다.

1. `src` 배열의 `srcPos` 인덱스부터 `srcPos + length - 1` 인덱스의 값들이 `length` 만큼의 길이를 가지는 temp 배열에 복사된 후
2. temp 배열을 `dest` 배열(`src`와 동일)의 `destPos` 인덱스부터 `destPos + length - 1` 인덱스까지 대입합니다.

"as if"라는 부분이 애매해 소스코드를 대강 훑어보아도 src와 dest가 같은 객체를 참조하고 있어도 임시 배열을 생성하는 코드는 확인할 수 없었습니다.\
그냥 느낌만 그렇다고 하는 것 같네요^^*
~~아마도요~~
<br>
<br>
<br>
<br>

# Exceptions

documentation에 쓰여있는 exception 발생 조건들 또한 위 링크에서 본 소스코드에서 대강 확인 할 수 있었습니다.

1. **NullPointerException**
    1. `dest`가 `null`인 경우
    2. `src`가 `null`인 경우

2. **ArrayStoreException**
    1. `src`가 array가 아닌 객체를 참조하는 경우
    2. `dest`가 array가 아닌 객체를 참조하는 경우
    3. `src`와 `dest`가 참조하는 array의 원소의 primitive type이 다른 경우
    4. `src`는 primitive type array이지만 `dest`는 reference type array인 경우
    5. `src`는 reference type array이지만 `dest`는 primitive type array인 경우

3. **IndexOutOfBoundException**
    1. `srcPos`가 음수인 경우
    2. `destPos`가 음수인 경우
    3. `length`가 음수인 경우
    4. `srcPos + length`가 `src`의 크기보다 큰 경우
    5. `destPos + length`가 `dest`의 크기보다 큰 경우

4. **ArrayStoreException**
    1. `src`의 데이터중 `srcPos`와 `srcPos + length - 1` 사이에 있는 데이터 중 (복사될 데이터 중)\
    할당 변환을 통해 `dest`의 원소 타입으로 변경될 수 없는 데이터가 있을 경우

`ArrayStoreException`가 발생하는 경우

k는 `src[srcPos + k]`가 `dest`에 삽입될 수 없는 타입일 경우의 값일 때, (0 <= k < length)

`src[srcPos]`부터 `src[srcPos + k - 1]`은 이미 `dest[destPos]`부터 `dest[destPos + k - 1]`에 복사가 되었고\
`src[src + k]`부터 `src[src + length - 1]`은 `dest`에 복사되지 않을 것입니다.

해당 예외 처리 이전에 제한 사항이 이미 구체화 되어있으므로 이 예외는 보통 `src`와 `dest`가 모두 array of reference type인 경우에 발생한다고 합니다.

---

### ArrayStoreException

```java
public static void main(String[] args) {
  Object[] wowArray = new Object[]{1, 2, 3, "4", "5"};
  Integer[] integerArray = new Integer[5];

  try {
    System.arraycopy(wowArray, 2, integerArray, 0, 2);
  } catch (ArrayStoreException e) {
    System.out.println("---------");
    System.out.println("Exception: " + e.toString());
    System.out.println("---------");
  } finally {
    System.out.println("integerArray: " + Arrays.toString(integerArray));
  }
}

// output:
// ---------
// Exception: java.lang.ArrayStoreException: arraycopy: element type mismatch: can not cast one of the elements of java.lang.Object[] to the type of the destination array, java.lang.Integer
// ---------
// integerArray: [3, null, null, null, null]
```

`mixedArray`는 Integer type 1, 2, 3, 그리고 String type "4", "5"가 순서대로 담겨있습니다.

`integerArray`는 array of Integer이므로 String type이 들어올 수 없습니다.

`System.arraycopy(mixedArray, 2, integerArray, 0, 2)`는 `mixedArray`의 3, "4", "5"를 `integerArray`에 복사하고자 할 것인데

k == 1인 상황에서 `ArrayStoreException`이 발생할 것입니다.

즉, k < 1 인 3은 정상적으로 `integerArray`의 0번 인덱스(`destPos`)에 복사되고\
`integerArray`(`dest`)는 [3, null, null, null, null]가 되는 것입니다.
<br>
<br>
<br>
<br>

# Conclusion

어쩌다 보니 c++로 이루어진 소스코드까지 확인하게 되었는데 결론은 System.copyarray()를 만나고 상당히 기뻤다는 것입니다!

앞으로 java에서 array를 가공할 때 한층 더 쉬워질 것 같네요^^*
<br>
<br>

---
<br>
<br>

틀린 내용이 있다면 모진 회초리같은 댓글 부탁드리겠습니다!