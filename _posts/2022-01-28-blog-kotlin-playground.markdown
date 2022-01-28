---
layout: post
title:  "블로그에 코틀린 플레이그라운드 넣기"
date:   2022-01-28 20:21:00 +0900
categories: blog
---

항상 IDE가 있으면 좋겠지만 여의치 않을 경우에는 [코틀린 플레이그라운드](https://play.kotlinlang.org/)를 이용해 간단한 테스트를 해볼 수 있다.

그런데 자신의 블로그에도 코틀린 플레이그라운드 블록을 넣어서 코틀린 코드를 직접 수행해볼 수 있다.

<div class="kotlin-playground" >
    fun main() {
        val name = "stranger"        // Declare your first variable
        println("Hi, $name!")        // ...and use it!
        print("Current count:")
        for (i in 0..10) {           // Loop over a range from 0 to 10
            print(" $i")
        }
    }
</div>

어떻게 하는지 알아보자.

필요한 것은 두개의 코드 블록이다. 먼저 자바스크립트 블록.

```html
<script src="https://unpkg.com/kotlin-playground@1"></script>
<script>
  document.addEventListener('DOMContentLoaded', function() {
    KotlinPlayground('.kotlin-playground');
  });
</script>
```

이제 `kotlin-playground` 클래스에 대해 코틀린 플레이그라운드가 적용된다.

```html
<div class="kotlin-playground" >
    fun main() {
        val name = "stranger"        // Declare your first variable
        println("Hi, $name!")        // ...and use it!
        print("Current count:")
        for (i in 0..10) {           // Loop over a range from 0 to 10
            print(" $i")
        }
    }
</div>
```

div 태그에 `class="kotlin-playground"`를 기입해 클래스를 지정했다. 이 코드 블록을 합치면 div 태그에 포함된 내용이 코틀린 플레이그라운드 블록으로 포함된다.

<div class="kotlin-playground" >
    fun main() {
        val name = "stranger"        // Declare your first variable
        println("Hi, $name!")        // ...and use it!
        print("Current count:")
        for (i in 0..10) {           // Loop over a range from 0 to 10
            print(" $i")
        }
    }
</div>

우측 상단의 재생 버튼을 눌러 실행 결과를 확인해보자.

테마를 `darcula`로 변경할 수도 있다.

```html
<div class="kotlin-playground" theme="darcula">
    fun main() {
        val name = "stranger"        // Declare your first variable
        println("Hi, $name!")        // ...and use it!
        print("Current count:")
        for (i in 0..10) {           // Loop over a range from 0 to 10
            print(" $i")
        }
    }
</div>
```

`theme="darcula"` 속성을 추가했다.

<div class="kotlin-playground" theme="darcula">
    fun main() {
        val name = "stranger"        // Declare your first variable
        println("Hi, $name!")        // ...and use it!
        print("Current count:")
        for (i in 0..10) {           // Loop over a range from 0 to 10
            print(" $i")
        }
    }
</div>