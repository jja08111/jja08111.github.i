---
 title: "[Effective Kotlin] 5. 예외를 활용해 코드에 제한을 걸어라"
 date: 2023-02-19 19:12:00 +0900
 categories: kotlin
 tags:
   - kotlin
   - effective_kotlin
---

어느 시점에 특정한 형태로만 동작해야하는 코드가 있다면 예외를 활용해 제한을 거는 것이 좋다.
제한을 걸면 좋은 이유는 아래와 같다.

- 문서를 읽지 않은 개발자도 문제를 확인할 수 있다.
- 문제가 있을경우 예상하지 못한 동작을 하지 않고 예외를 던진다. 예상치 못한 동작을 하는 것은 예외를 던지는 것보다 굉장히 위험하다.
- 코드가 어느 정도 자체적으로 검사된다.
- 스마트 캐스트를 이용할 수 있어 타입 캐스트를 적게 할 수 있다.

나는 과거에 "프로그램이 던져진 예외로 인해 죽으면 사용자에게 치명적이지 않나"라고 생각했던 적이 있다.
하지만 여러 프로젝트를 개발하다보니 잘못된 상태로 프로그램이 살아있는 것보다 예외로 죽는 것이 낫다는 것을 알게되었다.

코틀린에는 아래와 같이 쉽게 동작을 제한할 수 있는 함수를 제공한다.

- `require`: 아규먼트를 제한
- `check`: 상태와 관련된 동작 제한
- `assert`: 테스트모드에서 상태 제한
- return 또는 throw와 함께 사용하는 Elvis 연산자

# 아규먼트

함수를 정의할 때 아규먼트에 제약이 필요한 경우는 흔하다. 예를 들어 아래와 같은 상황이 있다.

- 팩토리얼을 계산하는 함수에서 양의 정수를 아규먼트로 받아야하는 경우
- 이메일 형식에 맞는 문자열 아규먼트인지 확인이 필요한 경우

이러한 경우에 적절한 함수는 `require`이다. 이 함수는 제한을 만족하지 못할 때 `IllegalArgumentException`을 던진다.

```kotlin
fun factorial(n: Int): Long {
  require(n >= 0)
  // ...
}

fun sendEmail(user: User, message: String) {
  requireNotNull(user.email)
  require(isValidEmail(user.email))
  // ...
}
```

또한 아래와 같이 람다를 활용하여 lazy message를 정의할 수 있다. 이는 디버깅에 도움을 준다.

```kotlin
fun factorial(n: Int): Long {
  require(n >= 0) {
    "Cannot calculate factorial of $n because it is smaller then 0"
  }
  // ...
}
```

# 상태

객체 내부 혹은 외부와 관련하여 특정 상태일때만 해당 함수를 사용할 수 있게 해야하는 경우가 존재한다. 예를 들어 아래와 같은 경우가 있다.

- 어떤 객체가 초기화 되어야만 처리를 하게 하고 싶은 함수
- 사용자가 로그인 되어야만 처리를 하게 하고 싶은 함수
- 객체를 사용할 수 있는 시점에 사용하게 하고 싶은 함수

`check`는 `require`과 유사하지만 `IllegalStateException`을 던지는 차이가 있다. 이 함수는 마찬가지로 lazy message를 정의할 수 있다.
이 함수는 사용자가 규약을 어기고 사용하면 안되는 곳에서 함수를 호출하는 지점에 사용한다. 사용자를 100% 신뢰하기보다 문제가 생기는 부분에서 미리 예외를 던지는 것이다.

# 테스트

단위 테스트를 위해서는 보통 assert 계열 함수를 사용한다. 단위 테스트는 버그가 있음을 확인해주고 추후 리팩토링으로 인해 발생하는 문제를 예방할 수 있다.

```kotlin
class StackTest {

  @Test
  fun `Stack pops correct number of elements`() {
    val stack = Stack(20) { it }
    val ret = stack.pop(10)
    assertEquals(10, ret.size)
  }
}
```

단위 테스트는 구현의 정확성을 확인하는 가장 기본적인 방법이다. 그런데 위의 테스트는 보편적인 하나의 경우만 테스트하고 있다.
이 경우 함수 구현체 내부에 아래와 같이 `assert`를 넣는 것도 방법이다.

```kotlin
fun pop(num: Int = 1): List<T> {
  // ...
  assert(ret.size == num)
  return ret
}
```

이러한 조건은 -ea JVM 옵션을 활성화해야 확인할 수 있다.

# nullability와 스마트 캐스팅

코틀린에서 `require`와 `check` 블록을 사용한 후의 코드가 참이라면, 해당 조건은 이후로도 참이라고 가정한다.
따라서 이를 활용하여 타입을 비교하거나 null 여부를 확인했다면 스마트 캐스트가 진행된다.

```kotlin
fun sendEmail(user: User, message: String) {
  requireNotNull(user.email)
  val email: String = user.email
  // ...
}
```

혹은 Elvis 연산자를 활용하여 함수를 조기종료 혹은 예외를 던질 수 있다.

```kotlin
fun sendEmail(user: User, message: String) {
  val email: String = user.email ?: return
  // ...
}
```

이때 `run`을 조합하면 여러 처리를 함께할 수 있다. 예를 들어 함수가 중지된 이유를 로그에 남기는 작업을 하는 것이다.

```kotlin
fun sendEmail(user: User, message: String) {
  val email: String = user.email ?: run {
    log("Email not sent, no email address")
    return
  }
  // ...
}
```
