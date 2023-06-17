---
layout: post
title:  "컴포즈 레이아웃이 Surface만큼 커져요"
date:   2023-06-17 20:11:00 +0900
categories: compose
---

`Surface`안에 넣은 넣은 컴포저블 함수는 가끔 이상하게 렌더링 된다.

아래 코드를 생각해보자.

```kotlin
@Composable
fun ButtonExample(onButtonClicked: () -> Unit) {
    Surface(
        modifier = Modifier
            .background(Color.Red)
            .fillMaxSize()
    ) {
        Button(onClick = {}) {
            Text(text = "Send")
        }
    }
}
```

`Surface`를 최대 크기로 설정하고 배경을 빨간 색으로 설정했다.

`Button`은 적당한 사이즈로 렌더링되길 기대한다.

렌더링 된 결과는 우리의 예상과는 다르다.

![서피스 렌더링](/assets/surface.png)

`Surface`안에 있는 `Button`이 지나치게 커졌다.

이유는 `Surface`는 `Surface`의 최소 사이즈 제약을 자식에게 전파하기 때문이다.

`Surface`는 기술적으로는 `Box`인데 최소 사이즈 제약 전파가 체크되어 있다.

```kotlin
@Composable
private fun Surface(
    modifier: Modifier,
    shape: Shape,
    color: Color,
    contentColor: Color,
    border: BorderStroke?,
    elevation: Dp,
    clickAndSemanticsModifier: Modifier,
    content: @Composable () -> Unit
) {
    val elevationOverlay = LocalElevationOverlay.current
    val absoluteElevation = LocalAbsoluteElevation.current + elevation
    val backgroundColor = if (color == MaterialTheme.colors.surface && elevationOverlay != null) {
        elevationOverlay.apply(color, absoluteElevation)
    } else {
        color
    }
    CompositionLocalProvider(
        LocalContentColor provides contentColor,
        LocalAbsoluteElevation provides absoluteElevation
    ) {
        Box(
            modifier
                .shadow(elevation, shape, clip = false)
                .then(if (border != null) Modifier.border(border, shape) else Modifier)
                .background(
                    color = backgroundColor,
                    shape = shape
                )
                .clip(shape)
                .then(clickAndSemanticsModifier),
            propagateMinConstraints = true
        ) {
            content()
        }
    }
}
```

`propagateMinConstraints = true` 때문에 자식에게는 최소 사이즈 제한이 전파된다. 이 경우에는 `Surface`가 자식인 `Button`에게 꽉 채우는 것을 강요하게 되었다.

이런 현상을 막는 방법은 두 가지가 있다.

 1. `Surface`와 `Button` 사이에 다른 레이아웃을 끼워 넣는 방법.
 2. 이 `Surface`가 꼭 필요한게 아니라면 제거를 고려하기.

필자는 1번의 방법을 선택해 보았다.

```kotlin
@Composable
fun ButtonExample(onButtonClicked: () -> Unit) {
    Surface(
        modifier = Modifier
            .background(Color.Red)
            .fillMaxSize()
    ) {
        Box{
            Button(onClick = {}) {
                Text(text = "Send")
            }
        }
    }
}
```

`Surface`와 `Button` 사이에 `Box`가 끼어들었고 최소 사이즈 제약은 `Box`에게만 영향을 준다.

## 구글러의 답변

왜 이런 형태로 구현이 되었을까? [구글러의 답변](https://issuetracker.google.com/issues/177726827#comment8)을 살펴보자.

왜 `Surface` 안의 박스는 `propagateMinConstraints=true`인가?

`Surface`는 진짜 레이아웃이 아니다. 우리는 `FloatingActionButton`에서 이슈를 겪었다. 명세에 따라 최소 width와 height를 적용했는데 사용자는 원한다면 더 큰 사이즈를 설정할 수 있다. 이제 `FloatingActionButton`에서 안의 콘텐트(아이콘)은 `Surfae`의 전체 사이즈로 채워야 한다. 리플(ripple)을 적용하고 리플을 `Surface`의 모양에 따라 자를 수 있다. 우리가 `Modifier.fillMaxSize()`를 적용했다면 `FloatingActionButton`에 지정된 최대 사이즈가 없기에 전체 화면을 채울 것이다.

시스템의 동작 방식 때문에 이 정보가 `Box`에 의해 전파되지 않으며 `Modifier.fillMinSize()`같은 것은 존재하지 않는다. 그래서 우리는 `propagateMinConstraints=true`를 하는 아이디어를 냈고, 이제 `Surface` 안의 콘텐트는 `Surface`에 적용된 최소 사이즈를 채워야 한다.

솔직히 설명이 충분히 명확한지 모르겠다. :)

진짜 레이아웃과 여러 요소가 Surface안에 있다면 `Box`를 수동으로 추가해야 할 수 있다.