---
 title: "[Effective Kotlin] 6. 사용자 정의 오류보다는 표준 오류를 사용하라"
 date: 2023-02-26 23:12:00 +0900
 categories: kotlin
 tags:
   - kotlin
   - effective_kotlin
---

`require`, `check`, `assert` 함수를 사용하면 대부분의 오류를 처리할 수 있다. 하지만 때로는 예측하지 못한 상황을 나타내야 하는 경우가 존재한다.
예를 들어 JSON 형식을 파싱하는 함수를 구현하는 경우이다. 만약 JSON 입력이 정상적이지 않다면 `JSONParsingException`와 같은 사용자 정의을 던지는게 바람직 할 것이다.

```kotlin
inline fun <reified T> String.readObject(): T {
  // ...
  if (incorrectSign) {
    throw JSONParsingException()
  }
  // ...
  return result
}
```

하지만 가능하면 사용자 정의 오류보다는 표준 라이브러리의 오류를 사용하는 것이 더 좋다. 표준 라이브러리는 많은 개발자가 알고 있고 이미 널리 알려진 규약이기 때문에 쉽게 이해할 수 있기 때문이다.
일반적으로 사용되는 예외를 몇 가지 정리하면 다음과 같다.

- `IllegalArgumentException`, `IllegalStateException`: `require`, `check` 함수를 사용해 던질 수 있는 예외들이다.
- `IndexOutOfBOundsException`: 인덱스 파라미터의 값이 범위를 벗어난 것을 나타낸다. 일반적으로 컬렉션 또는 배열에서 사용된다.
- `ConcurrentModificationException`: 동시 수정을 금지했는데 발생했을 때 던진다. 예를 들어 컬렉션 순회 중 항목을 삭제하는 등의 사례가 있다.
- `UnsupportedOperationException`: 사용자가 사용하려했던 메소드가 현재 객체에서는 사용할 수 없을 때 이용한다. 참고로 이렇게 구현한다면 ISP 원칙에 어긋난다.
  하지만 listOf로 만들어진 컬렉션 같이 immutable임을 강제하기 위해서는 사용될 수 있다.
- `NoSuchElementException`: 사용자가 사용하려 했던 요소가 존재하지 않을 때를 나타낸다.
