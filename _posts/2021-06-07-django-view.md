---
title: "Django View(1)"
categories:
  - Django
tags:
  - DRF
  - DRF view
  - MTV pattern
---

** Django가 아닌 Django Rest Framework에 관련된 포스팅입니다.\
** DRF를 사용하기 위해서는 pip install을 해주셔야 합니다.

## Django Rest Framework View

Django Rest Framework는 Django를 베이스로 RESTful한 서버를 쉽게 구축할 수 있도록 도와주는 프레임워크 입니다.

DRF의 view는 각각 endpoint를 가지고 있으며, 한개 이상의 HTTP 메서드를 허용하며 그 메서드에 맞는 기능을 수행합니다.


## DRF View inheritance hierarchy
아래의 이미지는 DRF의 View class들이 어떤식으로 상속되어있는지 보여주는 다이어그램입니다.

![drf_view_inheritance_diagram]({{ site.url }}{{ site.baseurl }}/assets/images/drf_view_inheritance_diagram.png)

다이어그램 가장 왼쪽을 보게되면 ListAPIView, CreateAPIView 등 다양한 View class들이 있는 것을 확인할 수 있습니다.

각각의 view class들은 이름에 포함된 기능들을 수행할 수 있습니다.

예를들어 ListAPIView는 pagination이 적용된 list를 반환하는 기능만을 가지고 있고, CreateAPIView는 데이터를 생성하는 기능만 수행할 수 있고, RetrieveUpdateAPIView는 조회 및 수정 기능을 모두 가지고 있습니다.

즉, 이 경우에는 하나의 endpoint를 GET method를 사용하여 호출하면 Retrieve
기능을 수행하며, PUT 혹은 PATCH method를 상요하여 호출하면 Update 기능을 수행하게 됩니다.

이게 어떻게 가능한가 보면 각각의 View는 자신이 수행해야 할 기능을 가지고 있는 Mixin들을 상속받고 있습니다.