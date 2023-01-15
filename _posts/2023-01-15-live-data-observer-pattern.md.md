---
 title: "[Android] LiveData 내부 탐색하기"
 date: 2023-01-15 10:12:00 +0900
 categories: android
 tags:
   - android
   - live_data
   - observer_pattern
   - design_pattern
---

학교 디자인 패턴 강의에서 *observer pattern*을 배웠다. 안드로이드에서 흔히 사용되는 패턴이라 실제로 어떻게 구현되어있는지 궁금해졌다. lifecycle-livedata 2.5.1 버전 모듈에 있는 `LiveData`를 분석해보겠다.

# 코드 분석

`LiveData` 내부에 observe 함수를 보면 `LifecycleOwner`와 `Observer`를 매개변수로 받는다. `Observer`는 아래와 같은 인터페이스이다.

```java
public interface Observer<T> {
    void onChanged(T t);
}
```

함수 내부를 보면 `mObservers`라는 멤버 변수에 observer를 키로 하고 wapper를 값으로 하여 넣고 있다.

```java
@MainThread
public void observe(
    @NonNull LifecycleOwner owner,
    @NonNull Observer<? super T> observer
) {
    assertMainThread("observe");
    if (owner.getLifecycle().getCurrentState() == DESTROYED) {
        return;
    }
    LifecycleBoundObserver wrapper = new LifecycleBoundObserver(owner, observer);
    ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);

    // ...
}
```

그렇다면 `setValue` 함수를 살펴보자. 값을 설정하고 `dispatchingValue` 함수를 호출한다.

```java
@MainThread
protected void setValue(T value) {
    assertMainThread("setValue");
    mVersion++;
    mData = value;
    dispatchingValue(null);
}
```

`dispatchingValue` 함수를 보면 mObservers의 모든 항목들을 순회하며 `considerNotify` 함수를 호출한다.

```java
void dispatchingValue(@Nullable ObserverWrapper initiator) {
    if (mDispatchingValue) {
        mDispatchInvalidated = true;
        return;
    }
    mDispatchingValue = true;
    do {
        mDispatchInvalidated = false;
        if (initiator != null) {
            considerNotify(initiator);
            initiator = null;
        } else {
            for (Iterator<Map.Entry<Observer<? super T>, ObserverWrapper>> iterator =
                    mObservers.iteratorWithAdditions(); iterator.hasNext(); ) {
                considerNotify(iterator.next().getValue());
                if (mDispatchInvalidated) {
                    break;
                }
            }
        }
    } while (mDispatchInvalidated);
    mDispatchingValue = false;
}
```

`considerNotify` 함수를 보면 `observer`가 활성화 되어있을 때 `onChanged` 함수를 호출하여 옵저버들에게 변경되었음을 알린다.

```java
private void considerNotify(ObserverWrapper observer) {
    if (!observer.mActive) {
        return;
    }
    if (!observer.shouldBeActive()) {
        observer.activeStateChanged(false);
        return;
    }
    if (observer.mLastVersion >= mVersion) {
        return;
    }
    observer.mLastVersion = mVersion;
    observer.mObserver.onChanged((T) mData);
}
```

이들을 클래스 다이어그램으로 간략하게 나타내면 아래와 같다.

![diagram](/assets/images/live_data_class_diagram.png)

# 라이프 사이클

그런데 `LiveData`는 안드로이드 컴포넌트의 라이프사이클에 맞게 비활성화하는 기능을 제공한다. 예를 들어 엑티비티가 파괴된 경우 옵저버 콜백을 호출하지 않는 것이다.

아까 위에서 보았던 `observe` 함수 내부를 다시 보면 `observer`를 `wrapper` 클래스로 감싼 후 `lifecycleOwner`의 `lifecycle`으로 `addObserver`를 호출한다.

```java
@MainThread
public void observe(
    @NonNull LifecycleOwner owner,
    @NonNull Observer<? super T> observer
) {
    // ...

    owner.getLifecycle().addObserver(wrapper);
}
```

`addObserver`의 구현을 보기 위해 `Lifecycle` 추상 클래스의 구현체인 `LifecycleRegistry`를 보자. `mObserverMap`에 `observer`를 키로하고 `statefulObserver`를 put한다.

```java
@Override
public void addObserver(@NonNull LifecycleObserver observer) {
    enforceMainThreadIfNeeded("addObserver");
    State initialState = mState == DESTROYED ? DESTROYED : INITIALIZED;
    ObserverWithState statefulObserver = new ObserverWithState(observer, initialState);
    ObserverWithState previous = mObserverMap.putIfAbsent(observer, statefulObserver);

    // ...
}
```

그렇다면 이 `LifecycleRegistry`를 어디서 가지고 있을까? `AppCompatActivity`가 `FragmentActivity`를 상속하며 이곳에서 아래의 필드를 가진다.

```java
final LifecycleRegistry mFragmentLifecycleRegistry = new LifecycleRegistry(this);
```

`FragmentActivity`를 보면 엑티비티 라이프 사이클 콜백이 호출될 때마다 아래와 같이 `handleLifecycleEvent`를 호출한다.

```java
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    mFragmentLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_CREATE);
    // ...
}
```

`LifecycleRegistry`의 `handleLifecycleEvent`는 다시 `moveToState`를 호출한다.

```java
public void handleLifecycleEvent(@NonNull Lifecycle.Event event) {
    enforceMainThreadIfNeeded("handleLifecycleEvent");
    moveToState(event.getTargetState());
}
```

`moveToState` 함수를 보면 새로운 상태로 변경하고 `sync` 함수를 호출한다.

```java
private void moveToState(State next) {
    if (mState == next) {
        return;
    }
    if (mState == INITIALIZED && next == DESTROYED) {
        throw new IllegalStateException("no event down from " + mState);
    }
    mState = next;
    if (mHandlingEvent || mAddingObserverCounter != 0) {
        mNewEventOccurred = true;
        // we will figure out what to do on upper level.
        return;
    }
    mHandlingEvent = true;
    sync();
    mHandlingEvent = false;
    if (mState == DESTROYED) {
        mObserverMap = new FastSafeIterableMap<>();
    }
}
```

`sync` 함수에서는 옵저버의 상태를 확인하여 싱크가 맞지 않다면 `backwardPass` 혹은 `forwardPass`를 호출한다.

```java
private void sync() {
    LifecycleOwner lifecycleOwner = mLifecycleOwner.get();
    if (lifecycleOwner == null) {
        throw new IllegalStateException("LifecycleOwner of this LifecycleRegistry is already"
                + "garbage collected. It is too late to change lifecycle state.");
    }
    while (!isSynced()) {
        mNewEventOccurred = false;
        if (mState.compareTo(mObserverMap.eldest().getValue().mState) < 0) {
            backwardPass(lifecycleOwner);
        }
        Map.Entry<LifecycleObserver, ObserverWithState> newest = mObserverMap.newest();
        if (!mNewEventOccurred && newest != null
                && mState.compareTo(newest.getValue().mState) > 0) {
            forwardPass(lifecycleOwner);
        }
    }
    mNewEventOccurred = false;
}
```

`forwardPass`를 보면 이때마다 `dispatchEvent`를 호출한다.

```java
 private void forwardPass(LifecycleOwner lifecycleOwner) {
    Iterator<Map.Entry<LifecycleObserver, ObserverWithState>> ascendingIterator =
            mObserverMap.iteratorWithAdditions();
    while (ascendingIterator.hasNext() && !mNewEventOccurred) {
        Map.Entry<LifecycleObserver, ObserverWithState> entry = ascendingIterator.next();
        ObserverWithState observer = entry.getValue();
        while ((observer.mState.compareTo(mState) < 0 && !mNewEventOccurred
                && mObserverMap.contains(entry.getKey()))) {
            pushParentState(observer.mState);
            final Event event = Event.upFrom(observer.mState);
            if (event == null) {
                throw new IllegalStateException("no event up from " + observer.mState);
            }
            observer.dispatchEvent(lifecycleOwner, event);
            popParentState();
        }
    }
}
```

`dispatchEvent` 함수는 다시 `LifecycleEventObserver.onStateChanged` 함수를 호출한다.

```java
static class ObserverWithState {
    State mState;
    LifecycleEventObserver mLifecycleObserver;

    ObserverWithState(LifecycleObserver observer, State initialState) {
        mLifecycleObserver = Lifecycling.lifecycleEventObserver(observer);
        mState = initialState;
    }

    void dispatchEvent(LifecycleOwner owner, Event event) {
        State newState = event.getTargetState();
        mState = min(mState, newState);
        mLifecycleObserver.onStateChanged(owner, event);
        mState = newState;
    }
}
```

드디어 `LifecycleEventObserver`의 구현체인 `LifecycleBoundObserver`를 보면, `DESTROYED` 상태마다 `LiveData.removeObserver`를 호출하는 것을 볼 수 있다!

```java
@Override
public void onStateChanged(
    @NonNull LifecycleOwner source,
    @NonNull Lifecycle.Event event
) {
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

# 마무리

강의 때 간략하게나마 구현해봤던 observer pattern이 프레임워크에서 어떻게 구현되고 이용되는지 확인할 수 있었다.
