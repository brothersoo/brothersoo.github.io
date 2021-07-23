---
title: "Comparator, Comparable"
categories:
  - Java
tags:
  - Java
  - sort
---

> Java의 comparable과 comparator를 너무나도 아름답게 잘 정리한 포스팅을 발견했습니다.
>
> [자바 [JAVA] - Comparable 과 Comparator의 이해](https://st-lab.tistory.com/243)
>
> 위 블로그를 한번 정독하시면 comparable과 comparator에 대해 잘 이해하실 수 있습니다.\
> 이분 블로그에 아름다운 포스팅이 많더라구요^^

<br>
<br>

# Comparable & Comparator

우선 자바로는 알고리즘 문제뿐이 풀어보지 못한 저로써는 comparable과 comparator은 custom한 sorting을 위해서만 사용해보았습니다.
~~(원래 정렬에서만 사용되는것인지는 잘 모르겠네요^^*)~~

보통 ArrayList 혹은 Priority queue의 우선순위 설정을 위해서 많이 사용해보았습니다. ~~(그마저도 거의 comparator만 사용했습니다^^)~~

위 추천드린 블로그 포스트의 시작 부분에도 쓰여있듯이 comparable과 comparator은 interface입니다.

구현체를 만들어주어야 사용할 수 있습니다.
<br>
<br>
<br>
<br>

# Comparable

Comparable은 보통 클래스에서 상속받습니다.

Comparable을 상속받을때 필수로 overriding 해주어야 하는 메서드는 int compareTo() 입니다.

만약 메뉴의 가격을 비교하고자 한다면 아래와 같은 Menu 클래스에 Comparable을 상속받고 compareTo()를 overriding하면 됩니다.

```
class Menu implements Comparable<Menu> {

  Long id;
  String name;
  int price;

  public Menu(Long id, String name, int price) {
    this.id = id;
    this.name = name;
    this.price = price;
  }

  @Override
  public int compareTo(Menu m) {
    return this.price - m.price;
  }
}
```

이렇게 Menu class에 compareTo를 구현하면 Menu끼리 가격으로 손쉽게 비교를 할 수 있습니다.

```
public static void main(String[] args) {

  Menu a = new Menu(1L, "돈까스", 8000);
  Menu b = new Menu(2L, "피자", 15000);

  if (a.compareTo(b) > 0) {
    System.out.println("a is more expensive than b!");
  };
}
```

위 경우는 단순히 가격만 비교하기 때문에 굳이 compareTo()가 필요 없어 보입니다.

한가지 요소로만 단순 비교하지 않는다면 compareTo()를 요리조리킹 잘 조작하시면 됩니다.

```
class Menu implements Comparable<Menu> {

  Long id;
  String name;
  int price;

  public Menu(Long id, String name, int price) {
    this.id = id;
    this.name = name;
    this.price = price;
  }

  @Override
  public int compareTo(Menu m) {
    if (this.price == m.price) {
      return this.name.compareTo(m.name);
    } else {
      return this.price - m.price;
    }
  }
}
```

어라? Intellij가 this.name을 입력하니 다음 자동추천으로 compareTo를 보여줍니다.

this.name은 저희가 Compareable 설정을 해준것이 없어 보이는데 어떻게 된걸까요?

name 필드는 String으로 정의했었네요...

아하! String 클래스를 확인해보니 Compareable<String\>을 상속받고있군요! ~~저는 방금 알았습니다ㅎㅎ~~

확인해보니 String 뿐만이 아닌 Integer, Character 등 reference type은 모두 자신과의 Comparable을 상속받고 있는 것을 확인할 수 있었습니다.

---

다시 본론으로 들어와 위와 같이 Menu에 compareTo를 정의하게 된다면 가격이 다를시에는 가격으로 비교, 가격이 같다면 이름으로 비교를 하게 되는 것이죠.

만약 Menu에 Comparable을 상속받지 않아 compareTo를 재정의 하지 않고 Collections.sort()를 사용해 MenuList를 정렬한다고 하면 `Menu cannot be cast to class java.lang.Comparable` 와 같은 `ClassCastException`이 발생하게 됩니다!

Menu의 어떤 값을 기준으로 정렬할 것인지를 모르니 에러가 뜨는 것이 당연해 보입니다.

하지만 저희는 멋진 기준을 가지고 compareTo() 메서드를 재정의 해주었으니 Collections.sort()를 사용할 수 있게 되었습니다!

```
List<Menu> menuList = new ArrayList<>();
menuList.add(new Menu(1L, "파전", 8000));
menuList.add(new Menu(2L, "사탕", 200));
menuList.add(new Menu(3L, "피자", 15000));
menuList.add(new Menu(4L, "김치찌개", 5000));
menuList.add(new Menu(5L, "돈까스", 8000));

Collections.sort(menuList);
// 사탕    200
// 김치찌개    5000
// 돈까스    8000
// 파전    8000
// 피자    15000
```

가격 오름차순으로 잘 정렬되었고 가격이 같은 돈까스와 파전은 가나다 순으로 멋지게 정렬됨을 확인할 수 있었습니다^^b
<br>
<br>
<br>
<br>


# Comparator

앞서 말씀드렸다시피 Comparator 역시 Comparable과 마찬가지로 interface입니다.

위 Comparable과 같이 class에 직접 상속받아도 괜찮지만 보통은 Collections.sort()와 같은 비교 조건을 넣어줄 수 있는 메서드에 무명객체로 파람을 넘겨줍니다.

간단하게 람다 함수를 써주어도 great입니다!

```
List<Menu> menuList = new ArrayList<>();
menuList.add(new Menu(1L, "돈까스", 8000));
menuList.add(new Menu(2L, "사탕", 200));
menuList.add(new Menu(3L, "피자", 15000));
menuList.add(new Menu(4L, "김치찌개", 5000));
menuList.add(new Menu(5L, "파전", 8000));

Collections.sort(menuList, Collections.sort(menuList, new Comparator<Menu>() {
      @Override
      public int compare(Menu o1, Menu o2) {
        if (o1.price == o2.price) {
          return o1.name.compareTo(o2.name);
        } else {
          return o1.price - o2.price;
        }
      }
    });)

Collections.sort(menuList, (Menu m1, Menu m2) -> {
      if (m1.price == m2.price) {
        return m1.name.compareTo(m2.name);
      } else {
        return m1.price - m2.price;
      } 
    });
```

무명객체를 사용하든 람다식을 사용하든 결과는 같지만 람다식이 조금 더 깔끔해 보입니다!

같은 기준으로 정렬을 여러번 해주어야 한다면 객체를 따로 생성하여 Comparable을 상속받는 것이 좋아 보이지만

정렬의 기준이 계속 바뀌거나 한번만 정렬을 해준다면 간단하게 comparator 무명객체 혹은 람다식을 사용하는 것이 좋아 보입니다^^* ~~저의 개인적인 생각입니다~~
<br>
<br>
<br>
<br>


# Return int?

위에서 comparable과 comparator를 구현한 것을 보게된다면 모두 두 객체의 어떠한 요소를 뺀 값을 반환하고 있습니다.

1. 양수를 반환
2. 음수를 반환
3. 0을 반환

양수를 반환하는 경우에는 스왑이 이루어집니다.

삽입정렬, 퀵정렬, 합병정렬 등 수많은 대다수의 정렬은 두 데이터의 스왑이 계속 일어납니다.

여기서 스왑의 조건을 저희가 customizing하는 것이죠. 가슴이 웅장해집니다.

음수, 혹은 0을 반환하는 경우에는 스왑이 이루어지지 않습니다.

Comparable을 상속받은 경우라면 this 객체가 선행 객체, compareTo()의 param으로 넘겨준 객체가 후행 객체입니다.

즉 `return a.value - b.value;`를 하게 된다면 a.value가 b.value보다 클 시 양수를 return하므로 스왑이 일어나게 됩니다.

큰 값인 a.value가 작은 값인 b.value 보다 뒤로 가게 되니 오름차순으로 정렬하게 되는 것입니다!

만약 a.value가 b.value보다 작았다면 음수를 반환할 것이고 스왑이 일어나지 않을것입니다. 오름차순이 유지됩니다!

그럼다면 0을 반환할 경우는 어떨 때 있을까요?

검색을 하다가 좋은 글을 발견했습니다.

[[Stack Overflow] why we need to return 0 when Comparator.compare become equal](https://stackoverflow.com/questions/58267950/why-we-need-to-return-0-when-comparator-compare-become-equal)

첫번째 답변을 보니 Comparable을 상속받고 stream을 사용하여 아주 멋드러지게 정렬하는 것을 볼 수 있습니다.

저는 아직 stream에 대한 학습이 진행되지 않아 정말 멋지게 보이네요^^

어서 stream 공부를 해보아야겠습니다.

하여튼 결국 이분도 0을 return하는 것은 음수를 return하는 것과 거의 대부분 유사한 행동을 보이고,

비교의 값이 0이 되는 조건이라면 다른 조건을 추가해 줄 수 있다고 하는 것 같습니다. ~~영어 실력이 출중하지 못해 아닐수도 있습니다~~

결론은 이미 저희가 앞에서 price가 같다면 name으로 비교하는 것과 같은 내용으로 보입니다. ~~아닐수도 있습니다~~

설정한 조건들이 모두 동일하다면 스왑이 이루어지지 않을 것입니다.
<br>
<br>
<br>

# Caution!!!

맨 위에서 추천드린 포스트를 읽어보면 중간에 붉은색으로 강조된 주의해야 할 점이 있습니다.

바로 return 할 연산의 결과가 integer의 범위를 넘어가버리는 경우입니다.

integer의 범위는 -2,147,483,648 ~ 2,147,483,647으로 -2,147,483,648 - 1은 -2,147,483,649가 아닌 2,147,483,647가 됩니다.

integer는 -2,147,483,649를 표현할 수 없기 때문이죠.

대부분의 경우에서는 이 범위를 넘어가지 않겠지만 혹시 이러한 경우가 생길 수 있다면 빼기 연산을 반환하는 것이 아닌 비교 연산 후 조건에 따라 정확하게 1, 0, -1을 반환해 주는 것이 좋다고 합니다.

```
menuList.add(new Menu(1L, "돈까스", -2,147,483,648));
menuList.add(new Menu(2L, "사탕", 1));

Collections.sort(menuList, (Menu m1, Menu m2) -> {
      return m1.price - m2.price;
    });
```

위와 같은 경우에 돈까스.price - 사탕.price은 2,147,483,647가 됩니다.

즉 양수를 return하므로 스왑이 이루어지게 되고 사탕 -> 돈까스 순으로 정렬이 됩니다.

그러나 저희가 의도한 것은 오름차순이므로 이는 잘못된 결과를 도출한 것입니다.

그러니 정확한 비교를 위해서는 아래와 같이 작성해야 합니다.

```
Collections.sort(menuList, (Menu m1, Menu m2) -> {
      if (m1.price > m2.price) {
        return 1;
      } else if(m1.price == m2.price) {
        return 0;
      } else {
        return -1;
      }
    });
```

그러나 저는 보통 비교 연산을 하지 않고 곧바로 빼기 연산한 값은 반환해줍니다.

귀찮으니까요^^*
<br>
<br>

----

틀린내용이 있다면 꼭 댓글로 말씀 부탁드리겠습니다!