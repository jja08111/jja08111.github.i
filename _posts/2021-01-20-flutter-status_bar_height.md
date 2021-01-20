---
title: "[Flutter] MediaQuery.of(context).padding.top return 0.0"
date: 2021-01-20 18:00:00 -0400
categories: flutter
tags:
- flutter
- media_uery
- status_bar
---

# Problem
I wanted to get status bar height. So I used `MediaQuery.of(context).padding.top`, but it returns `0.0`.  
상단의 상태바 높이를 구하려고 `MediaQuery.of(context).padding.top`를 이용했으나 계속 `0.0`을 반환하였습니다.

# Cause
When there is `SafeArea` in the ancestor widget, the padding is disapear.  
`SafeArea`가 조상 위젯에 존재하여, 즉 `SafeArea`로 감쌌기 때문에 padding이 사라졌던 것입니다.

# Solve 
solve by removing `SafeArea`.  
`SafeArea`를 제거하여 해결했습니다.
