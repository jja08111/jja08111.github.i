---
title: "[Flutter] Flutter 구조와 Navigator"
date: 2020-10-25 23:14:00 -0400
categories: flutter
tags:
- flutter
- navigator
- observer 
toc: false
---

# Widget
`flutter`는 거의 모든 것이 `Widget`으로 이루어져 있다. `Widget`은 구현하다 보면 굉장히 깊은 트리구조가 된다.  
하나의 루트에서 시작에 수 많은 자손을 가진 트리가 되는 것이다.

# Route 
`Navigator`를 이용하여 다른 페이지를 전환할 수 있다. 
이때 위에서 설명한 깊은 트리구조의 어느 자손에서 `pop`을 한다면 이 트리는 통째로 `pop`될 것이다. 
또한 새로운 위젯을 `push`하면 새로운 루트가 형성될 것이다. 현재 화면에는 최상단 스택이 표시된다. 

# Observer
`Observer`를 이용하면 어떤 값이 변경 될 때 `push`하여 스택이 추가되는 것이 아니라 분기점이 하나 더 생길 뿐이다.
그래서 메인 위젯까지 `Navigator.pop`을 하여 날려버리지 않도록 주의해야 한다. 
