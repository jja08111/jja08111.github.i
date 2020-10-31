---
title: "[Flutter] Incorrect use of ParentDataWidget. 디버깅"
date: 2020-10-31 20:14:00 -0400
categories: flutter
tags:
- flutter
- error
- Expanded
toc: false
---

위젯을 형성하던 중 `Incorrect use of ParentDataWidget.`라는 에러를 얻었다. 
구글을 검색해 보니 [이곳](https://stackoverflow.com/questions/54905388/incorrect-use-of-parent-data-widget-expanded-widgets-must-be-placed-inside-flex)에 방법이 있었다.  
특별한 것은 아니었으며 내용은 아래와 같다. 

> `Expanded` 위젯은 `Column`,`Row`,`Flex` 위젯 내에서만 이용 가능하다. 

이를 이용하여 해결했다. 
