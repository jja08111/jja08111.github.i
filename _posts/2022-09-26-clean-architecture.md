---
title: "[Android] LiveData 내부 탐색하기"
date: 2022-11-02 10:12:00 +0900
categories: android
tags:
  - android
  - live_data
  - observer
---

# 코드 분석

LiveData 내부에 observe 함수를 보면
mObservers에 put함
lifecycleOwner의 lifecycle에 addObserver를 호출함.

Lifecycle 구현체인 LifecycleRegistry에서 `mObserverMap`에 observer를 put함

setValue에서 dispatchingValue를 호출, 여기서 considerNotify를 호출 -> `observer.mObserver.onChanged((T) mData);` 호출

### 라이프 사이클

LiveData observe 함수에 lifecycleOwner를 전달함

AppCompatActivity가 FragmentActivity를 상속
여기는 아래의 필드를 가짐
`final LifecycleRegistry mFragmentLifecycleRegistry = new LifecycleRegistry(this);`

라이프 사이클이 바뀔때마다 handleLifecycleEvent 호출 backwardPass or forwardPass -> `dispatchEvent` -> `onStateChanged`
이때마다 LifecycleBoundObserver에서 상태마다 제거 혹은 활성화 비활성화하는 것을 볼수있음

```java
@Override
public void onStateChanged(@NonNull LifecycleOwner source,
        @NonNull Lifecycle.Event event) {
    Lifecycle.State currentState = mOwner.getLifecycle().getCurrentState();
    if (currentState == DESTROYED) {
        removeObserver(mObserver);
        return;
    }
    Lifecycle.State prevState = null;
    while (prevState != currentState) {
        prevState = currentState;
        activeStateChanged(shouldBeActive());
        currentState = mOwner.getLifecycle().getCurrentState();
    }
}
```

# 그럼 Flow는?
