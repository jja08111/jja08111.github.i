---
title: "[Android] API 키 숨기기"
date: 2022-07-28 09:00:00 +0900
categories: android
tags:
  - android
---

동아리에서 벡엔드를 직접 구축하며 앱을 개발하고 있다. 앱에서는 벡엔드 팀에게서 전달받은 API 키를 이용한다.
하지만 이 API 키가 온라인에 노출되는 경우 누군가 우리 서버를 작동하지 못하게 엄청난 부하를 거는 등 심각한 문제가 발생할 수 있다.
그렇다면 어떻게 API 키를 노출하지 않고 로컬에서만 이용할 수 있을까? 방법은 다음과 같다.

# local.properties 파일 수정

이 파일은 안드로이드 스튜디오에서 자동으로 생성되며 `.gitignore`에 명시된 파일이다. 이것을 이용할 것이다(따로 `apikey.properties`와 같이 생성해서 이용해도 상관없다).
해당 파일에 아래와 같이 API 키 값을 추가해주자.

```properties
api_key="https://111.123.111/"
```

# build.gradle 수정

`local.properties`에 명시한 API 키 값을 앱 내에서 이용하기 위해서는 앱 수준의 `build.gradle`을 수정해야한다. 아래와 같이 수정하면 앱 빌드시 build config에 필드가 추가된다.

```gradle
def localPropertiesFile = rootProject.file("local.properties")
def localProperties = new Properties()
localProperties.load(new FileInputStream(localPropertiesFile))

android {

    defaultConfig {

        ...

        buildConfigField("String", "API_KEY", localProperties['api_key'])
    }
}
```

# 사용법

사용법은 간단하다. 앱을 빌드하면 다음과 같은 클래스가 생성되며 `BuildConfig.API_KEY`와 같이 이용하면 된다.

```java
public final class BuildConfig {
  ...
  // Field from default config.
  public static final String API_KEY = "https://123.111.123/";
}
```

```kotlin
const val BASE_URL = BuildConfig.API_KEY
```

# Github Action에서는 어떻게?

만약 CI 시스템으로 Github Actioin을 사용하고 있는 경우 별다른 조치를 하지 않는다면 빌드와 테스트에 실패할 것이다. Action 환경에서는 `local.properties`가 없기 때문이다.

해결 방법은 다음과 같다.

- Github Repository의 Setting으로 들어가 [Secret에 API_KEY를 등록한다.](https://docs.github.com/en/actions/security-guides/encrypted-secrets#creating-encrypted-secrets-for-a-repository)
- 기존에 사용하던 workflow 파일에 API_KEY에 엑세스하는 작업을 추가해준다. `echo` 명령어로 마지막 줄에 텍스트를 추가하고 있다.

```yml
jobs:
  build:
    ...

    steps:

      ...

      - name: Access API_KEY
        env:
          API_KEY: $
        run: echo API_KEY=\"API_KEY\" > ./local.properties

      Do build, testing...
```

# 참조

- [Storing secret keys in Android](https://guides.codepath.com/android/Storing-Secret-Keys-in-Android)
- [Accessing Android app secret from Github actions using gradle](https://blog.jakelee.co.uk/accessing-android-app-secret-from-github-actions-using-gradle/)
