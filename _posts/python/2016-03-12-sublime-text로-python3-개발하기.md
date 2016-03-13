---
layout: post
title: SublimeText3 에서 Python3 개발 환경 구축하기(SublimeREPL)
excerpt: "python"
tags: [python, sublime text]
comments: true
---

`SublimeText3`는 아주 강력한 ~~이쁜~~ Text Editor라서 3년 전쯤 충동적으로 구매를 했었는데,  
이번 `Django` 프로젝트를 시작하면서 처음 개발 환경으로 사용해보았다.
> 기존에는 syntax highlighting 기능이 다른 에디터에 비해서 이쁘장하게 나와서, 간단한 `script`나 `json`, `xml`등의 문서를 보는데 주로 사용했었다.

`Sublime Text 3`에 프로젝트를 세팅해서 빌드를 진행해 보았을 때 아래 2가지 문제가 발견되었다.

1. 기존 `ctrl+b`로는 `input()` 와 함수에 사용자 입력을 줄 수 없다.
2. `ctrl+b`로 빌드 시에 assert 구문의 출력이 콘솔에 보이지 않았다.
3. `/usr/bin/python`을 2.7로 두어야 했는데 내 django 프로젝트는 python3.4로 개발되고 있다.

> 아래와 같은 코드를 작성했을 경우, `SublimeText`에서 제공하는 기본 빌드로는 정상적으로 구동확인이 어렵다.
{% highlight python %}
a = input()
b = input()
print(a+b)
{% endhighlight %}

그래서 선택한 것이 `SublimeREPL` 이다.

### SublimeREPL

설치 방법은 아래와 같다.  
 - `Package Control`은 설치되어 있다고 가정한다.

1. `ctrl+shift+p`로 `command palette`를 연다. 
2. `Package Control - Install Package`를 입력하고 `Enter`
3. `SublimeREPL`을 입력하고 `Enter`를 눌러 설치한다.  
![SublimeREPL 설치]({{site.url}}/assets/images/post/python/2016-03-12/install-repl.png)
4. 설치 완료
5. `sublime text` 재실행(안해도 될지도..)
6. `ctrl+shift+p`로 `command palette`를 열고 SublimeREPL: Python - run current file을 통해 빌드 및 실행을 하면 아래와 같이 사용자 입력이 가능하다.  
![SublimeREPL 실행]({{site.url}}/assets/images/post/python/2016-03-12/sublimeREPL-run.png)

### virtualenv - new

`virtualenv`는 Python 개발 환경을 프로젝트 별로 각각 다르게 가져가는 경우에 사용하면 유용한 도구입니다.  
> [Django Girls Tutorial - Django 설치하기](http://tutorial.djangogirls.org/ko/django_installation/index.html)에 잘 설명되어 있으니 참고 바랍니다.

`Sublimetext`에서는 이 `virtualenv`를 편하게 선택하고 사용할 수 있는 기능을 지원합니다.  

1. `Package Control - Install Package` 로 들어간 이후 `virtualenv` 설치
2. `ctrl+shift+p` 에서 `virtualenv-new` 실행
3. 이름을 입력하고 `vitautlenv` 생성  
![virtualenv-new]({{site.url}}/assets/images/post/python/2016-03-12/virtualenv-new.png)
4. 팝업이 뜨면 사용할 Python version 선택
5. 새로운 virtualenv 생성 완료

### SublimeREPL: Python - Run current file


생성된 virtualenv를 `SublimeText` 상에서 activate 시켜도 SublimeREPL: Python - Run current file에는 적용되지 않는 문제가 발견되었다.~~해결방법을 찾으려 했으나 실패.~~  
그래서 아래와 같이 `terminal`상에서 virtualenv를 activate 시키고 `subl`를 실행하여 적용시켰다.

{% highlight sh %}
> source ~/.virtualenv/pomodoro_web/bin/activate
> subl
{% endhighlight %}

`SublimeREPL: Python - Run current file`이 virtualenv의 python 환경을 따르는 것을 확인 할 수 있다.  
![run-current-file]({{site.url}}/assets/images/post/python/2016-03-12/REPL_run_current_file.png)





















