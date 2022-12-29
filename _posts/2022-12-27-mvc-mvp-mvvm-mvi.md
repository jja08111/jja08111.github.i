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

그런데 Android에서는 Activity나 Fragment들이 View와 Control 모두 가지고 있다.
예를 들면 아래와 같은 코드이다.

```kotlin
class MainActivity : AppCompactActivity() {

  override fun onCreate(savedInstanceState: Bundle?) {
    // ...

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
- View와 Controller의 결합도가 높아 Controller를 테스트하기 어렵다.

<br>

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

  fun onClickSomething() {
    // ...

    doSomething()
  }

  fun doSomething() {
    // ...

    viewInterface.showSnackBar(R.string.some_string)
  }
}
```

## 장점

- Presenter가 View에 대한 의존성이 없기 때문에 Presenter를 테스트하기 좋다.
- 코드가 적절히 분리되어 유지보수하기 좋다.

## 단점

- View와 Presenter가 1:1 의존 관계이기 때문에 유사한 유사한 로직을 가진 뷰들이 있을 때에도 계속해서 Presenter를 만들어야하는 단점이 존재한다.

<br>

# MVVM

MVP에서 1:1로 View와 Presenter가 결합되는 단점을 해결한 패턴이 MVVM 패턴이다. MVVM은 Model, View, ViewModel로 구성된다.
Presenter 대신 ViewModel이 존재한다.

MVVM은 View와 ViewModel이 N:1 관계를 갖는다. 즉, 하나의 ViewModel이 여러 View에서 사용될 수 있다는 의미이다.
이것이 어떻게 가능할까? 방법은 observable 값을 이용하는 것이다. 다시 말해 ViewModel에 관찰 가능한 값을 두고 View에서 이를 구독하는 것이다.

observable 값이 없다면 의존성 관계는 아래와 같을 것이다. 이벤트를 뷰모델에 전달하고 뷰모델의 로직이 실행되어 뷰에 값의 변경을 알려야 하기 때문이다.

![vvm](https://user-images.githubusercontent.com/57604817/209697713-cf474f33-5663-47c5-bc3b-56ef61a92911.png)

이는 [Observer 패턴](https://en.wikipedia.org/wiki/Observer_pattern)을 이용하여 문제를 해결할 수 있다.
아래의 다이어그램을 보면 `FlowCollector` 인터페이스를 통해 ViewModel에서 View를 의존하는 문제가 해결된 것을 볼 수 있다.

![vvm-observer](https://user-images.githubusercontent.com/57604817/209753151-684a0712-e20d-4885-85c0-b42fd529cf58.png)

안드로이드 코드로 예를 들어보겠다. ViewModel은 관찰 가능한 값인 `StateFlow`를 가진다.
`MainActivity`에서는 `MainViewModel`을 가지며 `collect` 함수를 호출하여 `uiState` 갱신을 구독한다.
만약 `onClickSomething`가 호출되어 `viewModel.doSomething`이 실행된다면 viewModel의 observable 값인 `uiState`가 갱신될것이고 이를 구독하는 뷰에 알림이 간다.
뷰는 알림을 받아 처리하면 된다.

```kotlin
data class MainUiState(
  @StringRes val userMessage: Int? = null
)

class MainViewModel : ViewModel() {

  private val _uiState = MutableStateFlow(MainUiState())
  val uiState get() = _uiState.asStateFlow()

  fun doSomething() {
    // ...

    showUserMessage()
  }

  private fun showUserMessage() {
    _uiState.update {
      it.copy(userMessage = R.string.some_string)
    }
  }

  fun userMessageShown() {
    _uiState.update {
      it.copy(userMessage = null)
    }
  }
}

class MainActivity : AppCompactActivity() {

  private val viewModel: MainViewModel by viewModels()

  override fun onCreate(savedInstanceState: Bundle?) {
    // ...

    lifecycleScope.launch {
      repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.uiState.collect(::updateUi)
      }
    }
  }

  private fun updateUi(uiState: MainUiState) {
    // ...
  }

  private fun onClickSomething() {
    viewModel.doSomething()
  }
}
```

## 장점

- ViewModel이 View에 독립적이기 때문에 중복되는 로직을 모듈화 할 수 있다. 이는 테스트하기에도 좋다.
- 코드가 적절히 분리되어 유지보수하기 좋다.

## 단점

- 상태 값을 잘못 관리하면 문제가 생긴다. 물론 앞서 보인 코드처럼 UiState를 두어 좋은 구조로 구현하면 큰 문제는 생기지 않는다. 하지만 이곳저곳 상태 값이 흩어지는 등 잘못 구현하는 경우 문제가 생긴다.
- 부수효과(Side-Effect) 관리가 어렵다. 예를 들어 스낵바를 보이는 것을 구현한다고 해보자. `userMessage`에 문자열 리소스를 넣어 uiState를 갱신하면 스낵바가 보인다. 그리고 **보여졌을 때 이 값을 다시 `null`로 만들어야 한다**.
  그렇지 않으면 또 다시 스낵바가 보이는 문제가 발생할 수 있다. 이렇듯 부수효과 관리가 까다롭다.

<br>

# MVI

MVVM에서 상태 관리를 잘못할 수 있는 부분과 부수효과를 관리하기 어려운 단점을 개선한 패턴이 MVI 패턴이다. MVI는 MVVM에서 ViewModel 대신 Intent가 존재한다.

MVI는 아래의 그림과 같이 사이클이 존재하는 단방향 그래프 구조이다. user가 intent를 발생시켜 model에 전달 되고 다시 이는 view에 전달된다. view는 user에게 보여지고 다시 유저는 이벤트를 발생시킨다.
intent가 model에 전달되는 것 외에도 sideEffect가 model에 전달된다.

![mvi](https://user-images.githubusercontent.com/57604817/209839928-d195d19b-e5a8-4dc8-8a6c-c1cbef9cc9bc.png)

직접 위의 구조를 구현하면 아래와 같다. 중복되는 코드들을 `Container`라는 클래스에 모아 다시 사용할 수 있다. `Container`를 살펴보면 uiState 그리고 sideEffect가 존재하는 것을 볼 수 있다.
uiState는 `StateFlow`라 특정한 값을 가지고 있는 반면 sideEffect는 `Channel`이어서 값을 보내기만 하며 가지고 있지는 않는다. `reduce` 함수는 상태를 갱신한다. `postSideEffect`는 부수효과를 전달한다.
`intent` 함수는 정해진 `scope`에서 suspend 함수를 실행한다.

```kotlin
class MainViewModel(
  private val postRepository: PostRepository = PostRepository()
) : ViewModel() {

  val container = Container<MainUiState, MainSideEffect>(
    initialState = MainUiState(),
    scope = viewModelScope
  )

  init {
    fetchOverviews()
  }

  private fun fetchOverviews() = container.intent {
    val result = postRepository.getOverviews()
    if (result.isSuccess) {
      reduce {
        copy(overviews = result.getOrNull()!!)
      }
    } else {
      TODO()
    }
  }

  fun onPostClicked(overview: PostOverview) = container.intent {
    postSideEffect(MainSideEffect.NavigateToDetails(overview.id))
  }
}

class Container<STATE, SIDE_EFFECT>(
    initialState: STATE,
    private val scope: CoroutineScope
) {

  private val _uiState = MutableStateFlow(initialState)
  val uiState: StateFlow<STATE> = _uiState.asStateFlow()

  private val _sideEffect = Channel<SIDE_EFFECT>(Channel.BUFFERED)
  val sideEffect: Flow<SIDE_EFFECT> = _sideEffect.receiveAsFlow()

  fun intent(transform: suspend Container<STATE, SIDE_EFFECT>.() -> Unit) {
    scope.launch(SINGLE_THREAD) {
      this@Container.transform()
    }
  }

  suspend fun reduce(reducer: STATE.() -> STATE) {
    withContext(SINGLE_THREAD) {
      _uiState.value = _uiState.value.reducer()
    }
  }

  suspend fun postSideEffect(event: SIDE_EFFECT) {
    _sideEffect.send(event)
  }

  companion object {
    @OptIn(DelicateCoroutinesApi::class)
    private val SINGLE_THREAD = newSingleThreadContext("mvi")
  }
}

data class MainUiState(
    val overviews: List<PostOverview> = emptyList()
)

sealed class MainSideEffect {
    data class NavigateToDetails(val postId: String) : MainSideEffect()
}
```

하지만 위처럼 직접 Container를 구현하기보다 [Orbit](https://github.com/orbit-mvi/orbit-mvi)이라는 라이브러리를 사용하는 것이 좋다.
Orbit에는 직접 구현했던 Container 보다 더나은 기능들이 있기 때문이다.

- 더 엄격한 DSL scoping: 예를 들어 reduce 스코프 안에서 reduce를 또 호출하지 못하도록 막아준다.
- 개선된 스레딩 모델: 더 최적화된 방법으로 스레드를 다룬다.
- 유닛 테스트
- 테스트 프레임워크
- Idling resource 지원
- Saved state 지원

Orbit을 이용하여 코드를 수정한다면 아래와 같다. `reduce` 함수는 내부적으로 `StateFlow.update`를 호출하기 때문에 멀티 스레드에 안정적이다.
`intent`는 container를 생성할 때 ViewModel의 `viewModelScope`가 전달되어 내부적으로 이 코루틴 스코프를 이용한다.

```kotlin
class MainViewModel(
  private val postRepository: PostRepository = PostRepository(),
) : ViewModel(), ContainerHost<MainUiState, MainSideEffect> {

  override val container: Container<MainUiState, MainSideEffect>
    get() = container(MainUiState())

  init {
    fetchOverviews()
  }

  private fun fetchOverviews() = intent {
    val result = postRepository.getOverviews()
    if (result.isSuccess) {
      reduce {
        state.copy(overviews = result.getOrNull()!!)
      }
    } else {
      // ...
    }
  }

  fun onPostClicked(overview: PostOverview) = intent {
    postSideEffect(MainSideEffect.NavigateToDetails(overview.id))
  }
}
```

위의 뷰모델을 사용하는 뷰는 아래와 같이 코드를 작성하면 된다.

```kotlin
  // ...

  viewModel.observe(
    this@MainActivity,
    state = ::render,
    sideEffect = ::handleSideEffect
  )
}

private fun render(uiState: MainUiState) {
  adapter.submitList(uiState.overviews)
}

private fun handleSideEffect(sideEffect: MainSideEffect) {
  when (sideEffect) {
    is MainSideEffect.NavigateToDetails -> {
      navigateToPostView(sideEffect.postId)
    }
  }
}
```

## 장점

- 뷰의 생명주기 동안 일관성 있는 상태를 갖는다.
- 불변 Model은 통해 멀티 스레드 안정성과 안정적인 동작을 제공한다.
- 누가 작성해도 어느정도 보장된 수준의 코드 퀄리티가 나온다.

## 단점

- 배우는데 시간이 필요하다. 코루틴을 정확히 이해해고 있어야 한다.
- 작은 행위에도 SideEffect 클래스를 만들어야 하기 때문에 번거로울 수 있다.

<br>

# 참조

- https://thdev.tech/androiddev/2016/10/23/Android-MVC-Architecture/
- https://www.youtube.com/watch?v=E6obYmkkdko
- https://medium.com/myrealtrip-product/android-mvi-79809c5c14f0
