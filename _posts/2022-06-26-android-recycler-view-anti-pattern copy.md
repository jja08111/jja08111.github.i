---
title: "[Android] RadioButton 묶음을 Recycler view로 구현하기"
date: 2022-06-24 10:20:00 -0400
categories: android
tags:
  - android
  - fragment
  - error_handling
---

# Do Not

어댑터를 ViewModel 클래스와 긴밀하게 결합하므로 ViewModel을 RecyclerView 어댑터에 전달하는 것은 좋지 않습니다.

https://developer.android.com/jetpack/guide/ui-layer/events?hl=ko

구글에서는 UiState를 이용하는 방식을 권장하고 있습니다.
