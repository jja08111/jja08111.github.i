---
title: "[Flutter] int to double"
date: 2020-10-31 20:14:00 -0400
categories: flutter
tags:
- flutter
- convert
toc: false
---

`int`형을 `double`형으로 변경하는 방법이다. 아주 간단하다. 
아래와 같이 원하고자 하는 변수 혹은 값에 .toDouble()을 붙여주면 된다. 
```dart
int a=3;
int b=2;
double c = a.toDouble() / b.toDouble();
```
