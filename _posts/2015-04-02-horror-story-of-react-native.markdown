---
layout: post
title:  "Reactive Native 머지 잔혹사"
date:   2015-04-02 05:47:00 +0900
categories: react ios
---

[krazyeom](http://www.appilogue.kr/)씨의 [PR](https://github.com/facebook/react-native/pull/598), [Haruair](http://haruair.com/)씨의 [PR](https://github.com/facebook/react-native/pull/340)과 저의 [PR](https://github.com/facebook/react-native/pull/344)은 공통점이 있다. [머 ](https://github.com/facebook/react-native/commit/db3a724bb25d0a6c17d563be624c04621643995a)[지 ](https://github.com/facebook/react-native/commit/9a4ee17adba756e676ea2585e13ca4e5884801a5)[가 ](https://github.com/facebook/react-native/commit/31c4ff0dd62ee705353c2b4de916a151f89410d4) 되었지만 흔적조차 찾기어렵다는 것. 리액트 네이티브는 기여를 해도 기여자의 이름이 남지 않는다.

````
Sadly we get the name of the people that imported the pull request and not the author of the pull request. I'm looking to see if there's a quick fix we can do as it really sucks to lose attribution.
- Christopher Chedeau (vjeux)
````
[#557](https://github.com/facebook/react-native/pull/557)

이런 일이 생기는 이유는 모든 기여를 손으로 머지하고 그 머지한 사람의 이름만 기록하기 때문이다. 머지 과정을 복잡하게 하는 이유는 안드로이드용 코드와 같은 작업 단계의 코드를 숨기기 위함으로 보이고. 인터널 마스터를 따로 운영하는 구글의 경우에는 아주 깔끔하게 머지하는데 페북의 운영은 아쉬움이 많다.

기분 좋은 정책은 아니고 어이가 없기도 하다. 오픈소스에 기여하고 싶은 사람들은 대부분 기여자로 오르고 싶을텐데 현재의 리액트 네이티브 프로젝트는 주의할 필요가 있다.
