---
title: "저를 소개합니다."
permalink: /about/
layout: single
share: false
author_profile: true
taxonomy: about
# excerpt: "**Minseong Kim**"
header:
  overlay_image: /assets/images/algorithm-bg.jpg
  overlay_filter: 0.2
  caption: "Photo credit: [**Unsplash**](https://unsplash.com)"
---

## About Me

### Introduction

- 안녕하세요 김민성입니다.
- 문제 해결의 즐거움을 느끼며 최근에는 Flutter와 Android를 공부하고 있습니다.

### Contact

- Email | jja08111@gmail.com
- Github | [https://github.com/jja08111](https://github.com/jja08111)

## Projects

### [한성대 공지 App](https://play.google.com/store/apps/details?id=com.foundy.hansungnotification)

<img src="https://play-lh.googleusercontent.com/N1ie6859fXSZVP-iOc82OVXOXK5noIhR7pp0hWNPlfRo1qV_kXEvTVDvLV_M-0kMigQ=w416-h235" width="500">

- 모바일에서 학교 공지사항을 편하게 볼 수 있는 Android 앱
- 키워드를 등록하여 공지 알림을 받을 수 있음
- [Clean Architecture](https://jja08111.github.io/android/android-clean-architecture/), MVVM 패턴 적용
- [CI 구축(App build, Unit test, Espresso test)](https://jja08111.github.io/android/android-ci-with-github-action/)
- [Repository Link](https://github.com/jja08111/HansungNotification)

### [뭐먹을까 App](https://play.google.com/store/apps/details?id=com.foundy.what_should_i_eat)

<img src="https://play-lh.googleusercontent.com/qGAAl6UdgAIPJTLys2tRZ1DWIatBqMyk-SJC8QMRd9IOT1KxAnUGgGAhmvVvGxS6VLM=w416-h235" width="500">

- 친구들과 모였을 때 식사 메뉴를 정하는데 낭비되는 시간이 아까워 만들게 된 Flutter 앱
- 적극적으로 **테스트 코드**를 작성 (100여개)
- **Firebase database를 이용**하여 사용자 데이터 동기화 및 백업
- CI/CD 적용(Github Action, Fastlane)
- **Tensorflow lite를 이용**하여 Naver API에서 얻어온 사진을 [음식, 음식 아님]으로 분류해본 경험
- 동시에 최대 16개의 이미지를 분류하는 작업 때문에 발생했던 **퍼포먼스 문제를 Isolate를 활용**하여 **10프레임 -> 50프레임 이상으로 개선**한 [경험](https://jja08111.github.io/flutter/flutter-tflite-with-isolate/)

### [꿀밤(Bedtime) App](https://play.google.com/store/apps/details?id=io.github.jja08111.good_night_app)

<img src="https://user-images.githubusercontent.com/57604817/117240536-fcbe5d80-ae6b-11eb-8a6f-788bb70ea558.png" width="500">

- 수면 알람을 이용하여 규칙적인 수면 습관 형성을 도와주는 Flutter 앱
- 수면 체크리스트를 통해 더욱 깊은 수면을 도와줌
- **Android에서 동작하는 Flutter 알람 앱**을 개발한 [경험](https://jja08111.github.io/flutter/flutter-alarm-app/)

## Open Source

### [markdown-toolbar-compose](https://github.com/jja08111/markdown-toolbar-compose)

![markdown-toolbar-preview](https://github.com/jja08111/markdown-toolbar-compose/raw/main/images/preview.gif)

- [동아리 앱](https://github.com/hansung-pocs/blog-android)을 개발하며 같이 개발
- Jetpack compose를 이용하여 개발

### [time_chart](https://pub.dev/packages/time_chart)

![img](https://github.com/jja08111/time_chart/raw/main/assets/images/time_chart/weekly_time_chart.gif?raw=true)![img](https://github.com/jja08111/time_chart/raw/main/assets/images/time_chart/monthly_time_chart.gif?raw=true)

- 플러터 플러그인에서 시간을 나타내는 차트가 없어서 직접 개발
- 이분탐색을 활용해 1,000개의 데이터도 버거워하던 **차트 렌더링 성능을 10,000개 이상의 데이터도 무난히 수용할 수 있도록 개선한 경험**

## Education

- 한성대학교 재학 ( 2018.03 ~ )
