---
title: "[Flutter] CustomPaint를 터치 가능하게 해보자"
date: 2020-11-06 20:14:00 -0400
categories: flutter
tags:
- flutter
- touchable
- rect
- debug
---

[수면 데이터 차트](https://github.com/jja08111/Learning-Flutter/tree/main/Sleep-data-chart)에서 바를 터치하면 수면량을 표시하고 싶었다. 
구글링 결과 [Touchable](https://github.com/nateshmbhat/touchable)이라는 좋은 라이브러리가 있었다. 이는 [이곳](https://medium.com/flutter-community/bring-your-custompainter-paintings-to-life-in-flutter-using-touchable-c2413cd1897)에서 영어로 자세히 설명이 적혀있다. 

# 초간단 사용법 
아래와 같이 `CustomPaint`를 `CanvasTouchDetector`로 감싸주고 직접 정의한 `MyPainter`로 `context`를 넘기면 된다. 

```dart
CanvasTouchDetector(
  builder: (context) => CustomPaint(
    painter: MyPainter(context),
  ),        
)
```

그 후 직접 정의한 `MyPainter`내의 `paint`함수에서 `TouchyCanvas`를 정의하여 이용하면 된다.  
```dart
void paint(Canvas canvas, Size size) {
  TouchyCanvas touchyCanvas = TouchyCanvas(context, canvas);

  touchyCanvas.drawCircle( ... , onTapDown: (_) {
    print('You clicked BLUE circle');
  });

  touchyCanvas.drawCircle( ... , onLongPressStart: (_) {
    print('long pressed on GREEN circle');
  });
}
```
`onTapDown`외에도 다양한 것들이 있다. 

# 디버깅 
하지만 나는 바로 적용에 실패했다... 이유를 찾아보니 다음과 같았다.  

1. 나는 테두리가 둥근 `Bar`를 그리고 싶었고 따라서 `RRect`를 이용하여 `touchyCanvas.drawRRect`를 하려 했다. 
2. `RRect.fromLTRBR`로 초기화하는데 아래처럼 Left와 Right가 반대로 하고있었다. 막대는 제대로 출력 되어 계속 찾지 못했다.
```dart
// 잘못된 코드
RRect rect = RRect.fromLTRBR(right, top, left, bottom, Radius.circular(8.0));
// 옳은 코드 
RRect rect = RRect.fromLTRBR(left, top, right, bottom, Radius.circular(8.0));
```
3. 범위가 없다고 인식해서인지 터치 시 아무 반응이 없었다. 

코드를 고치고 나니 터치에 제대로 반응하고 있다.

# 예제 
차후에 직접 작성한 그래프를 가지고 돌아오겠다.
