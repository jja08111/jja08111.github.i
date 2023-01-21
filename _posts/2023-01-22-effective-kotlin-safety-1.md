---
 title: "[Effective Kotlin] 1. 가변성을 제한하라"
 date: 2023-01-21 22:12:00 +0900
 categories: kotlin
 tags:
   - kotlin
   - effective_kotlin
---

상태를 관리하기 어려운 이유는 다음과 같다.

- 상태들의 관계를 이해해야 하고 상태 변경이 많아지면 추적이 어렵다.
- 가변성(mutability)가 있으면 코드의 실행을 추론하기 어렵다.
- 멀티스레드 환경에서 동기화가 필요하다.
- 테스트하기 어렵다. 모든 상태를 테스트하기 위해 수많은 조합을 테스트해야 한다.
- 상태 변경이 일어날 때 변경 사항을 다른 요소에 알려야 하는 경우가 있다.

# 코트린에서 가변성 제한하기

## 읽기 전용 프로퍼티(val)

`val` 선언된 프로퍼티는 마치 값(value)처럼 동작하며, 일반적인 방법으로는 값이 변하지 않는다.

하지만 읽기 전용 프로퍼티가 **완전히 변경 불가능한 것은 아니라는 것을 주의**해야한다.
예를 들어 mutable 객체를 사용하는 경우 내부적으로 값이 변할 수 있다.
이는 _읽기 전용_(재할당 불가)과 _가변성_(값이 변할 수 없는 것)의 차이를 알아야 한다.

```kotlin
val list = mutableListOf(1, 2, 3)
list.add(4);

print(list) // [1, 2, 3, 4]
```

val은 읽기 전용 프로퍼티지만, 변경할 수 없음(불변, immutable)을 의미하는 것이 아니다.

val은 정의 옆에 상태가 바로 적히므로 코드의 실행을 예측하기 쉽다. 또한 val은 스마트 캐스트 등의 추가적인 기능을 활용할 수 있다.

## 가변 컬랙션과 읽기 전용 컬렉션 구분하기

코틀린에는 mutable 컬렉션과 immutable 컬렉션이 나뉘어 존재한다. 읽기 전용 컬렉션이 내부의 값을 변경할 수 없다는 것을 의미하지는 않는다.
대부분의 경우 변경이 가능하다. 하지만 읽기 전용 인터페이스가 이를 지원하지 않아 변경할 수 없다.

이처럼 컬렉션을 완전한 불변(immutable)하게 만들지 않고, 읽기 전용으로 설계한 것은 굉장히 중요한 부분이다.
그런데 개발자가 다운캐스팅을 통해 읽기 전용 객체를 쓰기 가능한 실제 인스턴스의 객체로 접근하는 것은 매우 위험하다.
아래와 같이 함수를 통해 리스트를 얻었을 때 다운 캐스팅을 하여 쓰기 작업을 한다면 계약 위반이며 추상화를 무시하는 행위이다.

```kotlin
val list: List = fetchSomeList()

// DON'T DO IT!
if (list is MutableList) {
  list.add(4)
}
```

위와 같은 방법 대신 복제를 통해서 새로운 mutable 컬렉션을 만드는 API를 활요하는 것이 안전하다.

```kotlin
val list: List = fetchSomeList()

val mutableList = list.toMutableList()
mutableList.add(4)
```

## 데이터 클래스의 copy

우선 immutable 객체의 장점은 아래와 같다.

- 한 번 정의된 상태가 유지되므로, 코드를 이해하기 쉽다.
- 병렬 처리를 안전하게 할 수 있다.
- 방어적 복사본을 만들 필요가 없다.
- set 혹은 map의 키로 사용하기 아주 적절하다.

이러한 장점이 있으나 immutable 객체는 변경할 수 없다는 단점이 있다.
코틀린은 `data` 한정자를 사용하면 `copy` 메소드를 만들어 주기 때문에 이를 잘 활용하면 단점을 해소할 수 있다.

# 다른 종류의 변경 가능 지점

변경 가능한 리스트를 만드는 방법은 아래와 같이 두 가지가 존재한다.

```kotlin
val list1: MutableList<Int> = mutableListOf()
val list2: List<Int> = listOf()
```

위의 두 가지 모두 `+=` 연산자를 사용할 수 있다. 하지만 내부 동작은 다르다.

```kotlin
list1 += 1 // list1.plusAssign(1)
list2 += 1 // list2 = list2.plus(1)
```

첫 번째 방법은 내부적으로 동기화 처리가 되어있는 지 알 수 없다. 두 번째는 직접 동기화 처리를 할 수 있다.
따라서 두 번째가 비교적 안정적이다. 그렇기 때문에 mutable 컬렉션 대신 **mutable 프로퍼티**를 이용하자.

# 변경 가능 지점 노출하지 말기

아래와 같이 mutable 객체를 외부에 노출하는 것은 위험하다.

```kotlin
data class User(val name: String)

class UserRepository {
  private val users: MutableMap<Int, String> = mutableMapOf()

  fun loadAll() = users
}
```

이를 처리하는 방법은 두 가지가 존재한다. 하나의 방법은 *방어적 복제*를 이용하는 것이다.

```kotlin
class UserRepository {
  private val users = MutableUser()

  fun loadAll() = users.copy()
}
```

아니면 가변성을 제한하는 방법이다.

```kotlin
data class User(val name: String)

class UserRepository {
  private val users: MutableMap<Int, String> = mutableMapOf()

  fun loadAll(): Map<Int, String> = users
}
```

# 정리

개인적으로 컬렉션에 상태를 저장할 때 mutable 컬렉션 대신 mutable 프로퍼티를 사용하라는 부분이 인상적이었다.
하지만 가끔 효율성 때문에 mutable 객체를 사용하는 것이 좋다고 한다.
