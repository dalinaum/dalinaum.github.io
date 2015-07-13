---
layout: post
title:  "Refusing to install as a dependency of iteslf"
date:   2015-07-14 02:11:00 +0900
categories: npm
---

```
Refusing to install BLAH-BLAH as a dependency of itself
```

npm으로 패키지를 설치할 때 이런 에러메시지가 발생할 수 있다. 가장 대표적인 이유는 자신이 진행하고 있는 프로젝트가 패키지 네임과 동일한 이름을 사용하는 것이다. `package.json` 파일을 열어서 확인해보자.
