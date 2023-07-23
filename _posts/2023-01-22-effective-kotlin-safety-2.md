---
 title: "[Effective Kotlin] 2. 변수의 스코프를 최소화하라"
 date: 2023-01-22 22:12:00 +0900
 categories: kotlin
 tags:
   - kotlin
   - effective_kotlin
---

왜 변수의 스코프를 최소화해야할까? 그 이유는 프로그램을 추적하고 관리하기 쉽기 때문이다.
코드를 분석할 때는 어떤 시점에 어느 요소가 있는지 파악해야한다. 이때 변수가 많아지면 프로그램을 이해하기 어려워진다.
따라서 가능한 mutable 프로퍼티는 좁은 스코프로 설정하는 것이 추후에 추적하기 쉽다.

첫 번째로 반복문에서 반복문 내부에서만 사용하는 변수는 내부에서 선언을 하는 것이다.

```kotlin
// Bad
var user: User
for (i in users.indices) {
  user = users[i]
  print("User at $i is $user")
}

// Good
for (i in users.indices) {
  val user = users[i]
  print("User at $i is $user")
}

// Better
for ((i, user) in users.withIndex()) {
  print("User at $i is $user")
}
```

Kotlin은 if-else, when, try-catch, Elvis를 표현식으로 사용할 수 있기 때문에 변수를 정의할 때 적극적으로 이용하면 좋다.

```kotlin
// Bad
val user: User
if (hasValue) {
  user = getValue()
} else {
  user = User()
}

// Good
val user: User = if (hasValue) {
  getValue()
} else {
  User()
}
```

여러 프로퍼티를 한꺼번에 설정해야 하는 경우에는 구조분해 선언이 유용하다.

```kotlin
// Bad
fun updateWeather(degrees: Int) {
  val description: String
  val color: Int
  if (degrees < 5) {
    description = "cold"
    color = Color.BLUE
  } else if (degrees < 23) {
    description = "mild"
    color = Color.YELLOW
  } else {
    description = "hot"
    color = Color.RED
  }
  // ...
}

// Good
fun updateWeather(degrees: Int) {
  val (description, color) = when {
    degrees < 5 -> "cold" to Color.BLUE
    degrees < 23 -> "mild" to Color.YELLOW
    else -> "hot" to Color.RED
  }
  // ...
}
```

# 캡쳐링

스코프 범위가 넓어 발생하는 위험을 살펴보기 위해 시퀀스 빌더를 이용하여 에라토스테네스의 체를 구현해보자.

그전에 시퀀스란 무엇일까? 컬렉션과 다르게 시퀀스는 요소를 직접 가지고 있지 않고, 반복하는 도중 요소를 생산한다.
[시퀀스 프로세싱 예제](https://kotlinlang.org/docs/sequences.html#sequence-processing-example)를 살펴보면 시퀀스가 작업을 누적해두고 한 번에 모두 처리하는 것을 확인할 수 있다.

그럼 이제 본론으로 돌아와 시퀀스 빌더로 소수를 구하는 예제를 살펴보자.

```kotlin
val primes: Sequence<Int> = sequence {
  var numberes = generateSequence(2) { it + 1 }

  while (true) {
    val prime = numbers.first()
    yield(prime)
    numbers = numbers.drop(1).filter { it % prime != 0 }
  }
}

print(primes.take(10).toList())
// [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]
```

그런데 항상 아래와 같이 prime을 반복문 외부에서 선언을 하여 최적화를 하려는 사람들이 있다.

```kotlin
val primes: Sequence<Int> = sequence {
  var numberes = generateSequence(2) { it + 1 }

  var prime: Int
  while (true) {
    prime = numbers.first()
    yield(prime)
    numbers = numbers.drop(1).filter { it % prime != 0 }
  }
}
```

위의 코드를 실행하면 아래와 같이 엉뚱한 결과가 나온다.

```kotlin
print(primes.take(10).toList())
// [2, 3, 5, 6, 7, 8, 9, 10, 11, 12]
```

이유는 prime이라는 변수를 filter에 전달한 람다에서 캡쳐했기 때문이다. 시퀀스는 앞서 말했듯이 필터링이 지연된다.
따라서 동일한 `prime` 참조 값으로만 필터링 되어 엉뚱한 결과가 나오는 것이다.

그런데 책을 보면 이상한 부분이 있다. 바로 4가 필터링 되었다는 부분이다. 이부분은 틀렸다.
[다른 분이 작성하신 글](https://two22.tistory.com/73)을 살펴보면 4는 필터링된 것이 아니라 drop된 것이다.
while문이 두 번째 반복되는 시점에는 아래와 같은 상황일 것이다. 2가 drop되고 3이 필터링되며 4가 drop되고 5가 필터를 통과하여 5가 반환된다.

```kotlin
numbers.drop(1).filter { it % 3 != 0 }
       .drop(1).filter { it % 3 != 0 }
       .first()
```

결국 중요한 부분은 잘못된 시퀀스 이해로 변수가 캡쳐되어 문제가 발생했다는 부분이다. 이러한 문제를 피하기 위해 가변성을 피하고 스코프 범위를 좁게 만들자.
