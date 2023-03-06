---
layout: post
title:  "Compose: rememberUpdatedState"
date:   2022-12-12 02:11:00 +0900
categories: compose
---

Compose의 여러 코드들을 조금씩 살펴볼 예정이다. 이번에 살펴볼 코드는 [rememberUpdatedState](https://developer.android.com/jetpack/compose/side-effects#rememberupdatedstate)

```kotlin
@Composable
fun <T> rememberUpdatedState(newValue: T): State<T> = remember {
    mutableStateOf(newValue)
}.apply { value = newValue }
```

일반적인 `remember { mutableStateOf(value) }` 패턴에 `apply`가 붙어 있다. 리컴포지션이 발생하면 이전에 `remember`로 저장한 상태가 불러진다. 그 상태는 상태가 만들어졌을 때의 값을 가지고 있기 때문에 오래된 값을 가지고 있다. `apply`를 통해 이전 상태의 값을 새 값으로 업데이트한다.