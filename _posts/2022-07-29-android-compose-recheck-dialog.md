---
title: "[Android] 뒤로가기 할 때 AlertDialog 띄우기 in Compose"
date: 2022-07-29 20:00:00 +0900
categories: android
tags:
  - android
  - compose
---

어느 화면에서 뒤로가기 버튼을 눌렀을 때 다이어로그를 띄워 한 번 더 확인하는 것을 compose에서 구현하고 싶었습니다. 구현한 결과물을 먼저 보이겠습니다.

![result](https://camo.githubusercontent.com/df94efe7668dea52e33e31febc43ad189f551b1ae11c2ba9d489999c8a0a38fe/68747470733a2f2f696d352e657a6769662e636f6d2f746d702f657a6769662d352d306539636432353136342e676966)

이는 사용자가 게시글을 수정하는 등의 작업을 할 때 실수로 뒤로가기 버튼을 눌러 작성한 내용을 잃지 않도록 하기 위한 방법입니다.

이를 위해서는 [BackHandler](https://sungbin.land/jetpack-compose-뒤로가기-이벤트-처리하기-69cbc47268ea)가 필요합니다. 이 컴포저블 함수는 콜백을 등록하여 뒤로가기 액션 대신 콜백을 실행할 수 있게 해줍니다.

이를 활용하여 `RecheckHandler`라는 새로운 컴포저블을 만들어 다양한 곳에서 재활용할 수 있도록 하겠습니다. 앞서 설명한 `BackHandler`를 이용하여 아래와 같이 구현합니다.

```kotlin
@Composable
fun RecheckHandler(
    enableRechecking: Boolean = true
) {
    BackHandler(enableRechecking) {
        ...
    }
}
```

<br>

이제 뒤로가기 버튼을 누르면 다이어로그를 보여야 합니다. Compose에서는 상태를 이용하여 다이어로그를 띄우면 됩니다. 적용하면 아래와 같습니다.

```kotlin
@Composable
fun RecheckHandler(
    enableRechecking: Boolean = true
) {
    var enabledAlertDialog by remember { mutableStateOf(false) }

    if (enabledAlertDialog) {
        RecheckDialog(
            onDismissRequest = { enabledAlertDialog = false },
            onOkClick = { /* TODO implement */ }
        )
    }

    BackHandler(enableRechecking) {
        enabledAlertDialog = true
    }
}
```

<br>

이제 확인 버튼을 누른 경우를 처리하면 됩니다. 이때 `LocalOnBackPressedDispatcherOwner`를 이용한다면 `BackHandler`의 콜백이 호출되기 때문에 다른 방법을 이용해야 합니다.
저는 아래와 같이 `navigateUp`을 매개변수로 전달받을 수 있게 하여 상황에 맞게 뒤로 갈 수 있도록 했습니다. 아래가 구현 완성본입니다.

```kotlin
@Composable
fun RecheckHandler(
    navigateUp: () -> Unit,
    enableRechecking: Boolean = true
) {
    var enabledAlertDialog by remember { mutableStateOf(false) }

    if (enabledAlertDialog) {
        RecheckDialog(
            onDismissRequest = { enabledAlertDialog = false },
            onOkClick = { navigateUp() }
        )
    }

    BackHandler(enableRechecking) {
        enabledAlertDialog = true
    }
}
```

<br>

# 사용법

아래와 같이 해당 컴포저블 함수 내에 위치시면 됩니다.

```kotlin
@Composable
fun SomeScreen(
    navigateUp: () -> Unit,
    ...
) {
    RecheckHandler(
        navigateUp = navigateUp,
        enableRechecking = someValue
    )

    Scaffold() {
      ...
    }
}
```

<br>

## Activity finish 이용

다이어로그의 확인 버튼을 눌렀을 때 Activity를 끝내야 할 경우 다음과 같이 `Activity`의 `finish` 함수를 전달하면 됩니다.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    ...

    setContent {
        SomeScreen(navigateUp = ::finish, ...)
    }
}
```

<br>

# Compose의 NavController 이용

간단히 `NavController`의 `navigateUp`를 전달하면 됩니다.

```kotlin
SomeScreen(navigateUp = navController::navigateUp, ...)
```
