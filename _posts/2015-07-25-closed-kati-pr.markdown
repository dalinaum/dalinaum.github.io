---
layout: post
title:  "Kati 커밋 거절"
date:   2015-07-25 17:54:00 +0900
categories: build
---

일관성없는 Markdown 포맷을 고치는 [커밋](https://github.com/dalinaum/kati/commit/82a3e803fc35a1bf8936b612b77afac900e7e7fd)과 Homebrew 설치를 추가하는 [커밋](https://github.com/dalinaum/kati/commit/307d3edd80d391797a0673aaecf4d4405647b37c)을 Google의 Kati에 보냈는데 [완강한 거부](https://github.com/google/kati/pull/12#issuecomment-124793479)를 받았다.

```
Hmm I'm not sure if I like this change.

I don't want to add a section whenever a new package manager starts supporting kati.
I'm not sure it's a good idea to use kati provided by a package manager. kati is in a very early stage of development so many changes could be done in near future.
I think a user of package manager knows how to install a program with it.
Once kati becomes more stable, it's OK to mention package managers. I think just adding a sentence like "or, you can use your package manager like homebrew." would be sufficient.

As for the formatting change, I'd slightly prefer the current style because it looks prettier as a text file.
```

아무 것도 수용할 수 없다는 표현이다.

기분은 상했지만 목표를 좁게 수정해서 일관성없는 부분만 수정하고 원래 컨텐츠를 전혀 수정하지 않는 [커밋](https://github.com/dalinaum/kati/commit/7c80f7a1111336e0d2cb2ad5ef3907003b2af502)을 보내는 타협안을 제시했는데 답변은 [이슈 닫음](https://github.com/google/kati/pull/12#issuecomment-124817492)이었다.

정말 실망스럽다. 예전에 [Angular.js PR](https://github.com/angular/angular.js/pull/7589) 때 이후로 정말 나쁜 PR 경험을 해본다. (그러고 보니 둘다 구글 제품이다.)
