---
title: "[Android] Mockk를 이용하여 테스트하기"
date: 2022-04-06 09:20:00 -0400
categories: android
tags:
  - android
  - compose
  - mock
  - test
---

mockk를 이용하여 테스트 코드를 만드는 방법을 소개하겠다. 먼저 다음과 같이 의존성을 추가해준다. mockk_version에 최신 버전을 넣어주면 된다.

### /app/build.gradle

```gradle
dependencies {
    ...

    testImplementation "io.mockk:mockk:$mockk_version"
    androidTestImplementation "io.mockk:mockk-android:$mockk_version"
}
```

나는 테스트해야 하는 것이 다음과 같았다.
View model에서 스크래핑을 위해 스크래핑 repository의 함수를 호출하고 그 안에서는 `Jsoup.connect()`를 호출한다.
이때 인터넷 연결이 안되어서 에러를 던지는 경우 UI에 문제가 있음을 보인다.
이를 테스트 코드로 작성하면 아래와 같다. 함수의 시작 부분의 두 줄에서 mock을 형성하고 있는 것을 볼 수 있다.

```kotlin
@OptIn(ExperimentalPagerApi::class)
@Test
fun showErrorPage_whenInternetIsNotConnected() {
    mockkStatic(Jsoup::class)
    every { Jsoup.connect(URL) } throws InternetNotConnectedException()

    val scaffoldState = createMockScaffoldState()
    val homeViewModel = HomeViewModel(
        scaffoldState = scaffoldState,
        searchUrl = URL,
        coroutineContext = coroutineContext,
    )

    composeTestRule.setContent {
        FakeHomeScreen(homeViewModel = homeViewModel, scaffoldState = scaffoldState)
    }

    runBlocking {
        awaitFrame()
        homeViewModel.apply {
            assertEquals(
                scaffoldState.snackbarHostState.currentSnackbarData != null,
                true
            )
            composeTestRule
                .onAllNodesWithText("인터넷에 연결되지 않았습니다.")
                .onFirst()
                .assertIsDisplayed()
        }
    }
}
```

그런데 막상 실행해보니 아래와 같은 에러만 잔뜩 보인다. 아예 mockk가 작동하지 않는 것이었다.

```console
java.lang.ExceptionInInitializerError
at com.foundy.hansungcafeteria.ui.HomeViewModelTest.showErrorPage_whenInternetIsNotConnected(HomeViewModelTest.kt:200)
... 38 trimmed
Caused by: io.mockk.proxy.MockKAgentException: MockK could not self-attach a jvmti agent to the current VM. This feature is required for inline mocking.
This error occured due to an I/O error during the creation of this agent: java.io.IOException: Unable to dlopen libmockkjvmtiagent.so: dlopen failed: library "libmockkjvmtiagent.so" not found

... 생략
```

구글에 찾아보니 다음과 같은 [해결 방법](https://github.com/mockk/mockk/issues/297#issuecomment-901924678)이 있었다.
방법은 gradle에 다음과 같은 코드를 추가해주는 것이다.

```gradle
android {
    testOptions {
        packagingOptions {
            jniLibs {
                useLegacyPackaging true
            }
        }
    }
}
```

그 후 성공적으로 테스트를 통과했다.

![result](/assets/images/2022-04-06-11.22.05.png)
