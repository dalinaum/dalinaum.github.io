---
layout: post
title:  "M1 맥북에서 안드로이드 에뮬레이터 사용하기"
date:   2021-03-15 20:45:00 +0900
categories: android
---

구글의 [안드로이드 에뮬레이터 M1 프리뷰](https://github.com/google/android-emulator-m1-preview) 저장소가 있다. 역시나 프리뷰이구나 생각할 수 있는데 인상적인 문구 하나가 추가되어 있다.

![더 이상 프리뷰 필요없어요.]({{ "/assets/9-no-preview.png" | absolute_url }})

이제 [더 이상 프리뷰가 필요없다](https://github.com/google/android-emulator-m1-preview/pull/29/files)는 이야기다. 이제 정식 에뮬레이터를 띄워보자.

안드로이드 스튜디오 시작 화면에서 SDK Manager를 선택하자.

![SDK Manager]({{ "/assets/9-sdk-manager.png" | absolute_url }})

SDK Manager에서 `Android 11.0(R)`을 클릭하자.

![SDK Manager에서 항목 선택]({{ "/assets/9-sdk-manager2.png" | absolute_url }})

`Android SDK Platform 30`, `Sources for Android 30`, `Google Play ARM 64 v8a System Image`을 선택하자.

사실 필요한 것은 `Google Play ARM 64 v8a System Image`인데 안드로이드 개발을 할 때 SDK와 소스는 있으면 좋아서 선택했다.

이제 시작화면에서 `ADK Manager`를 선택하자.

![ADV Manager 선택]({{ "/assets/9-adv-manager.png" | absolute_url }})

`Create Virtual Machine`을 눌러 가상 머신 생성을 하자.

![가상 머신 생성]({{ "/assets/9-adv-manager2.png" | absolute_url }})

하드웨어 선택은 `Next`로 넘어가 보자. 기본으로는 `Pixel 2`인데 사이즈를 결정한다.

![가상 머신 생성]({{ "/assets/9-select-hardware.png" | absolute_url }})

시스템 이미지에서 `Other Images` 탭을 누르고 다운로드 버튼이 없는 `R`을 고르고 다음을 누르자.

![시스템 이미지]({{ "/assets/9-system-image.png" | absolute_url }})

설치가 완료되면 켜보자.

![에뮬레이터 시작]({{ "/assets/9-start-emul.png" | absolute_url }})

불행히도 아래와 같은 팝업이 뜨며 실패한다.

![에뮬레이터 실패]({{ "/assets/9-failure.png" | absolute_url }})

걱정하지 말자. [아직은 다들 그렇게 뜬다](https://www.reddit.com/r/androiddev/comments/m51kcw/android_emulator_on_m1_mac/gqxt844/?utm_source=reddit&utm_medium=web2x&context=3).

왜 이런 모양으로 릴리즈를 했냐 구글을 마음 속으로 (지금 혼자 계신다면 소리내어) 욕한 다음에 다시 켜자.

![에뮬레이터 성공]({{ "/assets/9-m1-emul.png" | absolute_url }})

정말 M1 에뮬레이터가 맞는지는 `활성 상태 확인`을 들어가서 확인해볼 수 있다. (상단 우측 돋보기에서 `활성 상태 확인`을 입력하라.)

![활성 상태 확인]({{ "/assets/9-active-state.png" | absolute_url }})

에뮬레이터는 Apple(M1)으로 안드로이드 스튜디오는 Intel로 돌고 있는 것을 확인할 수 있다.