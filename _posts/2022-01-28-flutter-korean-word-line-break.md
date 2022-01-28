---
title: "[Flutter] 한글 어절 단위 줄바꿈"
date: 2022-01-28 012:50:00 -0400
categories: flutter
tags:
  - flutter
  - korean
---

Flutter에서 한글은 영어처럼 어절 단위로 줄바꿈이 되지 않는다. 문제를 해결하기 위해 찾아보니 [word_break_text](https://pub.dev/packages/word_break_text)라는 패키지가 존재했다. 굉장히 간단한 방법으로 문제를 해결하고 있었다.

하지만 이 패키지에도 문제가 존재했다. 그 문제는 어절의 너비가 주어진 너비 제약보다 큰 경우 부자연스럽게 줄바뀜이 진행되는 것이었다. 그 이유는 `Wrap`을 사용하는데 어절 단위로만 문장을 쪼개고 있기 때문이었다. 아래와 같은 사진이 해당 문제이다.

![문제](/assets/images/korean_word_before)

이 문제를 해결하기 위해 먼저 `LayoutBuilder`로 주어진 제약 너비를 구하고 `TextPainter`를 통해 빌드 전에 텍스트의 너비를 구했다. 그 후 어절마다 너비를 초과하는지 확인하며 초과한다면 초과하지 않게 어절을 쪼갠다. 쪼갤 위치를 찾는 것은 이분탐색을 이용했다.(어절이 보통 크지 않아 의미있는 방법 같지는 않다.) 해결된 후에는 아래와 같이 나온다.

![해결](/assets/images/korean_word_after)

이 내용은 [PR](https://github.com/ChangJoo-Park/word_break_text/pull/1)로 개시했다.
