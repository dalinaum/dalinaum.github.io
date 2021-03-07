---
layout: post
title:  "Windows에 여러 자바 설치하기"
date:   2021-03-07 23:25:00 +0900
categories: java
---

[맥에 여러 자바 버전을 설치](http://dalinaum.github.io/mac/java/2020/05/20/multiple-jdks.html)했었는데 이제 윈도우즈에서도 여러 자바 버전을 사용할 필요가 생겼다. (이게 다 [코틀린 빌드](https://github.com/JetBrains/kotlin) 때문이다.)

[sdkman](https://sdkman.io/)을 이용해 필요한 자바 버전을 윈도에 설치하려 한다. sdkman은 유닉스와 맥에서도 잘 돌기 때문에 유닉스와 맥에서도 유사할 것이다. `sdkman`이 `bash`와 `zip`을 요구하기 때문에 Git bash 터미널를 열었다. 아마도 Github Desktop 설치 중에 설치가 된 것 같은데 직접 설치해도 동일할 것이다.

이제 `bash` 의존성은 해결했으니 `zip` 의존성을 해결하자. zip을 수동으로 설치하고 싶지 않기 때문에 윈도용 패키지 관리자 [Scoop](https://scoop.sh/)을 이용하기로 한다.

파워셀에서 아래 두 커맨드를 입력하면 설치가 된다. 커맨드라인(`cmd`)이 아니라는 점을 유의하자.

```sh
Set-ExecutionPolicy RemoteSigned -scope CurrentUser
iwr -useb get.scoop.sh | iex
```

이제 zip을 설치하자.

```sh
scoop install zip
```

그리고 Git bash에 접속해 sdkman을 설치한다. 파워 셀이 아니라는 것을 유의하자.

```sh
curl -s "https://get.sdkman.io" | bash
```

git bash를 다시 열거나 터미널에 아래의 내용을 입력하면 sdkman을 사용할 수 있다.

```sh
source "/c/Users/User/.sdkman/bin/sdkman-init.sh"
```

sdkman이 동작하면 `sdk list java`를 이용해 설치가능한 자바를 확인할 수 있다.

```sh
$ sdk list java
================================================================================
Available Java Versions
================================================================================
 Vendor        | Use | Version      | Dist    | Status     | Identifier
--------------------------------------------------------------------------------
 AdoptOpenJDK  |     | 15.0.2.j9    | adpt    |            | 15.0.2.j9-adpt
               |     | 15.0.2.hs    | adpt    |            | 15.0.2.hs-adpt
               |     | 11.0.10.j9   | adpt    |            | 11.0.10.j9-adpt
               |     | 11.0.10.hs   | adpt    |            | 11.0.10.hs-adpt
               |     | 11.0.9.open  | adpt    |            | 11.0.9.open-adpt
               |     | 8.0.282.j9   | adpt    |            | 8.0.282.j9-adpt
               |     | 8.0.282.hs   | adpt    |            | 8.0.282.hs-adpt
               |     | 8.0.275.open | adpt    |            | 8.0.275.open-adpt
 Alibaba       |     | 11.0.9.4     | albba   |            | 11.0.9.4-albba
 Amazon        |     | 15.0.2.7.1   | amzn    |            | 15.0.2.7.1-amzn
               |     | 11.0.10.9.1  | amzn    |            | 11.0.10.9.1-amzn
               |     | 8.282.08.1   | amzn    |            | 8.282.08.1-amzn
 Azul Zulu     |     | 15.0.2       | zulu    |            | 15.0.2-zulu
               |     | 15.0.2.fx    | zulu    |            | 15.0.2.fx-zulu
               |     | 11.0.10      | zulu    |            | 11.0.10-zulu
               |     | 11.0.10.fx   | zulu    |            | 11.0.10.fx-zulu
               |     | 8.0.282      | zulu    |            | 8.0.282-zulu
               |     | 8.0.282.fx   | zulu    |            | 8.0.282.fx-zulu
               |     | 6.0.119      | zulu    |            | 6.0.119-zulu
 BellSoft      |     | 15.0.2.fx    | librca  |            | 15.0.2.fx-librca
               |     | 15.0.2       | librca  |            | 15.0.2-librca
               |     | 11.0.10.fx   | librca  |            | 11.0.10.fx-librca
               |     | 11.0.10      | librca  |            | 11.0.10-librca
               |     | 8.0.282.fx   | librca  |            | 8.0.282.fx-librca
               |     | 8.0.282      | librca  |            | 8.0.282-librca
 GraalVM       |     | 21.0.0.2.r11 | grl     |            | 21.0.0.2.r11-grl
               |     | 21.0.0.2.r8  | grl     |            | 21.0.0.2.r8-grl
               |     | 20.3.1.2.r11 | grl     |            | 20.3.1.2.r11-grl
               |     | 20.3.1.2.r8  | grl     |            | 20.3.1.2.r8-grl
               |     | 19.3.5.r11   | grl     |            | 19.3.5.r11-grl
               |     | 19.3.5.r8    | grl     |            | 19.3.5.r8-grl
               |     | 19.1.0       | grl     |            | 19.1.0-grl
 Java.net      |     | 17.ea.12     | open    |            | 17.ea.12-open
               |     | 17.ea.11     | open    |            | 17.ea.11-open
               |     | 17.ea.10     | open    |            | 17.ea.10-open
               |     | 17.ea.9      | open    |            | 17.ea.9-open
               |     | 17.ea.2.pma  | open    |            | 17.ea.2.pma-open
               |     | 17.ea.2.lm   | open    |            | 17.ea.2.lm-open
               |     | 16.ea.36     | open    |            | 16.ea.36-open
               |     | 15.0.2       | open    |            | 15.0.2-open
               |     | 11.0.10      | open    |            | 11.0.10-open
               |     | 11.0.2       | open    |            | 11.0.2-open
               |     | 8.0.282      | open    |            | 8.0.282-open
               |     | 8.0.265      | open    |            | 8.0.265-open
 Mandrel       |     | 21.0.0.0     | mandrel |            | 21.0.0.0-mandrel
               |     | 20.3.1.2     | mandrel |            | 20.3.1.2-mandrel
 SAP           |     | 15.0.2       | sapmchn |            | 15.0.2-sapmchn
               |     | 11.0.10      | sapmchn |            | 11.0.10-sapmchn
 TravaOpenJDK  |     | 11.0.9       | trava   |            | 11.0.9-trava
================================================================================
Use the Identifier for installation:

    $ sdk install java 11.0.3.hs-adpt
================================================================================
```

Java 6, 7, 8, 9, 11을 설치하겠다.

일단 목록을 보고 6, 8, 11버전을 설치한다.

```
sdk install java 6.0.119-zulu
sdk install java 8.0.282-open
sdk install java 11.0.10-open
```

`/c/Users/<USER_NAME>/.sdkman/`에 설치된다.

`<USER_NAME>`이 `User`라면 다음과 같은 경로에 설치된다.

 * `c:\Users\User\.sdkman\candidates\java\11.0.10-open`
 * `c:\Users/User\.sdkman\candidates\java\6.0.119-zulu`
 * `c:\Users/User\.sdkman\candidates\java\8.0.282-open`

불행하게도 7과 9가 SDKMAN에 받을 수 없다. 자바 JDK를 설치할 때는 행복한 적이 별로 없는 것 같다. 결국 7과 9는 수동으로 받아야 한다.

OpenJDK 버전과 Oracle 버전이 있다. OpenJDK버전은 아래 링크에서 받을 수 있다.

 * [OpenJDK 7](https://jdk.java.net/java-se-ri/7) - Accept를 누르고 Windows i586 Binary를 선택하자.
 * [OpenJDK 9](https://jdk.java.net/java-se-ri/9) - Accept를 누르고 Windows x64 Java Development Kit를 선택하자.

오라클 버전은 회원가입/로그인을 해야 받을 수 있다.

 * [JDK 7](https://www.oracle.com/java/technologies/javase/javase7-archive-downloads.html) - Windows x64를 선택하자.
 * [JDK 9](https://www.oracle.com/java/technologies/javase/javase9-archive-downloads.html) - Windows를 선택하자. (이제 32비트 버전은 없는 모양.)

나는 OpenJDK 7, 9를 받아서 다음과 같이 설치했다.

 * `c:\opt\jdk7`
 * `c:\opt\jdk9`

 내 아이디는 `User`이기 때문에 아래와 같은 것이고 실제 사용에서는 `User`를 다른 경로로 바꾸어야 한다.

```sh
JDK_16="c:\Users/User\.sdkman\candidates\java\6.0.119-zulu"
JDK_17="c:\opt\jdk7"
JDK_18="c:\Users/User\.sdkman\candidates\java\8.0.282-open"
JDK_9="c:\opt\jdk9"
```

JAVA_HOME은 11버전이나 8버전을 상황에 따라 설정한다.

```sh
JAVA_HOME="c:\Users\User\.sdkman\candidates\java\11.0.10-open"
JAVA_HOME="c:\Users/User\.sdkman\candidates\java\8.0.282-open"
```

`시스템 속성` > `환경 변수`에서 `사용자 변수`에 항목들을 설정하자. 찾기 어려우면 Windows 10 하단의 검색창에서 `시스템 환경 변수 편집`을 입력하고 시작하자.