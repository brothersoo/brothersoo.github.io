---
title: "Django prefetch_related(2)"
categories:
  - Django
tags:
  - Django
  - query optimization
  - ORM
toc: true
---
<br>
# Simple example - 다대다 관계

```
class Chef(models.Model):
    name = models.CharField()
    grade = models.IntegerField()
    licences = models.ManyToMany(Licence)

    class Meta:
        db_table = 'chef'


class Licence(models.Model):
    name = models.CharField()
    difficulty = models.IntegerField()
    preference = models.IntegerField()

    class Meta:
        db_table = 'licnece'
```

Chef와 Licence를 보면 한명의 요리사는 여러개의 자격증을 가질 수 있고, 하나의 자격은 여러명의 요리사가 가질 수 있습니다.

등급이 3등급인 요리사들이 가지고 있는 자격증의 데이터까지 한번에 가져오고 싶다면 아래와 같이 작성하면 됩니다.

```
Chef.obejcts.filter(grade=3).prefetch_related('licence')
```

반대로 어려움 등급이 2등급인 자격증을 가지고 있는 요리사의 데이터를 한번에 가져오고 싶다면 아래와 같이 작성하면 됩니다.

```
Licence.objects.filter(difficulty=2).prefetch_related('chef_set')
```

역참조 관계와 동일하게 `{역참조_관계의_클래스명.lower()}` 를 사용하시면 되고, 이를 사용하기 싫으시면 ManyToManyField()에 related_name을 인자로 넘겨주면 됩니다.

위 예시 또한 Chef의 데이터를 불러오는 쿼리 한개, Licence의 데이터를 불러오는 쿼리 한개, 총 두개의 쿼리를 실행하게 됩니다.
<br>
<br>
<br>
<br>

# N + 1 problem 예시

각 요리사의 자격들을 포함한 요리사들의 리스트를 불러오는 ChefListAPIView api를 예시로 들어보겠습니다.

```
class ChefListAPIView(generics.ListAPIView):
    queryset = Chef.objects.all()
    serializer_class = ChefListSerializer


class ChefListSerializer(serializers.ModelSerializer):
    licence = LicenceSerializer(many=True, read_only=True)

    class Meta:
        model = chef
        fields = ('id', 'name', 'grade', 'licence')


class LicenceSerializer(serializers.ModelSerializer):

    class Meta:
        model = Licence
        fields = ('id', 'name', 'difficulty', 'preference')
```

ChefListAPIView에 정의된 바와 같이 get_queryset()을 통해 모든 chef들을 담은 queryset이 정의될 것입니다.

최종적으로 결과값을 전송할때 각각의 chef object는 ChefListSerializer를 거칠 것이고, 그때마다 chef가 가지고 있는 각각의 licence들은
또다시 LicenceSerializer를 거칠 것입니다.

이것이 왜 문제가 되는것인가 보면 만약 첫번째 chef(이하 Joseph)가 5개의 자격증을 가지고 있다면, 다섯개의 자격들에 대한 데이터 처리를 위해 각각의 자격들은
모두 LicenceSerializer를 거칠 것입니다.

그때마다 Joseph의 자격들에 대한 데이터를 가져오기 위해 추가적으로 쿼리를 실행할 것이고, Joseph의 자격증에만 벌써 다섯개의 쿼리를 사용했습니다.

다섯개면 그나마 다행인 상황이고 Joseph이 만약 세계 최고의 요리사의 길을 걷기 위해 1000개의 요리 자격증을 땄다면 추가로 1000개의 쿼리를 실행하게 될 것입니다.

거기다 요리사는 Jospeh 뿐만이 아니기 때문에 쿼리 수는 엄청나게 추가가 될 것입니다.
<br>
<br>
<br>
<br>

# prefetch_related() in sql

prefetch_realted를 사용하였을 때 실행되는 쿼리문의 수는 1 + (prefetch_related()를 한 수) 입니다. 위 사용법 예시에서는 두개가 실행되겠습니다.

각각의 테이블에 헤당하는 쿼리셋이 각자 호출되고, JOIN 연산은 db가 아닌 python에서 진행됩니다!

`Chef.objects.prefetch_related('licence')`의 실제 실행되는 쿼리문을 살펴보면 아래와 같습니다.

```
-- first query
SELECT
    "chef"."id",
    "chef"."name",
    "chef"."grade",
    "chef"."licence_id"
FROM
    "chef"

-- second query
SELECT
    "licence"."id",
    "licence"."name",
    "licence"."difficulty",
    "licence"."preference"
FROM
    "licence"
INNER JOIN
    "chef_licences"
ON
    "licence"."id" = "chef_licences"."licence_id"
WHERE
    "chef_licences"."chef_id" IN (1, 2, 3, 4, 5, 6, 7)
```
<br>
<br>
<br>

# prefetch_related에서 무슨일이?

음... 이상한 점이 한두가지가 아닙니다.

* **"chef_licences"?**

두번째 쿼리의 INNER JOIN clause를 보게 되면 chef_licences라는 이상한 테이블명이 하나 있습니다.

이는 django migration을 진행할 때, 자동으로 생성되는 Many to Many 관계의 중간자 테이블 입니다.

Chef 모델이 licences라는 ManyToManyField를 가지고 있으므로 chef_licences라는 테이블이 생성됩니다.

만약 반대로 Licence 모델이 chefs라는 ManyToManyField를 가지고 있었다면 licence_chefs라는 테이블이 생성되었을 것입니다.
<br>
<br>
<br>

* **INNER JOIN clause?**

JOIN 연산은 db가 아닌 python이 진행한다고 했었는데 위 두번째 쿼리문에는 INNER JOIN clause가 존재합니다.

음... Django Document가 거짓말을 하고있는 것일까요... 저는 아니라고 생각합니다.

위 두번째 sql의 INNER JOIN이 어떤 테이블간에 이루어지고 있는지를 확인해 보면 아래와 같은 것을 알 수 있습니다..

```
FROM
    "licence"
INNER JOIN
    "chef_licences"
```

![chef_licences]({{ site.url }}{{ site.baseurl }}/assets/images/chef_licences.png)

Joinning이 이루어지고 있는 테이블은 licence 와 chef_licences입니다. 위 이미지에서 파란 점선으로 표시된 관계는 db에서 이루어졌고,

붉은 실선으로 표시된 licence와 chef_licence(결국 chef)와의 관계가 python level에서 이루어지는 것입니다.
<br>
<br>
<br>

* **WHERE IN clause?**

이 WHERE IN clause는 무엇을 뜻하는 것일까요?

사실 저도 이 글을 쓰면서 WHERE IN clause가 붙는다는 것을 알게 되었습니다...

저 WHERE IN clause가 왜 있나 검색을 많이 했는데 사실 아직 완벽하게 이해는 하지 못했습니다...

그러나 검색 중 두개의 좋은 글을 확인할 수 있었습니다.

1. **[Django issue ticket](https://code.djangoproject.com/ticket/25464)**

    해당 티켓을 보면 `Category.objects.filter(type=5).prefetch_related('items')`이라는 queryset을 실행할 때 type이 5인 Category가
    10만개 있다면 WHERE IN clause에 10만개의 items id가 들어가게 된다고 나와있습니다.\
    <br>
    이러한 WHERE IN clause는 다음과 같은 이유로 불필요할 수 있다고 적혀있습니다.
      1. 불필요하고
      2. DB가 실행해야 할 SQL statement가 multi-megabyte 수준으로 커질 가능성이 있고
      3. DB query planner가 쿼리 실행 계획을 짜는데 혼선을 줄 가능성이 있고
      4. doesn't scale(확장하지 않는다? 사실 무슨말인지 잘 모르겠습니다...)

    <br>
    티켓을 만드신 분이 제안한 PR은 `filter_on_instances`라는 parameter를 select_related()에 넘겨주어 WHERE IN clause를 생성하지 않도록
    설정할 수 있게 하는 것이었습니다.

    [PR 페이지](https://github.com/django/django/pull/5356)를 보면 반응도 굉장히 긍정적이었는데 closed 처리되고 더이상의 논의는 없는 것 같습니다.

    Django 3.2 버전의 django/db/models/fields/related_descriptors.py 소스 코드를 확인해 보아도 get_prefetch_queryset() 메서드에
    파라미터가 아직 추가되지 않은 것으로 보아 WHERE IN clause를 추가하지 않도록 하는 기능은 추가되지 않은 것 같습니다.

    Django document를 보면 다음과 같이 적혀있습니다.
    > prefetch_related in most cases will be implemented using an SQL query that uses the ‘IN’ operator. This means that for a large QuerySet a large ‘IN’ clause could be generated, which, depending on the database, might have performance problems of its own when it comes to parsing or executing the SQL query. Always profile for your use case!

    조심하라고 하네요!

2. **[Stack Overflow: query slower with prefetch related than without](https://stackoverflow.com/questions/25179934/query-slower-with-prefetch-related-than-without)**

    Craig Labenz님이 작성하신 답변을 보면 아래와 같은 글을 확인할 수 있습니다.
    > If you examine the queries Django executes, you'll see that prefetch_related() leads to an additional query with a WHERE IN (ids) clause. Those values are then mapped back to the original object in Python, preventing additional database load.

    공식 document는 아니지만 Labenz님의 말을 믿어보자면 WHERE IN clause는 불필요한 추가적인 db 호출을 방지하기 위해 존재한다고 합니다.

    솔직히 잘 이해가 가지 않네요...

<br>
---
<br>

### 여러 테스트를 해본 결과 WHERE IN clause가 있는 이유를 금방 찾을 수 있었습니다.
<br>

`Chef.objects.prefetch_related('licence')` 라는 코드를 실행했을 때의 쿼리를 보면 아래와 같은 WHERE IN clause가 발생하였고,

```
WHERE
    "chef_licences"."chef_id" IN (1, 2, 3, 4, 5, 6, 7)
```

`Chef.objects.filter(grade=2).prefetch_related('licence')`라는 코드를 실행했을 때의 쿼리를 보면 아래와 같은 WHERE IN clause가 발생하였습니다.
```
WHERE
    "chef_licences"."chef_id" IN (1, 2, 3)
```

id가 1, 2, 3인 요리사들은 등급이 2등급인 요리사들입니다.

몇번에 테스트에서 filter를 걸지 않았을 때 모든 베이스 테이블에 있는 데이터들의 id들이 WHERE IN clause 내에 포함되지 않아 헷갈렸습니다.

정확히 아시는분 계실까요?!