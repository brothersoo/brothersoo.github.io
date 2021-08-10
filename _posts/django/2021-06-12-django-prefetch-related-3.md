---
title: "Django prefetch_related(3)"
categories:
  - Django
tags:
  - Django
  - query optimization
  - ORM
permalink: /django/:title
---
<br>
# Custom prefetch_related

실제로 prefetch_related를 사용하여 추가로 불러오는 데이터셋에 추가 처리를 해야 하는 경우가 많이 발생합니다.

예를 들어 Store에서 근무하는 요리사들을 모두 prefetch_related를 사용하여 한번에 불러올 때 요리사에도 특정 조건을 부여하고 싶은 경우도 있을 수 있습니다.

```python
# Modeling

class Chef(models.Model):
    name = models.CharField()
    grade = models.IntegerField()
    licences = models.ManyToMany(Licence)
    store = models.ForeignKey(Store)

    class Meta:
        db_table = 'chef'


class Licence(models.Model):
    name = models.CharField()
    difficulty = models.IntegerField()
    preference = models.IntegerField()

    class Meta:
        db_table = 'licnece'


class Store(models.Model):
    name = models.CharField()
    address = models.CharField()

    class Meta:
        db_table = 'store'

class Menu(models.Model):
    name = models.CharField()
    store = models.ForeignKey(Store, on_delete=models.CASCADE)
    chef = models.OneToOneField(Chef, on_delete=models.CASCADE)

    class Meta:
        db_table = 'menu'
```

```python
from django.db.models.query import Prefetch


#  모든 Store 데이터와 모든 Chef 데이터를 가져옵니다.
Store.objects.prefetch_related('chef_set')


#  모든 Store 데이터와 등급이 2등급인 Chef 데이터를 가져옵니다. Chef를 id로 오름차순 정렬하는 것은 덤!
Store.objects.prefetch_related(
  Prefetch(
    'chef_set',
    queryset=Chef.objects.filter(grade=2).order_by(id)
  )
)


# Nested relation
Store.objects.prefetch_related(
  Prefetch(
    'chef_set',
    queryset=Chef.objects.select_related(
      'menu'
    ).filter(
      grade=2
    ).prefetch_related(
      Prefetch(
        'licence_set',
        queryset=Licence.objects.filter(preference=1)
      )
    )
  )
)
```