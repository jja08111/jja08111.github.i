---
title: "[Android] MVC, MVP, MVVM, MVI"
date: 2022-12-27 01:12:00 +0900
categories: android
tags:
  - android
  - architecture
---

안드로이드의 주요 디자인 패턴에는 MVC, MVP, MVVM, MVI 등이 존재한다. 각각의 특징 및 장단점을 비교해보겠다.

# MVC

MVC는 Model, View, Control로 구성된다.

- Model: 데이터를 가진다.
- View: 사용자 인터페이스를 담당한다.
- Control: 사용자에게 입력을 받아 이를 Model에 의해 View를 정의한다.

의존성은 아래와 같다. View와 Controller가 강하게 결합되어 Controller를 테스트하기란 쉽지 않다.

![mvc](https://user-images.githubusercontent.com/57604817/209694614-e3c06751-c1cc-4513-89c6-e169513eceb4.png)

흐름은 다음과 같다.

1. Control측에 사용자 이벤트가 발생한다.
2. 데이터 업데이트가 필요한지 Control이 Model에게서 확인한다. 있다면 Model로부터 데이터를 업데이트한다.
3. View는 Model 혹은 Control으로부터 갱신 필요 여부 이벤트를 받는다.
4. Model에서 데이터를 받아와 View를 갱신한다.

그런데 Android에서는 View와 Control이 Activity나 Fragment와 같은 View들이 모두 가지고 있다.
예를 들면 아래와 같은 코드이다.

```kotlin
class MainActivity : AppCompactActivity() {

  override fun onCreate(savedInstanceState: Bundle?) {

    ...

    val fab = findViewById<FloatingActionButton>()
    fab.setOnClickListener {
      // 데이터 갱신 요청
      // Model에 접근하여 최신 데이터 요청(ex: getItems())
      // 전달받은 값을 이용하여 View 갱신
    }
  }
}
```

즉, 위와 같이 엑티비티 안에 대부분의 코드가 작성되며 일부분의 코드가 Model에 존재한다.

## 장점

- 하나의 클래스에서 메소드들만 적절히 분리하여 작성하기 때문에 개발 기간이 빠를 수 있다.
- 안드로이드에 낯선 사람이 접해도 쉽게 파악할 수 있다.

## 단점

- 하나의 클래스에서 많은 것을 처리하기 때문에 요구사항이 많은 경우 클래스의 크기가 비대해진다. 이는 가독성이 떨어지고 유지보수에 좋지 않다.
- 코드 재활용성이 떨어지기 때문에 동일한 로직을 가진 코드가 여기저기 흩어지게 되고 이 또한 유지보수하기 나쁘다.
- View와 Model의 결합도가 높다. 앞선 코드를 보면 View에서 Model을 직접 호출하는 것을 볼 수 있다. View가 Model에 의존성을 가지게 되며 테스트하기 어렵게 된다.

<br></br>

# MVP

Controller가 너무 많은 일을 하던 MVC의 단점을 해결한 방법이 MVP 패턴이다. MVP는 Model, View, Presenter로 구성된다.
Controller 대신 Presenter가 존재하는 것을 알 수 있다.

Presenter는 View와 1대1로 존재한다. View에서 발생하는 액션은 모두 Presenter에게 전달되며 Presenter에서 값을 변경하면 View에서 변경 된 값을 보인다.
이는 View 인터페이스를 Presenter에 넘기는 방식으로 구현할 수 있다. 이 인터페이스가 핵심이다. 인터페이스가 없었다면 아래와 같은 서로 의존하는 관계가 되었을 것이다.

![vp](https://user-images.githubusercontent.com/57604817/209694634-e9786dcd-2a24-4dd6-abc3-ce8fa03cd3f9.png)

인터페이스를 이용한다면 아래와 같이 의존성 관계를 개선할 수 있다. Presenter가 더이상 View를 의존하지 않는다. 이는 Presenter를 테스트하기 쉽게 만들어준다.

![vp-with-interface](https://user-images.githubusercontent.com/57604817/209695114-5df38213-7353-4bee-8136-eda6dc25be53.png)

안드로이드 코드로 예를 들어보겠다. 스낵바를 보이는 함수를 가진 인터페이스가 존재한다. MainActivity는 이 인터페이스를 구현하며 presenter를 가지고 있어 생성자에 `this`로 전달한다.
추후 클릭 이벤트가 발생하면 presenter에 알리고 presenter에서는 viewInterface를 통해 view에게 다시 알린다.

```kotlin
interface ViewInterface {
  fun showSnackBar(@StringRes messageRes: Int);
}

class MainActivity : AppCompactActivity(), ViewInterface {

  private val presenter = Presenter(this)

  override fun onCreate(savedInstanceState: Bundle?) { ... }

  private fun onClickSomething() {
    presenter.onClickSomething()
  }

  override fun showSnackBar(@StringRes messageRes: Int) {
    // show snackBar
  }
}

class Presenter(val viewInterface: ViewInterface) {

  val model = Model()

  fun onClickSomething() {

    ...

    doSomething()
  }

  fun doSomething() {

    ...

    viewInterface.showSnackBar(R.string.some_string)
  }
}
```

## 장점

- Presenter가 View에 대한 의존성이 없기 때문에 Presenter를 테스트하기 좋다.

## 단점

- View와 Presenter가 1:1 의존 관계이기 때문에 유사한 로직을 가진 뷰들이 있을 때 계속해서 Presenter를 만들어야하는 단점이 존재한다.

<br></br>

# MVVM

MVP에서 1:1로 View와 Presenter가 결합되는 단점을 해결한 패턴이 MVVM 패턴이다. MVVM은 Model, View, ViewModel로 구성된다.
Presenter 대신 ViewModel이 존재한다.

MVVM은 View와 ViewModel이 N:1 관계를 갖는다. 즉, 하나의 ViewModel이 여러 View에서 사용될 수 있다는 의미이다.
이것이 어떻게 가능할까? 방법은 observable 값을 이용하는 것이다. 다시 말해 ViewModel에 관찰 가능한 값을 두고 View에서 이를 구독하는 것이다.

observable 값이 없다면 의존성 관계는 아래와 같을 것이다. 이벤트를 뷰모델에 전달하고 뷰모델의 로직이 실행되어 뷰에 값의 변경을 알려야 하기 때문이다.

![vvm](https://user-images.githubusercontent.com/57604817/209697713-cf474f33-5663-47c5-bc3b-56ef61a92911.png)

이는 [Observer 패턴](https://en.wikipedia.org/wiki/Observer_pattern)을 이용하여 문제를 해결할 수 있다.
아래의 다이어그램을 보면 `Observer` 인터페이스를 통해 ViewModel에서 View를 의존하는 문제가 해결된 것을 볼 수 있다.

![vvm-observer](https://user-images.githubusercontent.com/57604817/209699179-6fece0b6-3e68-43fe-9daf-70107027893a.png)

안드로이드 코드로 예를 들어보겠다. ViewModel는 관찰 가능한 값인 `LiveData`를 가진다. 이는 `StateFlow`로 대체할 수 있다.
`MainActivity`에서는 `MainViewModel`을 가지며 `observe` 함수를 호출하여 `uiState` 갱신을 구독한다.
만약 `onClickSomething`가 호출되어 `viewModel.doSomething`이 실행된다면 viewModel의 observable 값인 `uiState`가 갱신될것이고 이를 구독하는 뷰에 알림이 간다.
뷰는 알림을 받아 처리하면 된다.

```kotlin
data class MainUiState(
  @StringRes val userMessage: Int? = null
)

class MainViewModel : ViewModel() {

    private val _uiState = MutableLiveData(MainUiState())
    val uiState: LiveData<MainUiState> = _uiState

    fun doSomething() {

      ...

      showUserMessage()
    }

    private fun showUserMessage() {
      requireNotNull(_uiState.value).let {
        _uiState.postValue(it.copy(userMessage = R.string.some_string))
      }
    }
}

class MainActivity : AppCompactActivity() {

  private val viewModel: MainViewModel by viewModels()

  override fun onCreate(savedInstanceState: Bundle?) {

    ...

    viewModel.uiState.observe(this, ::updateUi)
  }

  private fun updateUi(uiState: MainUiState) {
    ...
  }

  private fun onClickSomething() {
    viewModel.doSomething()
  }
}
```

## 장점

- ViewModel이 View에 독립적이기 때문에 중복되는 로직을 모듈화 할 수 있다. 이는 테스트하기에도 좋다.

## 단점

- 상태 값이 많아지면 관리하기 어려워진다.

<br></br>

# MVI

Coming soon..

# 참조

https://thdev.tech/androiddev/2016/10/23/Android-MVC-Architecture/
