---
layout: post
title: graphene-django의 Datetime을 한국시간으로 표현
excerpt: "python"
tags: [python]
comments: true
---

django는 기본적으로 `USE_TZ=True` 를 사용할 때에는 UTC로 그 값을 저장합니다.

그런데 한국 시간 기준으로 서비스를 운영하는 경우(예를 들어 한국에서 주최하는 행사의 일시)에는 UTC보다는 한국 시간으로 Timezone을 변환해서 전달하는게 더 직관적일 수 있습니다.

- 물론 USE_TZ=False를 하면 문제가 해결되기는 합니다. 하지만 mysql은 timezone_aware를 지원하지 않기 때문에, 개발 환경에서 mysql을 사용하는 경우에는 이 방법을 사용할 수 없습니다.

따라서 이번 포스트에서는 graphene에서 Asia/Seoul로 시간을 변환해서 반환하는 Field를 정의하는 방법을 설명합니다.

## Field 정의

```py
import datetime
import graphene
import pytz
from graphene_django import DjangoObjectType

class SeoulDateTime(graphene.types.Scalar):
    '''
    It is used to replace timezone to Seoul for graphene queries.
    When using the time field, specify scala as follows and return it in Korean time.
    e.g)
    started_at = graphene.Field(SeoulDateTime)
    finished_at = graphene.Field(SeoulDateTime)
    '''

    @staticmethod
    def serialize(obj):
        timezone = pytz.timezone('Asia/Seoul')
        return obj.astimezone(tz=timezone).isoformat()

    @staticmethod
    def parse_literal(node):
        return datetime.datetime.strptime(
            node.value, "%Y-%m-%dT%H:%M:%S.%f")

    @staticmethod
    def parse_value(value):
        return datetime.datetime.strptime(value, "%Y-%m-%dT%H:%M:%S.%f")
```

## Schema 정의

```py
from graphene_django import DjangoObjectType
from api.models.program import Presentation

class PresentationNode(DjangoObjectType):
    class Meta:
        model = Presentation
        description = """
        Program which speakers present their presentations at Pycon Korea.
        It is one of the the most important program in Pycon Korea.
        """
    started_at = graphene.Field(SeoulDateTime)
    finished_at = graphene.Field(SeoulDateTime)

class Query(graphene.ObjectType):
    presentations = graphene.List(PresentationNode)

    def resolve_presentations(self, info):
        return Presentation.objects.all()
```

## Reference

자세한 코드는 [PyCon Korea API 서버](https://github.com/pythonkr/pyconkr-api)에서 볼 수 있습니다

https://www.notion.so/amazingguni/graphene-django-Datetime-743af44ee03c48539aef74561f3cad1d
