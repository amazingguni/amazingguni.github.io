---
layout: post
title: Django 프로젝트 테스트 환경 구축 2 - unittest란?
excerpt: "Django"
tags: [python, django, test]
comments: true
---

*Django 프로젝트 테스트 환경 구축* 은 *파이썬을 이용한 클린 코드를 위한 테스트 주도 개발* 를 보며 정리를 위해 작성하는 블로그이다.  
해당 블로그 및 예제는 `Ubuntu 15.10`에서 작성되었다.  


### unittest란?
`unittest` 모듈은 JUnit과 동일한 기능을 제공하는 Python 모듈이다.(PyUnit이라고도 불린다)  
python 2나 3을 사용하지 않는다면 대부분 기본적으로 설치되어 있어 import만으로 사용이 가능하다.

### Sample Code
{% highlight py %}
from selenium import webdriver
import unittest

class CommonTest(unittest.TestCase):
    def setUp(self):
        self.browser = webdriver.Firefox()
        self.browser.implicitly_wait(3)

    def tearDown(self):
        self.browser.quit()

    def test_site_title(self):
        self.browser.get('http://localhost:8000')
        self.assertIn('Pomodoro web', self.browser.title)

if __name__ == '__main__':
    unittest.main(warnings='ignore')
{% endhighlight %}

1. `unittest.TestCase`를 상속한 클래스 형태의 테스트를 만든다.
2. `setUp()`: 테스트 코드 실행 전에 실행할 설정 작업들을 모아둔 함수이다.(precondition을 설정해야 할 경우)
 * 위에서는 browser engine을 설정하였다
 * `implicitly_wait()`는 3초간 대기한 이후에 진행되도록 sleep을 하는 코드이다.
3. `tearDown()`: 테스트 코드를 실행한 이후에 마무리를 하는 코드를 입력한다.
 * `test code`나 `setup` 함수에서 에러가 발생한다고 하더라도 무조건 실행되는 코드이다.(try-catch-finally의 finally같은 느낌?)
 * 해당 코드에서는 browser를 종료하는 코드가 들어갔다.
4. test함수는 항상 `test`를 앞에 붙여주어야 한다.
 * 이 규칙만 따른다는 class 내에 테스트 함수를 여러개 만들 수 있다.
5. self.assertIn(a, b): a가 b에 있는지 확인한다.
 * 위의 예제에서는 a와 b가 string 이기 때문에 해당 문자열이 포함되었는지 확인한다.

> 자세한 assert문의 종류에 대해서는 [unittest 공식 문서](https://docs.python.org/3/library/unittest.html)를 참고 바란다.  
`assertEqual`, `assertTrue`, `assertFalse` 등은 자주 씌이는 assert문이다.

위 코드를 실행한 결과는 아래와 같다.
{% highlight py %}
F
======================================================================
FAIL: test_site_title (__main__.CommonTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "functional_tests.py", line 13, in test_site_title
    self.assertIn('Pomodoro web', self.browser.title)
AssertionError: 'Pomodoro web' not found in 'Welcome to Django'

----------------------------------------------------------------------
Ran 1 test in 3.390s

FAILED (failures=1)
{% endhighlight %}

웹 페이지의 이름을 정하지 않았기 때문에 당연히 실패로 끝난다.

