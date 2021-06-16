---
title: "Django prefetch_related(4)"
categories:
  - Django
tags:
  - Django
  - query optimization
  - ORM
toc: true
---

<br>

# prefetch_related가 적용되지 않습니다.

저의 경험 중 prefetch_related를 분명히 APIView의 queryset에 적용하여 관계 테이블의 데이터를 prefetch 하였지만,

해당 관계된 데이터를 불러올때 쿼리를 추가로 사용하는 이상한 상황이 있었습니다.

왜 그랬을까요?
<br>
<br>
<br>
<br>

# Django caching and querysets

잠깐 queryset caching에 대해 이야기해보겠습니다.

기본적으로 django ORM의 queryset은 evaluate 되었을 때 객체 내에 그 실행 결과를 caching합니다.

원래 related fields는 caching하지 않지만 select_related 혹은 prefetch_related를 하면 관계된 데이터까지 캐싱하게 됩니다.

정확한 django queryset cahcing은 오른쪽 링크를 확인해주세요. [Django caching and querysets](https://docs.djangoproject.com/en/3.2/topics/db/queries/#caching-and-querysets)

위 링크의 내용을 간략하게 설명한다면, queryset은 무작정 캐싱되지 않습니다. queryset 객체를 생성하고 그 객체를 다시 불러올때 비로소 그 객체에 캐싱된
데이터를 불러오게 됩니다.

```
# 위 print문과 아래 print문에서 사용된 Menu queryset은 별개이므로 queryset 캐싱의 이점을 전혀 활용하지 못했습니다.
print([menu.name for menu in Menu.objects.all()])    # hits DB!
print([menu.price for menu in Menu.objects.all()])   # hits DB!

# 위 print문과 아래 priint문에서 동일한 queryset 객체를 사용했기 때문에 caching된 데이터를 활용했습니다.
queryset = Menu.objects.all()            # Queryset 객체 생성. Cache는 현재 비어있습니다.
print([menu.name for menu in queryset])  # hits DB and cache saved!
print([menu.price for menu in queryset]) # Cache의 데이터를 사용!
```

주석으로도 쓰여있듯이 queryset 객체를 생성하였을때는 query evaluation이 일어나지도 않고 cache가 저장되지도 않습니다.\
처음으로 queryset을 evaluate 할 때 cahce가 queryset 객체에 저장됩니다.
<br>
<br>
<br>
<br>

# 다시 돌아와...

그래서 제가 저질렀던 잘못은 무었이었을까요? 그 당시의 코드를 간략하게 재현해보겠습니다.

**View**

```
class StoreAPIView(generics.RetrieveAPIView):
  queryset = Store.objects.prefetch_related(
    Prefetch(
      'menu_set',
      queryset=Menu.objects.filter(price__lte=30000)
    )
  )
  serializer_class = StoreSerializer
```

**Serializer**

```
class StoreSerializer(serializers.ModelSerialzier):
  avg_menu_price = serializers.SerializerMethodField()

  def get_avg_menu_price(self, obj):
    return obj.menu_set.aggregate(Avg('price'))

  class Meta:
    fields = ('id', 'name', 'avg_menu_price')
```

**코드 설명**

view의 queryset을 보면 Store와 관계된 menu들을 prefetch 하여 한번에 가져오고 있습니다.

그렇다면 이 queryset을 사용할 때 Store를 불러오는 query문 하나, Menu를 불러오는 query문 하나, 총 두개의 쿼리를 사용할 것이라고 예상할 수 있습니다.

Menu의 데이터를 prefetch로 eager loading 한 이유는 serializer를 보면 알 수 있습니다.

StoreSerializer가 반환하는 세개의 fields 중 avg_menu_price를 보면 가게에서 판매하는 메뉴들의 평균 금액을 반환하고 있습니다.

저의 예상대로라면 prefetch_related로 menu_set을 eager loading 하였으니 get_avg_menu_price 메서드에서는 query evaluation이 단 한개도
일어나지 않을 것입니다.
<br>
<br>
<br>
<br>

# 100개... 10000개... 혹은 그 이상...

위 예시의 View를 사용하면 총 몇개의 query evaluation이 일어날까요?

그건 가게의 수에 따라 달라질 것입니다.

가게가 100개라면 102개, 1000개라면 1002개, 가게가 많으면 많을수록 실행되는 쿼리의 수는 늘어나게 됩니다.

Serializer의 get_avg_menu_price 메서드에서 가게의 수만큼 쿼리가 실행될 것입니다.

음... 분명 view의 queryset에서 menu_set을 prefetch했기 때문에 obj.menu_set.aggregate(Avg('price'))를 실행해도 캐싱된 menu_set을 사용할 줄
알았습니다.

그러나 위에서 설명했다시피 cache는 queryset 객체에 저장되기 때문에 다른 queryset에서는 캐싱된 데이터를 당연히도 사용하지 못합니다.
<br>
<br>
<br>
<br>
<br>
다시 코드를 보겠습니다.

View에서 prefetch한 Menu에 대한 queryset과 serializer에 작성된 store과 관계된 menu queryset이 동일할까요?

Serializer의 queryset에는 aggregate라는 다른 orm 메서드가 뒤에 붙어있습니다.

이렇게 되면 obj.menu_set과는 다른 새로운 queryset을 생성하게 됩니다.

View에서 prefetch한 데이터를 사용할 수가 없는 것입니다.

결국 prefetch_related를 사용하여 추가로 쿼리만 늘려놓고 실제 추가한 쿼리는 아무런 도움이 되지 못했습니다.
<br>
<br>
<br>
<br>

# 결론

결론을 말씀드리자면 prefetch된 데이터를 사용하기 위해서는 prefetch 한 queryset과 동일한 queryset 객체를 사용해야 합니다.

prefetch_related는 메인 테이블에 대한 쿼리문 따로, prefetch한 related 테이블에 대한 쿼리문 따로 실행한다고 했었습니다.

쿼리문은 따로 실행되지만 그렇다고 해서 각 쿼리문에 대한 queryset 객체가 따로 생성되는 것은 아닙니다.

prefetch_related를 사용할 시 prefetch_related를 사용한 queryset 객체의 cache에 prefetch 된 관계 데이터들이 저장되는 것입니다.

prefetch_related를 사용했음에도 쿼리의 수가 원하는 대로 줄지 않는다면 prefetch 한 queryset을 사용하고 있는 것인지 확인해보세요!

aggregate 뿐만이 아니라 annotate, filter와 같은 다른 메서드를 추가해도 다른 queryset이 됩니다!



