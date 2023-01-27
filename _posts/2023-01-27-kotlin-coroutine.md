---
 title: "Kotlin Coroutine"
 date: 2023-01-27 21:12:00 +0900
 categories: kotlin
 tags:
   - kotlin
   - coroutine
---

코루틴은 코틀린에서 제공하는 비동기 솔루션이다.
코드를 실행하는 동시에 다른 코드를 실행하는 점이 경량 스레드라고 생각할 수도 있지만 스레드와는 차이점이 존재한다.
코루틴은 특정 스레드에 속하지 않는다. 즉, 코루틴은 특정 스레드에 실행되고 다른 스레드로부터 재게될 수 있다.

다른 비동기 솔루션에는 RxJava가 존재한다. 하지만 코루틴을 선호하는 이유는 언어 자체에서 지원을 해주기 때문이다.

# 장점

- 경량: 코루틴은 실행 중인 스레드를 차단(block)하지 않는 정지(suspension)을 지원한다. 따라서 단일 스레드에서 많은 코루틴을 실행할 수 있다. 그리고 정지(suspension)은 많은 동시 작업을 지원하면서도 차단(block)보다 메모리를 절약한다.
- 메모리 누수 감소: 코루틴은 코루틴 스코프에서만 실행 될 수 있기 때문에 메모리 누수의 위험성을 덜어준다.
- 기본으로 제공되는 취소 지원: 실행 중인 코루틴 계층 구조를 통해 자동으로 취소가 전달된다.

# 코루틴 스코프

코루틴 스코프는 빌더에 의해 생성될 수 있다. 코루틴 스코프는 자식들이 모두 완료되기 전까지 완료되지 않는다.

기본적인 스코프는 `runBlocking`과 `coroutineScope`가 존재한다. 둘은 내부의 코루틴이 종료될 때까지 대기한다는 점이 동일하나 차이가 존재한다. `runBlocking`은 현재 스레드를 **차단(block)**하며, 반면에 `coroutineScope`는 코루틴을 **일시 정지(suspend)**하여 스레드가 다른 곳에서 사용할 수 있게 한다.

코드로 살펴보자. 아래는 `runBlocking`을 사용하였다. 실행 결과 Hello가 먼저 출력되는 것을 볼 수 있다. 스레드가 차단되었기 때문에 1초 대기 후 *"Hello World!"*가 출력되는 것이다. `runBlocking`은 값비싼 스레드를 차단한다는 것을 확인할 수 있다. 그래서 실제 상용 프로그램 개발을 할때는 많이 사용되지 않는다.

```kotlin
fun main() {
    runBlocking {
        launch {
            delay(1_000)
            println("Hello")
        }
    }
    println("World!")
}

// Hello
// World!
```

이번에는 `coroutineScope`를 살펴보자. 실행 결과 World가 먼저 출력된 것을 볼 수 있다.

```kotlin
fun main() = runBlocking {
    doWorld()
}

suspend fun doWorld() = coroutineScope {
    launch {
        delay(1000L)
        println("Hello!")
    }
    println("World")
}

// World
// Hello!
```

또한 동시에 처리가 가능하다. 아래의 코드를 보면 두개의 코루틴을 생성하는 모습을 볼 수 있다.

```kotlin
fun main() = runBlocking {
    doWorld()
    println("Done")
}

suspend fun doWorld() = coroutineScope {
    launch {
        delay(2000L)
        println("World 2")
    }
    launch {
        delay(1000L)
        println("World 1")
    }
    println("Hello")
}

/*
Hello
World 1
World 2
Done
*/
```

`launch`를 통해 코루틴을 생성하면 `Job` 객체가 반환되는데, 이를 통해 `join`, `cancel`등의 작업을 할 수 있다.

```kotlin
fun main() = runBlocking {
    val job = launch { // launch a new coroutine and keep a reference to its Job
        delay(1000L)
        println("World!")
    }
    println("Hello")
    job.join() // wait until child coroutine completes
    println("Done")
}

/*
Hello
World!
Done
*/
```

안드로이드에서는 흔히 사용되는 코루틴 스코프가 존재한다. `viewModelScope`, `lifecycleScope`와 같은 [수명 주기 인식 구성요소와 함께 사용되는 코루틴 스코프](https://developer.android.com/topic/libraries/architecture/coroutines?hl=ko)가 존재한다. 때문에 생명주기가 존재하는 엑티비티, 프레그먼트, 뷰모델 등에서 안전하게 코루틴을 사용할 수 있다.

# async, await

만약 실행에 필요한 비동기 요청들이 의존성이 없어 동시에 처리하려면 어떻게 해야할까? `launc`를 여러번 수행해도 되지만 `async`를 이용하는 방법도 존재한다. `async`는 `launch`와 유사하게 독립된 코루틴을 실행한다.
다른 부분은 `launch`는 `Job`을 반환하는데, `async`는 결과를 나중에 제공해주는 `Deferred`를 반환한다.

아래의 코드를 보면 1초가 걸리는 두 개의 함수를 동시에 실행하여 대기한 후 결과를 반환한다. 만약 `async` 없이 호출했다면 첫 번째 함수를 호출하고 1초를 대기하고 다시 두 번째 함수를 호출하고 1초를 대기해 총 2초가 걸릴 것이다.

```kotlin
fun main() = runBlocking<Unit> {
    val time = measureTimeMillis {
        val one = async { doSomethingUsefulOne() }
        val two = async { doSomethingUsefulTwo() }
        println("The answer is ${one.await() + two.await()}")
    }
    println("Completed in $time ms")
}

suspend fun doSomethingUsefulOne(): Int {
    delay(1000L) // pretend we are doing something useful here
    return 13
}

suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L) // pretend we are doing something useful here, too
    return 29
}
```

이를 이용하여 아래와 같이 async-style 함수를 만들 수 있으나 이처럼 **사용하지 않는** 것을 강력히 권장한다.
그 이유는 예외 발생시 메모리 누수 문제가 있기 때문이다. 예를 들어 첫 번째 함수를 실행하던 도중 예외가 던저졌다고 가정하자. 그렇게 첫 번째 코루틴 스코프는 취소된다. 하지만 두 번재 코루틴은 그대로 백그라운드에서 계속 진행한다. 결과가 의미 없음에도 말이다. 따라서 **절대로 아래와 같이 사용하지 말자**.

```kotlin
@OptIn(DelicateCoroutinesApi::class)
fun somethingUsefulOneAsync() = GlobalScope.async {
    doSomethingUsefulOne()
}

// The result type of somethingUsefulTwoAsync is Deferred<Int>
@OptIn(DelicateCoroutinesApi::class)
fun somethingUsefulTwoAsync() = GlobalScope.async {
    doSomethingUsefulTwo()
}

fun main() {
  // ...
  val one = somethingUsefulOneAsync()
  val two = somethingUsefulTwoAsync()
  // but waiting for a result must involve either suspending or blocking.
  // here we use `runBlocking { ... }` to block the main thread while waiting for the result
  runBlocking {
      println("The answer is ${one.await() + two.await()}")
  }
}
```

반대로 스코프를 아래와 같이 적절히 이용한다면 하나의 예외가 발생하였을 때 예외가 전파되어 스코프 내의 모든 코루틴이 취소된다. 그렇기 때문에 아래의 방법을 이용하자.

```kotlin
suspend fun concurrentSum(): Int = coroutineScope {
    val one = async { doSomethingUsefulOne() }
    val two = async { doSomethingUsefulTwo() }
    one.await() + two.await()
}
```

한 곳에서 예외가 발생했을 때 실제로 어떤 일이 일어나는 지는 아래의 코드를 통해서 확인할 수 있다.

```kotlin
fun main() = runBlocking<Unit> {
    try {
        failedConcurrentSum()
    } catch(e: ArithmeticException) {
        println("Computation failed with ArithmeticException")
    }
}

suspend fun failedConcurrentSum(): Int = coroutineScope {
    val one = async<Int> {
        try {
            delay(Long.MAX_VALUE) // Emulates very long computation
            42
        } finally {
            println("First child was cancelled")
        }
    }
    val two = async<Int> {
        println("Second child throws an exception")
        throw ArithmeticException()
    }
    one.await() + two.await()
}

/*
Second child throws an exception
First child was cancelled
Computation failed with ArithmeticException
*/
```

# [Flow](https://jja08111.github.io/kotlin/kotlin-flow/)

# Channel

`Channel`은 코루틴간의 통신을 위해 사용될 수 있다. 이는 `BlockingQueue`와 매우 유사하다.

아래의 코드는 코루틴에서 5개의 데이터를 전송하고 다른 쪽에서 5개를 수신하고 있다.

```kotlin
fun main() = runBlocking {
    val channel = Channel<Int>()
    launch {
        // this might be heavy CPU-consuming computation or async logic, we'll just send five squares
        for (x in 1..5) channel.send(x * x)
    }
    // here we print five received integers:
    repeat(5) { println(channel.receive()) }
    println("Done!")
}

/*
1
4
9
16
25
Done!
*/
```

queue와 다르게 channel은 닫혀질 수 있다. 반복문에서 iteratorable처럼 사용될 수 있다.

```kotlin
val channel = Channel<Int>()
launch {
    for (x in 1..5) channel.send(x * x)
    channel.close() // we're done sending
}
// here we print received values using `for` loop (until the channel is closed)
for (y in channel) println(y)
println("Done!")

/*
1
4
9
16
25
Done!
*/
```

# 내부 동작

Kotlin compiler가 coroutine 코드를 어떻게 변환하는지 살펴보자.

코루틴은 suspension 함수당 하나의 상태 머신을 만들어 동작한다. 아래의 코드는 두 개의 suspension 지점이 있어 초기 상태를 포함한 총 세 개의 상태를 가진다.

```kotlin
val a = a()
val y = foo(a).await() // suspension point #1
b()
val z = bar(a, y).await() // suspension point #2
c(z)
```

위의 코드는 아래의 pseudo-Java 코드와 유사하게 컴파일된다. 상태 머신을 구현한 익명 객체가 생성된다. 필드는 현재 머신의 상태를 가지며, 각 상태들에 공유된다. 상태를 `label`이라는 필드로 나타내고 있으며 반복적으로 다음 상태로 바뀌는 것을 확인할 수 있다. 중간에 중단되고 다시 `resumeWith`가 호출되면 재개된다. 마지막 상태인 `2`에 도달하면 마지막 `c(z)`를 호출하고 상태를 `-1`로 바꾸어 더이상 반복하지 않는다.

```kotlin
class <anonymous_for_state_machine> extends SuspendLambda<...> {
    // The current state of the state machine
    int label = 0

    // local variables of the coroutine
    A a = null
    Y y = null

    void resumeWith(Object result) {
        if (label == 0) goto L0
        if (label == 1) goto L1
        if (label == 2) goto L2
        else throw IllegalStateException()

      L0:
        // result is expected to be `null` at this invocation
        a = a()
        label = 1
        result = foo(a).await(this) // 'this' is passed as a continuation
        if (result == COROUTINE_SUSPENDED) return // return if await had suspended execution
      L1:
        // external code has resumed this coroutine passing the result of .await()
        y = (Y) result
        b()
        label = 2
        result = bar(a, y).await(this) // 'this' is passed as a continuation
        if (result == COROUTINE_SUSPENDED) return // return if await had suspended execution
      L2:
        // external code has resumed this coroutine passing the result of .await()
        Z z = (Z) result
        c(z)
        label = -1 // No more steps are allowed
        return
    }
}
```

# 참조

- [Kotlin/KEEP/proposals/coroutines.md](https://github.com/Kotlin/KEEP/blob/master/proposals/coroutines.md)
- [Android의 Kotlin 코루틴](https://developer.android.com/kotlin/coroutines?hl=ko)
