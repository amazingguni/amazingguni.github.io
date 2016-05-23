---
layout: post
title: The Python ssl extension was not compiled. Missing the OpenSSL lib?
excerpt: "python"
tags: [python, pydev]
comments: true
---

`pydev`를 사용해 특정 버전을 설치할 때 아래와 같은 에러와 함께 실패할 경우에 대한 트러블 슈팅입니다.

### 증상

``` sh
$ pyenv install -v 3.5.1
Installing Python-3.5.1...
WARNING: The Python bz2 extension was not compiled. Missing the bzip2 lib?
WARNING: The Python readline extension was not compiled. Missing the GNU readline lib?
ERROR: The Python ssl extension was not compiled. Missing the OpenSSL lib?
 
 Please consult to the Wiki page to fix the problem.
 https://github.com/yyuu/pyenv/wiki/Common-build-problems
 BUILD FAILED (Ubuntu 16.04 using python-build 20150601-11-g36c5cbf)
```

### 해결

*libssl-dev* 를 설치한 이후에 다시 시도합니다

``` sh
$ sudo apt-get install libssl-dev
$ pyenv install -v 3.5.1
# install 이 진행될 것입니다
```

### Reference

[Install pyenv on Ubuntu 15.04 – The Code Heaven](https://bjlee72.wordpress.com/2015/07/09/install-pyenv-on-ubuntu-15-04/)

