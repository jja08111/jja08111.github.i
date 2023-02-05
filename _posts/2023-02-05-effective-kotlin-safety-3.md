---
 title: "[Effective Kotlin] 3. 최대한 플랫폼 타입을 사용하지 말라"
 date: 2023-02-05 12:12:00 +0900
 categories: kotlin
 tags:
   - kotlin
   - effective_kotlin
---

들어가기에 앞서 플랫폼 타입이란 무엇일까? 플랫폼 타입이란 null-safety가 없는 자바, C 등의 프로그래밍 언어를 코틀린을 연결해서 사용할 때 등장하는 타입이다.

예를 들어 아래와 같은 자바 코드를 코틀린에서 이용하는 경우를 생각해보자. 자바에는 `@Nullable`, `@NotNull`과 같은 어노테이션이 있으나 사용하지 않고 있는 모습이다.

```java
public class JavaTest {

  public string giveName() {
    // ...
  }
}
```

자바에서는 모든 것이 nullable일 수 있으므로 안전하게 접근하고자 한다면 nullable로 가정하고 다뤄야 한다. 하지만 이는 여간 번거로운 일이 아닐 수 없다.
만약 `List<List<User>>`를 반환한다고 하면 아래와 같은 복잡한 처리가 필요할 것이다.

```kotlin
val users: List<List<User>> = UserRepo().groupedUsers!!.map {
  it!!.filterNotNull()
}
```

때문에 가능하면 자바코드에서 `@Nullable`, `@NotNull`를 사용하는 것이 좋다.

만약 자바 코드를 직접 수정할 수 없다면 가능한 빠르게 플랫폼 타입을 제거하는 것이 좋다. 아래의 두 가지 경우를 살펴보면 둘 모두 NPE가 발생한다.

```kotlin
fun startedType() {
  val value: String = JavaClass().value
  println(value.length)
}

fun platformType() {
  val value = JavaClass().value
  println(value.length)
}
```

하지만 `startedType`은 값을 가져오는 시점에서 NPE가 발생하고 `platformType`은 값을 사용할 때 NPE가 발생한다.
이는 추후에 문제가 발생했을 때 전자는 버그를 추적하기 쉬우나 후자의 경우는 추적하기 어렵다. 이처럼 플랫폼 타입이 전파되는 일은 굉장히 위험하다.

그러니 가능한 빠른 시점에 플랫폼 타입을 제거하자.
