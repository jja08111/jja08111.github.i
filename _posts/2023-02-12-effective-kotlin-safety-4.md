---
 title: "[Effective Kotlin] 4. inferred 타입으로 리턴하지 말라"
 date: 2023-02-12 12:12:00 +0900
 categories: kotlin
 tags:
   - kotlin
   - effective_kotlin
---

kotlin에는 타입 추론(type inferred)이 존재한다. 타입 추론을 통해 코드 작성량을 줄일 수 있으나 이는 위험한 부분이 존재한다.

다음과 같은 인터페이스가 존재한다고 생각해보자.

```kotlin
interface CarFactory {
  fun produce(): Car
}
```

그리고 디폴트로 생성되는 자동차가 있다고 가정해보자.

```kotlin
val DEFAULT_CAR: Car = Fiat126P()
```

프로젝트가 커지면서 어느순간 누군가 기본 차량을 반환할 때 반환값을 명시할 필요가 없다고 판단하여 아래와 같이 작성했다고 해보자.

```kotlin
interface CarFactory {
  fun produce() = DEFAULT_CAR
}
```

또 마찬가지로 `DEFAULT_CAR`가 명시적으로 `Car`로 반환하던 것을 타입 추론으로 대체하였다고 해보자.

```kotlin
val DEFAULT_CAR = Fiat126P()
```

이제 문제가 발생했다. 더이상 `CarFactory`는 `Fiat126P` 이외의 자동차는 생산하지 못한다.

# 정리

타입을 확실하게 지정해야하는 경우 **명시적으로 타입을 지정**하자. inferred 타입은 프로젝트가 진전되는 과정에서 예측할 수 없는 결과를 야기할 수 있다.
