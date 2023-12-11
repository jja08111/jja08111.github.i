---
 title: "Compose의 안정성 시스템"
 date: 2023-08-21 11:00:00 +0900
 categories: android
 tags:
   - android
   - compose
---

Jetpack Compose에는 안정성 시스템이 존재한다. 이는 리컴포지션을 생략 가능한지 판단할 때 사용된다.
리컴포지션이 발생하여 컴포저블 함수의 스냅샷 상태가 변경되었다면 해당 컴포저블은 리컴포지션이 필요하다.
만약 변경되지 않았다면 불필요하게 리컴포지션을 진행할 필요가 없다. 타입이 안정적이지 않다면 항상 리컴포지션을 진행한다.

# Stable

값의 변경을 어떤 기준으로 확인할까? 아래의 안정성(Stable) 기준이 필요하다.

1. 동일한 두 인스턴스로 equals를 수행하였을 때 **항상** 동일한 결과가 주어진다.
2. 모든 공개 프로퍼티는 변경될 때 Composition에 알려진다.
3. 모든 공개 프로퍼티는 Stable하다.

첫 번째 항목을 생각해보자. 두 인스턴스가 존재하고 equals를 수행할 때마다 다른 결과가 나온다고 가정하자.
이 경우 캐시된 값과 현재 값의 동등성을 보장할 수 없기 때문에 캐시된 값을 재사용할 수 없다.
때문에 이는 unstable 하다고 판정되어 리컴포지션을 생략할 수 없다.

두 번째 항목을 생각해보자.

```kotlin
class Person(
    var name: String,
    val address: String
)
```

위의 `Person` 클래스를 보면 모든 프로퍼티가 쓰기 가능하지만 Compose에 [관찰될 수 있는 값](https://developer.android.com/reference/kotlin/androidx/compose/runtime/MutableState)은 아니다.
때문에 아래와 같이 name이 변경 되어도 리컴포지션은 발생하지 않는다.

```kotlin
@Composable
fun PersonDetail(person: Person) {
    Text(
        modifier = Modifier.clickable { person.name = "new name" },
        text = person.name
    )
}
```

그리고 아래와 같이 `person`이 변경되지 않는 경우에도 `checked`가 변경될 때마다 항상 `PersonDetail` 컴포저블에서 리컴포지션이 발생한다.

```kotlin
@Composable
fun PersonDetailRow(person: Person, modifier: Modifier = Modifier) {
    var checked by remember { mutableStateOf(false) }

    Row(modifier) {
        PersonDetail(person)
        Switch(checked = checked, onCheckedChange = { checked = !checked })
    }
}
```

궁금한점이 생겼다. 메모이제이션된 `Person` 객체로 equals를 수행하면 항상 동일한 값이 나오기 때문에 재사용해도 문제가 없지 않나?
그렇다면 왜 이를 unstable로 취급할까? 이는 잘못된 어노테이션 사용 항목에서 설명된다.

세 번째 항목은 공개 프로퍼티 중 하나라도 unstable하다면 두 인스턴스의 동등성을 보장할 수 없기 때문에 필요하다.

# `@Stable`, `@Immutable`을 잘못 사용하면?

두 어노테이션은 주의해서 사용해야한다. 리컴포지션이 발생해야하는 시점에서 누락되는 문제가 발생할 수 있기 때문이다.

예를 들어 stable이 아닌 `Person` 객체에 `@Stable`을 사용한 아래의 경우를 살펴보자.

```kotlin
@Stable
data class Person(
    var name: String,
    val address: String
)

@Composable
fun PersonDetail(person: Person) {
    Text(text = person.name)
}

@Composable
fun PersonDetailRow(person: Person, modifier: Modifier = Modifier) {
    var checked by remember { mutableStateOf(false) }

    Row(modifier) {
        PersonDetail(person)
        Switch(
            checked = checked,
            onCheckedChange = {
                checked = !checked
                person.name = "wow"
            }
        )
    }
}
```

스위치를 누르면 `checked` 값이 갱신되고 `person`의 `name`은 `wow`가 된다.
이때 State 값인 `checked`가 갱신되었기 때문에 리컴포지션이 발생된다.

하지만 `Person`의 `name`이 변경되었음에도 UI에는 변경된 `name`이 표시되지 않는다.
그 이유는 `Person` 객체가 `@Stable`로 마킹되었으나, equals를 동일한 인스턴스로 수행했기 때문이다. 실제로 equals를 오버라이딩하여 출력하면 동일한 `name`을 갖고 있다.(의문이 드는 부분은 equals를 항상 false만 반환하도록 오버라이딩 하여도 바뀐 `name`이 UI에 보이지 않는다... 왜일까? equals 말고도 다른 연산이 수행되나? hashCode로 비교를 수행하나?)

이는 아래와 같이 `mutableStateOf`를 사용하여 `Person`을 안정적으로 만들어 해결할 수 있다.
`name` 자체를 관찰 가능한 값으로 만들어 Text에서 변경을 알아 낼 수 있도록 하는 것이다. 하지만 이보다 immutable한 `data class`를 사용하는 것이 바람직하다.

_(2023/12/11 내용 수정됨)_

```kotlin
@Stable
class Person(
    name: String,
    val address: String,
) {
    var name: String by mutableStateOf(name)
}
```

# 람다 함수

람다 함수는 다음으로 나눠진다.

- 값을 캡쳐하지 않는 람다 함수
- 안정적인 값을 캡쳐하는 람다 함수
- 안정적이지 않은 값을 캡쳐하는 람다 함수

값을 캡쳐하지 않는 람다 함수의 경우는 아래와 같다.

```kotlin
val foo = { println("bye, world!") }
```

이는 컴파일 후 아래와 같이 싱글톤으로 만들어진다. 아래의 싱글톤 클래스는 안정성 조건을 모두 만족하므로 stable하다.

```kotlin
class Foo : Function0<Unit> {
    override operator fun invoke() {
        println("bye, world!")
    }
}

val fooInstance = Foo()
```

안정적인 값을 캡쳐하는 람다 함수의 경우는 아래와 같다.

```kotlin
val greeting = "Hello"
val bar = { println("$greeting world!") }
```

이는 컴파일 후 아래와 같은 클래스로 변형된다. 안정성 조건을 모두 만족하므로 stable하다.
싱글톤이 아닌 경우 컴포즈 컴파일러에 의해 `remember`를 이용하여 항상 인스턴스를 생성하지 않도록 최적화된다.

```kotlin
class Bar(private val greeting: String) : Function0<Unit> {
    override operator fun invoke() {
        println("$greeting world!")
    }
}
```

아래처럼 안정적이지 않은 값을 캡쳐하는 람다 함수의 경우는 안정성 3번 조건을 만족하지 않기 때문에 unstable하다.

```kotlin
// 컴파일 전
val list = listOf("hi")
val bar = { list.forEach { println(it) } }

// 컴파일 후
class Bar(private val list: List<String>) : Function0<Unit> {
    override operator fun invoke() {
        // ...
    }
}
```

그런데 아래와 같이 안정적이지 않은 클래스의 함수 레퍼런스를 사용하는 경우는 함수 레퍼런스가 안정적으로 취급되기 때문에 리컴포지션을 막을 수 있다.
정확하지는 않지만 함수 레퍼런스는 리플랙션을 이용하여 고정된 함수 위치를 반환하기 때문이 아닐까 추측한다.

```kotlin
@Composable
fun Content(unstableClass: UnstableClass) {
    LambdaComposable(unstableClass::doSomthing)
}
```

# 성능 문제가 발생했을 때 디버깅하는 법

먼저 어디서 리컴포지션이 빈번하게 발생하는지 확인해야 한다. 이는 [Layout Inspector](https://developer.android.com/jetpack/compose/tooling/layout-inspector)를 이용하면 된다.

```kotlin
@Immutable // WARNING to use!
data class Person(
    val name: String,
    val address: String,
    val someList: List<Int>
)

@Composable
fun PersonDetail(person: Person) {
    Text(text = person.name)
}

@Composable
fun PersonDetailRow(person: Person, modifier: Modifier = Modifier) {
    var checked by remember { mutableStateOf(false) }

    Row(modifier) {
        PersonDetail(person)
        Switch(
            checked = checked,
            onCheckedChange = { checked = !checked }
        )
    }
}
```

위의 예제 코드를 보면 스위치를 토글하였을 때 `PersonDetail`는 리컴포지션이 발생하지 않는다. 하지만 `@Immutable`을 제거하면 리컴포지션이 발생한다.
그 이유는 `MutableList`가 `List`를 구현해서 `List`는 unstable하기 때문이다.

> For example, you could write `val set: Set<String> = mutableSetOf("foo")`. The variable is constant and its declared type is not mutable, but its implementation is still mutable. The Compose compiler cannot be sure of the immutability of this class as it only sees the declared type. It therefore marks tags as unstable.

**위 예제는 "`@Stable`, `@Immutable`을 잘못 사용하면?"처럼 문제가 있기 때문에 사용에 주의할 필요가 있다.**

때문에 위 방법보다는 아래처럼 wrapper 방식 혹은 [ImmutableCollection](https://github.com/Kotlin/kotlinx.collections.immutable)을 사용하는 것을 권장한다.

```kotlin
@Immutable
data class SnackCollection(
   val snacks: List<Snack>
)
```

# 참조

- [https://developer.android.com/jetpack/compose/performance/stability](https://developer.android.com/jetpack/compose/performance/stability)
- [https://developer.android.com/jetpack/compose/performance/stability/diagnose](https://developer.android.com/jetpack/compose/performance/stability/diagnose)
- [https://developer.android.com/jetpack/compose/performance/stability/fix](https://developer.android.com/jetpack/compose/performance/stability/fix)
- [https://sungbin.land/jetpack-compose-람다-최적화에-대한-고찰-b8854e38067a](https://sungbin.land/jetpack-compose-람다-최적화에-대한-고찰-b8854e38067a)
