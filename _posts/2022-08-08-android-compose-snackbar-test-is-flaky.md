---
title: "[Android] 불안정한 compose snackbar 테스트 문제 해결"
date: 2022-08-08 16:00:00 +0900
categories: android
tags:
  - android
  - compose
  - test
  - flaky
---

요즘 개발하던 앱이 Github Action에서 수행하는 instrument 테스트에 가끔씩 실패했다. 실패한 테스트는 compose로 구현한 화면에서 특정 액션 후 스낵바가 보이는지 확인하는 테스트였다.
이게 가끔씩 실패하니 해결하려고 시도해도 해결되었는지 파악조차 어려웠다.

![action_failure](https://user-images.githubusercontent.com/57604817/183372648-1cf050bc-94c3-4127-bc99-531f9d78358b.png)

안드로이드 팀은 테스트에 실패하면 PR을 머지할 수 없도록 했다. 그렇기 때문에 불안정한 테스트 때문에 실패하는 경우 다시 테스트를 돌리고 끝나기를 기다려야한다. 이는 상당히 거슬리는 문제였다.

**환경**

- compose: 1.2.0

# Minimal reproducible example

일단 최소화된 예제를 만들어 정말로 스낵바로 인해 문제가 발생하는지 확인하기로 했다. 아래처럼 간단히 컴포저블 함수와 테스트 코드를 작성했다.

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun SomeScreen() {
    val snackbarHostState = remember { SnackbarHostState() }
    val coroutineScope = rememberCoroutineScope()

    Scaffold(
        snackbarHost = { SnackbarHost(hostState = snackbarHostState) }
    ) {
        Box(Modifier.padding(it)) {
            TextButton(
                onClick = {
                    coroutineScope.launch {
                        snackbarHostState.showSnackbar("message")
                    }
                }
            ) {
                Text(text = "Button")
            }
        }
    }
}
```

```kotlin
class SomeScreenTest {

    @get:Rule(order = 1)
    val composeRule = createComposeRule()

    @Test
    fun showSnackBar_WhenClickButton() {
        with(composeRule) {
            setContent { SomeScreen() }

            onNodeWithText("Button").performClick()

            onNodeWithText("message").assertIsDisplayed()
        }
    }
}
```

위의 테스트 코드를 수행하면 대부분 성공하지만 가끔 실패했다. 이 불안정함을 확실히 제거하기 위해서 방법이 필요했다.

# Parameterized

일단 문제가 발생하는지 안하는지 확실히 하고 싶었다. 생각해낸 방법은 동일한 테스트를 한 번에 100번 연속 수행하여 한 번도 실패하지 않으면 문제가 해결된 것이라 여기는 것이다.

처음에는 코드를 복사 붙여넣기하여 15개 정도를 만들어 수행했다. 하지만 코드를 수정할 때마다 반복해서 복붙해야 하니 너무 번거로웠다.

다른 방법을 찾아보니 `Parameterized`라는 클래스를 발견했다. 이는 테스트를 수행할 때 동일한 테스트를 parameter만 다르게 전달하여 테스트할 수 있게 해주는 클래스이다.
이를 이용하여 빈 parameter를 아래처럼 100개 만들었다.

```kotlin
@RunWith(Parameterized::class)
class SomeScreenTest {

    companion object {
        private const val NUM_REPEATS = 100

        @JvmStatic
        @Parameterized.Parameters
        fun data(): Collection<Array<Any?>> {
            val out: MutableCollection<Array<Any?>> = ArrayList()
            for (i in 0 until NUM_REPEATS) {
                out.add(arrayOfNulls(0))
            }
            return out
        }
    }

    ...
}
```

위의 코드를 추가하고 3번 테스트를 돌려보았다. 총 300회의 테스트 중 5회 실패했다.

![before1](https://user-images.githubusercontent.com/57604817/183372678-c0ba0acd-5940-4262-9348-9aa50371f8df.png)
![before2](https://user-images.githubusercontent.com/57604817/183372684-7c4a14a0-c390-4677-ba43-caae7b4cbda8.png)
![before3](https://user-images.githubusercontent.com/57604817/183372708-23a8419b-6b60-45eb-a8a6-9515bb7f1338.png)

생각보다 적은 숫자이지만 Github Action 환경에서는 꽤 빈번하여 꼭 해결하고 싶었다.

# mainClock

스낵바가 애니메이션을 가지고 있기 때문에 "애니메이션으로 인해 발생한 문제이지 않을까?" 추측했다. 찾아보니 `Flutter`에서 tester를 bump 하듯이 안드로이드 compose에서도 `mainClock`을 이용하여 프레임을 밀리초 단위로 조종할 수 있었다.
일반적인 상황에서는 `autoAdvance`가 `true`로 되어 `mainClock`을 이용할 필요는 없다. 하지만 나의 경우 필요하다고 생각되어 [자동 동기화 사용중지](https://developer.android.com/jetpack/compose/testing#disable-autosync)과 [이슈 트래커에 존재하는 예제](https://issuetracker.google.com/issues/217880227#comment7)를 참고하여 직접 프레임을 조종하기로 했다.

아래가 그 결과이다. 먼저 `autoAdvance = false`를 통해 자동 동기화를 비활성화한다. 그리고 스낵바를 띄우는 버튼을 누르고 `advanceTimeByFrame`를 호출하여 리컴포지션을 유발한다.
그 후 `waitForIdle`를 통해 애니메이션 셋업을 기다리고 애니메이션 시작을 위해 다시 `advanceTimeByFrame`를 호출한다. 마지막으로 스낵바가 보이는 애니메이션이 대략 200밀리초이니 넉넉히 500밀리초 흘려보낸다.

```kotlin
@Test
fun showSnackBar_WhenClickButton() {
    with(composeRule) {
        setContent { SomeScreen() }

        mainClock.autoAdvance = false

        onNodeWithText("Button").performClick()

        mainClock.advanceTimeByFrame() // trigger recomposition
        waitForIdle() // await layout pass to set up animation
        mainClock.advanceTimeByFrame() // give animation a start time
        mainClock.advanceTimeBy(500)

        onNodeWithText("message").assertIsDisplayed()
    }
}
```

위의 테스트를 동일하게 300번 수행해보니 단 한번도 실패하지 않았다.

![after1](https://user-images.githubusercontent.com/57604817/183372712-42259d7a-aca0-4963-996d-2e018a974f9f.png)
![after2](https://user-images.githubusercontent.com/57604817/183372719-ba61adcc-576b-4451-bf1f-1f8ac626e2c9.png)
![after3](https://user-images.githubusercontent.com/57604817/183372724-c8c66f77-0590-45a6-a71b-7d0dd37c511a.png)

그런데 스낵바를 확인하는 테스트가 여러 곳에 존재했다. 반복되는 코드를 줄이기 위해 아래와 같이 extension을 만들어 사용했다.

```kotlin
fun ComposeTestRule.assertSnackBarIsDisplayed(
    before: () -> Unit,
    message: String
) {
    mainClock.autoAdvance = false

    before()

    mainClock.advanceTimeByFrame() // trigger recomposition
    waitForIdle() // await layout pass to set up animation
    mainClock.advanceTimeByFrame() // give animation a start time
    mainClock.advanceTimeBy(500)

    onNodeWithText(message).assertIsDisplayed()

    mainClock.autoAdvance = true
}
```

```kotlin
@Test
fun showSnackBar_WhenClickButton() {
    with(composeRule) {
        setContent { SomeScreen() }

        assertSnackBarIsDisplayed(
            before = {
                onNodeWithText("Button").performClick()
            },
            message = "message"
        )
    }
}
```

# 마치며

불안정한 테스트가 존재하면 수많은 시간이 낭비될 수 있으며 테스트에 대해 확신을 가지기 어려워진다. 하지만 다양한 변수가 존재하는 UI 테스트에서는 어쩔 수 없는 것 같다.
불안정한 테스트 코드를 해결할 방법이 도무지 없는 경우 [Barista의 @AllowFlaky를 이용하는 것도 하나의 방법](https://proandroiddev.com/managing-flaky-tests-in-jetpack-compose-89c598590068)이다.
