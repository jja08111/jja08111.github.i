---
 title: "[Kotlin] Flow 구현체 파헤치기"
 date: 2023-07-17 21:00:00 +0900
 categories: kotlin
 tags:
   - kotlin
   - flow
---

앱 개발에 있어 자주 사용되는 `Flow` 구현을 파헤쳐보겠다.

# Flow

`Flow`는 Cold Stream이고 단순한 예제는 다음과 같다.
5번 방출을 하는 flow를 선언하였고 이를 collect하여 순차적으로 값들을 출력한다.

```kotlin
flow {
    repeat(5) { n ->
        emit(n)
    }
}.collect { n ->
    println(n)
}

// 출력 결과
// 0
// 1
// 2
// 3
// 4
```

`flow` 함수 구현은 아래와 같다. `SafeFlow` 클래스를 생성하는 모습이다. 이때 `FlowCollector` 컨텍스트로 된 block 함수가 인자로 전달된다.

```kotlin
fun <T> flow(@BuilderInference block: suspend FlowCollector<T>.() -> Unit): Flow<T> = SafeFlow(block)
```

`SafeFlow`를 살펴보기 전에 `Flow`를 확인해보자. 먼저 `Flow`의 선언부이다. `collect`라는 함수가 있고 매개변수로 `FlowCollector`가 있다.
`FlowCollector`는 `emit` 함수가 존재한다. `FlowCollector`가 `fun interface`인 이유는 Kotlin에서 SAM(Single Abstract Method) 특성을 사용하기 위해서이다.

```kotlin
interface Flow<out T> {
    suspend fun collect(collector: FlowCollector<T>)
}

fun interface FlowCollector<in T> {
    suspend fun emit(value: T)
}
```

그렇다면 저 interface들이 어떻게 이용될까? `SafeFlow`를 보면 `AbstractFlow`를 상속하였다.
`AbstractFlow`는 `Flow`를 구현하였고 `collector`를 전달받아 `SafeCollector`로 감싸서 `collectSafely`를 호출한다.
`SafeFlow`의 `collectSafely` 함수는 앞서 `flow` 함수로 생성했던 그 `block`을 실행한다.
이때 block으로 작성했던 `FlowCollector.emit()`이 5번 호출되고, 방출된 값을 출력하게 된것이다.

```kotlin
class SafeFlow<T>(private val block: suspend FlowCollector<T>.() -> Unit) : AbstractFlow<T>() {

    override suspend fun collectSafely(collector: FlowCollector<T>) {
        collector.block()
    }
}

public abstract class AbstractFlow<T> : Flow<T>, CancellableFlow<T> {

    public final override suspend fun collect(collector: FlowCollector<T>) {
        val safeCollector = SafeCollector(collector, coroutineContext)
        try {
            collectSafely(safeCollector)
        } finally {
            safeCollector.releaseIntercepted()
        }
    }

    public abstract suspend fun collectSafely(collector: FlowCollector<T>)
}
```

무언가 알듯말듯하다. 이해하기 쉽도록 예제 코드를 수정하여 다시 살펴보자. 아래는 기존 예제 코드를 생략하지 않고 작성한 코드이다.

```kotlin
val myFlow = flow(
    block = {
        repeat(5) {
            emit(it)
        }
    }
)
myFlow.collect(
    collector = object : FlowCollector<Int> {
        override suspend fun emit(value: Int) {
            println(value)
        }
    }
)
```

`Flow`를 생성하면 `SafeFlow`의 생성자 매개변수로 아래의 블록이 전달되고

```kotlin
block = {
    repeat(5) {
        emit(it)
    }
}
```

`Flow`의 `collect`를 호출하면 익명의 클래스가 인자로 전달되어 해당 블록이 실행된다. 주석 처리된 부분을 보면 이해하기 수월하다.

```kotlin
class SafeFlow<T>(private val block: suspend FlowCollector<T>.() -> Unit) : AbstractFlow<T>() {

    override suspend fun collectSafely(collector: FlowCollector<T>) {
        // collector = object : FlowCollector<Int> {
        //     override suspend fun emit(value: Int) {
        //         println(value)
        //     }
        // }
        //
        // block = {
        //     repeat(5) {
        //         emit(it)
        //     }
        // }
        collector.block()
    }
}
```

# SharedFlow

이번에는 Hot Stream인 SharedFlow를 파헤쳐보자

TODO
