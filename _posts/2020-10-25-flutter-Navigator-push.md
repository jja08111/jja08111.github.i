---
title: "[Flutter] Flutter 구조와 Navigator"
date: 2020-10-25 23:14:00 -0400
categories: flutter
tags:
- flutter
- navigator
toc: false
---

# 수정예정.

`flutter`의 `Widget`은 계층구조로 이루어져 있다. 그리고 이의 깊이는 사용하다 보면 굉장히 깊어진다.  

`Navigator`를 이용하여 스택에 화면을 쌓을 수 있다. 
가장 최상단에 있는 스택이 현재 화면에 나타나게 된다. 

만약 `Navigator.push`를 하게 된다면 스택을 하나 더 쌓게 되는 것이다. 

그런데 아주 깊어진 위젯트리의 리프에서 `Navigator.pop`을 하게 된다면 이 위젯 트리는 통째로 날라간다. 

`Observer`를 이용시 새로운 화면, 즉 스택이 추가되는 것이 아니라 분기점이 하나 더 생길 뿐이다.
그래서 메인 위젯을 날려버리지 않도록 주의해야 한다. 
