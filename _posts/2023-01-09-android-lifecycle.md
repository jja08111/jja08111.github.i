---
title: "[Android] Activity Lifecycle"
date: 2023-01-09 22:20:00 -0400
categories: android
tags:
  - android
  - activity
  - fragment
  - lifecycle
---

안드로이드의 Activity에는 수명 주기가 존재한다.
수명 주기를 이해하지 못한다면 앱의 상태에 문제가 생기고 버그를 만들 수 있다.
지금부터 안드로이드에 존재하는 수명 주기들을 파헤쳐보자.

# Activity

먼저 엑티비티이다. 엑티비티의 수명 주기를 잘못 다룬다면 아래와 같은 문제들이 발생할 수 있다.

- 사용자가 앱을 사용하는 도중에 전화가 걸려오거나 다른 앱으로 전환할 때 비정상 종료되는 문제
- 사용자가 앱을 활발하게 사용하지 않을 때에도 시스템 리소스를 낭비하는 문제
- 사용자가 앱을 나갔다가 나중에 다시 돌아올 때 이전 상태가 사라지는 문제
- 화면을 회전하거나 어두운 테마로 전환했을 때 비정상 종료되거나 상태가 사라지는 문제

엑티비티 클래스에는 아래의 사진과 같이 콜백 함수들이 존재한다.
엑티비티는 새로운 상태에 들어가면 시스템은 각 콜백을 호출한다.

![activity_lifecycle](https://user-images.githubusercontent.com/57604817/211316998-565af4fd-3fc6-4476-8a4f-ef4a7133f858.png)

## onCreate()

이 콜백은 시스템이 먼저 엑티비티를 생성할 때 실행되는 것으로, 필수적으로 구현해야 한다. 엑티비티가 생성되면 `Created` 상태가 된다.
이곳에서 엑티비티의 전체 수명 주기 동안 한 번만 발생해야 하는 **기본 앱 시작 로직을 실행한다.**
예를 들어 바인딩을 하거나 `ViewModel`과 연결하는 등 클래스 멤버 변수를 인스턴스화 할 수 있다.
이 메소드의 매개변수에는 `savedInstanceState`가 존재하는데, 이는 엑티비티의 이전 상태가 포함된 `Bundle` 객체이다.

엑티비티의 수명 주기와 연결된 수명 주기 인식 구성요소가 있다면 이 구성 요소는 `ON_CREATE` 이벤트를 수신한다.

엑티비티는 생성된 후 곧바로 `onStart()`와 `onResume()`을 호출한다.

## onStart()

엑티비티는 `Started` 상태에 들어가면 시스템은 이 콜백을 호출한다.
이것이 호출되면 엑티비티가 사용자에게 표시되고, 앱은 엑티비티를 포그라운드에 보내 사용자와 상호작용 할 수 있도록 **준비**한다.
보통 이 메소드에서 앱이 UI를 관리하는 코드를 초기화한다.

엑티비티가 시작됨 상태로 전환하면 이 엑티비티의 수명 주기와 연결된 모든 수명 주기 인식 구성요소는 `ON_START` 이벤트를 수신한다.

이 메소드는 매우 빠르게 완료되고 `Resumed` 상태로 들어가며, 곧바로 `onResume()` 메소드가 실행된다.

## onResume()

엑티비티가 `Resumed` 상태에 들어가면 포그라운드에 표시되고 시스템이 `onResume()` 콜백을 호출한다.
이 상태에 들어갔을 때 **앱이 사용자와 상호작용한다.** 어떤 이벤트가 발생하여 앱에서 포커스가 떠날 때까지 앱이 **이 상태에 머무른다.**

엑티비티가 재개됨 상태로 전환되면 이 엑티비티의 수명 주기와 연결된 모든 수명 주기 인식 구성요소는 `ON_RESUME` 이벤트를 수신한다. 이 상태에서 카메라 미리보기 시작과 같은 기능을 수행할 수 있다.

이 상태에서 방해되는 이벤트가 발생하면 엑티비티는 `Paused` 상태에 들어가고, 시스템이 `onPasue()` 콜백을 호출한다.

엑티비티가 `Paused` 상태에서 `Resumed` 상태로 돌아오면 시스템은 `onResume()` 메서드를 다시 한 번 호출한다. 따라서 `onResume()`을 구현하여 `onPause()` 중에 해제했던 구성요소를 초기화하는 등의 작업을 해야한다.

## onPause()

이 콜백은 시스템이 사용자가 엑티비티를 떠나는 첫 번째 신호로써 호출된다.
이는 엑티비티가 포그라운드에 있지 않게 되었다는 것을 나타낸다.
(다만, 멀티 윈도우 모드에서는 예외이다.)

엑티비티가 `Paused` 상태에 들어서는 이유는 여러 가지가 있다.

- 일부 이벤트가 앱 실행을 방해하는 경우
- 새로운 반투명 활동(예: 대화 상자)가 열릴 때
- 멀티 윈도우 모드에서 포커스를 갖지 않을 때

엑티비티가 `Paused` 상태로 전환하면 이 활동의 수명 주기와 연결된 모든 수명주기 인식 구성요소는 `ON_PAUSE` 이벤트를 수신한다. 여기에서 카메라 미리보기 정지와 같은 포그라운드가 아닐 때 실행할 필요가 없는 기능을 모두 정지 할 수 있다.

`onPause()`는 아주 잠깐 실행된다. 따라서 이곳에서 데이터를 저장하거나, 네트워크 호출을 하거나, DB 트랜잭션을 실행해서는 **안된다.** 그 대신, 부하가 큰 종료 작업 `onStop()`에서 실행하여야 한다.

사용자에게 완전히 보이지 않을 때까지 이 상태에 머무른다. 엑티비티가 다시 시작되면 시스템은 다시 `onResume()`을 호출한다.

## onStop()

엑티비티가 **사용자에게 표시되지 않으면 `Stopped` 상태**에 들어간다.
예를 들어 새로 시작된 엑티비티가 화면 전체를 차지하는 경우이다.

활동이 중단됨 상태로 전환하면 이 활동의 수명 주기와 연결된 모든 수명 주기 인식 구성요소는 `ON_STOP` 이벤트를 수신한다.

이 메소드에서는 앱이 사용자에게 보이지 않는 동안 필요 없는 리소스를 해제하거나 조정해야 한다.
예를 들어 애니메이션을 일시중지하거나 세밀한 위치 업데이트에서 대략적인 위치 업데이트로 전환할 수 있다.

또한 이 메소드를 사용하여 CPU를 비교적 많이 소모하는 종료 작업을 실행해야 한다. 예를 들어 작성 중인 초안 정보를 DB에 저장하는 작업이 적절하다.

엑티비티가 `Stopped` 상태에 들어가면 `Activity` 객체는 메모리 안에 머무르게 된다. 그리고 이 상태에 있을 때 시스템이 해당 엑티비티가 포함된 프로세스를 소멸시킬 수 있다.

엑티비티는 `Stopped` 상태에서 다시 시작되어 사용자와 상호작용을 하거나 실행을 종료하고 사라진다. 엑티비티가 다시 시작되면 시스템은 `onRestart()`를 호출한다. Activity가 실행을 종료하면 시스템은 `onDestroy()`를 호출한다.

## onDestroy()

이 메소드는 엑티비티가 소멸되기 전에 호출된다. 예를 들어 아래와 같은 경우이다.

- 사용자가 엑티비티를 닫거나 엑티비티에서 `finish()`가 호출되어 엑티비티가 종료되는 경우
- 구성 변경(예: 기기 회전 혹은 멀티 윈도우 모드)으로 인해 시스템이 일시적으로 활동을 소멸시키는 경우

엑티비티가 `Destroyed` 상태로 전환하면 이 엑티비티의 수명 주기와 연결된 모든 수명 주기 인식 구성요소는 `ON_DESTROY` 이벤트를 수신한다.

엑티비티가 종료되는 경우 `onDestroy()`는 엑티비티가 수신하는 마지막 수명 주기 콜백이다.
구성 변경으로 인해 `onDestroy()`가 호출되는 경우 시스템은 즉시 새 엑티비티 인스턴스를 생성한 다음, 그 새로운 인스턴스의 `onCreate()`를 새로운 구성에서 호출한다.

# 엑티비티 전환시 흐름

엑티비티 A에서 엑티비티 B를 실행하는 경우 발생하는 작업 순서는 아래와 같다.

1. A의 `onPause()` 메소드 실행됨
2. B의 `onCreate()`, `onStart()`, `onResume()` 메소드가 순차적으로 실행됨 -> 이제 포커스는 B에 있음
3. 그런 다음, A가 더이상 화면에 표시되지 않는 경우 A의 `onStop()` 메소드가 실행됨

# 수명 주기 인식 구성요소

흔히 엑티비티에 수명 주기 메소드에 특정 작업들을 반복적으로 구현한다.
이는 동일한 코드가 반복적으로 작성되고 오류가 발생할 위험이 크다.
수명 주기 인식 구성요소를 사용하면, 수명 주기 메소드에서 구성요소 자체로 종속 구성요소의 코드를 옮길 수 있다.

예를 들어 안좋은 구현은 아래와 같다. 매번 `myLocationListener`를 사용할 때마다 `start()`, `stop()`이 반복적으로 호출될 것이다.

```kotlin
internal class MyLocationListener(
        private val context: Context,
        private val callback: (Location) -> Unit
) {

    fun start() {
        // connect to system location service
    }

    fun stop() {
        // disconnect from system location service
    }
}

class MyActivity : AppCompatActivity() {
    private lateinit var myLocationListener: MyLocationListener

    override fun onCreate(...) {
        myLocationListener = MyLocationListener(this) { location ->
            // update UI
        }
    }

    public override fun onStart() {
        super.onStart()
        myLocationListener.start()
        // manage other components that need to respond
        // to the activity lifecycle
    }

    public override fun onStop() {
        super.onStop()
        myLocationListener.stop()
        // manage other components that need to respond
        // to the activity lifecycle
    }
}
```

이는 아래와 같이 개선할 수 있다.

```kotlin
class MyObserver : DefaultLifecycleObserver {
    override fun onStart(owner: LifecycleOwner) {
        // ...
    }

    override fun onStop(owner: LifecycleOwner) {
        // ...
    }
}

myLifecycleOwner.getLifecycle().addObserver(MyObserver())
```
