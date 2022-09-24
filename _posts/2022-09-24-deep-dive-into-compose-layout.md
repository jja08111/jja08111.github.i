---
title: "Deep dive into Jetpack Compose layouts"
date: 2022-09-24 21:12:00 +0900
categories: android
tags:
  - android
  - compose
  - jetpack
  - layout
---

안드로이드의 새로운 UI 툴킷인 Jetpack Compose의 레이아웃을 깊이 탐구해보자.

# Layout Model

Compose의 Layout을 알아보기 전에 Compose는 어떻게 State를 UI로 변환하는지 알아보자. Compose가 State를 UI로 변환하는 과정은 아래와 같이 3단계로 구성된다.

1. Composition
2. Layout
3. Drawing

Composition 단계에서는 UI를 내보내(emit) 아래와 같은 UI트리를 만든다.

```console
SearchResult
  Row
    Image
    Column
      Text
      Text
```

Layout 단계에서는 트리로부터 UI의 각 부분을 측정하고 2D 영역의 화면에 배치한다.
즉, 각 노드가 너비와 높이를 결정하고 x, y 좌표를 알아낸다.

Drawing 단계에서는 UI를 렌더링한다.

이제 Layout에 자세히 알아보겠다. Layout 단계는 측정(Measure), 배치(Place) 단계로 나뉜다. 각 노드는 각 하위 요소(children)를 측정하고 자신의 크기를 결정하여 하위 요소(children)을 배치해야 한다. 요약하자면 아래의 순서이다.

1. Measure children
2. Decide own size
3. Place children

![layout_example](https://user-images.githubusercontent.com/57604817/192106388-c7138666-4ebf-495c-82f9-b240e63cf1f5.png)

앞서 본 트리로 과정을 설명 해보겠다. 가장 루트인 Row를 첫 번째로 측정한다. 그 후 자식인 Image를 측정한다. Image는 자식이 없으므로 스스로 측정해서 크기를 보고한다. 모델에서는 하위요소를 측정하는 배치 명령을 반환한다. 이를 모든 하위 요소의 측정이 끝날때 까지 위의 사진의 순서대로 반복한다.

모든 요소의 크기가 측정되면 다시 트리가 작동한다. 배치(place) 단계에서 모든 배치 명령이 실행되는 것이다.

![low_level_layout](https://user-images.githubusercontent.com/57604817/192106394-18c48a7c-7572-48e0-8d4c-2957eeab3cd9.png)

트리의 상위 수준의 요소 뿐만 아니라 하위 요소까지 나타낸다면 위의 사진과 같다. Layout 컴포저블이 하나씩 위치한 것을 볼 수 있다. 이 Layout은 LayoutNode를 내보낸다(emit).

# Layout composable

Layout 컴포저블의 작동 원리를 살펴보자.

```kotlin
@Composable
fun Layout(
  content: @Composable () -> Unit,
  modifier: Modifier = Modifier,
  measurePolicy: MeasurePolicy
) {
  ...
}
```

`Layout` 함수는 `content`, `modifier`, `measurePolicy` 세 개의 매개변수가 존재한다. 여기서 중요한 것은 `measurePolicy`이다. 레이아웃은 이것을 이용하여 항목을 측정하고 배치한다.

아래와 같은 커스텀 함수를 보며 이해해보자.

```kotlin
@Composable
fun MyCustomLayout(
  modifier: Modifier = Modifier,
  content: @Composable () -> Unit
) {
  Layout(
    modifier = modifier,
    content = content
  ) { measurables: List<Measurable>,
      constraints: Constraints ->
    ...
  }
}
```

`MyCustomLayout` 함수는 `Layout` 앞서 보인 `measurePolicy`를 후행 람다로 제공하여 필요한 측정 함수를 구현한다.

이 함수는 `constraints` 객체를 전달받아 레이아웃에 크기를 알려준다. `Constraints` 클래스는 너비와 높이의 최댓값과 최솟값을 모델링한다. `Constraints`의 최소 크기와 최대 크기에 제한을 걸지 않아 원하는 크기로 두거나, 제한을 두어 정확한 크기를 지정할 수 있다.

`measurables`는 측정 가능한 입력된 하위 요소를 나타낸다.

## 구현 방법

```kotlin
@Composable
fun MyCustomLayout(
  modifier: Modifier = Modifier,
  content: @Composable () -> Unit
) {
  Layout(
    modifier = modifier,
    content = content
  ) { measurables: List<Measurable>,
      constraints: Constraints ->
    val placeables = measurables.map { measurable ->
      measurable.measure(constraints)
    }
  }
}
```

먼저 하위 요소를 측정한다. 각각의 `measurable`들에 제약을 전달하여 측정한다. 측정을 하면 `placeable`을 반환한다. `placeable`은 측정된 하위 요소이고 크기가 결정되어 있다.

```kotlin
@Composable
fun MyCustomLayout(
  modifier: Modifier = Modifier,
  content: @Composable () -> Unit
) {
  Layout(
    modifier = modifier,
    content = content
  ) { measurables: List<Measurable>,
      constraints: Constraints ->
    val placeables = measurables.map { measurable ->
      measurable.measure(constraints)
    }
    val width =  // calculate from placeables
    val height = // calculate from placeables
    layout(width, height) {
      // TODO place items
    }
  }
}
```

`placeable`을 사용해서 레이아웃의 크기를 계산한 다음 레이아웃의 크기가 얼마인지 `layout` 메서드를 호출하여 크기를 보고한다.

```kotlin
layout(width, height) {
  placeables.forEach { placeable ->
    placeable.place(...)
  }
}
```

layout의 `placementBlock`은 일반적으로 후행 람다로 구현하여 각 항목을 원하는 곳에 배치한다.

# Layout Examples

작성 중...
