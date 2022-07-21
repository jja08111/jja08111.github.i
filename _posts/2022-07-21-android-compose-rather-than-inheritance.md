---
title: "[Android] 코드 중복을 Hilt를 이용하여 구성(Composition) 방식으로 제거"
date: 2022-07-21 09:00:00 +0900
categories: android
tags:
  - android
  - hilt
  - dagger
  - composition
---

최근 [한성대 공지 어플](https://github.com/jja08111/HansungNotification)을 상속 대신 구성을 활용하게 된 경험이 있어 글로 정리하고자 한다.

# 배경

한성대 공지 어플에는 공지사항을 즐겨찾기 하는 기능이 있다. 이 기능은 홈 화면, 검색 화면 등 다양한 화면에서 작동한다.
해당 기능을 제공하는 곳에서는 즐겨찾기 추가, 삭제, 이 공지사항이 즐겨찾기인지 아닌지 파악하는 작업을 해야한다.

그렇게 필요한 `ViewModel`들을 만들다보니 아래의 코드가 중복되고 있다는 것을 발견했다.

**SomeViewModel.kt**

```kotlin
    ...

    val noticeFlow = getNoticeListUseCase().cachedIn(viewModelScope).map {
        it.map(::createNoticeItemUiState)
    }

    fun createNoticeItemUiState(notice: Notice): NoticeItemUiState {
        return NoticeItemUiState(
            notice,
            onClickFavorite = { isFavorite ->
                viewModelScope.launch(dispatcher) {
                    if (isFavorite) {
                        addFavoriteNoticeUseCase(notice)
                    } else {
                        removeFavoriteNoticeUseCase(notice)
                    }
                }
            },
            isFavorite = { isFavoriteNoticeUseCase(notice) }
        )
    }

    ...
```

이렇게 계속하다가는 더 많은 곳에서 코드가 중복되고 수정할 때 마다 모든 곳을 수정해야 할 것 같았다. 그래서 중복을 제거하기 위해 여러 고민을 했다.
일단은 `StateFlow`를 Repository로 이동시켜 하나만 두도록 했다. 그리고 `NoticeItemUiState`를 생성하는 코드를 어떻게 할 지 생각했다.

# 달콤한 유혹 - 상속(Inheritance)

첫 번째 생각은 코드 재활용을 위해 "_상속을 이용해볼까?_" 였다. 아래와 같은 `ViewModel`을 만들고 필요한 곳에서 이를 상속하면 어떨까 잠시 고민했다.

```kotlin
abstract class FavoriteViewModel @Inject constructor(
  ...
) : ViewModel() {
  ...
}
```

하지만 객체지향 기본 내용인 _코드를 재사용하기 위해 상속하면 안된다는_ 말이 떠올랐다. 위의 사례는 `IS-A` 관계도 아니었다. 무분별한 상속은 부모 자식간에 결합도가 커지고 확장성이 떨어지는 문제가 있다.
따라서 구성(Composition)을 하기로 결정했다.

# 구성(Composition)

구성은 상속과 다르게 `HAS-A` 관계라고 생각하면 된다. 그런데 막상 구성 방식으로 구현하려 해보니 쉽지 않았다. Hilt를 사용하고 있었고 비동기 호출이 있어 `viewModelScope`를 전달받아야 했다.

어떻게 런타임에 주입할 수 있을지 찾아보니 `AssistedFactory`라는 어노테이션이 있는 것을 발견했다. 이는 모든 생성자 매개변수를 한 번에 의존성 주입하지 않고 필요한 부분은 따로 런타임에 주입할 수 있게 해준다.

구현된 결과는 아래와 같다. `viewModelScope`를 런타임에 주입 가능하다. `dispatcher`는 테스트를 위해 존재한다.
`triggerCollection`는 `Flow`가 `collect`를 하지 않으면 방출이 시작되지 않기 때문에 의도적으로 방출을 시작하기 위해 있다.

```kotlin
@AssistedFactory
interface NoticeItemUiStateCreatorFactory {
    fun create(
        viewModelScope: CoroutineScope,
        dispatcher: CoroutineDispatcher = Dispatchers.Main,
        triggerCollection: Boolean = true
    ): NoticeItemUiStateCreator
}

class NoticeItemUiStateCreator @AssistedInject constructor(
    readFavoriteListUseCase: ReadFavoriteListUseCase,
    private val addFavoriteNoticeUseCase: AddFavoriteNoticeUseCase,
    private val removeFavoriteNoticeUseCase: RemoveFavoriteNoticeUseCase,
    private val isFavoriteNoticeUseCase: IsFavoriteNoticeUseCase,
    @Assisted private val viewModelScope: CoroutineScope,
    @Assisted private val dispatcher: CoroutineDispatcher,
    @Assisted triggerCollection: Boolean
) {

    init {
        if (triggerCollection) {
            viewModelScope.launch(dispatcher) {
                readFavoriteListUseCase().collect()
            }
        }
    }

    /**
     * Favorite에 대한 상태를 가진 [NoticeItemUiState]를 [notice]로부터 생성한다.
     */
    fun create(notice: Notice): NoticeItemUiState {
        return NoticeItemUiState(
            notice,
            onClickFavorite = { isFavorite ->
                viewModelScope.launch(dispatcher) {
                    if (isFavorite) {
                        addFavoriteNoticeUseCase(notice)
                    } else {
                        removeFavoriteNoticeUseCase(notice)
                    }
                }
            },
            isFavorite = { isFavoriteNoticeUseCase(notice) }
        )
    }
}
```

위의 코드는 아래와 같이 해당 `ViewModel`들에서 이용하고 있다.
아래를 살펴보면 `noticeItemUiStateCreatorFactory`를 주입받고 있다. 그리고 `create(viewModelScope)`를 호출하여 `NoticeItemUiStateCreator`를 생성하는 모습을 볼 수 있다.

```kotlin
@HiltViewModel
class NoticeViewModel @Inject constructor(
    getNoticeListUseCase: GetNoticeListUseCase,
    noticeItemUiStateCreatorFactory: NoticeItemUiStateCreatorFactory
) : ViewModel() {

    ...

    private val noticeItemUiStateCreator = noticeItemUiStateCreatorFactory.create(viewModelScope)

    init {
        viewModelScope.launch {
            getNoticeListUseCase().cachedIn(viewModelScope).map { pagingData ->
                pagingData.map(noticeItemUiStateCreator::create)

                ...

            }
        }
    }
}
```

이제 새로운 `ViewModel`에서 `NoticeItemUiState`를 생성해야 할 때 간단히 코드를 작성하여 생성할 수 있게 되었다.

# 참조

- [Assisted Injection - Dagger](https://dagger.dev/dev-guide/assisted-injection.html)
