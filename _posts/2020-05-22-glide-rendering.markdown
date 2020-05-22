---
layout: post
title:  "Glide 라이브러리 렌더링의 이해"
date:   2020-05-22 03:45:00 +0900
categories: android
---

안드로이드 이미지 라이브러리 [Glide](https://bumptech.github.io/glide/)가 어떻게 이미지를 표시하게 되는지 그 과정을 간략히 설명한다.

## 로딩 상태에 대응하는 Target

Glide에서 이미지를 로딩을 시작하면 [Target](https://github.com/bumptech/glide/blob/master/library/src/main/java/com/bumptech/glide/request/target/Target.java)을 통해 진행 상황을 알려온다.

```java
public interface Target<R> extends LifecycleListener {
  void onLoadStarted(@Nullable Drawable placeholder);
  void onLoadFailed(@Nullable Drawable errorDrawable);
  void onResourceReady(@NonNull R resource, @Nullable Transition<? super R> transition);
  void onLoadCleared(@Nullable Drawable placeholder);
  void getSize(@NonNull SizeReadyCallback cb);
  void removeCallback(@NonNull SizeReadyCallback cb);
  void setRequest(@Nullable Request request);
  Request getRequest();
}
```

이미지를 가져오기 시작할 때 `onLoadStarted`가 호출되고 실패시 `onLoadFailed`, 성공시 `onResourceReady`가 호출된다. `LifecycleListener`을 상속받았다는 것을 통해 안드로이드 라이프사이클에 대응하는 것도 `Target`의 책임이라는 것을 알 수 있다.

로딩 후 ImageView를 갱신하는 관련 Target이 몇가지 있는데 [ImageViewTarget](https://github.com/bumptech/glide/blob/master/library/src/main/java/com/bumptech/glide/request/target/ImageViewTarget.java)을 상속받는다.

리소스가 준비되면 `onResourceReady` 메서드가 호출된다.

```java
@Override
public void onResourceReady(@NonNull Z resource, @Nullable Transition<? super Z> transition) {
  if (transition == null || !transition.transition(resource, this)) {
    setResourceInternal(resource);
  } else {
    maybeUpdateAnimatable(resource);
  }
}
```

`setResourceInternal`를 따라가보자.

```java
private void setResourceInternal(@Nullable Z resource) {
  // Order matters here. Set the resource first to make sure that the Drawable has a valid and
  // non-null Callback before starting it.
  setResource(resource);
  maybeUpdateAnimatable(resource);
}
```

`setResource`가 호출된다.

`ImageViewTarget`의 자식 [BitmapImageViewTarget](https://github.com/bumptech/glide/blob/master/library/src/main/java/com/bumptech/glide/request/target/BitmapImageViewTarget.java)의 구현을 보자.

```java
@Override
protected void setResource(Bitmap resource) {
  view.setImageBitmap(resource);
}
```

이미지 뷰의 `setImageBitmap`으로 로딩된 비트맵을 먼저 등록했다.

```java
private void maybeUpdateAnimatable(@Nullable Z resource) {
  if (resource instanceof Animatable) {
    animatable = (Animatable) resource;
    animatable.start();
  } else {
    animatable = null;
  }
}
```

[Animatable](https://developer.android.com/reference/android/graphics/drawable/Animatable)인 경우 `Animatable#start`를 호출한다. `Animatable`은 안드로이드 표준 객체인데 에니메이션을 정지하고 시작할 수 있는 기능을 제공하는 인터페이스다.

## 에니메이션 시작 GifDrawable#start

```java
public class GifDrawable extends Drawable
    implements GifFrameLoader.FrameCallback, Animatable, Animatable2Compat {
}
```

[GifDrawable](https://github.com/bumptech/glide/blob/master/library/src/main/java/com/bumptech/glide/load/resource/gif/GifDrawable.java)이 `Animatable`이기 때문에 구현된 `start` 메서드가 호출된다.

```java
@Override
public void start() {
  isStarted = true;
  resetLoopCount();
  if (isVisible) {
    startRunning();
  }
}
```

시작 상태로 체크하고, 루프 카운트를 리셋하고 visible한 상태일 때만 `startRunning`을 호출한다.

```java
private void startRunning() {
  Preconditions.checkArgument(
      !isRecycled,
      "You cannot start a recycled Drawable. Ensure that"
          + "you clear any references to the Drawable when clearing the corresponding request.");
  // If we have only a single frame, we don't want to decode it endlessly.
  if (state.frameLoader.getFrameCount() == 1) {
    invalidateSelf();
  } else if (!isRunning) {
    isRunning = true;
    state.frameLoader.subscribe(this);
    invalidateSelf();
  }
}
```

`Precondition.checkArgument`는 조건이 맞지 않을 때 예외를 던지는 헬퍼 메서드다.

단일 프레임 리소스일 경우 `Drawable#invalidateSelf`를 호출해서 다시 그린다.

에니메이션인 경우에는 실행 상태가 아닐 때 실행 상태로 바꾸고 `state.frameLoader.subscribe(this)`를 호출한 후 다시 그린다.

## Gif의 여러 프레임을 처리하는 GifFrameLoader

`GifDrawable`은 `state: GifState`를 필드로 가지고 있는데 `GifState`는 `ConstantState`를 상속받는다. `ConstantState`를 상속받기 때문에 `Drawble`의 행동과 마찬가지로, 같은 리소스를 로딩했을 때 여러 `GitDrawable`이 같은 `state`를 공유할 것이라 예상할 수 있다. `state`를 통해 공유하는 것은 `frameLoader: GifFrameLoader`이다.

[GifFrameLoader](https://github.com/bumptech/glide/blob/master/library/src/main/java/com/bumptech/glide/load/resource/gif/GifFrameLoader.java)의 `subscribe`는 다음과 같다.

```java
void subscribe(FrameCallback frameCallback) {
  if (isCleared) {
    throw new IllegalStateException("Cannot subscribe to a cleared frame loader");
  }
  if (callbacks.contains(frameCallback)) {
    throw new IllegalStateException("Cannot subscribe twice in a row");
  }
  boolean start = callbacks.isEmpty();
  callbacks.add(frameCallback);
  if (start) {
    start();
  }
}
```

이미 `clear`되었으면 더 이상 진행할 수 없고 동일한 `GifDrawable`은 등록할 수 없다.

`callbacks`는 `GifDrawable`의 리스트인데 우리의 `GifDrawable`을 등록하고, 첫번째 `GifDrawable`이면 `start`를 호출한다.

```java
private void start() {
  if (isRunning) {
    return;
  }
  isRunning = true;
  isCleared = false;

  loadNextFrame();
}
```

### 다음 프레임 처리하기 loadNextFrame

이미 러닝 상태면 종료하고 러닝 상태가 아니면 러닝 상태로 바꾼다. 여기에서 핵심은 `loadNextFrame`을 호출하는 것이다.

```java
private void loadNextFrame() {
  if (!isRunning || isLoadPending) { // (1)
    return;
  }
  if (startFromFirstFrame) { // (2)
    Preconditions.checkArgument(
        pendingTarget == null, "Pending target must be null when starting from the first frame");
    gifDecoder.resetFrameIndex();
    startFromFirstFrame = false;
  }
  if (pendingTarget != null) { // (3)
    DelayTarget temp = pendingTarget;
    pendingTarget = null;
    onFrameReady(temp);
    return;
  }
  isLoadPending = true; // (4)
  int delay = gifDecoder.getNextDelay();
  long targetTime = SystemClock.uptimeMillis() + delay;

  gifDecoder.advance();
  next = new DelayTarget(handler, gifDecoder.getCurrentFrameIndex(), targetTime);
  requestBuilder.apply(signatureOf(getFrameSignature())).load(gifDecoder).into(next);
}
```

1. 실행 중이 아니거나 로딩이 아직 끝나지 않았다면 진행하지 않는다.
1. 첫 프레임 부터 시작하라는 지시를 받았다면 `pendingTarget` (처리해야 하는 타겟)이 없는 경우인지 확인하고, 디코더를 -1 번째 프레임으로 돌린다. 이렇게 -1로 돌리는 이유는 아래 호출할 `gifDecoder.advance()`가 프레임을 더해 0 번째 프레임으로 이동하기 때문이다. `startFromFirstFrame`을 처리했기 때문에 리셋한다.
1. `pendingTarget` (처리해야 하는 타겟)이 있다면 이를 `onFrameReady`메서드를 통해 처리한다.
1. 이제 (다음 프레임) 로딩 중이다고 체크하고 디코드를 다음 프레임으로 진행시킨다. `requestBuilder.apply`를 통해 비동기로 로딩을 진행시키고 로딩의 상황을 `DelayTarget`이 수행하도록 구성한다.

### 다음 이미지 로딩을 다루는 타겟 DelayTarget

아래는 `DelayTarget`의 일부이다.

```java
@VisibleForTesting
static class DelayTarget extends CustomTarget<Bitmap> {
  ...
  @Override
  public void onResourceReady(
      @NonNull Bitmap resource, @Nullable Transition<? super Bitmap> transition) {
    this.resource = resource;
    Message msg = handler.obtainMessage(FrameLoaderCallback.MSG_DELAY, this);
    handler.sendMessageAtTime(msg, targetTime);
  }
}
```

로딩이 비동기로 완료되면 `onResourceReady`가 호출될텐데 `loadNextFrame`에서 설정된 `targetTime`의 값으로 핸들러를 수행시킨다.

핸들러는 `GifFrameLoader` 객체 안에 `FrameLoaderCallback`이란 이름으로 존재한다.

```java
private class FrameLoaderCallback implements Handler.Callback {
  static final int MSG_DELAY = 1;
  static final int MSG_CLEAR = 2;

  @Synthetic
  FrameLoaderCallback() {}

  @Override
  public boolean handleMessage(Message msg) {
    if (msg.what == MSG_DELAY) {
      GifFrameLoader.DelayTarget target = (DelayTarget) msg.obj;
      onFrameReady(target);
      return true;
    } else if (msg.what == MSG_CLEAR) {
      GifFrameLoader.DelayTarget target = (DelayTarget) msg.obj;
      requestManager.clear(target);
    }
    return false;
  }
}
```

`MSG_DELAY`로 전달했기 때문에 결국 로딩된 이미지는 일정 딜레이 후에 `GifFrameLoader#onFrameReady`로 전달된다.

### 로딩된 이미지를 다루는 GifFrameLoader#onFrameReady

아래 두 경우에 `GifFrameLoader#onFrameReady`가 호출된다.

1. `loadNextFrame` 수행 시 `pendingTarget`이 있는 경우.
1. `loadNextFrame`에서 리퀘스트 빌더를 통해 다음 프레임을 로딩하고 핸들러로 전달된 경우.

그럼 `pendingTarget`은 언제 설정될까? `onFrameReady`가 호출되었는데 다음 프레임을 처리할 수 없을 때 설정된다.

```java
@VisibleForTesting
void onFrameReady(DelayTarget delayTarget) {
  ...
  if (!isRunning) {
    pendingTarget = delayTarget;
    return;
  }
  ...
}
```

실행 상태가 아닐 경우는 다음 프레임을 진행하면 안되니 `pendingTarget`에 올려두고 더 이상 진행하지 않는다. 다음에 `loadNextFrame`이 호출될 때 `pendingTarget`을 발견하고 다시 `onFrameReady`로 전달될 것이다.

```java
@VisibleForTesting
void onFrameReady(DelayTarget delayTarget) {
  ...

  if (delayTarget.getResource() != null) {
    recycleFirstFrame(); // (1)
    DelayTarget previous = current; // (2)
    current = delayTarget; // (3)
    for (int i = callbacks.size() - 1; i >= 0; i--) { // (4)
      FrameCallback cb = callbacks.get(i);
      cb.onFrameReady();
    }
    if (previous != null) {
      handler.obtainMessage(FrameLoaderCallback.MSG_CLEAR, previous).sendToTarget(); // (5)
    }
  }

  loadNextFrame(); // (6)
}
```

1. 첫번째 프레임이 있다면 더 이상 필요없으니 제거한다.
1. 현재 프레임을 과거 프레임으로 바꾼다.
1. 로딩된 이미지를 현재 프레임으로 설정한다.
1. 콜백 (`GifDrawable`)에게 `onFrameReady`로 통보한다.
1. 이전 프레임은 핸들러에 `MSG_CLEAR`를 전달해서 제거한다.
1. `loadNextFrame`을 호출해 다음 프레임을 읽도록 한다. (성공하면 `onFrameReady`로 다시 돌아올 것이다.)

## 이미지 변경을 뷰에 전달하는 GifDrawable#onFrameReady

`GifDrawable`의 `onFrameReady`는 아래와 같다.

```java
@Override
public void onFrameReady() {
  if (findCallback() == null) {
    stop();
    invalidateSelf();
    return;
  }

  invalidateSelf(); // (1)

  if (getFrameIndex() == getFrameCount() - 1) { // (2)
    loopCount++;
  }

  if (maxLoopCount != LOOP_FOREVER && loopCount >= maxLoopCount) { // (3)
    notifyAnimationEndToListeners();
    stop();
  }
}
```

1. 기본적으로 `Drawable#invalidateSelf`를 호출해서 갱신한다.
1. 몇회나 반복했는지 체크한다.
1. 과도하게 반복했으면 에니메이션을 종료한다.

## 다음 프레임을 가져오는 loadNextFrame의 버그

이전의 코드는 `loadNextFrame`에서 크래쉬가 날 수 있는 버그가 있다.

```java
private void loadNextFrame() {
  ...
  if (startFromFirstFrame) {
    Preconditions.checkArgument(
        pendingTarget == null, "Pending target must be null when starting from the first frame");
    gifDecoder.resetFrameIndex();
    startFromFirstFrame = false;
  }
  ...
}
```

`startFromFirstFrame`이 호출되었고 `pendingTarget`이 있는 상황에서 `loadNextFrame`이 호출되면 `Preconditions.checkArgument`가 예외를 발생시킨다.

`GifDrawable#startFromFirstFrame`이 호출되면 `GitFrameLoader#setNextStartFromFirstFrame`가 호출된다.

```java
void setNextStartFromFirstFrame() {
  Preconditions.checkArgument(!isRunning, "Can't restart a running animation");
  startFromFirstFrame = true;
  if (pendingTarget != null) {
    requestManager.clear(pendingTarget);
    pendingTarget = null;
  }
}
```

이 시점에서 이미 `pendingTarget`이 호출되어 있다면 문제가 없다. `requestManager.clear(pendingTarget)`을 통해 캔슬되기 때문이다.

하지만 `startNextStartFromFirstFrame`이 먼저 호출되고, 러닝 상태가 아닌 경우에, 또 이미지 로딩이 완료되어 `onFrameReady`가 호출된 경우엔 어떻게 될까?

러닝 상태를 끄는 방법은 두가지가 있다.

1. `GifDrawable#stop`
2. `GifDrawable#setVisible`

예컨데 `setVisible` 설정으로 잠시 invisible에 간 동안 로딩이 완료되어 핸들러에서 의해 `onFrameReady`가 호출되었다면 `pendingTarget`은 존재하고 `startFromFirstFrame`이 설정된 상태가 된다.

러닝 상태를 켜는 방법도 두가지가 있다.

1. `GifDrawable#start`
2. `GifDrawable#setVisible`

에니메이션 중인 이미지 뷰에 대해 `setVisible`을 조작했다가 런타임 예외가 발생할 수 있는 셈이다.

해당 문제를 수정하기 위해 `onFrameReady` 코드를 고쳤다. 버그가 있는 코드는 아래와 같다.

```java
@VisibleForTesting
void onFrameReady(DelayTarget delayTarget) {
  ...
  if (!isRunning) {
    pendingTarget = delayTarget;
    return;
  }
  ...
}
```

아래는 수정된 코드이다.

```java
@VisibleForTesting
void onFrameReady(DelayTarget delayTarget) {
  ...
  if (!isRunning) {
    if (startFromFirstFrame) {
      handler.obtainMessage(FrameLoaderCallback.MSG_CLEAR, delayTarget).sendToTarget();
    } else {
      pendingTarget = delayTarget;
    }
    return;
  }
  ...
}
```

러닝 상태가 아닐 때 무조건 `pendingTarget`을 설정하지 않고 `startFromFirstFrame`인 경우에는 로딩된 이미지를 `MSG_CLEAR` 메시지를 핸들러로 보내 삭제 시켰다. 어차피 첫 프레임부터 재생시킬 것이기 때문에 준비된 다음 프레임은 아무런 의미가 없기 때문이다.

해당 패치는 내가 작성해 [bumptech/Glide#4193](https://github.com/bumptech/glide/pull/4193)과 [zjupure/GlideWebpDecoder#58](https://github.com/zjupure/GlideWebpDecoder/pull/58)에 적용되었고 아직 배포 전이다.