---
title: "DRF Serializer(2) save and update"
categories:
  - Django Rest Framework
tags:
  - DRF
permalink: /django/:title
---

## save(), update() method

CreateAPIView 혹은 UpdateAPIView를 사용한다면 생성 혹은 수정 작업을 하기 위해서일 것입니다.

이런 생성과 수정 또한 serializer에서 수행이 가능합니다.

Create의 예시로 CreateAPIView와 그가 상속받는 CreateModelMixin의 소스를 확인한다면 아래와 같은 코드를 확인할 수 있습니다.

```
class CreateAPIView(mixins.CreateModelMixin,
                    GenericAPIView):
    """
    Concrete view for creating a model instance.
    """
    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)
```

```
class CreateModelMixin:
    """
    Create a model instance.
    """
    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        self.perform_create(serializer)
        headers = self.get_success_headers(serializer.data)
        return Response(serializer.data, status=status.HTTP_201_CREATED, headers=headers)

    def perform_create(self, serializer):
        serializer.save()

    def get_success_headers(self, data):
        try:
            return {'Location': str(data[api_settings.URL_FIELD_NAME])}
        except (TypeError, KeyError):
            return {}
```

CreateModelMixin의 코드를 살펴보면
1. `serializer = self.get_serializer(data=request.data)`:\
프런트에서 넘겨준 데이터를 지정한 serializer에 대입하게 됩니다.
2. `serializer.is_valid(raise_exception=True)`:\
serializer의 validate() 메서드를 호출합니다. Serializer에서 validate() 메서드를 오버라이딩하지 않는다면 serializer의 default
validate()가 실행되는데 이 validate는 아무 검증도 하지 않고 데이터를 반환합니다.
3. `self.perform_create(serializer)`:
위 코드에서 확인할 수 있듯이 serializer.save()를 실행합니다.\
self.instance가 정의되어 있지 않다면 serializer.save()는 serializer의 create() 메서드를 호출하고,\
self.instance가 정의되어 있다면, serializer.save()는 serializer의 update() 메서드를 호출합니다.\
serializer 내에 create() 혹은 update()가 반드시 오버라이딩 되어있어야 합니다. 그렇지 않다면 에러를 발생합니다.
4. `headers = self.get_success_headers(serializer.data)`:\
위 코드에서 정의된 것과 같이 settings.py 파일에 URL_FIELD_NAME이 정의되어있다면 헤더 정보에 담아줍니다.
5. `return Response(serializer.data, status=status.HTTP_201_CREATED, headers=headers)`:\
생성된 데이터를 serializer에 정의된 바와 같이 status, headers와 함께 반환합니다.