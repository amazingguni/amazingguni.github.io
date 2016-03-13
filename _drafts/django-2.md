---
layout: post
title: Django 프로젝트 테스트 환경 구축 1 - 첫 Django Project
excerpt: "Django"
tags: [python, django, test]
comments: true
---

*Django 프로젝트 테스트 환경 구축* 은 *파이썬을 이용한 클린 코드를 위한 테스트 주도 개발* 를 보며 정리를 위해 작성하는 블로그이다.  
해당 블로그 및 예제는 `Ubuntu 15.10`에서 작성되었다.  


### Django란 무엇인가?
테스트 환경에 대해 이야기하기에 앞서 *Django*가 무엇인지 간단하게 짚고 넘어가보자.  
*Django*는 *Python*으로 만들어진 *Web Application Framework* 이고 [오픈소스](https://github.com/django/django)로 공개되어 있고 **공짜로 이용 가능**하다. 
또한, 웹 개발자들이 아이디어를 구상하고 구현하기까지 고려해야할 여러가지 것들을 대신 해주어 개발 속도가 **놀라울 정도로 빨라진다**~~고 한다~~.  

### Django 설치
`Python 3`를 사용할 예정이기 때문에 아래 명령어로 설치한다.
{% highlight sh %}
sudo apt-get install python3
{% endhighlight %}

python3 package 설치를 위해 `pip3`를 설치한다.  

- Python3 package를 설치하기 위한 툴

{% highlight sh %}
sudo apt-get install python3-pip
{% endhighlight %}

`Django`를 설치한다.
{% highlight sh %}
sudo pip3 install django
{% endhighlight %}

### Django 프로젝트 생성
`pomodoro_web`이라는 이름으로 프로젝트를 생성한다.
   
- 추후 해당 내용으로 블로그를 이어 나갈 예정이다.
{% highlight sh %}
django-admin startproject pomodoro_web
{% endhighlight %}

위 명령어를 실행하고 나면 아래와 같이 프로젝트가 생성된다.
{% highlight sh %}
pomodoro_web/
├── manage.py
└── pomodoro_web
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
{% endhighlight %}

- `manage.py`는 *Django project*로 다양한 작업들을 할 수 있도록 도와주는 Utility이다.  
  - 자세한 옵션에 대해서는 Django 공식 홈페이지의 [django-admin and manage.py](https://docs.djangoproject.com/en/1.9/ref/django-admin/)를 참고바란다.  
- `pomodoro_web`  
  - 프로젝트 이름과 동일한 Directory가 생성되는 것을 볼 수 있는데 사이트 전체에 Global하게 적용되는 설정들을 저장하는 곳이라고 보면 된다.  
  - 또한 해당 디렉토리는 *Python package name*으로 사용되어 *Django Project*내에서 import할 수 있다.(e.g., `import pomodoro_web.urls`)  
- `pomodoro_web/settings.py`는 *Django project*의 [다양한 설정](https://docs.djangoproject.com/en/1.9/topics/settings/)들을 저장한다.  
- `pomodoro_web/urls.py`는 *Django project*의 URL 정의를 저장한다.  
- ~~`pomodoro_web/wsgi.py`는 아직 잘 모르겠다.(TODO)~~  

<br>
위에서 설명한 `manage.py`를 이용해서 server를 가동하면 당신의 첫번째 *DJango Project*가 실행된다.
{% highlight sh %}
python3 manage.py runserver

March 06, 2016 - 09:08:29
Django version 1.9.2, using settings 'pomodoro_web.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
{% endhighlight %}
> Port를 변경하고자 할 경우에는 아래와 같이 parameter로 추가하여 호출하면 된다.
`python3 manage.py runserver 8081`

비록 내용은 없지만 `http://localhost:8000/`에 접속하면 *Django Default 접속 화면*을 볼 수 있다.

### Django 어플리케이션
*Django*는 웹 서비스를 `app`의 형태로 구조화할 수 있다.  
`app`을 생성할 때는 아래 명령어를 사용한다.  

- 해당 블로그에서는 task들을 보여주는 app을 만들 예정이기 때문에 *tasks*로 명명했다.
{% highlight sh %}
python3 manage.py startapp tasks
{% endhighlight %}

프로젝트 내에 아래와 같은 `tasks` 디렉토리가 생성된 것을 확인할 수 있다.
{% highlight sh %}
tasks
├── admin.py
├── apps.py
├── __init__.py
├── migrations
│   └── __init__.py
├── models.py
├── tests.py
└── views.py
{% endhighlight %}

