---
title: "[Android] color theme 설정하기(XML)"
date: 2022-07-10 19:20:00 +0900
categories: android
tags:
  - android
  - theme
  - material
---

앱을 개발할 때 일관성 있는 테마를 구축하는 것은 중요합니다. 제가 최근에 테마 설정을 하며 알아간 부분을 정리하고자 합니다.

# 컬러 코드 생성

가장 첫 번째 할일은 앱의 브랜딩 컬러 생성입니다. Material 팀에서 공식으로 지원하는 [Material theme builder](https://material-foundation.github.io/material-theme-builder/)에서 자동으로 색상을 생성할 수 있습니다. 이곳에서 이미지 혹은 특정 색상을 기준으로 색상을 생성하고 Export를 하시면 됩니다. 저는 Android Views(XML) 기준으로 설명하겠습니다. 해당 파일을 열어보면 다음과 같은 파일들이 존재합니다.

```
README.md
values/
values-night/
```

`values`에는 `themes.xml`과 `colors.xml`, `values-night`에는 `themes.xml`가 있습니다. 이 생성된 코드들을 이용하여 앱에 적용해보겠습니다.

# Theme 적용

먼저 기존에 있던 `colors.xml` 내부에 필요없는 색상을 전부 지웁니다. 그 후 방금 전에 다운받은 `colors.xml`에서 모든 컬러들을 다음과 같이 넣어줍니다.
코드를 보면 primary부터 background, surface등 모든 material color들이 있습니다.

_참고: Primary와 Secondary, Tertiary의 경우 브랜딩 컬러이며, onPrimary와 같이 on으로 시작하는 색상은 해당 컬러 위에 존재하는 아이템의 컬러를 말합니다._

**values/colors.xml**

```xml
<resources>
    <color name="seed">#6750A4</color>
    <color name="md_theme_light_primary">#375CA9</color>
    <color name="md_theme_light_onPrimary">#FFFFFF</color>
    <color name="md_theme_light_primaryContainer">#D9E2FF</color>
    <color name="md_theme_light_onPrimaryContainer">#001945</color>
    <color name="md_theme_light_secondary">#575E71</color>
    <color name="md_theme_light_onSecondary">#FFFFFF</color>

    ...

</resources>
```

그 다음 `themes.xml`을 적용해줍니다. 저는 2022년 기준 최신인 Material3의 테마를 상속하여 스타일을 만들었습니다.

**values/themes.xml**

```xml
<resources xmlns:tools="http://schemas.android.com/tools">
    <style name="Theme.YourApp" parent="Theme.Material3.DayNight.NoActionBar">
        <item name="colorPrimary">@color/md_theme_light_primary</item>
        <item name="colorOnPrimary">@color/md_theme_light_onPrimary</item>
        <item name="colorPrimaryContainer">@color/md_theme_light_primaryContainer</item>
        <item name="colorOnPrimaryContainer">@color/md_theme_light_onPrimaryContainer</item>
        <item name="colorSecondary">@color/md_theme_light_secondary</item>
        <item name="colorOnSecondary">@color/md_theme_light_onSecondary</item>
        <item name="colorSecondaryContainer">@color/md_theme_light_secondaryContainer</item>
        <item name="colorOnSecondaryContainer">@color/md_theme_light_onSecondaryContainer</item>

        ...

    </style>
</resources>
```

그 후 `values-night/themes.xml`도 마찬가지로 적용하여 어두운 테마를 적용할 수 있습니다.

_참고: 만약 api level로 인해 여러개의 theme파일이 있는 경우 `Base.Theme.YourApp`로 위의 기준 스타일을 만들고 다른 곳에서 상속하여 사용하시면 편리합니다._

# Theme 사용

## 정적 사용

테마를 적용했으니 이제 사용하면 됩니다. 색상을 하드코딩하여 사용하는 대신에 `?attr/...`를 이용하여 사용하는 것을 권장드립니다.
예를 들어 primary 컬러를 사용하는 경우 `?attr/colorPrimary`를 사용하시면 됩니다. 대부분 `?attr/color..`로 시작하며 예외적으로 background만 `?android:attr/colorBackground`입니다.

## 동적 사용

[MaterialColors.getColor](https://developer.android.com/reference/com/google/android/material/color/MaterialColors)를 사용하면 됩니다. `androidx.appcompat.R.attr.colorPrimary`와 같이 `AttrRes`를 전달하면 됩니다.

# 마무리

지금까지 기본적인 내용을 다루었습니다. 더 깊은 내용은 [Material design 3 - Color](https://m3.material.io/styles/color/overview)를 방문하여 확인할 수 있습니다.
