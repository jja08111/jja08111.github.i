---
title: "[Android] RadioButton 묶음을 Recycler view로 구현하기"
date: 2022-06-24 10:20:00 -0400
categories: android
tags:
  - android
  - fragment
  - error_handling
---

인터넷이 연결되어있지 않을 때 문제가 있다는 프레그먼트를 사용자에게 보이도록 하려한다. 좋은 방법은 무엇일까?

첫 시도 -> 에러 발생시 프레그먼트를 `replace()`를 이용하여 통째로 교체 -> 너무 복잡함

나은 방법 찾아봄 -> `View`의 `visibility`속성을 이용하여 평상시에는 에러 프레그먼트를 `gone`으로 두고 에러 발생시 `visible`로 변경하는 방식 적용!

그러면 메모리에 불필요한 부분 차지하지 않나? 궁금쓰

추가로 footer를 적용
