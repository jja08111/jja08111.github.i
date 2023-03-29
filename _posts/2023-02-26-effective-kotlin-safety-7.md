---
 title: "[Effective Kotlin] 7. 결과 부족이 발생할 경우 null과 Failure를 사용하라"
 date: 2023-03-19 10:12:00 +0900
 categories: kotlin
 tags:
   - kotlin
   - effective_kotlin
---

함수가 원하는 결과를 만들어 낼 수 없을 때가 있다. 예를 들어 서버로부터 데이터를 읽어 들이려고 했으나, 인터넷 연결 문제로 읽지 못한 경우이다.
이러한 상황을 처리하는 방법은 아래와 같이 크게 두 가지가 있다.

- `null` 혹은 `실패를 나타내는 sealed 클래스(일반적으로 Failure라는 이름을 붙임)`를 반환한다.
- 예외를 던진다.

위의 두 가지는 중요한 차이점이 있다. 예외는 정말 특별한 상황을 나타내야하며 정보를 전달하기 위해서 사용하여서는 안된다. 이유를 정리하면 다음과 같다.

- 많은 개발자가 예외가 전파되는 과정을 제대로 추적하지 못한다.
- 코틀린의 모든 예외는 unchecked 예외이다. 따라서 제대로 처리되지 않을 가능성이 높다.
- 예외는 예외적인 상황을 처리하기 위해서 만들어졌으므로 명시적인 테스트만큼 빠르게 동작하지 않는다.
- try-catch 블록 내부에 코드를 배치하면, 컴파일러가 할 수 있는 최적화가 제한된다.

반면, `null`과 `Failure`는 예상되는 오류를 컴파일 타임에 명시적으로 파악할 수 있다.
따라서 충분히 예측할 수 있는 범위의 오류는 `null`과 `Failure`을 사용하고 예측하기 어려운 범위의 오류는 예외를 던지는 것이 좋다.

```kotlin
inline fun <reified T> String.readObjectOrNull(): T? {
  // ...
  if (incorrectSign) {
    return null
  }
  // ...
  return result
}

inline fun <reified T> String.readObject(): Result<T> {
  // ...
  if (incorrectSign) {
    return Result.failure(JsonParsingException())
  }
  // ...
  return Result.success(result)
}
```

나는 개인적으로 Kotlin에 내장된 `Result` 클래스를 즐겨 사용한다. 이 클래스는 `runCatching`과 같은 유용한 API들을 많이 제공해주기 때문이다.

그렇다면 `null` 혹은 `Failure` 중 무엇을 사용할지 어떻게 결정할까? `null`은 예외 상황에 대해 추가적인 정보가 없을 때 사용하고 `Failure`는 추가 정보가 필요할 때 사용하면 된다.
예를 들어 서버에 잘못된 요청을 하여 서버에서 전달해준 메시지와 함께 예외를 반환해야하는 경우 `Failure`가 적절하다.
