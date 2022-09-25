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

Jetpack Compose의 레이아웃을 깊이 탐구해보자.

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

직접 Column을 구현하며 Layout을 이해해보자.

```kotlin
@Composable
fun MyColumn(
  modifier: Modifier = Modifier,
  content: @Composable () -> Unit
) {
  Layout(
    modifier = modifier,
    content = content
  ) { measurables, constraints ->
    val placeables = measurables.map { measurable ->
      measurable.measure(constraints)
    }
    val height = placeables.sumOf { it.height }
    val width = placeables.maxOf { it.width }
    layout(width, height) {
      var y = 0
      placeables.forEach { placeable ->
        placeable.placeRelative(x = 0, y = y)
        y += placeable.height
      }
    }
  }
}
```

항목의 높이를 더한 값을 높이로 설정하고 가장 넓은 너비를 너비 값으로 설정한다. 그 후 width와 height를 보고하고 y값을 늘려주며 각 항목들을 배치한다.

이번에는 예시로 Grid를 보겠다.

```kotlin
@Composable
fun VerticalGrid(
  modifier: Modifier = Modifier,
  columns: Int = 2,
  content: @Composable () -> Unit
) {
  Layout(
    content = content,
    modifier = modifier
  ) { measurables, constraints ->
    val itemWidth = constraints.maxWidth / columns
    // Keep given height constraints, but set an exact width
    val itemConstraints = constraints.copy(
      minWdith = itemWidth,
      maxWidth = itemWidth
    )
    // Measure each item with these constraints
    val placeables = measurables.map { it.measure(itemConstraints) }
    ...
  }
}
```

각 열은 레이아웃의 최대 너비를 일정한 비율로 나눈다. 구해진 값을 이용하여 새로운 constraints 값을 얻는다. 높이의 제약은 유지하되 정확한 너비를 지정한다.
그 후 이 제약을 적용해 각 항목을 측정하고 그리드에 배치한다.

상위와 하위 요소 간의 협상은 없다. 상위 요소는 허용 가능한 크기를 전달하고 이를 제약으로 표현한다. 하위 요소가 그중에서 크기를 선택하면 상위 요소는 이를 받아서 처리해야 한다.

이러한 디자인에는 싱글 패스로 UI 트리 전체를 측정하고 여러 번의 측정 사이클을 돌리지 않아도 되는 장점이 있다. 기존 View에서는 이게 문제였다. 여러 번 측정값을 입력하는 중첩된 계층 구조에서는 리프 뷰에서 측정값의 2차적 값이 발생하기도 했다.
Compose는 설계부터 이를 차단했다.

다음으로 [Jetsnack 샘플 앱](https://github.com/android/compose-samples/tree/main/Jetsnack)에 존재하는 [BottomNavItem](https://github.com/android/compose-samples/blob/5bc3451b11f9aae111b8a6dca0199a317c997a2b/Jetsnack/app/src/main/java/com/example/jetsnack/ui/home/Home.kt#L278-L317)을 살펴보자.

```kotlin
@Composable
fun BottomNavItem(
  icon: @Composable BoxScope.() -> Unit,
  text: @Composable BoxScope.() -> Unit,
  @FloatRange(from = 0.0, to = 1.0) animationProgress: Float
) {
  Layout(
    content = {
      Box(
        modifier = Modifier.layoutId("icon"),
        content = icon
      )
      Box(
        modifier = Modifier.layoutId("text"),
        content = text
      )
    }
  ) { measurables, constraints ->
    val iconPlaceable = measurables.first { it.layoutId == "icon" }.measure(constraints)
    val textPlaceable = measurables.first { it.layoutId == "text" }.measure(constraints)

    placeTextAndIcon(
        textPlaceable,
        iconPlaceable,
        constraints.maxWidth,
        constraints.maxHeight,
        animationProgress
    )
  }
}
```

BottomNavItem는 animationProgress가 0.0이면 아이콘만 렌더링하고 1.0인 경우에는 레이블까지 렌더링한다. Custom Layout은 measure블록에서 measurable을 식별하기 위해서 layoutId를 지정한다. 이는 순서 정렬에 의지하는 것보다 훨씬 안정적이다.

```kotlin
private fun MeasureScope.placeTextAndIcon(
  textPlaceable: Placeable,
  iconPlaceable: Placeable,
  width: Int,
  height: Int,
  @FloatRange(from = 0.0, to = 1.0) animationProgress: Float
): MeasureResult {
  val iconY = (height - iconPlaceable.height) / 2
  val textY = (height - textPlaceable.height) / 2

  val textWidth = textPlaceable.width * animationProgress
  val iconX = (width - textWidth - iconPlaceable.width) / 2
  val textX = iconX + iconPlaceable.width

  return layout(width, height) {
    iconPlaceable.placeRelative(iconX.toInt(), iconY)
    if (animationProgress != 0f) {
      textPlaceable.placeRelative(textX.toInt(), textY)
    }
  }
}
```

`placeTextAndIcon`의 경우 코드 가독성을 위해 따로 함수를 만들었다. 수학적 계산을 통해 애니메이션 진행 값에 따라 텍스트와 아이콘을 배치한다.

Compose의 레이아웃은 성능이 뛰어나 측정이나 배치에 애니메이션을 적용하거나 제스처로 실행할 수 있다. View 시스템에서는 성능 문제로 인해 레이아웃 애니메이션을 권장하지 않아서 구현이 어려웠다.
