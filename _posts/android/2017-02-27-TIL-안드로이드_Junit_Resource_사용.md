---
layout: post
title: Today I learned - Android Junit Test에서 Resource 사용하기
excerpt: ""
tags: [TIL, android, resource, junit]
comments: true
---

# 개요

Android Application 개발 중에 고용량 프로필 이미지가 서버에 업로드가 되지 않는 문제가 발견되었습니다. 이 문제를 해결하기 위해 테스트 케이스를 작성하던 중 알게 된 `안드로이드 유닛 테스트에서 파일을 사용하는 방법` 을 정리합니다.

# 사용 방법

우선 프로젝트 구조는 일반적인 Application과 같습니다.

```
.
├── LICENSE
├── README.md
├── app
│   ├── app.iml
│   ├── build
│   │   ├── generated
│   │   ├── intermediates
│   │   ├── outputs
│   │   └── tmp
│   ├── build.gradle
│   ├── proguard-rules.pro
│   └── src
│       ├── androidTest
│       ├── main
│       └── test
```

먼저 `app/src/test` 에 `resources` 디렉토리를 만듭니다.

```bash
mkdir app/src/test/resources
```

디렉토리 안에 사용할 이미지 파일을 넣습니다.

```bash
cp 04110411.JPG app/src/test/resources
```

그리고 테스트 케이스에서 다음과 같이 File 객체를 얻어옵니다.

*app/src/test/java/com/amazingguni/FileTest.java*

```java
@Test
public void profileUpload_hugefile() throws RestClientException, URISyntaxException {
      File file = new File(getClass().getResource("/04110411.JPG").toURI());
}
```

문제 없이 동작하는 것을 볼 수 있습니다.

# 고찰

그동안 테스트 케이스에서 필요했던 파일은 대부분 텍스트였기 때문에, 불편함을 감수하고 단순 스트링으로 테스트 케이스를 구성했습니다. 이번 기회에 resource로써 파일에 접근하는 방법을 알게되어 테스트 케이스 작성에 큰 도움이 될 것으로 기대합니다.
