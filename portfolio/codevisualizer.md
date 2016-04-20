---
layout: page
title: CodeVisualizer
permalink: /portfolio/codevisualizer/
---

2012년 7월즈음 진행한 `Code visualizer` 프로젝트에 대한 log이다.

[http://dev.naver.com/projects/codevisualizer](http://dev.naver.com/projects/codevisualizer)

### 프로젝트를 시작한 이유

2012년 즈음 [code.org](https://code.org/)의 동영상 하나가 sns에 돌아다녔었다.

> Every student in every school should have the opportunity to learn computer science

이 문구를 보는 순간 대학교 2학년 때 들었던 자료구조 수업이 떠올랐다.  
그 당시 아딜 무하마드 칸이라는 외국인 교수님에게 그 수업을 들었었고 그 당시에는 영어에 자신이 없었다.   
그 때문에 큰 기대를 하지 않고 덜덜 떨면서 첫 수업을 들어갔었다. 하지만 뜻밖에도 칸 교수님께서는 학생들이 영어라는 장벽으로 인해 자료구조 과목을 이해하지 못할 것을 염려해 굉장히 쉬운 단어로 화이트보드에 강의 내용을 적으면서 수업을 진행했었다. 특히 모든 자료구조들을 그림과 함께 적절한 예시(예를 들자면 서랍에 양말을 정리하는 등의..)를 들어가며 설명해 주셨는데 다른 방식에 비해 굉장히 효과적으로 이해할 수 있었다. 

![linked list]({{site.url}}/images/note/linkedlist.png)

그 수업은 내가 프로그래머라는 직업을 선택하게 되는, 아니 어쩌면 인생을 바꿨을지도 모를 소중한 시간이 되었고, 그러한 경험으로 인해 프로그래밍 교육에는 `적절한 도식화`가 반드시 필요하다는 생각이 내 머리속에 들어가 있었다.  

그러다 소프트웨어 멤버십 활동 중 위에서 기술한 *모든 학생들은 프로그래밍 교육을 받을 기회가 있어야 한다*는 문구를 보게 되었고, 자극을 받아 맘이 맞는 팀원들을 구해 과제로 진행하였다.

> 아직까지도 완성하고 싶은 ~~미완성으로 끝났었다~~ 또 가장 기억 속에 남아있는 프로젝트라고 생각한다.

### 사용 기술

* python : pdb 수정 및 data parsing 모듈
* nodejs : 웹소켓을 이용한 간단한 relay 서버
* ace.js / kinetic.js / jqueryUI mobile / php: web frontend 제작

### 개요

모든 사람들이 나와 같은 과정으로 프로그래밍을 배웠을 거라 생각하지는 않지만, 나는 프로그래밍을 C로 처음 접했다.  
수업이 시작되면 항상 앞의 스크린에는 알수 없는 용어들이 가득했다.  

~~아래 그림을 프로그래밍을 처음 접한다고 생각하고 생각하고 봐주길 바란다.~~

![first programming]({{site.url}}/images/note/first_programming.png)


물론 교수님들께서 강의 자료에 다양한 그림들을 첨부하여 이해를 돕고자 했지만, 많은 학생들이 수업을 따라가지 못하고 `열혈강의`라는 책으로 따로 공부를 했던 것으로 기억한다.
그 당시 `열혈강의`의 인기 비결은 그림으로 c의 어려운 개념들(pointer라든지..)을 설명하고 있었기 때문이라고 개인적으로는 확신한다.

그래서 기존의 책이나 강의 미리 쓰여진 코드를 시각화하는 것에 그치지 않고 자신이 작성한 코드가 지금 어떠한 형태로 동작하고 있는지 보여주어 프로그래밍을 더욱 빠르게 익힐 수 있는 서비스가 필요하다는 생각을 하였고, 본 프로젝트를 기획하게 되었다.

### 그런데 왜 python을 시각화의 대상으로 하였나

`Code Visualizer` python debugger인 pdb를 사용하여 개발되었는데 python을 선택한 이유는 크게 3가지이다.

* compile 기반 언어(c/c++, java, etc..)를 선택했을 경우에는 서버측에서 사용자 소스가 수정될 때마다 컴파일 과정을 추가로 해주어야 한다. 이 과정을 제거하고자 하였다.
* pdb는 그 자체도 컴파일되지 않은 코드(즉, 수정이 쉬운)였기 때문에 구현이 상대적으로 편리할 것으로 예상되었다.
    * 먼저 python으로 개발하여 사용성을 확인하고 다른 언어로 확장할 예정이였다.
* 해외 아이비 리그에서 python을 프로그래밍 기초 교육에 활용한다는 소식이 들려왔었다.
    * ~~어쩌면 좋은 핑계거리가 되었을지도 모르겠다.~~

### System Architecture

* 크게 server와 client(front-end)로 이루어져 있다.
* server에서는 python debugger인 pdb를 통해 현재 frame/variable간의 관계와 가지고 있는 값들을 분석한다.
* client는 사용자가 직접 조작하고 결과를 보는 웹 사이트로 아래 view들을 제공한다.
    * DrawView: 사용자가 작성한 소스 코드의 현재 위치에서 각 frame/variable간의 관계를 시각적으로 표현한다.
    * Source view: 사용자가 소스를 입력할 수 있는 editor를 제공한다.
    * File tree view: user의 소스코드를 tree형태로 출력하며 선택하여 editor로 open이 가능하다.
    * Console view: 출력되는 메시지나 에러 메시지를 보여준다.
* 둘 간의 통신은 websocket으로 이루어진다.

![first programming]({{site.url}}/images/note/architecture.png)

### UI

전체적인 UI는 아래와 같다.

![codevisualizer_ui_overview]({{site.url}}/images/note/codevisualizer_ui_overview.png)

메모리상의 변화가 주요 기능이기 때문에 `DrawView`를 확대해서 볼 수 있는 기능도 지원하고 있다.

![codevisualizer_ui_drawview]({{site.url}}/images/note/codevisualizer_ui_drawview.png)


### 주요 기능

Drawview에서의 시각화가 가장 중요한 기능이라고 할 수 있다.

변수가 저장되는 영역은 크게  `Frame`/`heap`로 볼 수 있다.

* `Frame`은 호출된 하나의 함수의 scope이다. primary type이 주로 저장된다
    * number, boolean, reference 변수(object, array, string 타입을 가리키는) 등
* `Heap`에는 reference 변수들의 실제 값이 위치한다.

`Code Visualizer`에서는 좌측에 각각의 `Frame`을 표현하고 우측에 `Heap`을 표현하였다.

> 아래 그림에서는 `execfile(c의 main()과 같음)`에서 `add()`를 호출하였다.

![codevisualizer_ui_frame]({{site.url}}/images/note/codevisualizer_ui_frame.png)

각각의 python 변수는 아래와 같이 표현하였다.

* 좌측 상단에 변수 이름을 출력하고 도형 안에 해당 변수의 값을 출력
* tuple, list, dictionary 등의 자료형은 key(혹은 index) 기준으로 table 형태로 출력
* Object는 member 변수를 내부에 출력하고 reference관계를 화살표로 표현
![codevisualizer_var]({{site.url}}/images/note/codevisualizer_var.png)
![codevisualizer_object]({{site.url}}/images/note/codevisualizer_object.png)

### 자료구조

노드는 직관적으로 볼 수 있도록 생성 즉시 적절한 위치에 배치된다.  
물론 굳이 자료구조 형태가 아니라도 아래 나오는 규칙에 따라 연관된 노드의 위치가 결정된다.  

#### Linked List

![codevisualizer_linked_list]({{site.url}}/images/note/codevisualizer_linked_list.png)

한 node 내에 자신과 동일한 객체가 1개가 존재하고 선형적인 관계일 경우 직선으로 추가 노드를 배치한다.  
직선으로 배치할 경우 부자연스러운 느낌이 들어 약간 꺽이도록 배치하였다.

#### Tree

![codevisualizer_tree]({{site.url}}/images/note/codevisualizer_tree.png)

한 node 내에 자신과 동일한 객체가 2개 이상 존재하고 선형적인 관계일 경우 자식 노드의 갯수에 맞게 트리의 형태로 우측에 노드를 배치한다.  

#### Graph

![codevisualizer_graph]({{site.url}}/images/note/codevisualizer_graph.png)

한 node 내에 자신과 동일한 객체가 2개 이상 존재하고 선형적인 관계가 아닐 경우(각각 가리키는) 전체 노드의 배치를 고려하여 빈자리(?)에 배치한다.

> 약간은 규칙 없이 배치하기 때문에 reference 관계가 한눈에 들어오지 않도록 그려지는 경우가 있는데, 각각의 note는 draggable하기 때문에 사용자가 직접 위치를 변경할 수 있다.

### 해당 프로젝트에서 나의 역할

* Node.js websocket server
* pdb 수정을 통한 debug 정보 수집 모듈 구현
* web client에서 `draw view`를 제외한 부분

### 데모 영상

저화질만 있는 점이 상당히 너무나 아쉽다.

> 언젠가는 완성해서 초고화질로 만들어서 업데이트...하고 싶다.~~안되려나..~~

<iframe width="420" height="315" src="https://www.youtube.com/embed/mZRMk0uNa4I" frameborder="0" allowfullscreen></iframe>

### 개선할 점

언젠가 이 프로젝트를 다시 할 날이 온다면 개선할 점을 정리해본다.

* WAS 구성
    * 이 프로젝트 당시에는 웹에 대한 지식이 지금에 비해서 많지 않아 html/js 구현물을 apache에 올리고 별도 웹서버(node.js)와 websocket 통신을 ajax로 하도록 구현했었다.
    * 굉장히 비효율적이였기 때문에 다시 만든다면.. websocket이 크게 나쁘지는 않았지만 django나 flask 같은 python web server framework를 사용하고 싶다.
* Javascript framework 사용
    * UI 컴퍼넌트에 너무 집중한 나머지.. 다양한 framework들을 섞어서 사용하였다. 그렇기 때문에 디자인/코드에 통일성이 떨어지는 아쉬움이 있다.
    * `angular.js`나 `react`같은 framework를 사용하는 것을 고려하고 있다.
* kinetic.js -> d3.js
    * 속도 측면이나 지원 기능에서 차이가 있다고 하더라..(확인해보지는 않았다.)



