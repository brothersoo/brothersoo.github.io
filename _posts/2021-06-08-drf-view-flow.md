---
title: "DRF View(2) Flow"
categories:
  - Django Rest Framework
tags:
  - DRF
  - REST
---

## DRF View의 흐름

정말 간단한 ListAPIView의 예시를 하나 작성해보자면
```
from rest_framework import generics

class SomeListAPIView(generics.ListAPIView):
    queryset = Person.objects.all()
    serializer_class = SomeSerializer
```
이게 다입니다.

urls.py에 설정한 SomeListAPIView의 endpoint를 GET method로 호출하게 된다면 Person 테이블의 모든 데이터를 조회할 수 있게 됩니다.

처음 DRF를 접했을 때 이 View가 도대체 어떻게 돌아가는건지 너무나도 헷갈렸습니다. Serializer는 무엇이고, 두줄 뿐인 코드에서 어떻게 이런 값들이 반환될
수 있는 것인가 했습니다.

물론 python은 완전한 객체 지향 언어가 아니지만, 객체 지향스럽게 코딩을 할 수 있고, DRF의 view는 class based programming으로 짜여졌습니다.
객체 지향 언어에 익숙하지 않았던 터라 상속받은 ListAPIView가 어떻게 생겼는지 확인할 생각을 하지 못했습니다.

저는 mac os위에서 vs code로 코딩을 하여 cmd+좌클릭을 하면 해당 소스 코드를 확인할 수 있습니다.
ListAPIView에 들어가보게 되면
```
class ListAPIView(mixins.ListModelMixin,
                  GenericAPIView):
    """
    Concrete view for listing a queryset.
    """
    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)
```
위와 같이 생겼습니다.

get() 메서드만 있으니 GET method만 사용하여 api를 호출할 수 있습니다.

그렇다면 self.list()는 어떻게 생겼나 보기 위해 들어가보면,
```
class ListModelMixin:
    """
    List a queryset.
    """
    def list(self, request, *args, **kwargs):
        queryset = self.filter_queryset(self.get_queryset())

        page = self.paginate_queryset(queryset)
        if page is not None:
            serializer = self.get_serializer(page, many=True)
            return self.get_paginated_response(serializer.data)

        serializer = self.get_serializer(queryset, many=True)
        return Response(serializer.data)
```
이렇게 생긴 코드를 확인하실 수 있습니다.

여기서 ListAPIView의 get() 메서드 내에서 사용한 self.list()는 상속받은 ListModelMixin의 list()라는 것을 확인할 수 있습니다.

이제 ListAPIView를 호출했을 때 어떠한 흐름으로 작업이 이루어지는지 대강 알 수 있게 되었습니다.

1. self.get_queryset()을 통한 queryset 가져오기
2. Filtering 조건을 정의해놓았다면 queryset filtering 하기
3. Filter된 queryset paginate 하기. Custom하게 정의한 pagination_class가 없다면 default pagination 진행.
4.
    1. Pagination의 결과가 있다면 queryset serialize 후 pagination 양식에 맞추어 response
    2. Pagination의 결과가 없다면 queryset serialize 후 response

get_queryset(), filter_queryset(), get_serializer() 등의 메서드들은 GenericAPIView에 포함되어 상속받아 사용할 수 있습니다.

ListAPIView 뿐만이 아니라 ListCreateAPIView, RetrieveUpdateView, DestroyView와 같은 다른 view들도 위와 같은 방법으로 어떤 흐름으로
작동되는지 확인하실 수 있습니다.