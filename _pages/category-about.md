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
- 문제 해결의 즐거움을 느끼며 Android와 Flutter로 앱을 개발하고 있습니다.

### Contact

- Email | jja08111@gmail.com
- Github | [https://github.com/jja08111](https://github.com/jja08111)

## Projects

### POCS 블로그 앱

<img src="https://user-images.githubusercontent.com/57604817/207652827-40b6390d-0dd4-4bc0-b0b4-e49b115996ab.png" width="500">

<a href="https://github.com/hansung-pocs/blog-android"><img alt="GitHub" src="https://img.shields.io/badge/GitHub-181717.svg?&style=for-the-badge&logo=GitHub&logoColor=white"/>
<a href="https://play.google.com/store/apps/details?id=com.pocs.blog"><img alt="Playstore" src="https://img.shields.io/badge/Playstore-1d7c48.svg?&style=for-the-badge&logo=googleplay&logoColor=white"/>

> 개발 기간: 2022.07 ~ 2022.08 (2개월)

- 한성대학교 소모임 POCS를 위한 Android 커뮤니티 앱
- 안드로이드, 웹프론트, 벡엔드 총 10명이 함께 기획하고 개발한 프로젝트.
- 함께 성장 하기 위해 적극적으로 [코드리뷰](https://jja08111.github.io/develop/code-review/)를 진행.
- 앱에 필요한 [markdown-toolbar-compose](https://github.com/jja08111/markdown-toolbar-compose) 라이브러리를 개발하여 maven central에 배포
- 기술: `Android` `Kotlin` `Clean Architecture` `MVVM` `Hilt` `Dagger2` `Retrofit` `OkHttp3` `Compose` `ViewBinding` `JUnit4` `Esspresso` `GitHub Actions`

### 한성대 공지 앱

<img src="https://play-lh.googleusercontent.com/N1ie6859fXSZVP-iOc82OVXOXK5noIhR7pp0hWNPlfRo1qV_kXEvTVDvLV_M-0kMigQ=w416-h235" width="500">

<a href="https://github.com/jja08111/HansungNotification"><img alt="GitHub" src="https://img.shields.io/badge/GitHub-181717.svg?&style=for-the-badge&logo=GitHub&logoColor=white"/>
<a href="https://play.google.com/store/apps/details?id=com.foundy.hansungnotification"><img alt="Playstore" src="https://img.shields.io/badge/Playstore-1d7c48.svg?&style=for-the-badge&logo=googleplay&logoColor=white"/>

> 개발 기간: 2022.06 (2주)

- 모바일에서 학교 공지사항을 빠르고 쉽게 볼 수 있는 Android 앱
- Firebase push notification을 이용하여 키워드 알림 기능 구현
- 기술: `Android` `Kotlin` `Clean Architecture` `MVVM` `Hilt` `Dagger2` `Retrofit` `OkHttp3` `ViewBinding` `JUnit4` `Esspresso` `GitHub Actions` `Firebase`

### 뭐먹을까 앱

<img src="https://play-lh.googleusercontent.com/qGAAl6UdgAIPJTLys2tRZ1DWIatBqMyk-SJC8QMRd9IOT1KxAnUGgGAhmvVvGxS6VLM=w416-h235" width="500">

<a href="https://play.google.com/store/apps/details?id=com.foundy.what_should_i_eat"><img alt="Playstore" src="https://img.shields.io/badge/Playstore-1d7c48.svg?&style=for-the-badge&logo=googleplay&logoColor=white"/>

> 개발 기간: 2022.01 ~ 2022.02 (2개월)

- 친구들과 모였을 때 식사 메뉴를 정하는데 낭비되는 시간이 아까워 만들게 된 Flutter 앱
- **Firebase realtime database를 이용**하여 사용자 데이터 동기화 및 백업
- **Tensorflow lite를 이용**하여 Kakao API에서 얻어온 사진을 [음식, 음식 아님]으로 분류해본 경험
- 동시에 최대 16개의 이미지를 분류하는 작업 때문에 발생했던 **퍼포먼스 문제를 Isolate를 활용**하여 **10프레임 -> 50프레임 이상으로 개선**한 [경험](https://jja08111.github.io/flutter/flutter-tflite-with-isolate/)
- 적극적으로 **테스트 코드**를 작성했으나 의존성을 잘 관리하지 못하여 Fake 및 Mock을 만들기 어려워 테스트 코드 작성이 어려웠음
- CI/CD 적용(Github Action, Fastlane)
- 기술: `Flutter` `Dart` `Getx` `Firebase` `Tensorflow` `Kakao-API` `GitHub-Actions`

### 꿀밤(Bedtime) 앱

<img src="https://user-images.githubusercontent.com/57604817/117240536-fcbe5d80-ae6b-11eb-8a6f-788bb70ea558.png" width="500">

<a href="https://play.google.com/store/apps/details?id=io.github.jja08111.good_night_app"><img alt="Playstore" src="https://img.shields.io/badge/Playstore-1d7c48.svg?&style=for-the-badge&logo=googleplay&logoColor=white"/>

> 개발 기간: 2020.12 ~ 2021.03 (4개월) + 유지보수 4개월

- 수면 알람을 이용하여 규칙적인 수면 습관 형성을 도와주는 Flutter 앱
- 수면 체크리스트를 통해 더욱 깊은 수면을 도와줌
- **Android에서 동작하는 Flutter 알람 앱**을 개발한 [경험](https://jja08111.github.io/flutter/flutter-alarm-app/)
- 사용자가 요구한 기능(ex: 사용자 커스텀 알람 음악 기능, 수면 알람에서 취침 알람만 비활성화하는 기능 등)을 추가하며 **앱을 유지보수 한 경험**
- 기술: `Flutter` `Dart` `Provider` `Android` `SQLite` `Firebase`

## Open Source

### Markdown Toolbar Compose

![markdown-toolbar-preview](https://github.com/jja08111/markdown-toolbar-compose/raw/main/images/preview.gif)

<a href="https://github.com/jja08111/markdown-toolbar-compose"><img alt="GitHub" src="https://img.shields.io/badge/GitHub-181717.svg?&style=for-the-badge&logo=GitHub&logoColor=white"/>
<a href="https://maven-badges.herokuapp.com/maven-central/io.github.jja08111/markdown-toolbar-compose"><img alt="Maven" src="https://img.shields.io/badge/Maven-181717.svg?&style=for-the-badge"/>

- [동아리 앱](https://github.com/hansung-pocs/blog-android)을 개발하며 같이 개발
- Jetpack compose를 이용하여 개발

### Time Chart

![img](https://github.com/jja08111/time_chart/raw/main/assets/images/time_chart/weekly_time_chart.gif?raw=true)![img](https://github.com/jja08111/time_chart/raw/main/assets/images/time_chart/monthly_time_chart.gif?raw=true)

<a href="https://github.com/jja08111/time_chart"><img alt="GitHub" src="https://img.shields.io/badge/GitHub-181717.svg?&style=for-the-badge&logo=GitHub&logoColor=white"/>
<a href="https://pub.dev/packages/time_chart"><img alt="pub" src="https://img.shields.io/badge/pub-0175C2.svg?&style=for-the-badge&logo=dart&logoColor=white"/>

- 플러터 플러그인에서 시간을 나타내는 차트가 없어서 직접 개발
- 이분탐색을 활용해 1,000개의 데이터도 버거워하던 **차트 렌더링 성능을 10,000개 이상의 데이터도 무난히 수용할 수 있도록 개선한 경험**

## Activity

- 한성대학교 컴퓨터공학부 소모임 POCS 활동 ( 2022.06 ~ )

## Education

- 한성대학교 재학 ( 2018.03 ~ )
