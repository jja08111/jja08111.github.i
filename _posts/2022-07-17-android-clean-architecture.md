---
title: "[Android] Clean Architecture"
date: 2022-07-18 09:00:00 +0900
categories: android
tags:
  - architecture
  - clean_architecture
  - android
---

최근 [한성대 공지 어플](https://github.com/jja08111/HansungNotification)을 개발하면서 공부한 [Clean Architecture](http://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)에 대해 정리하고자 한다.

아키텍처에 관심을 가지게 된 이유는 기존에 앱을 개발하면서 느낀 점이 있기 때문이다. 처음으로 만들었던 [꿀밤 앱](https://play.google.com/store/apps/details?id=io.github.jja08111.good_night_app&hl=ko&gl=US)이 점점 거대해지고 복잡성은 늘어나며 유지보수가 어려워지고 중구난방의 의존성 때문에 테스트하기도 어려워졌다. 그러던 중 클린 아키텍처로 구현된 [아냥이 어플](https://github.com/juhwankim-dev/pushNotificationApp)을 발견하여 _"이거다!!"_ 싶어 새로 개발하는 앱에 적용하게 되었다.(아냥이 앱은 모듈을 나누지는 않았다. 개인적으로 진행한 프로젝트는 모듈까지 나누어 관리했다.)

시작하기 전 핵심을 말하자면 클린 아키텍처는 **관심사의 분리**를 통해 UI, DB, Framework간의 결합도를 낮추고 유지 보수하기 좋으며, 테스트하기 용이한 아키텍처이다.

# 종속성 규칙

먼저 아래의 다이어그램을 보자

![](https://user-images.githubusercontent.com/57604817/179393181-8521a33b-a344-4f12-afb5-7cb3e77c44f2.png)

큰 원과 내부의 작은 원들이 있다. 그리고 큰 원이 작은 원을 가르킨다. 이 화살표는 의존성을 말한다. 즉 바깥의 원이 내부의 원을 의존한다는 이야기이다. 반대로 말하면 **내부의 원은 그 어떤 바깥의 원을 알지 못한다.** 이 부분이 핵심이다.

내부의 원은 추상화된 규칙들이 정의되며 외부로 갈 수록 구체적인 구현과 세부사항, 프레임 워크들이 존재한다.
이러한 종속성 규칙은 다음과 같은 장점을 준다.

- 테스트하기 좋다. 비즈니스 규칙을 UI, DB, 웹 서버 또는 기타 외부 요소 없이 테스트 할 수 있다.
- UI 변경이 수월하다. UI를 시스템의 나머지 부분을 변경하지 않고도 쉽게 수정할 수 있다. 예를 들어 비즈니스 규칙은 그대로 두고 기존 뷰를 컴포즈로 변경하기 수월하다. 왜냐하면 비즈니스 규칙은 UI에 대해 전혀 알지 못하기 때문이다.
- DB에 독립적이다. Oracle 또는 SQL Server를 Mongo와 같은 다른 것으로 교체할 수 있다. 비즈니스 규칙은 DB에 의존하지 않기 때문이다.

이를 안드로이드에서 `:domain`, `:data`, `:presentation` 모듈로 나누어 아래와 같이 나타낼 수 있다.

![](https://camo.githubusercontent.com/ff6f7705da1debdd88d7b5ebf0b95fc53fce0dbb091b56a6293d696252d9c84d/68747470733a2f2f7777772e636861726c657a7a2e636f6d2f776f726470726573732f77702d636f6e74656e742f75706c6f6164732f323032312f30392f7777772e636861726c657a7a2e636f6d2d636c65616e6172636869746563747572652d636972636c652e706e67)

_(이미지 출처: [찰스의 안드로이드](https://www.charlezz.com/?p=45391))_

# :domain 모듈

domain 모듈에는 앱의 핵심 비즈니스 규칙을 담고 있으며 Entity와 Usecase가 존재한다. 먼저 Entity에 대해 말해보겠다. 클린 아키텍처에서의 엔터티는 전사적 차원의 비즈니스 규칙을 캡슐화 한 것이다. 엔터티는 메소드가 있는 개체이거나 데이터 구조 및 함수 집합일 수 있다. 안드로이드에서 예를 든다면 Repository interface와 모델을 정의하는 data class를 말할 수 있다.

엔터티는 가장 일반적이고 높은 수준의 규칙을 캡슐화한다. 이 엔터티가 수정된다면 응용프로그램의 모든 부분이 수정될 것이다. 하지만 엔터티 외부의 것이 수정된다고 하여도 엔터티는 변경될 가능성이 적다.

이번에는 Usecase이다. 이 계층은 시스템의 모든 **사용 사례**를 캡슐화하고 구현한다. 여기서 Entity를 사용하여 Usecase의 목적을 달성한다.
DB, UI가 수정된다고 하여도 이 계층에 영향을 미치지 않을 것이다. 왜냐면 마찬가지로 외부 계층에 대해 알지 못하기 때문이다.

여기까지가 안드로이드에서 `domain` 모듈에 속한다. 이 모듈의 `build.gradle`에는 순수 kotlin을 제외한 어떠한 의존성도 존재하여서는 안된다. 예를 들어 안드로이드의 `LiveData`가 있어서는 안된다(그래서 클린 아키텍처에서는 `StateFlow`를 사용하라고 한다).

# :data 모듈

data 모듈은 DB, 서버와 같이 데이터 소스를 다루는 코드가 존재한다. 예를 들면 retrofit을 이용하여 API와 통신한 뒤 데이터를 받아 적절히 변환하여 데이터를 반환한다.

그런데 곧 설명할 `:presentation` 모듈에서 `:data` 모듈을 직접 참조하지 않고 어떻게 접근이 가능한 것일까? 이에 대한 답은 앞서 보인 모듈 의존 그림을 통해 알 수 있다. 바로 DIP(의존성 역전 원칙)을 이용하는 것이다.
간단히 설명하자면 `:domain` 모듈에서 인터페이스를 만들고 `:data` 모듈에서는 해당 인터페이스를 구현한 객체를 만들면 되는 것이다. 그리고 `:presentation` 모듈에서는 Hilt등을 활용하여 `:domain` 모듈에 위치한 UseCase를 의존성 주입을 통해 얻으면 된다.

# :presentation 모듈

presentation 모듈은 UI와 관련된 코드를 다룬다. ViewModel에서는 View에 대한 어떠한 참조도 있으면 안된다. ViewModel은 UseCase를 통해 얻은 데이터를 View에서 바로 이용할 수 있도록 만드는 로직을 가지고 있다. View는 최대한 단순하게 UI 관련된 코드만 담당해야한다. 이는 클린 아키텍처와 MVVM을 결합한 것과 동일하다.

# 모델

각 모듈마다 사용하는 모델들이 따로 존재하는 것이 좋다. User를 예로 들면 presentation에서는 `UserUiState`, domain에서는 `User`, data에서는 `UserDto`와 같다. 각 계층간에 데이터가 이동해야 하는 경우 Mapper를 이용하여 해당 모델로 변환이 필요하다.

# 마치며

아키텍처를 신경쓰지 않는다면 어느정도 규모가 커질 때 어느순간 모래성처럼 와르르 무너질 수 있다. 아키텍처를 고민하여 확장성이 뛰어나고 유지보수 하기 좋은 앱을 만들자.

# 참조

- http://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html
- https://techblog.woowahan.com/2647/
- https://www.charlezz.com/?p=45391
- https://k-elon.tistory.com/38
