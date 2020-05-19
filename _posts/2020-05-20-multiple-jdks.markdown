---
layout: post
title:  "HomeBrew로 Mac에 여러 자바 설치하기"
date:   2020-05-20 01:26:00 +0900
categories: mac java
---

가끔 JDK 버전을 여러개 설치해야 하는 경우가 있다. 나는 [Kotlin](https://github.com/JetBrains/kotlin) 빌드 때문에 여러 버전을 설치하게 되었다.

설치에 앞서 설명할 단어가 몇개 있다. [Zulu](https://www.azul.com/downloads/zulu-community/?architecture=x86-64-bit&package=jdk)와 [AdoptOpenJDK](https://adoptopenjdk.net/)는 OpenJDK 배포판이다. 오픈소스 자바를 빌드해서 배포한다고 보면 된다. Oracle의 자바로 설치하면 좋은데 Oracle 사이트에서 받아야 할 경우도 있고 원하는 버전을 받기 어려울 수 있다. 실용적으로는 OpenJDK로도 충분하기 때문에 OpenJDK를 최대한 이용하겠다.

Oracle Java 6은 [homebrew/cask-versions](https://github.com/Homebrew/homebrew-cask-versions)에 있지만 Java 7은 없다. 그래서 Zulu 7을 받는다. Homebrew의 [Cask](https://github.com/Homebrew/homebrew-cask)는 바이너리 배포판을 설치하기 위한 Homebrew 확장이며 homebrew/cask-versions는 과거 버전들을 설치할 수 있는 Cask이다.

```sh
brew tap homebrew/cask-versions
brew cask install java6
brew cask install zulu7
```

Java 8은 시스템에 설치되었으니 생략. 미래에 디폴트 자바 버전이 바뀌면 해당 버전을 Homebrew를 이용해야 할 것이다.

[zulu9은 지원되지 않는다고 삭제됨](https://github.com/Homebrew/homebrew-cask-versions/pull/7268). homebrew/cask-versions 정책이 좀 화가난다. 있던 포뮬러가 상황에 따라 삭제되어 나중에 다시 이용할 수 없다.

어쩔 수 없이 AdoptOpenJDK로 가자. AdoptOpenJDK에는 역으로 Java 7은 없기 때문에 Zulu로 받을 수 밖에 없다. Zulu와 AdoptOpenJDK을 모두 사용해야 한다.

```sh
brew tap adoptopenjdk/openjdk
brew cask install adoptopenjdk9
```

참고로 나는 AdoptOpenJDK가 이상하게 너무나 느렸다. 어쩔 수 없으니 기다리자.

### jenv 설정

jenv를 설정한다면 아래와 같이 등록하자. [jenv](https://www.jenv.be/)는 파이선의 pyenv나 루비의 rbenv 비해서는 사용성이 나쁜데 일단 패키지를 설치한 후 경로만 등록할 수 있다. 당연히 pyenv나 rbenv는 설치와 설정이 한번에 된다. 아무래도 자바가 오픈소스스럽게 배포하지 않고 Oracle 사이트에서 배포하는 방식도 있고 OpenJDK도 여러 구성원들이 배포하고 있어 배포가 깔끔하지 못한 것 같다.

```sh
jenv add /Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home
jenv add /Library/Java/JavaVirtualMachines/zulu-7.jdk/Contents/Home
jenv add /Library/Java/JavaVirtualMachines/adoptopenjdk-9.jdk/Contents/Home
```

맥에서는 AdoptOpenJDK는 등록이 거부될텐데 `System Preferences.app`을 열고 `Sucurity & Privacy`의 하단에서 차단 정보를 보고 `Open Anyway`를 누르고 `App Store`만 허가한 것을 `App Store and identified developers`로 변경하자.

만약 jenv를 설치하길 원한다면 `brew install jenv`로 설치할 수 있다.

### 환경 변수 설정

설치한 자바에 맞추어 환경 변수를 아래와 같이 설정했다.

```sh
export JAVA_HOME="/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home"
export JDK_16="/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home"
export JDK_17="/Library/Java/JavaVirtualMachines/zulu-7.jdk/Contents/Home"
export JDK_18="/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home"
export JDK_9="/Library/Java/JavaVirtualMachines/adoptopenjdk-9.jdk/Contents/Home"
```

### 에필로그: SDKMAN!

이 글을 적는 동안은 [SDKMAN!](https://sdkman.io/)의 존재를 몰랐다. 내 손이 너무 좋아서 머리를 고생 안시켰네. 다음 설정 때는 SDKMAN!을 가능한 사용해야 겠다.

 - [SDK! 으로 Java 버전 관리하기](https://phoby.github.io/sdkman/)