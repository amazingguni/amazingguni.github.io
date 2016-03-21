---
layout: post
title: Django 프로젝트 테스트 환경 구축 3 - models.py
excerpt: "Django"
tags: [python, django, test, database, model]
comments: true
---

*Django 프로젝트 테스트 환경 구축* 은 *파이썬을 이용한 클린 코드를 위한 테스트 주도 개발* 를 보며 정리를 위해 작성하는 블로그이다.  
해당 블로그 및 예제는 `Ubuntu 15.10`에서 작성되었다.  

###Model?
`Model`은 간단히 말해 data이다.  
2가지 개념으로 나누어 볼 수 있는데:

1. Class 자체는 table을 나타낸다고 할 수 있다. 할일을 담아주는 `Task` table일 수도 있고 게시판 데이터를 담고 있는 table일 수도 있다.  
django에서는 `Model`을 상속받은 이 class를 통해서 데이터 list를 가져 올 수 있다.(물론 추가/수정/삭제 도 가능하다)
2. `Model`을 상속받은 class의 instance는 *To do list*의 하나의 item일 수도 있고, 혹은 게시판의 한개의 글일 수도 있다.  
즉 database table에서의 한개의 row를 나타낸다고 보는 것이 지금으로써는 맞는 말인 것 같다.


###Tutorial
해당 블로그에서 만들고자 하는 web site는 일종의 `To-do-list`이기 때문에 그에 맞는 `Model`을 생성하는 예제를 통해 어떤 개념인지 가볍게 알아보도록 하자.  

우선, `To-do-list`에는 `Title`이 필요하다.  `Title`은 `text`이기 때문에 아래와 같이 만들 수 있다.

> default parameter는 해당 field의 기본값(없을경우)를 나타낸다. 아래 경우에는 title이 없다면 공백문자열이 들어간다.

``` python
#pomodoro_web/tasks/models.py
from django.db import models #models 상속을 위해 import

class Task(models.Model):#models.Model을 상속해야 한다.
    title = models.TextField(default='')
```
등록된 날짜를 저장하고자 한다면 `created_date` field도 필요하다.

> default value를 timezone.now로 두어 생성 시의 시간이 들어가도록 한다. timezone을 사용하기 위해 import 문에 추가한다.

``` python
#pomodoro_web/tasks/models.py
from django.db import models
from django.utils import timezone

class Task(models.Model):
    title = models.TextField(default='')
    created_date = models.DateTimeField(default=timezone.now)
```

여기까지 진행하면 model에 대한 정의(?)는 완료된다. spring에서 `mybatis`를 사용할 때와 같이 `create table`을 통해 query문을 생각하고 호출하는 절차 없이 아주 간편하게 database 구조를 설계할 수 있다!!  
다른 field를 사용하고자 한다면 [django 공식 홈페이지의 model reference](https://docs.djangoproject.com/en/1.9/ref/models/fields/)를 참고하도록 하자.

> 우리 회사에서는 `mybatis`를 사용하고 있는데 query 수정도 복잡할 뿐더러 디버깅이 어렵고 테스팅 코드 짜기에도 쉽지가 않다..  
Rawquery가 필요할 경우가 있다고는 하지만 `django의 model`도 rawquery를 지원하기 때문에 변명이 될 것 같지는 않다 ㅠㅠ

가벼운 테스트 코드를 통해 해당 model이 어떻게 사용되는지 보도록 하자.

###Usage
`test.py`에 해당 api를 사용하는 code를 작성해보자.  

``` python
from django.test import TestCase
from tasks.models import Task

class TaskModelTest(TestCase):
    def test_saving_and_retrieving_tasks(self):
        first_task = Task() #하나의 model를 생성하는데,,
        first_task.title = 'first task' # title은 'first task'이다
        first_task.save() #'정의한 data를 저장(commit?) 한다.

        second_task = Task()
        second_task.title = 'second task'
        second_task.save()

        saved_tasks = Task.objects.all() #task table에 있는 objects들을 다 불러왔는데
        self.assertEqual(saved_tasks.count(), 2) # 2개인지 확인하고

        first_saved_task = saved_tasks[0] # 첫번째 item을 담아서
        second_saved_task = saved_tasks[1]

        print('title:%s created_date:%s' %(first_saved_task.title, first_saved_task.created_date)) #출력해보았다.
        #저장한대로 출력이 되었다!!
        print('title:%s created_date:%s' %(second_saved_task.title, second_saved_task.created_date))
```

해당 코드를 작성한 이후 `python manage.py test`를 통해 testcode를 실행시킨다.

``` sh
Creating test database for alias 'default'...
/home/amazingguni/workspaces/pomodoro_web/pomodoro_web/urls.py:20: RemovedInDjango110Warning: Support for string view arguments to url() is deprecated and will be removed in Django 1.10 (got tasks.views.home_page). Pass the callable instead.
  url(r'^$', 'tasks.views.home_page', name='home')

......title:first task created_date:2016-03-20 14:02:04.607102+00:00
title:second task created_date:2016-03-20 14:02:04.607413+00:00
.
----------------------------------------------------------------------
Ran 7 tests in 0.030s

OK
Destroying test database for alias 'default'...
```

데이터의 저장이 성공적으로 완료되었기 때문에 해당 내용이 print 되는 것을 확인 할 수 있다.  


위의 `Creating test database for alias 'default'...`, `Destroying test database for alias 'default'...`를 보면 알 수 있듯이,
`django`에서는 test를 위해 임시 database를 만들어주고 다시 없애는 과정을 해주고 있다. 실제 프로젝트(runserver를 통해 구동되는)에서는 자동으로 database가 생성되지 않는다..  
그래서, `models.py`에 지정된 database를 생성하는 명령어를 아래와 같이 입력해주어야 정상적으로 database 생성이 완료된다.

``` sh
python manage.py makemigration
python manage.py migrate
```

성공적으로 완료되었다면 `django project`에서 `Task` model을 추가/수정/삭제/혹은 더 많은 것들을 할 수 있는 준비가 완료된 셈이다.


여기까지 간략한 model에 대한 설명을 마무리 짓도록 하겠다.

> 중간중간, `foreign key`라든지, `1대다 관계 매핑`이라든지 하는 부분에 대해서는 필요할 때 해당 포스트를 수정하거나 새로운 포스트를 작성할 예정이다.~~실은 지금 전혀 모르고 있기도 해서..~~


-----
####reference
[Models | Django documentation | Django](https://docs.djangoproject.com/en/1.9/topics/db/models/)  
[Django 모델 | Django Girls Tutorial](http://tutorial.djangogirls.org/ko/django_models/index.html)
