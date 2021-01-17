---
layout: post
title:  "RxJava 2의 연산자 결합"
date:   2021-01-13 22:45:00 +0900
categories: rx
---

[Vasya Drobushkov](https://medium.com/@krossovochkin)가 작성한 [Operator fusion in RxJava 2](https://proandroiddev.com/operator-fusion-in-rxjava-2-dcd6612cffae)를 허락을 맡고 번역합니다. 좋은 글을 작성하고 번역을 허락하여 감사합니다.

## 서문

RxJava는 강력한 라이브러리지만 몇가지 문제도 있습니다. 특히 성능과 메모리 문제는 라이브러리가 풀려는 문제와 기술적인 관점에서 솔루션이 설계되는 방식에서 비롯됩니다.

RxJava는 간접비용(overhead)를 최소화하기 위해 "연산자 결합" (Operator fusion)이라 불리는 여러 최적화를 가집니다. 그리고 우리는 이 글에서 거기에 대해 다룰 예정입니다.

하지만 먼저 RxJava 반응형 타입이 어떻게 동작하는지와 무슨 문제가 있는지 요약하겠습니다.

### Observable

![Observable]({{ "/assets/1-observable.png" | absolute_url }})

Observable를 사용할 때 크게 세가지 부분이 있습니다. Observable, Observer, Disposable. 

우리 모두 Observable을 알고 있고 어떻게 만드는지 압니다. (예: `Observable.just("Hello, World!")`) Observable은 각 Rx 체인의 생성 블록입니다.Observable을 작동하려면 `subscribe(...)` 메서드에 `Observer`를 전달해 구독해야합니다.

Observer는 기본적으로 `onSubscribe`, `onNext`, `onComplete` 콜백을 가진 인터페이스입니다.

Observer가 Observable을 구독하면 (Observer가 나중에 필요할 때 Rx 체인을 없앨(dispose) 수 있게) Observable은 Disposable 객체를 생성하고 `onSubscribe` 콜백을 통해 Observer로 전달합니다. 

이 작업이 완료되면, 통신이 연결되고 Observable이 추가적인 대기없이 `onNext`를 통해 값들을 보낼 수 있습니다.

그래서 Observable은 배압(backpressure)을 지원하지 않습니다. Observer가 Observable에게 더 이상 값을 보내지 말라 알릴 방법이 없습니다.

### Flowable

![Flowable]({{ "/assets/1-flowable.png" | absolute_url }})

Flowable은 모든 점이 비슷합니다. 하지만 Observer와 Disposable 대신 Subscriber와 Subscription를 가집니다.

Subscription은 추가로 `request(n)` 메서드가 있습니다. 이를 이용하여 Subscriber는 명시적으로 Flowable에게 요청된 양의 아이템을 보내게 요청할 수 있습니다.

값을 요청하지 않으면 Flowable은 어떤 값도 내보내지 않습니다. 이게 Flowable이 배압을 지원하는 이유입니다.

### 조립과 구독

RxJava 반응형 타입을 사용할 때 두 중요한 단계인 조립(assembly)과 구독(subscribe)이 있습니다.

조립할 때 Rx 체인이 만들어지고 구독할 때 우리는 Rx 체인을 시작합니다.

아래의 예제를 고려해보세요.

```kotlin
Observable.just(1, 2, 3)
    .map { it + 1 }
    .filter { it < 3 }
    .subscribe()
```

이 경우 위에서 아래로 이동하면 다음과 같인 일이 일어납니다.

  * ObservableJust 객체가 생성됩니다.
  * ObservableMap 객체가 생성되고 이전에 생성된 ObservableJust를 새  Observable에 전달합니다. (그래서 함께 조합됩니다.)
  * ObservableFilter 객체가 생성되고 이전에 생성된 (ObservableJust가 내장된) ObservableMap을 새 Observable에 전달합니다.
  * ObservableFilter를 구독하여 실제 구독(subscription)을 작동합니다.
  * ObservableFilter는 내부의 자체 옵저버를 만들어 ObservableMap을 구독합니다.
  * ObservableMap은 내부의 자체 옵저버를 만들어 ObservableJust를 구독합니다.
  * ObservableJust는 onSubscribe 이벤트를 다운스트림으로 보냅니다. (다른 옵저버블들도 이 이벤트를 체인의 최신 옵저버블까지 다운스트림으로 보냅니다.)
  * ObservableJust는 값을 발행하고 `onNext` 콜백을 통해 다운스트림으로 전달합니다.

![Rx 체인]({{ "/assets/1-rxchain.png" | absolute_url }})

이 짧은 Rx 체인에서 꽤 많은 일이 일어나는 것을 볼 수 있습니다. 이 체인이 Flowable 타입이었다면 `request(n)`의 추가 통신이 발생해 더 복잡해집니다.

### 큐와 동기화

연산자 안에 이벤트를 처리하기 위한 내부 큐가 있습니다.

큐에 대한 접근은 직렬화되어야 합니다. (이는 적절한 동기화를 통해서 접근해야한다는 것입니다.)

RxJava2는 (성능을 위해) Atomic(예: AtomicInteger)과 compareAndSet 메서드를 이용한 무한 반복에 기반한 블록킹 없는 동기화를 가지고 있습니다. 라이브리리에서 일반적으로 다음과 같은 코드를 볼 수 있습니다.

```java
for (; ; ) {
    long r = state.get();

    if ((r & COMPLETED_MASK) != 0L) {
        return;
    }

    long u = r | COMPLETED_MASK;
    // (active, r) -> (complee, r) 전환
    if (state.compareAndSet(r, u)) {
        // 만약 요청된 양이 0이 아니면 큐를 비웁니다.
        if (r != 0L) {
            postCompleteDrain(u, actual, queue, state, isCancelled);
        }

        return;
    }
}
```

체인의 각 연산자가 개별적으로 큐를 가진다면 원자 객체를 가진 연산자의 큐는 간접비용을 가집니다.

### 문제점

위의 모든 것을 고려해 RxJava가 가진 문제는 다음과 같습니다. 

  * 조립 시간 간접비용 - Rx 체인을 생성하면 꽤 많은 객체를 만들어야 하고 메모리 간접비용를 유발합니다.
  * 구독 시간 간접비용 - Rx 체인을 구독하고 많은 통신이 이루어지면 성능 간접비용이 발생합니다.
  * 할당과 직렬화 간접비용 - 매 연산자마다 큐와 원자 객체같은 내부 구조를 만드는 것은 메모리와 성능의 간접비용을 유발합니다.

## 연산자 결합

성능과 메모리 문제를 풀기 위해 연산자 결합이 있습니다.

두가지 형태의 결합이 있습니다.

  * 매크로 결합 (Macro fusion) - 몇 개의 연산을 하나로 합병하여 조합과 구독 동안 생성되는 객체의 수를 최소화 합니다.
  * 마이크로 결합 (Micro fusion) - 연산자 사이의 불필요한 동기화와 (큐와 같은) 내부 공유 구조를 제거합니다.

## 조립에서 매크로 결합

### 조립 결합

조립에서 매크로 결합은 Observable과 조합 때 생성된 객체의 수를 최소화하는 데 집중합니다. "조립"은 이런 것을 이야기 합니다.

```java
@CheckReturnValue
@SchedulerSupport(SchedulerSupport.NONE)
public final <R> Observable<R> map(Function<? super T, ? extends R> mapper) {
    ObjectHelper.requireNonNull(mapper, "mapper is null");
    return RxJavaPlugins.onAssembly(new ObservableMap<T, R>(this, mapper));
}
```

### 조립 결합의 기본

Observable들을 최적화하는 단순한 방법은 특별한 경우에 대한 검사를 추가하여, 일반적인 Observable 보다 구현 측면에서 단순한 Observable을 생성하는 것입니다.

예를 들어 `Observable.fromArray`를 살펴봅시다. 이는 항목의 개수가 0이나 1이면 각각 `Observable.empty` 또는 `Observable.just`로 다운그레이드할 수 있습니다.

```java
@CheckReturnValue
@SchedulerSupport(SchedulerSupport.NONE)
public static <T> Observable<T> fromArray(T... items) {
    ObjectHelper.requireNonNull(mapper, "mapper is null");
    if (items.length == 0) {
        return empty();
    } else if (items.length == 1) {
        return just(items[0]);
    }
    return RxJavaPlugins.onAssembly(new ObservableFromArray<T>(items));
}
```

### ScalarCallable

결합(fuseable) 패키지의 첫 "고급" 최적화는 ScalarCallable 인터페이스입니다.

```java
public interface ScalarCallable<T> extends Callable<T> {

    // 예외를 내지 않도록 오버라이드합니다.
    @Override
    T call();
}
```

기본적으로 자바의 Callable 인터페이스와 기본적으로는 동일한데 예외를 내지 않는 차이가 있습니다.

ScalarCallable란 반응형 타입은 조립 시간 동안 안전하게 추출할 수 있는 상수값을 가지고 있는 인터페이스입니다. 특히 이런 반응형 타입은 정확히 값 하나를 가지거나 항목 자체가 완전히 없습니다.

그래서 우리가 `call` 메서드를 호출할 때, 반환 값이 null이었다면 반응형 타입은 어떤 값도 가지지 않으며, null이 아닌 값이 있다면 하나의 값만 가집니다.

설명된 제약을 기반으로 Observable, Floawable, Maybe의 empty, just 연산자만이 이 인터페이스로 표시될 수 있습니다.

xMap 연산자(flatMap, switchMap, contactMap)로 예를 들겠습니다. 소스가 이 인터페이스로 표시될 수 있다면 이 최적화를 적용할 수 있습니다.

```java
@CheckReturnValue
@SchedulerSupport(SchedulerSupport.NONE)
public final <R> Observable<R> flatMap(Function<? super T, ? extends ObservableSource<? extends R>> mapper, boolean delayErrors, int maxConcurrency, int bufferSize) {

    if (this instanceOf ScalarCallable) {
        T v = ((ScalarCallable<T>)this).call();
        if (v == null) {
            return empty();
        }
        return ObservableScalarXMap.scalarXMap(v, mapper);
    }
    return RxJavaPlugins.onAssembly(new ObservableFlatmap<T, R>(this, ...));
}
```

이 경우에 소스가 ScalarCallable로 표시되었다면 (꽤 무거운) 전체 구현 대신에 xMap의 단순화된 버전으로 변환할 수 있습니다.

### FuseToXXX

`fuseable` 패키지에 세가지 인터페이스가 있습니다.

```java
public interface FuseToObservable<T> {
    Observable<T> fuseToObservable();
}

public interface FuseToFlowable<T> {
    Floawable<T> fuseToFlowable();
}

public interface FuseToMaybe<T> {
    Maybe<T> fuseToMaybe();
}
```

`FuseToObservable`을 살펴봅시다. 다른 인터페이스들도 전부 비슷합니다.

다음 Rx 체인을 고려해봅시다.

```java
Observable.range(1, 10)
    .count()
    .toObservable()
    .subscribe()
```

여기서 range를 만들고 내보낸 항목의 갯수를 세어봅시다. `count` 연산자는 `Single`을 반환합니다. 하지만 우리는 Observble을 받길 원하고요. (다른 Observable과 이 결과를 병합하는 예로 들 수 있습니다.) Rx 체인에 추가적인 `toObservable` 연산자를 추가해 더 보잡하고 무거워졌습니다.

FuseToObservable은 여기에 도움이 됩니다. 이 인터페이스가 하는 일은 다음과 같습니다. Observable이 아닌 반응형 타입을 반환하는 몇몇 연산자들은 내부에 Observable을 반환하는 구현을 가지고 있고 이 구현을 `toObservable` 호출해 추출할 수 있습니다.

`count` 예제를 살펴봅시다.

```java
@CheckReturnValue
@SchedulerSupport(SchedulerSupport.NONE)
public interface Single<Long> count() {
    return RxJavaPlugins.onAssembly(new ObservableCountSingle<T>(this));
}
```

기본적으로 `ObservableCountSingle`을 반환합니다. 하지만 이 연산자를 조금 더 깊게 살펴보면 이 구현이 `FuseToObservable` 인터페이스를 구현하고 결합 모드로 불릴 때 다른 구현을 제공하는 것을 알 수 있습니다.

```java
public interface class ObservableCountSingle<T> extends Single<Long> implements FuseToObservable<Long> {
    ...

    @Override
    public Observable<Long> fuseToObservable() {
        return RxJavaPlugins.onAssembly(new ObservableCount<T>(source));
    }
}
```

그리고 `toObservable`을 호출하면 그 구현이 추출되고 `toObservable`을 위한 Observable이 생성되지 않아 효율적입니다.

```java
public final Observable<T> toObservable() {
    if (this instanceOf FuseToObservable) {
        return ((FuseToObservable<T>)this).fuseToObservable();
    }
    return RxJavaPlugins.onAssembly(new SingleToObservable<T>(this));
}
```

## 구독 시 매크로 결합

구독 시 매크로 결합은 조립에 이루어지는 것과 동일한 최적화에 중점을 두지만 `subscribeActual` 메서드 내에서 수행됩니다.

```java
@Override
public void subscribeActual(Observer<? super U> t) {
    source.subscribe(new MapObserver<T, U>(t, function));
}
```

구독 전에 데이터가 불활실하기 때문에 조립 시점에 최적화가 불가능할 수 있습니다. 조립 보다 구독 때 최적화를 하는 것이 더 편리할 수 있습니다.

### 구독 결합의 기본

조립때와 마찬가지로 어떤 특수한 조건을 확인하여 일반적이지 않는 단순한 구현을 사용하는 간단한 최적화가 있습니다. 예를들어 Observable.amb는 제공된 소스의 갯수를 확인하여 무거운 AmbCoordinator를 인스턴스화할지 말지 결정합니다.

```java
public void subscribeActual(Observer<? super T> observer) {
    ObservableSource<? extends T>[] sources = this.sources;
    int count = 0;

    ...

    if (count == 0) {
        EmptyDisposable.complete(observer);
        return;
    } else if (count == 1) {
        sources[0].subscribe(observer);
        return;
    }

    AmbCoordinator<T> ac = new AmbCoordinator<T>(observer, count);
    ac.subscribe(sources);
}
```

### Callable

조립동안 ScalarCallable 인터페이스로 몇가지 최적화를 합니다. 구독할 때도 Callable 인터페이스를 이용한 비슷한 최적화가 있습니다.

> 주의: ScalarCallable이 Callable을 확장한 것 처럼 - ScalarCallable로 조립 때 적용한 최적화들은 구독 때 Callable에 적용할 수 있습니다.

Callable 인터페이스로 표시된 Observable에 대한 구독할 때 xMap 연산자를 사용할 때는 단순화된 구현으로 대체할 수 있습니다.

```java
@Override
public void subscribeActual(Observer<? super U> t) {

    if (ObservableScalarXMap.tryScalarXMapSubscribe(source, t, mapper)) {
        return;
    }

    source.subscribe(new MergeObserver<T, U>(t, mapper, delayErrors, maxConcurrency, bufferSize));
}
```

## 마이크로 결합

마이크로 결합은 어떤 동기화와 큐와 같은 공유 내부 구조를 줄여 간접비용을 최소화하는 것을 목표로 합니다.

### ConditionalSubscriber

`Flowable.filter`와 같은 연산자가 사용될 때 어떤 일이 일어나는지를 살펴봅시다.

![ConditionalSubscriber]({{ "/assets/1-conditionalsubscriber.png" | absolute_url }})

업스트림, filter 연산자, 다운스트림을 가지고 있습니다. 값이 5보다 작은지 검사하는 필터가 있다고 가정합시다. 구독이 되면 다운스트림은 업스트림에게 몇개의 항목을 요청해야 합니다.

  * 필터에게 다운스트립은 1개의 항목을 요청합니다.
  * 필터는 업스트림에게 1개의 항목을 요청합니다.
  * 업스트림은 항목을 생성하고 필터로 전달합니다. (숫자가 1이라고 합시다.)
  * 필터는 값이 술어(predicate)를 만족하는지 확인하여 다운스트림에 전달합니다.
  * 다운스트림은 항목을 받아들이고 필터에게 하나 더 요청합니다.
  * 필터는 업스트림에게 1개의 항목을 요청합니다.
  * 업스트림은 항목을 만들고 필터에게 전달합니다. (숫자가 10이라고 합시다.)
  * 필터는 값이 술어를 만족하지 못한 것을 확인하고 다운스트림에게는 전달하지 않습니다. 다운스트림이 하나의 항목을 요청하였지만 필터가 제공하지 않았고 그래서 필터는 업스트림에게 하나 더 요청합니다.
  * 스트림이 종료될 때까지 이 작업은 반복됩니다.

연산자 간 많은 통신이 있던 것을 볼 수 있고 더 중요한 것은 각 이벤트가 직렬화 된 방식으로 제공되기 때문에 약간의 간접비용이 있습니다.

그 사이에 두개의 필터 연산이 있다고 가정합시다. 통신은 상당한 간접비용을 증가시킬 수 있습니다.

![두개의 필터]({{ "/assets/1-two-filters.png" | absolute_url }})

ConditionalSubscriber는 이 상황을 구합니다.

```java
public interface ConditionalSubscriber<T> extends FlowableSubscriber<T> {
    boolean tryOnNext(T t);
}
```

일반적으로 Subscriber의 `onNext` 콜백은 아무 값도 반환하지 않습니다. 업스트림은 단순히 값을 콜백을 통해 제공하고 다운스트림의 새로운 요청을 기다립니다.

ConditionalSubscriber는 추가적인 `tryOnNext` 메서드를 가집니다. 이는 `onNext`와 비슷한데 값을 받을 수 있느냐 거절되었냐에 따라 불리언 값을 즉히 반환합니다.

이는 업스트림이 직접 응답을 수신할 때 `request(n)` 호출이 요청되는 회수를 줄일 수 있습니다.

`Flowable.filter` 구현의 예를 살펴보면 업스트림 filter가 직접 다운스트림 filter의 술어를 접근하는 것을 볼 수 있습니다.

```java
@Override
public boolean tryOnNext(T t) {
    ...

    boolean b;
    try {
        b = filter.test(t);
    } catch (Throwable e) {
        fail(e);
        return true;
    }
    return b && downstream.tryOnNext(t);
}
```

이는 몇 차례의 요청을 절약합니다.

![직접 다운스트림 접근]({{ "/assets/1-direct-access.png" | absolute_url }})

최적화가 연쇄 필터 연산자의 간접 비용을 줄이고자 하는게 목표라면 그다지 놀랍지는 않을 것입니다. (어쨌든 하나의 필터 연산자로 바꿀 수 있는 것 처럼 보이니깐요.) 좋은 점은 `Flowable.map` 역시 ConditionalSubscriber로 구현할 수 있고 여러 필터와 맵 연산자가 연쇄적으로 함께 있다면 간접 비용을 더 줄일 수 있습니다.

```java
@Override
public boolean tryOnNext(T t) {
    ...

    U v;

    tru {
        v = ObjectHelper.requireNonNull(mapper.apply(t), ...);
    } catch (Throwable ex) {
        fail(ex);
    }
}
```

### Queue fuseable

가장 복잡한 마이크로 결합은 연산자 사이의 공유 내부 큐들에 기반합니다. 전체 최적화는 `QueueSubscription` 인터페이스에 기반합니다.

```java
public interface QueueSubscription<T> extends QueueFuseable<T>, Subscription {
}
```

기본적으로 Queue와 Subscription이 한 인터페이스 아래 있습니다. 하지만 큐 인터페이스는 자바의 간단한 인터페이스가 아닙니다. 대신 추가적인 메서드 `requestFusion`을 가집니다.

```java
public interface QueueFuseable<T> extends SimpleQueue<T> {

    int requestFusion(int mode);

    int NONE = 0;
    int SYNC = 1;
    int ASYNC = 2;
    int ANY = SYNC | ASYNC;
    int BOUNDARY = 4;
}
```

Flowable과 Subscriber 사이에 `onXXX` 콜백을 사용하는 일반적인 커뮤니케이션과 비교해서, 업스트림은 Subscription 뿐만 아니라 QueueSubscription를 제공할 수 있습니다. QueueSubscription은 직접적으로 내부의 큐에 접근할 수 있습니다.

동작방식은 다음과 같습니다. 먼저 onSubscribe 동안 업스트림과 다운스트림은 결합에 합의하고 작동할 결합 모드(fusion mode)를 선택합니다. 결합이 합의되면 새로운 통신 구현이 사용됩니다. 그렇지 않다면 `onXXX` 콜백을 사용하는 전통적인 통신이 수립됩니다.

일반적으로 결합이 수립되면 다운스트림은 업스트림 큐의 `poll()` 메서드를 직접 호출하여 항목을 얻을 것입니다.

![poll 호출]({{ "/assets/1-poll.png" | absolute_url }})

세가지 결합 모드가 있습니다.

  * NONE - 결합을 하지 않는 것입니다.
  * SYNC - 동기적인 결합이 이루어집니다.
  * ASYNC - 비동기적인 결합이 이루어집니다.

ANY는 단순히 SYNC이거나 ASYNC입니다. (이는 업스트림이 무엇을 지원하냐에 따라 수립이 결정됩니다.)

### SYNC 결합

업스트림의 값이 이미 정적이거나 poll()이 호출될 때 동기적으로 생성된다면 SYNC 결합을 사용할 수 있습니다.

만약 업스트림과 다운스트림이 동기 결합 모드를 사용하기로 합의되면 아례의 계약을 따릅니다.

  * 다운스트림은 필요에 따라 직접 `poll()` 메서드를 호출합니다.
  * `poll()` 메서드는 예외를 던질 수 있습니다. 이는 `onError`와 동일합니다.
  * `poll()` 메서드는 null을 반환할 수 있습니다. 이는 `onComplete`와 동일합니다.
  * `poll()` 메서드는 null이 아닌 값을 반환할 수 있습니다. 이는 `onNext`와 동일합니다.
  * 업스트림은 어떤 `onXXX` 콜백도 호출하지 않습니다.

![SYNC 결합]({{ "/assets/1-sync-fusion.png" | absolute_url }})

SYNC 결합 모드를 지원하는 연산자의 예는 `Flowable.range`입니다.

```java
@Override
public final int requestFusion(int mode) {
    return mode & SYNC;
}

@Nullable
@Override
public final Integer poll() {
    int i = index;
    if (i == end) {
        return null;
    }
    index = i + 1;
    return i;
}
```

### ASYNC 결합

poll()을 호출할 때 나중에(eventually) 업스트림의 값이 사용가능해질때 ASYNC 결합 모드를 사용할 수 있습니다.

업스트림과 다운스트림이 비동기 결합 모드를 사용하기로 합의되면 아래의 계약을 따릅니다.

  * 업스트림은 일반적으로 `onError`와 `onComplete`를 보냅니다.
  * onNext은 업스트림의 값 뿐 아니라 null을 대신 가질 수 있습니다. 다운스트림은 이 onNext를 poll()을 호출할 수 있는 표식으로 취급할 수 있습니다.
  * poll()의 호출자는 예외를 처리해야 합니다.

> 맞습니다. RxJava에서 onNext는 null값을 가질 수 있습니다.

![ASYNC 결합]({{ "/assets/1-async-fusion.png" | absolute_url }})

ASYNC 결합 모드를 지원하는 연산자의 예는 `Flowable.filter`입니다.

```java
public T poll() throws Exception {
    QueueSubscription<T> qs = this.qs;
    Predicate<? super T> f = filter;

    for (;;) {
        T t = qs.poll();
        if (t == null) {
            return null;
        }

        if (f.test(t)) {
            return t;
        }

        if (sourceMode == ASYNC) {
            qs.request(1);
        }
    }
}
```

```java
@Override
public boolean tryOnNext(T t) {
    if (done) {
        return false;
    }

    if (sourceMode != NONE) {
        return downstream.tryOnNext(null);
    }

    boolean b;
    try {
        b = filter.test(t);
    } catch (Throwable e) {
        fail(e);
        return true;
    }
    return b && downstream.tryOnNext(t);
}
```

결합 모드를 지원하는 몇개의 연산자를 살폈지만 이 모드를 활성화하기 위해 먼저 결합을 요청해야 합니다. `Flowable.flatMap` 내에서 결합 모드가 요청하는 예입니다.

```java
@Override
public void onSubscribe(Subscription s) {
    if (SubscriptionHelper.setOnce(this, s)) {
        if (s instanceOf QueueSubscription) {
            QueueSubscription<U> qs = (QueueSubscription<U>) s;
            int m = qs.requestFusion(QueueSubscription.ANY | QueueSubscription.BOUNDARY);
            if (m == QueueSubscription.SYNC) {
                fusionMode = m;
                queue = qs;
                done = true;
                paent.drain();
            }
            if (m == QueueSubscription.ASYNC) {
                fusionMode = m;
                queue = qs;
            }
        }
        s.request(bufferSize);
    }
}
```

소스가 `QueueSubscription`을 구현한다면 구독 동안 결합 모드 ANY가 요청되는 것을 볼 수 있습니다. 소스에 의해 수락된 모드에 따라 다른 전략이 적용됩니다.

### QueueSubscription 스레드

큐 결합에서 스레드 문제를 유의해야 합니다. 다운 스트림이 업 스트림에 직접 접근할 수 있다면 업 스트림과 다운 스트림이 서로 다른 스레드에서 작동하는 경우 문제가 발생할 수 있습니다.

![다른 스레드에서 호출]({{ "/assets/1-threads.png" | absolute_url }})

map 안에서 무거운 연산이 이루어질 수 있습니다. 이 경우 (직접 폴링일 경우) 해당 계산이 다른 스레드로 유출될 수 있습니다. 이 문제를 해결하기 위해 추가적인 마커 옵션 BOUNDARY가 있습니다. 이는 poll 메서드의 호출자가 다른 스레드일 수 있음을 나타냅니다. 연산자는 BOUNDARY 옵션을 무시할 수 있고, 다른 스레드에서 큐의 접근을 허용할 수도, BOUNDARY 옵션이 요청될 경우 결합을 거부할 수 있습니다.

`Observable.map` 구현을 살펴보면 transitiveBoundaryFusion 헬퍼 함수를 사용하는 것을 볼 수 있습니다.

```java
@Override
public int requestFusion(int mode) {
    return transitiveBoundaryFusion(mode);
}
```

(transitiveBoundaryFusion) 내부는 BOUNDARY 모드를 허용하지 않습니다.

```java
protected final int transitiveBoundaryFusion(int mode) {
    QueueSubscription<T> qs = this.qs;
    if (qs != null) {
        if ((mode & BOUNDARY) == 0) {
            int m = qs.requestFusion(mode);
            if (m != NONE) {
                sourceMode = m;
            }
            return m;
        }
    }
    return NONE;
}
```

## 결론

이 글에서 RxJava의 몇가지 최적화의 개요를 살펴보고 몇가지 흥미로운 것을 봤습니다.

  * RxJava 2의 Observable은 배압을 지원하지 않습니다. (더 이상의 항목이 없다는 걸 업스트림에 알릴 방법이 없습니다.)
  * 몇가지 내부 최적화가 null 값을 콜백에 전하는 것에 기반하기 때문에 RxJava에서 null 값을 전달하는 것은 안됩니다.
  * 최적화를 모두 끄고 싶다면 `hide()` 연산자는 매우 중요합니다.
  * 연산자 결합은 화려하지만 여러 초적화 중의 하나에 불과합니다. 모든 연산자에서 사용할 수 있는 것은 아닙니다. 최적화 될 것 같아 보이지만 경우에 따라 어떤 최적화도 작동하지 않아 놀랄 수 있습니다. 그 이유는 이런한 최적화가 일부 중요한 부분과 공통 문제에 적용되고 일반적인 최적화는 매우 어렵기 때문입니다.

그래서 RxJava 내부의 모든 것이 효율적이다 생각하고 긴 Rx 체인을 만들지 마십시요. 당신의 코드를 벤치마크 하고 중요한 체인을 프로파일링해서 개별적으로 최적화할 수 있는 방법을 찾으세요.

아래의 글을 같이 읽으세요.

  * [Operator-fusion (Part 1)](https://akarnokd.blogspot.com/2016/03/operator-fusion-part-1.html)
  * [Operator fusion (part 2 - final)](https://akarnokd.blogspot.com/2016/04/operator-fusion-part-2-final.html)

행복한 코딩하세요.