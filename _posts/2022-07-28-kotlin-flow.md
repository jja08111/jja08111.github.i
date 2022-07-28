---
title: "[Kotlin] Flow 정리"
date: 2022-07-28 22:00:00 +0900
categories: kotlin
tags:
  - kotlin
  - flow
---

비동기 데이터 흐름인 Flow에 대해 정리한다.

# Flow

Flow 문서의 첫 문장은 다음과 같다.

> An asynchronous data stream that sequentially emits values and completes normally or with an exception.

순차적으로 값을 방출하고 완료되는 비동기 데이터 흐름이라고 한다.

Dart의 `Stream`과 유사하다. suspend 함수는 단일 값만 반환하는 것과 다르게 Flow는 여러개의 값을 순차적으로 반환한다.

예를 들면 단순한 API 요청을 통해 비동기로 하나의 값을 얻는 것과 다르게, Firebase의 database와 같이 실시간으로 변화하는 값들을 Flow를 통해 얻을 수 있다.

흐름은 기본 스레드를 차단하지 않고 다음 값을 생성할 네트워크 요청을 안전하게 만들 수 있다.

흐름에는 다음과 같은 세 가지 요소가 존재한다.

- 생산자: 스트림에 추가되는 데이터를 생산하며 코루틴을 이용하여 비동기적으로 생산할 수 있다.
- 중개자: 스트림에 내보내지는 값들을 수정하거나 스트림 자체를 수정할 수 있다.
- 소비자: 스트림 값을 사용한다.

Flow는 collect를 하지 않는 경우 값을 방출하지 않는다. 데이터를 hold하지 않으며, 어떠한 상태 값을 가지지 않는다. 이를 cold flow라고 한다.

Android에서는 Repository가 생산자의 역할을 하며 UI에서 소비자 역할을 담당한다.

흔히 사용되는 패턴은 Room 데이터베이스 변경을 실시간으로 업데이트 받는 것이다.

```kotlin
@Dao
abstract class ExampleDao {
    @Query("SELECT * FROM Example")
    abstract fun getExamples(): Flow<List<Example>>
}
```

# SharedFlow

SharedFlow는 Flow를 상속하여 `replayCache`라는 속성이 존재하는 흐름이다. 이 속성은 새로운 소비자가 나타났을 때 이전에 발생했던 이벤트를 새로운 소비자에게 얼마나 전달할지 결정한다. 그렇기 때문에 SharedFlow는 hot flow이다.

```kotlin
public interface SharedFlow<out T> : Flow<T> {

    public val replayCache: List<T>

    override suspend fun collect(collector: FlowCollector<T>): Nothing
}
```

`sharedIn`을 통해 콜드 플로우를 핫 플로우로 전환할 수 있다.

```kotlin
class NewsRemoteDataSource(...,
    private val externalScope: CoroutineScope,
) {
    val latestNews: Flow<List<ArticleHeadline>> = flow {
        ...
    }.shareIn(
        externalScope,
        replay = 1,
        started = SharingStarted.WhileSubscribed()
    )
}
```

`sharedIn`을 통해 이 흐름은 `externalScope`이 활성 상태에 있고 수집기가 있는 동안 활성 상태로 유지된다. `replay`값을 통해 이전 이벤트를 새 수집기에 얼마나 전달할지 결정한다.

# StateFlow

StateFlow는 ShareFlow에서 상태가 존재하는 흐름이다. 상태란 무엇일까? 간단히 말하자면 초기 값을 가지고 있으며 언제든지 특정한 값을 가지고 있는 것이다. StateFlow는 `value` 속성을 통해 현재 값을 얻을 수 있다.

```kotlin
class LatestNewsViewModel(
    private val newsRepository: NewsRepository
) : ViewModel() {

    // 다른 클래스에서 업데이트 할 수 없도록 변경 가능한 상태는 숨긴다.
    private val _uiState = MutableStateFlow(LatestNewsUiState.Success(emptyList()))
    // UI는 상태를 업데이트 하기 위해 이 StateFlow를 수집한다.
    val uiState: StateFlow<LatestNewsUiState> = _uiState

    init {
        viewModelScope.launch {
            newsRepository.favoriteLatestNews
                // 최신 favoriteNews 정보로 뷰를 업데이트한다.
                // MutableStateFlow의 value 속성에 값을 쓴다.
                // flow에 새로운 값을 추가하고 collector들을 모두 업데이트한다.
                .collect { favoriteNews ->
                    _uiState.value = LatestNewsUiState.Success(favoriteNews)
                }
        }
    }
}
```

Android에서는 관찰해야하고 변경 가능한 상태 값을 위해 쓰인다. 유사한 것으로 LiveData가 있으나 차이가 존재한다. LiveData는 초기값이 없으며 View lifecycle에 맞게 설계되어 있다. 하지만 StateFlow는 초기값이 존재하며, 항상 활성 상태이고 메모리 내에 있으며 참조가 없을 때까지 존재한다. 그래서 꼭 아래와 같이 `repeatOnLifecycle`을 사용하여 lifecycle이 STARTED일 때마다 새로 코루틴을 실행하고 lifecycle이 STOPPED일때 취소되도록 해야한다.

```kotlin
class LatestNewsActivity : AppCompatActivity() {
    private val latestNewsViewModel = // getViewModel()

    override fun onCreate(savedInstanceState: Bundle?) {
        ...
        // Start a coroutine in the lifecycle scope
        lifecycleScope.launch {
            // repeatOnLifecycle launches the block in a new coroutine every time the
            // lifecycle is in the STARTED state (or above) and cancels it when it's STOPPED.
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                // Trigger the flow and start listening for values.
                // Note that this happens when lifecycle is STARTED and stops
                // collecting when the lifecycle is STOPPED
                latestNewsViewModel.uiState.collect { uiState ->
                    // New value received
                    when (uiState) {
                        is LatestNewsUiState.Success -> showFavoriteNews(uiState.news)
                        is LatestNewsUiState.Error -> showError(uiState.exception)
                    }
                }
            }
        }
    }
}
```

# 테스트하기

일반적인 테스트는 모조 생산자를 만들어 진행한다.

```kotlin
class MyFakeRepository : MyRepository {
    fun observeCount() = flow {
        emit(1)
    }
}
```

SharedFlow를 이용하면 더욱 다양한 상황을 테스트 할 수 있다.

```kotlin
class MyFakeRepository : MyRepository {

    private val flow = MutableSharedFlow<Int>()

    suspend fun emit(value: Int) {
        flow.emit(value)
    }

    fun observeCount() = flow
}
```

viewModel을 테스트하는 경우 UI없이 진행하기 때문에 모조 수집기가 필요하다. 그럴때는 아래와 같이 빈 수집기를 작동시키면 된다.

```kotlin
@Test
fun shouldDoSomething_WhenSomethingIsVisible() = runTest {
    ...

    // StateFlow를 위해 빈 수집기를 작동시킨다.
    val collectJob = launch(testDispatcher) {
        searchViewModel.uiState.collect()
    }

    ...

    collectJob.cancel()
}
```
