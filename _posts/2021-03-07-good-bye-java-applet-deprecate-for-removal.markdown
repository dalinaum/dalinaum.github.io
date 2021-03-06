---
layout: post
title:  "굿 바이 자바 애플릿, Deprecate, for Removal"
date:   2021-03-07 02:58:00 +0900
categories: java
---

[JEP 398](https://openjdk.java.net/jeps/398)이 올라왔다. 자바 애플릿 API를 `제거를 위한 deprecate`를 하자(Deprecate the Applet API for Removal)는 제안이다. JEP는 자바를 앞으로 어떻게 개선하자는 제안을 문서화시킨 것이다.

그런데 `deprecate`면 deprecate지 `Deprecate for Removal`은 무엇일까?

일단 [Applet 클래스를 살펴보자](https://docs.oracle.com/javase/9/docs/api/java/applet/Applet.html).

```java
@Deprecated(since="9")
public class Applet
extends Panel
```

자바 9 버전의 문서에서 이미 `Deprecated`되었다. 버전도 친절하게 `since="9"`로 명기시켜두었다. 2016년에 [JEP 289](https://openjdk.java.net/jeps/289)를 통해 `deprecated`를 결정했다.

그럼 이제와서 deprecate하겠다는 것은 무엇일까? [Java 9부터 향상된 Deprecation](https://docs.oracle.com/javase/9/core/enhanced-deprecation1.htm#JSCOR-GUID-23B13A9E-2727-42DC-B03A-E374B3C4CE96)이 도입되었다.

```
In prior releases, APIs were deprecated but virtually never removed. Starting with JDK 9, APIs may be marked as deprecated for removal. This indicates that the API is eligible to be removed in the next release of the JDK platform. If your application or library consumes any of these APIs, then you should make a plan to migrate from them soon.
```

이전 릴리즈까지는 API가 `deprecated` 되었다는 표기만 했을 뿐 실제로 제거하지 않았지만 JDK9부터는 `deprecated for removal`라고 표시를 하고 JDK 플랫폼의 다음 릴리즈 때 제거할 수 있다. 이렇게 표기된 API를 쓴다면 가능한 빨리 다른 API로 변경할 준비를 해야한다.

사람들이 `deprecated`되었다고 붙여도 그걸 사용하고 그러니 이제 진짜 지원 중단된다 삭제되니깐 그만 쓰라는 경고를 하나 추가하고 진짜로 삭제하기 시작한 것이다.

어노테이션은 다음과 같은 형태로 개선(?)되었다.

```java
@Deprecated(since="9", forRemoval=true)
```

`forRemoval`의 기본값은 `false`이고 이것이 `true`이면 이 API로 부터는 가능한 빨리 벗어나야겠다.

아마 `Applet` 관련 객체들은 자바 9부터 15까지 `deprecated` 상태였지만 아마 다음 버전의 자바에서는 진짜로 다시 볼 수 없을지도 모르겠다.

덧붙임: `deprecated`에 대한 적절한 번역어가 무엇인지 알기 어려워 일단 `deprecated`라고 적어두었다. 최소한 음차라도 해야할텐데 뭐가 좋은 번역일지 모르겠네.