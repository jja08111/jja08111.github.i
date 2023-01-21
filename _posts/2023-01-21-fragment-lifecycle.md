---
 title: "[Android] Fragment Lifecycle"
 date: 2023-01-21 10:12:00 +0900
 categories: android
 tags:
   - android
   - fragment
   - lifecycle
---

프레그먼트는 생명주기를 가지고 있다. 생명주기를 다루기 위해 프레그먼트는 `Lifecycle`을 노출하며 `LifecycleOwner`를 implements 한다.

`Lifecycle` 상태에는 아래의 종류가 존재한다.

- `INITIALIZED`
- `CREATED`
- `STARTED`
- `RESUMED`
- `DESTROYED`

프레그먼트도 엑티비티와 마찬가지로 [수명주기 인식 구성 요소](https://developer.android.com/topic/libraries/architecture/lifecycle)를 이용할 수 있다. 다른 방법으로는 `LifecycleOberver`를 이용할 수 있다.

프레그먼트의 뷰는 각 프레그먼트의 생명주기에 의해 독립적으로 관리되는 생명 주기를 갖고 있다. 프레그먼트는 그들의 뷰를 위해 `LifecycleOwner`를 관리한다.

# 프레그먼트와 프레그먼트 매니저

프레그먼트가 인스턴스화되면 프레그먼트는 `INITIALIZED` 상태가 된다.
프레그먼트의 전환을 하기 위해서는 프레그먼트는 `FragmentManager`에 추가가 되어야한다. `FragmentManager`는 프레그먼트가 어느 상태에 있어야 하는지 그리고 그것들의 상태를 바꿀 책임이 있다.

그리고 `FragmentManager`는 프레그먼트의 호스트 엑티비티에 프레그먼트를 attach, detach 할 책임이 있다. 프레그먼트에는 `onAttach()`, `onDetach()` 콜백 메소드가 존재한다.

주의할 점은 프레그먼트가 프레그먼트 매니저에게서 제거되고 나서 프레그먼트의 인스턴스를 재사용하면 안된다는 것이다.

# 프레그먼트 생명주기와 콜백함수

프레그먼트의 생명주기는 프레그먼트 매니저의 상태를 넘어서 진행될 수 없다.
그리고 프레그먼트는 자신의 부모보다 상태가 넘어설 수 없다. 예를 들어 부모 혹은 엑티비티는 자식의 시작보다 빨라야한다.

그래서 XML에서 `<fragment>` 사용을 피해야한다. `<fragment>`는 이들의 상태를 `FragmentManager`의 상태보다 넘어설 수 있게 허용하기 때문이다. 대신 `FragmentContainerView`를 이용해야한다.

그럼 이제부터 각각의 상태마다 무슨 일이 벌어지는지 살펴보겠다.

## Upward

프레그먼트가 백스택에 추가되는 경우 `CREATED`, `STARTED`, `RESUMED` 순으로 상태가 변경된다.

### CREATED

프레그먼트가 `CREATED` 상태가 되었다면 프레그먼트는 `FragmentManager`에 추가되고 `onAttach()` 메소드가 이미 호출되었다.

이곳에서는 프레그먼트와 관련된 상태를 복구하기에 적절하다. 참고로 이때 **프레그먼트의 뷰는 생성되지 않는다**. 따라서 프레그먼트의 뷰와 관련된 복구 작업은 뷰가 생성되고 나서 진행되어야 한다.

### CREATED and View INITIALIZED

프레그먼트의 뷰 생명주기는 프레그먼트가 유효한 뷰 인스턴스를 반환하면 생성된다. 흔히 프레그먼트의 생성자로 `@LayoutId`를 전달하여 적절한 시기에 뷰를 inflate 하도록 한다. 아니면 `onCreateView()`를 구현할 수도 있다.

이때가 뷰와 관련된 초기화 작업을 진행하기 좋다. 예를 들어 리사이클러뷰의 어뎁터를 초기화 하거나 `LiveData`의 인스턴스를 관찰하는 등의 작업이 있다.

### Fragment and View CREATED

프레그먼트의 뷰가 생성되면 이전 뷰 상태가 존재한다고 했을 때 그것이 복구되며 뷰의 `Lifecycle`의 상태는 `CREATED`가 된다.

### Fragment and View STARTED

프레그먼트의 `STARTED` 상태일 때 생명주기 인식 구성요소를 연결하는것을 강력히 권장한다. 그 이유는 `STARTED` 상태일 때 프레그먼트의 뷰는 사용가능하고, 프레그먼트 자식에서 `FragmentTracsaction`을 수행하기 안전하기 때문이다.

프레그먼트의 뷰가 널이 아니라면 프레그먼트의 상태가 `STARTED`가 된 직후에 프레그먼트의 뷰 상태가 `STARTED`가 된다. 프레그먼트가 `STARTED` 상태가 되면 `onStart()` 콜백이 호출된다.

### Fragment and View RESUMED

프레그먼트가 보이기 시작하면 모든 `Animator`와 `Transition` 효과가 완료된다. 그리고 프레그먼트는 사용자와 상호작용할 준비가 된다. 프레그먼트의 생명주기는 `RESUMED` 상태가 되고 이때 `onResume()` 콜백이 호출된다.

## Downward

프레그먼트가 백스택에서 제거될 때 프레그먼트의 상태는 `RESUMED`, `STARTED`, `CREATED` 그리고 `DESTROYED`로 변한다.

### Fragment and View STARTED

사용자가 프레그먼트를 떠나기 시작하였으나 아직 프레그먼트가 보일 때, 프레그먼트와 이것의 뷰의 생명주기는 `STARTED` 상태가 된다. 이때 `onPause()` 콜백이 호출된다.

### Fragment and View CREATED

더이상 프레그먼트가 보이지 않게 되면 프레그먼트와 이것의 뷰의 생명주기는 `CREATED` 상태가 된다. 이때 `onStop()` 콜백이 호출된다. 이 상태는 부모 엑티비티 혹은 프레그먼트가 정지되었을 때 발생할 뿐만 아니라, 부모 엑티비티 혹은 프레그먼트의 상태 저장으로 인해 발생된다. 이는 `ON_STOP` 이벤트가 프레그먼트의 상태를 저장하기 **이전**에 발생한다는 것을 보장한다. 이때는 자식 `FragmentManager`에서 `FragmentTrasaction`을 수행하기 안전한 마지막 지점이다.

### Fragment CREATED and View DESTROYED

모든 종료 에니메이션과 전환이 완료되면 프레그먼트의 뷰는 윈도우에서 떨어지고 뷰의 생명주기는 `DESTROYED` 상태가 된다. 이때 프레그먼트의 `onDestroyView()`가 호출된다.

이 시점에 모든 프레그먼트의 뷰에 대한 참조는 제거되고 뷰는 GC에 의해 수집될 수 있게 된다.

### Fragment DESTROYED

프레그먼트가 제거되거나 `FragmentManager`가 파괴되면 프레그먼트의 생명주기 상태는 `DESTROYED`가 된다. 이때 `onDestroy()` 콜백이 호출된다.

# 참조

- [Android - Fragment lifecycle](https://developer.android.com/guide/fragments/lifecycle)
