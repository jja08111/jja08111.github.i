---
title: "[Kotlin] reified, in, out은 무엇일까?"
date: 2022-07-30 10:00:00 +0900
categories: kotlin
tags:
  - kotlin
---

최근 앱을 개발하며 마주친 `reified`, `in`, `out` 키워드를 보며 이들이 정확히 어떤 의미를 가지는지 궁금해졌다.

# reified

이는 제네릭 타입을 런타임에 알 수 있게 해준다. 컴파일 타임 때는 제네릭 코드의 T가 어떤 타입인 지 알 수 있다.
하지만 [런타임 때는 타입 정보를 지워버리기 때문](https://stackoverflow.com/a/339708/14434806)에 런타임 시에는 무슨 타입인지 알 수 없다. 그래서 아래와 같이 코딩하면 컴파일 에러가 발생한다.

```kotlin
fun <T> foo(bar: T) {
    val classType = T::class // <-- Cannot use 'T' as reified type parameter. Use a class instead.
}
```

이럴 때 `reified`를 사용하면 런타임에도 타입 정보를 얻을 수 있다. `refiied`는 `inline`과 같이 써야만 한다.

```kotlin
inline fun <reified T> foo(bar: T) {
    val classType = T::class
}
```

그런데 `inline`은 무엇일까? 간단히 짚고 넘어가자면 일반 함수는 컴파일 시 람다를 객체로 만들어 메모리 할당과 런타임 오버헤드를 초래할 수 있다.
이와 다르게 inline 함수는 해당 코드를 람다 표현 자체를 복사하여 앞서 말한 오버헤드를 제거할 수 있다.

# in, out
