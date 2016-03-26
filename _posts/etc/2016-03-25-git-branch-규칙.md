---
layout: post
title: 성공적인 Git branch 관리 Model
excerpt: "Etc"
tags: [git]
comments: true
---
Git의 가장 큰 장점인 branch라는 개념을 효율적으로 쓰는 방법에 대해 정리하였다.  

### Overview
요즈음 회사에서 진행하는 프로젝트들의 형상 관리 툴로 점점 `Git`을 선택하고 있어 기왕 사용하는거 일반적으로 권장하는 branch 관리 전략에 대해서 한번 정리를 할 필요성을 느끼고 있었던 중 우연한 기회에 아래 블로그에 대한 글을 발견하고 관련 내용을 정리하였다.

* [A successful Git branching model](http://nvie.com/posts/a-successful-git-branching-model/)
* [A successful Git branching model(번역)](http://dogfeet.github.io/articles/2011/a-successful-git-branching-model.html)

> 그동안은,, 뭐랄까 한번 데이고 나서 release 시에 해당 버전을 branch로 남겨두어 이후 발생하는 버그에 대한 `hotfix`를 진행하고 적절하게 `merge` 혹은 `cherrypick`을 진행하였다. release시마다 branch가 1개씩 늘어나게 운영을 했었기 때문에 효율적이지 못하게 운영하고 있다. ~~이러한 비효율을 경험했기 때문에 방법들에 대해 고민했을거라고 생각하는 면도 있지만.~~



현업이나 개인 펫 프로젝트에서 사용해가며 보완, 응용하여 나에게 맞는 스타일을 찾아가고자 한다.

### 주요 Branch

주요 브랜치 두 개는 항상 유지한다.

* master
* develop

기본적으로 `master`는 곧 베포할 코드를 보관하며 매일 빌드하는 개발중인 코드를 `develop`에 보관한다. `develop` branch에 보관하는 코드가 안정되고 release를 할 준비가 완료되면 `master`로 merge하고 `TAG`를 단다. `master`에 `develop` branch를 merge할 때는 항상 새로운 버전이 release 되는 것을 의미한다고 봐도 좋다.

이와 같은 규칙을 통해 `jenkins`와 같은 빌드 툴을 이용해 `master` branch에 commit이 될 때 `TAG`를 달고 binary를 생성하는 과정을 자동화하고, 이를 staging 서버에 베포할 수도 있다.

### 보조 Branch

`master`와 `develop` branch 말고도 유용한 몇가지 branch도 필요하다.

1. release
2. hotfix
3. feature

#### release

`release` branch는 베포를 준비하는 branch이다. `develop` branch에서 작업하던 코드를 release하기에 앞서 버전 넘버 부여, 버그 수정, 검증 등 제품 release 전에 해야 할 활동들을 하는 branch이다.
> `develop` branch와 분리되어 있기 때문에 `release`를 준비하면서 `develop` branch도 사용할 수 있다는 장점도 있다. 추후 merge에서 약간은 피곤해질 수 있겠지만.(코드 프리징을 하는 것을 권장해야 할 것 같다)

`release` branch를 만드는 시점은 `develop` branch가 베포할 수 있는 상태가 되었다고 판단 되었을 때이다. 이때, 베포하고자 하는 기능이 `develop` branch에 merge되어 있어야 하고, 이번에 베포되지 않을 기능의 경우에는 `release` branch를 만들 때까지 기다렸다가 포함해야 한다.

`release` branch는 프로젝트의 버전 정책에 맞는 `release-버전넘버`로 생성한다.  

- e.g. release-2.1

> 특정 기능을 베포하고자 할 때 뿐만 아니라, 주기적인 릴리즈(에자일에서 말하는) 시에도 `release` branch를 만들기도 한다.

`release` branch의 주기는 아래와 같다:

1. `develop` branch로부터 `release` branch 생성  

    ``` sh
    $ git checkout -b release-2.1 develop
    ```

2. 코드 내의 버전 정보와 같은 메타 데이터를 알맞게 수정한다.

    ``` sh
    # 버전을 바꾸는 script 실행 혹은 행위
    $ ./change-version.sh -v 2.1
    ```

3. release를 위한 검증을 진행한다.
4. 해당 release 버전 검증 중에 발견되는 버그는 우선 `release` branch에서 수정한다.
5. `3 ~ 4`를 반복한다.
6. release될 준비가 되었다고 판단되면 베포한다.
7. `release` branch를 `master` branch로 merge한다.

    ``` sh
    $ git checkout master
    $ git merge --no-ff release-2.1
    ```

8. 버전정보를 포함한 `TAG`를 작성한다.

    ``` sh
    $ git tag -a 2.1
    ```

9. `develop` branch에도 `release` branch의 수정사항을 merge한다.

    ``` sh
    $ git checkout develop
    $ git merge --no-ff release-2.1
    # conflict가 난다면 적절하게 수정...
    ```

10. `release` branch를 삭제한다.

    ``` sh
    $ git branch -d release-2.1
    ```
