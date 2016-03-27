---
layout: post
title: 성공적인 Git branch 관리 Model
excerpt: "Etc"
tags: [git]
comments: true
---
Git의 가장 큰 장점인 branch라는 개념을 효율적으로 쓰는 방법에 대해 정리하였다.  

## 개요

요즈음 회사에서 진행하는 프로젝트들의 형상 관리 툴로 점점 `Git`을 선택하고 있어 기왕 사용하는거 일반적으로 권장하는 branch 관리 전략에 대해서 한번 정리를 할 필요성을 느끼고 있었던 중 우연한 기회에 아래 블로그에 대한 글을 발견하고 관련 내용을 정리하였다.

* [A successful Git branching model](http://nvie.com/posts/a-successful-git-branching-model/)
* [A successful Git branching model(번역)](http://dogfeet.github.io/articles/2011/a-successful-git-branching-model.html)

> 그동안은,, 뭐랄까 한번 데이고 나서 release 시에 해당 버전을 branch로 남겨두어 이후 발생하는 버그에 대한 `hotfix`를 진행하고 적절하게 `merge` 혹은 `cherrypick`을 진행하였다. release시마다 branch가 1개씩 늘어나게 운영을 했었기 때문에 효율적이지 못하게 운영하고 있다. ~~이러한 비효율을 경험했기 때문에 방법들에 대해 고민했을거라고 생각하는 면도 있지만.~~



현업이나 개인 펫 프로젝트에서 사용해가며 보완, 응용하여 나에게 맞는 스타일을 찾아가고자 한다.

---
## 주요 Branch

주요 브랜치 두 개는 항상 유지한다.

* master
* develop

기본적으로 `master`는 곧 베포할 코드를 보관하며 매일 빌드하는 개발중인 코드를 `develop`에 보관한다. `develop` branch에 보관하는 코드가 안정되고 release를 할 준비가 완료되면 `master`로 merge하고 `TAG`를 단다. `master`에 `develop` branch를 merge할 때는 항상 새로운 버전이 release 되는 것을 의미한다고 봐도 좋다.

이와 같은 규칙을 통해 `jenkins`와 같은 빌드 툴을 이용해 `master` branch에 commit이 될 때 `TAG`를 달고 binary를 생성하는 과정을 자동화하고, 이를 staging 서버에 베포할 수도 있다.

---
## 보조 Branch

`master`와 `develop` branch 말고도 유용한 몇가지 branch도 필요하다.

1. feature
2. release
3. hotfix

### feature

`feature` branch는 곧 배포할(다음 혹은 언젠가) 기능을 개발하는 branch이다. 따로 분리된 branch에서 개발을 진행하며 기능이 완료되는 시점에 `develop` branch로 merge한다. 일반적으로 origin에 push하지 않고 local에서 생성하는 branch이다. 같이 작업한다면 push하여 작업해도 좋다. 이러한 feature 단위의 branch를 통해서 release 대상 feature를 선택하거나 버리기 용이하기 때문에 local에서 작업하더라도 feature branch를 생성해서 하는 것을 권장한다.

`feature` branch는 아래와 같이 생성하고 merge한다:

1. 새로운 기능 `myfeature` 를 추가하고자 한다.
2. `develop` branch에서 `myfeature` branch를 새로 만든다.

    ``` sh
    $ git checkout -b myfeature develop
    ```

3. 원하는 기능을 개발한다.
4. 기능이 완성되어 다음 베포에 넣기로 했다면 `develop` branch에 merge한다.

    ``` sh
    $ git checkout develop
    $ git merge --no-ff myfeature
    $ git branch -d myfeature
    $ git push origin develop
    ```

### release

`release` branch는 베포를 준비하는 branch이다. `develop` branch에서 작업하던 코드를 release하기에 앞서 버전 넘버 부여, 버그 수정, 검증 등 제품 release 전에 해야 할 활동들을 하는 branch이다.
> `develop` branch와 분리되어 있기 때문에 `release`를 준비하면서 `develop` branch도 사용할 수 있다는 장점도 있다. 추후 merge에서 약간은 피곤해질 수 있겠지만.(코드 프리징을 하는 것을 권장해야 할 것 같다)

`release` branch를 만드는 시점은 `develop` branch가 베포할 수 있는 상태가 되었다고 판단 되었을 때이다. 이때, 베포하고자 하는 기능이 `develop` branch에 merge되어 있어야 하고, 이번에 베포되지 않을 기능의 경우에는 `release` branch를 만들 때까지 기다렸다가 포함해야 한다.

`release` branch는 프로젝트의 버전 정책에 맞는 `release-버전넘버`로 생성한다. 
* e.g. release-1.2

> 특정 기능을 베포하고자 할 때 뿐만 아니라, 주기적인 릴리즈(에자일에서 말하는) 시에도 `release` branch를 만들기도 한다.

`release` branch의 주기는 아래와 같다:

1. `develop` branch로부터 `release` branch 생성  

    ``` sh
    $ git checkout -b release-1.2 develop
    ```

2. 코드 내의 버전 정보와 같은 메타 데이터를 알맞게 수정한다.

    ``` sh
    # 버전을 바꾸는 script 실행 혹은 행위
    $ ./change-version.sh -v 1.2
    $ git commit -a -m "Bumped version number to 1.2"
    $ git push origin release_1.2
    ```

3. release를 위한 검증을 진행한다.
4. 해당 release 버전 검증 중에 발견되는 버그는 우선 `release` branch에서 수정한다.
5. `3 ~ 4`를 반복한다.
6. release될 준비가 되었다고 판단되면 베포한다.
7. `release` branch를 `master` branch로 merge한다.

    ``` sh
    $ git checkout master
    $ git merge --no-ff release-1.2
    $ git push origin master
    ```

8. 버전정보를 포함한 `TAG`를 작성한다.

    ``` sh
    $ git tag -a 1.2
    $ git push origin 1.2
    ```

9. `develop` branch에도 `release` branch의 수정사항을 merge한다.

    ``` sh
    $ git checkout develop
    $ git merge --no-ff release-1.2
    # conflict가 난다면 적절하게 수정...
    $ git push origin develop
    ```

10. `release` branch를 삭제한다.

    ``` sh
    # release remote branch 삭제
    $ git push origin :release-1.2
    $ git branch -d release-1.2
    ```
    
11. 그리고 다시 `develop` branch에서 개발을 시작한다.    
    
### hotfix

베포 이후에 버그가 나오지 않는다면 참 좋겠지만, minor하건 critical하건 버그는 언제나 등장하기 마련이다. `hotfix` branch는 그런한 버그를 수정하기 위한 목적으로 만들어진다. hotfix 이후에는 새로운 베포 바이너리를 말아야(?)하기 때문에 `hotfix` branch도 버전 정보를 포함한다.

> 별도로 branch를 개설함으로써 한 개발자가 `hotfix` branch에서 버그를 수정하는 동안 다른 개발자들은 `develop` branch에서 개발을 계속 진행할 수 있다.

1.2 버전으로 베포한 버전에서 버그가 발견되어 `hotfix` branch를 개설하고 버그를 수정하는 과정은 아래와 같을 것이다.

1. 베포한 1.2 버전에서 검색이 되지 않는 버그가 발견되었다.
2. `hotfix` branch 생성

    ``` sh
    # 만약 최근 베포 버전 이전에서 발견된 버그일 경우에는 해당 TAG로 이동한 이후 생성한다.
    $ git checkout -b hotfix-1.2.1 master
    ```

3. 코드 내의 버전 정보와 같은 메타 데이터를 알맞게 수정한다. 일종의 베포이기 때문에 `release` 와 동일하게 버전을 수정해야 한다. 

    ``` sh
    # 버전을 바꾸는 script 실행 혹은 행위
    $ ./change-version.sh -v 1.2.1
    $ git commit -a -m "Bumped version number to 1.2.1"
    ```

4. 버그를 수정한다.
5. `master` branch에 merge하고 TAG를 추가한다.

    ``` sh
    $ git checkout master
    $ git merge --no-ff hotfix-1.2.1
    $ git push origin master
    $ git tag -a 1.2.1
    $ git push origin 1.2.1
    ```
    

6. `develop` branch에도 수정사항을 merge한다.
    
    ``` sh
    $ git checkout develop
    $ git merge --no-ff hotfix-1.2.1
    $ git push origin develop
    ```
    
7. `hotfix` branch를 삭제한다.

    ``` sh
    $ git branch -d hotfix-1.2.1
    ```

---
## 다른 이야기긴 하지만..

Commit 시에 아래의 룰을 지키는 것도 상당히 중요하다고 생각한다. ~~당연한 이야기지만..~~

1. 하나의 commit은 하나의 comment로 설명이 가능해야 한다. 조금 귀찮더라도 commit을 나누어 하는 것이 좋다.
   * 추후 history를 확인할 일이 생겼을 때,
   * 버그 fix한 commit을 공유할 때 다른 사람들이 이해하기가 쉽다.(변경사항 조회라든지..)

2. 팀 내의 공통으로 사용하는 formatter가 있어야 한다.
    * 없으면 conflict도 자주나고, blame하기도 어려워진다.



