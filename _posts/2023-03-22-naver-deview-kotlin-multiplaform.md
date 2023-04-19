---
 title: "[Deview] Kotlin Multiplatform 적용기 요약"
 date: 2023-03-22 10:12:00 +0900
 categories: kotlin
 tags:
   - kotlin
   - kotlin_multiplatform
---

졸업 작품에 적용할까 말까 고민했던 코들린 멀티 플랫폼의 적용기를 다룬 [네이버 deview 영상](https://www.youtube.com/watch?v=B27Yu9uQvqY)이 올라와서 시청하였다.
주요 부분을 요약하여 정리하고자 한다.

# Multiplatform을 고려하게된 이유

개발자들이 흔히 Multiplatform(크로스 플랫폼이라고도 불리우는 그것)을 환상이라고 표현한다. 다양한 플랫폼에서 동작하는 앱들을 하나의 코드 베이스로 구현하여 개발 속도를 빠르게 하고 유지보수 하기 편리한 그런 세상이다.
하지만 현실을 녹록치 않다. 네이티브 기능을 지원하지 않는 경우가 꽤 있고...

네이버에서는 60개의 서비스가 PRISM Player를 사용한다고 한다. 서비스가 많아질수록, 스펙이 많아질수록, 지원할 플랫폼이 많아질수록 개발자는 더 필요하다.
한 명의 개발자만 필요한 것이 아니라 플랫폼 개수만큼 개발자가 필요하다.

개발자가 많아지면 각 플랫폼의 개발자마다의 구현 능력이 다르다. 결과적으로 개발자가 많아지면 커뮤니케이션이 증가하고 이슈는 플랫폼 별로 나오는 힘든 부분들이 생긴다.
결국 개발자가 많아지는 것만이 좋은 해결책은 아니다.

그래서 네이버에서는 다양한 플랫폼과 디바이스를 지원하는 동영상 서비스를 위해 멀티 플랫폼을 적용했다고 한다.

# 왜 Kotlin Multiplatform인가?

초반에는 안드로이드 개발자들의 익숙함을 이유로 Kotlin Multiplatform를 생각했다고 한다.
그리고 Android, iOS, Web, Google cast, TV 등을 잘 지원하기 때문에 채택하였다고 한다.

그리고 Android Studio에서 대부분의 개발들이 잘 되었다고 한다.

또한 코틀린은 Kotlin/JVM 외에도 Kotlin/JS, Kotlin/Native를 지원한다. 따라서 멀티 플랫폼에 너무 의존하지 않기 때문에 실무에서도 적용 할 수 있었다고 한다.

# 적용

## 기존 프로젝트를 마이그레이션

- 1안. 기존 프로젝트의 기능 일부를 Kotlin Multiplatform으로 전환
  - 리스크 적음
  - 플랫폼 별로 다른 설계 및 스펙으로 이를 맞추는데 상당한 리소스 소요
- 2안(채택). 신규 Kotlin Multiplatform 프로젝트에 기존 기능 도입
  - 차라리 각 플랫폼의 노하우를 취합해 신규 설계하는 것이 나음
  - 호환성 이슈로 개선하지 못했던 API 개선 가능

이미 Kotlin으로 작성된 Android 코드를 많이 참고했다고 한다. 그리고 공유 코드를 극대화하기 위해 pure Kotlin을 많이 사용했다고 한다. 어쩔수 없는 경우는 아래와 같이 `expect/actual` 키워드를 활용하여 각 플랫폼 별로 코드를 작성했다고 한다.

```kotlin
// commonMain
expect fun getPlatform(): String

// androidMain
actual fun getPlatform(): String = "Android ${Build.VERSION.RELEASE}"

// iosMain
actual fun getPlatform(): String = UIDevice.currentDevice.run { "$systemName $systemVersion" }

// jsMain
actual fun getPlatform(): String = window.navigator.userAgent.toPlatform()
private fun String.toPlatform(): String = ...
```

## Kotlin/JS

Kotlin/JS 덕분에 여러 웹 브라우저는 물론이고 TV, TIZEN, Google Cast 플랫폼으로 확장할 수 있다.

external modifier를 통한 Javascript 코드를 접근할 수 있다.

```kotlin
public external interface Console {
  // ...
}

public external val console: Console
```

그리고 `js` 함수로 Javascript 코드를 직접 호출할 수 있다고 한다.

```kotlin
val shakaPlayer = js("new shaka.Player(element)")
```

또한 NPM 의존성을 사용할 수 있다. Dukat을 사용하면 자동으로 external 코드를 생성해준다.

제대로 지원되지 않는 문자열 디코딩을 위해 직접 구현했다고 한다. 이는 SDK에서는 라이브러리를 선택할 때 신중해야 했기 때문이라고 한다. 왜냐하면 여러 플랫폼을 지원해야 했고 버전 충돌 등으로 예기치 못한 이슈가 종종 발생하기 때문이다.

## 테스트 코드

하나의 테스트 코드로 **모든 플랫폼에서 테스트**를 진행할 수 있다.

## Swift가 아닌 Objective-C

Kotlin/Native는 Swift가 아닌 Objective-C로 컴파일된다. sealed class나 default argument를 지원하지 않아 불편하다.

네이버는 sealed class를 IceRock의 MOKO KSwift 플러그인의 도움을 받았다고 한다.
