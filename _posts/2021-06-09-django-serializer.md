---
title: "DRF Serializer(1)"
categories:
  - Django Rest Framework
tags:
  - DRF
---

## DRF serializer

Serialize는 직렬화라는 의미를 가지고 있고, 이는 복잡한 수준의 데이터를 단순화 하는 의미를 가지고 있습니다.

DRF에서의 serializer는 queryset, model instances과 같은 복잡한 데이터를 python에서 사용할 수 있도록 변환해줍니다.

Python data type으로 변경된 데이터는 프런트로 전송될 수 있도록 JSON 혹은 XML 등의 데이터 타입으로 쉽게 변환될 수 있습니다.

Serializing 뿐만이 아닌 deserializing도 가능합니다.

아래 코드는 serializer의 간단한 예제입니다.

```
from rest_framework import serializers

class CommentSerializer(serializers.Serializer):
    email = serializers.EmailField()
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()
```

이후 View에서 아래와 같은 코드를 수행하게 된다면 표기된 형식의 데이터를 얻을 수 있습니다.

```
serializer = CommentSerializer(comment)
serializer.data

# serializer.data
{
    'email': 'leila@example.com',
    'content': 'foo bar',
    'created': '2016-01-27T15:17:10.375877'
}
```