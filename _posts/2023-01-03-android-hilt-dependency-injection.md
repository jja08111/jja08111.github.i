---
title: "[Android] 의존성 주입"
date: 2023-01-03 01:20:00 -0400
categories: android
tags:
  - android
  - hilt
  - dagger
---

Dependency injection(의존성 주입)이 무엇일까? 그리고 왜 사용될까? 먼저 아래의 코드를 보자.
아래의 코드를 보면 `db`와 `codecs`가 의존성 주입 없이 곧바로 생성되고 있다.
만약 `MusicPlayer` 객체를 테스트한다고 생각해보자. `db`, `codecs` 변수들 때문에 테스하기 쉽지 않다.
또한 다른 유형의 db가 필요할 경우에 기존 코드를 수정하거나 새로운 `MusicPlayer`가 필요할 것이다.

```kotlin
class MusicPlayer {
    private val db = SQLiteDatabase()
    private val codecs = listOf(CodecH264(), CodecFLAC())

    fun play(id: String) {
        //...
    }
}

fun main() {
    val player = MusicPlayer()
    player.play("YHLQMDLG")
}
```

위와 다르게 아래와 같은 코드를 작성한다면 다음과 같은 장점이 있다.

- `Database`의 재사용이 가능하다. `Database`의 다양한 구현체를 `db`에 전달할 수 있기 때문이다.
- 테스트하기 용이하다. 테스트 더블을 생성자에 전달하여 쉽게 테스트를 진행할 수 있다.

```kotlin
class MusicPlayer(
    private val db: Database,
    private val codecs: List<Codec>
) {
    fun play(id: String) { ... }
}

fun main() {
    val db = SQLiteDatabase()
    val codecs = listOf(CodecH264(), CodecFLAC())
    val player = MusicPlayer(db, codecs)
    player.play("YHLQMDLG")
}
```

그런데 만약 아래와 같은 객체가 있다면 하나씩 직접 의존성을 주입하기 굉장히 까다로울 것이다. 그리고 대규모 앱의 경우 모든 종속 항목을 가져와 올바르게 연결하려면 **대량의 상용구** 코드가 필요할 수 있다.
다중 레이어 아키텍처에서는 최상위 레이어의 객체를 생성하려면 그 아래에 있는 레이어의 **모든 종속 항목을 제공**해야 한다.
구체적인 예로, 실제 자동차를 만들려면 엔진, 변속기, 섀시 및 기타 부품이 필요할 것이다. 그리고 엔진에는 실린더와 점화 플러그도 필요하다.

```kotlin
class KeywordViewModel(
    readKeywordListUseCase: ReadKeywordListUseCase,
    private val addKeywordUseCase: AddKeywordUseCase,
    private val removeKeywordUseCase: RemoveKeywordUseCase,
    private val subscribeToUseCase: SubscribeToUseCase,
    private val unsubscribeFromUseCase: UnsubscribeFromUseCase,
    private val isSignedInUseCase: IsSignedInUseCase,
    private val hasSearchResultUseCase: HasSearchResultUseCase,
) {
    ...
}
```

이때 필요한 것이 의존성 주입 라이브러리이다. 물론 수동으로 의존성 주입 컨테이너를 구현할 수도 있다. 하지만 [종속 항목 수동 삽입](https://developer.android.com/training/dependency-injection/manual) 글을 읽어 보면 굉장히 많은 상용구가 존재하여 불편하고, 수명 주기를 잘못 관리할 수 있어 메모리 누수 위험성이 있다.

# Hilt

Hilt는 Dagger가 제공하는 **컴파일 타임 주입**, 런타임 성능, 확장성 및 Android 스튜디오 지원의 이점을 누리기 위해 DI 라이브러리인 Dagger를 기반으로 빌드되었다.
또한 Hilt는 프로젝트의 모든 Android 클래스에 의존성을 주입하는 컨테이너를 제공하고 **수명 주기를 자동으로 관리**함으로써 애플리케이션에서 DI를 사용하는 표준 방법을 제공한다.

# Koin

Koin은 Hilt에 비해 러닝커브가 낮으며 별도의 어노테이션을 사용하지 않기 때문에 빌드 시간이 비교적 빠르다.

Hilt와 다르게 Koin은 **런타임 시 주입**을 하는 방식인 Sevice Locator 패턴을 사용한다.
`koin.get()` 함수 호출이 컴포넌트가 모듈에 선언되어 있는지 컴파일 타임에 확인할 수 없기 때문에 이는 앱이 점점 커지게 될 경우 런타임에 크래쉬를 발생 시킬 수 있다. 또한 런타임 시 주입을 하기 때문에 의존 관계를 파악하기 어렵다.

나는 컴파일 시에 의존성을 주입해주는 Hilt, Dagger를 선호한다. 안정성이 높기 때문이다.

# 결론

- 의존성 주입을 하면 다양한 구현체를 주입할 수 있어 코드 재활용성이 높아지고, 테스트 더블을 전달하여 다양한 시나리오를 쉽게 테스트 할 수 있다.
- 의존성 주입 라이브러리를 사용하여 보일러플레이트 코드 제거 및 손쉬운 수명 주기 관리 그리고 안정성을 향상 시킬 수 있다.
