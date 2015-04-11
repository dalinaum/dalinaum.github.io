---
layout: post
title:  "유닛 테스트는 충분하지 않습니다."
date:   2015-04-11 16:52:00 +0900
categories: programming
---

유닛 테스트가 있기 때문에 정적 타이핑이 필요없다는 논쟁들이 있습니다. 이 주제에 대해 [Evan R. Farrer](http://evanfarrer.blogspot.ca/)씨는 4개의 파이썬 프로젝트를 하스켈로 재작성을 하는 과정에서 발견한 버그의 사례를 이야기하며 유닛 테스트로는 충분하지 않다는 이야기를 합니다.

자세한 내용은 [블로그의 요약](http://evanfarrer.blogspot.ca/2012/06/unit-testing-isnt-enough-you-need.html), [논문](https://docs.google.com/open?id=0B5C1aVVb3qRONVhiNDBiNUw0am8), [소스코드](https://docs.google.com/open?id=0B5C1aVVb3qROS2dib0NtbzZCaVE)에서 확인하세요.

개인적인 의견으로는 생산성이 중요한 스타트업에서는 동적 언어를 사용하는 것이 좋고, 안정성이 중요한 대규모 프로젝트는 단계 별로 정적 언어의 사용량을 늘려가는 것이 합리적으로 보입니다. 
