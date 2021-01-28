---
layout: post
title:  "코루틴과 Retrofit를 쓸 때 디스패처는 필요없습니다."
date:   2021-01-28 23:55:00 +0900
categories: android
---

[Retrofit](https://square.github.io/retrofit/)을 사용하며 아래와 같은 인터페이스가 있다고 합시다.

```kotlin
interface Service {
    @GET("/") suspend fun body(): String
    @GET("/") suspend fun bodyNullable(): String?
    @GET("/") suspend fun response(): Response<String>

    @GET("/{a}/{b}/{c}")
    suspend fun params(
        @Path("a") a: String,
        @Path("b") b: String,
        @Path("c") c: String
    ): String
}
```

아래와 같이 호출하는 경우가 많을 것입니다.

```kotlin
val user: LiveData<String> = liveData(Dispatchers.IO) {
    val data = service.body()
    emit(data)
}
```

사실은 `Dispatchers.IO`는 의미가 없습니다. Retrofit의 구현을 살펴봅시다.

인터페이스의 메서드는 `HttpServiceMethod` 메서드에서 해당 메서드를 어떻게 처리할지 결정합니다.

```java
abstract class HttpServiceMethod<ResponseT, ReturnT> extends ServiceMethod<ReturnT> {
  static <ResponseT, ReturnT> HttpServiceMethod<ResponseT, ReturnT> parseAnnotations(
    ...

    // 서스펜드 펑션이 아니면
    if (!isKotlinSuspendFunction) {
      // CallAdapted()를 반환합니다.
      return new CallAdapted<>(requestFactory, callFactory, responseConverter, callAdapter);
    } else if (continuationWantsResponse) {
      // Response를 반환하는 경우 SuspendForResponse로 갑니다.  
      return (HttpServiceMethod<ResponseT, ReturnT>)
          new SuspendForResponse<>(
              requestFactory,
              callFactory,
              responseConverter,
              (CallAdapter<ResponseT, Call<ResponseT>>) callAdapter);
    } else {
      // 리스펀스를 반환하지 않는 경우 SuspendForBody로 갑니다. 
      return (HttpServiceMethod<ResponseT, ReturnT>)
          new SuspendForBody<>(
              requestFactory,
              callFactory,
              responseConverter,
              (CallAdapter<ResponseT, Call<ResponseT>>) callAdapter,
              continuationBodyNullable);
    }
  }
```

우리는 `SuspendForResponse`와 `SuspendForBody`만 살펴봅면 됩니다.

먼저 `SuspendForResponse`을 살펴봅시다.

```java
static final class SuspendForResponse<ResponseT> extends HttpServiceMethod<ResponseT, Object> {
  private final CallAdapter<ResponseT, Call<ResponseT>> callAdapter;

  ...
  @Override
  protected Object adapt(Call<ResponseT> call, Object[] args) {
    call = callAdapter.adapt(call);

    Continuation<Response<ResponseT>> continuation =
        (Continuation<Response<ResponseT>>) args[args.length - 1];

    try {
      return KotlinExtensions.awaitResponse(call, continuation);
    } catch (Exception e) {
      return KotlinExtensions.suspendAndThrow(e, continuation);
    }
  }
}
```

`KotlinExtensions.awaitResponse`를 호출하거나 예외가 발생하면 `KotlinExtensions.suspendAndThrow`를 호출합니다.

```java
static final class SuspendForBody<ResponseT> extends HttpServiceMethod<ResponseT, Object> {
  ...

  @Override
  protected Object adapt(Call<ResponseT> call, Object[] args) {
    call = callAdapter.adapt(call);

    ...
    try {
      return isNullable
          ? KotlinExtensions.awaitNullable(call, continuation)
          : KotlinExtensions.await(call, continuation);
    } catch (Exception e) {
      return KotlinExtensions.suspendAndThrow(e, continuation);
    }
  }
}
```

여기에 `awaitNullable`, `await`, `suspendAndThrow`가 보입니다.

```kotlin
suspend fun <T : Any> Call<T>.await(): T {
  return suspendCancellableCoroutine { continuation ->
    continuation.invokeOnCancellation {
      cancel()
    }
    enqueue(object : Callback<T> {
      override fun onResponse(call: Call<T>, response: Response<T>) {
        if (response.isSuccessful) {
          if (body == null) {
            ...
            // body가 없으면 에러와 함께 이어갑니다.
            continuation.resumeWithException(e)
          } else {
            // 정상적으로 body를 넘기고 이어 갑니다.
            continuation.resume(body)
          }
        } else {
          // 요청이 실패했으면 HttpException을 넘기며 이어갑니다.
          continuation.resumeWithException(HttpException(response))
        }
      }

      override fun onFailure(call: Call<T>, t: Throwable) {
        // 이것도 HttpException을 넘기며 이어갑니다.
        continuation.resumeWithException(t)
      }
    })
  }
}
```

`await`은 [Call.enqueue](https://square.github.io/retrofit/2.x/retrofit/retrofit2/Call.html#enqueue-retrofit2.Callback-)를 수행합니다. `Call.euqueue`는 Retrofit Call의 메서드로 자체적으로 비동기 호출을 하고 결과에 따라 콜백을 처리합니다.

즉, 우리가 만든 인터페이스의 수행은 코루틴의 흐름을 타는 것이 아니라 자바 스레드 풀에서 돌게 됩니다.

해당 메서드의 콜백에서 결과에 따라 [Continuation.resumWithException](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines/resume-with-exception.html), [Continuation.resume](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines/resume.html)를 수행합니다.

1. `resumeWithException`은 suspend 메서드 이전, 멈추었던 지점으로 돌아가는데 그 지점에서 예외를 던집니다.
2. `Continuation.resume`은 suspend 메서드 호출 이전, 멈추었던 지점으로 돌아가서 다음 코드를 수행합니다.

```kotlin
@JvmName("awaitNullable")
suspend fun <T : Any> Call<T?>.await(): T? {
  return suspendCancellableCoroutine { continuation ->
    continuation.invokeOnCancellation {
      cancel()
    }
    enqueue(object : Callback<T?> {
      override fun onResponse(call: Call<T?>, response: Response<T?>) {
        if (response.isSuccessful) {
          continuation.resume(response.body())
        } else {
          continuation.resumeWithException(HttpException(response))
        }
      }

      override fun onFailure(call: Call<T?>, t: Throwable) {
        continuation.resumeWithException(t)
      }
    })
  }
}
```

코드가 조금은 다르지만 결국 패턴은 비슷합니다. `Call.enqueue`로 Retrofit 자바 스레드 풀에서 수행시키고 처리가 끝났을 때 `Continuation`을 이용해서 suspend 메서드를 호출시킨 곳에 돌려 보냅니다.

```kotlin
suspend fun <T> Call<T>.awaitResponse(): Response<T> {
  return suspendCancellableCoroutine { continuation ->
    continuation.invokeOnCancellation {
      cancel()
    }
    enqueue(object : Callback<T> {
      override fun onResponse(call: Call<T>, response: Response<T>) {
        continuation.resume(response)
      }

      override fun onFailure(call: Call<T>, t: Throwable) {
        continuation.resumeWithException(t)
      }
    })
  }
}
```

역시 마찬가집니다.

결론적으로 말해 레트로핏을 쓸 때 별도의 디스패처를 쓸 필요가 없습니다.

```kotlin
val user: LiveData<String> = liveData {
    val data = service.body()
    emit(data)
}
```
