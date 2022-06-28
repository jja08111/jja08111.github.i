---
title: "[Android] Github action으로 CI 구현하기"
date: 2022-06-27 20:20:00 -0400
categories: android
tags:
  - android
  - ci
---

하루종일 씨름하다 드디어 Android CI를 구축했습니다. 정리할 겸 글을 씁니다.
[이곳](https://github.com/jja08111/HansungNotification/actions/runs/2573543283)에서 실제 구동한 흔적을 보실 수 있습니다.

제가 CI에서 진행하는 작업은 다음과 같습니다. 테스트들은 다양한 API level에서 병렬적으로 수행하도록 했습니다.

- Gradle build
- Unit test
- UI test

# 구현

먼저 `.github/workflows/ci.yml`과 같이 파일을 만듭니다. 저는 main 브런치에서만 수행하기 위해 앞 부분에 다음과 같이 작성했습니다.

```yaml
name: Android CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

  ...
```

그 후 빌드를 진행하는 job을 다음과 같이 만들었습니다. JDK를 셋업하고 `./gradlew build` 명령을 수행하고 있습니다.

```yaml

 ...

jobs:
  build:
    name: Development build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup JDK 11
        uses: actions/setup-java@v3
        with:
          java-version: "11"
          distribution: "temurin"
          cache: gradle

      - name: Build with Gradle
        run: ./gradlew build
```

그 다음 테스트를 위한 job을 만들었습니다. 다양한 API level에서 수행하도록 했습니다. 유닛 테스트는 `./gradlew test --stacktrace`으로 수행합니다.
UI 테스트의 경우 `reactivecircus/android-emulator-runner`를 이용하여 에뮬레이터를 구동시킨 후 `./gradlew connectedCheck` 명령으로 진행합니다.

```yaml

  ...
{% raw %}
test:
  name: Tests on Android (API level ${{ matrix.api-level }})
  runs-on: macos-latest
  strategy:
    matrix:
      api-level: [23, 29, 30, 31]
  steps:
    - name: Checkout
      uses: actions/checkout@v3
    - name: Setup JDK 11
      uses: actions/setup-java@v3
      with:
        java-version: "11"
        distribution: "temurin"
        cache: gradle
    - name: Setup Android SDK
      uses: android-actions/setup-android@v2

    - name: Run Unit tests
      run: ./gradlew test --stacktrace
    - name: Run UI test
      uses: reactivecircus/android-emulator-runner@v2
      with:
        api-level: ${{ matrix.api-level }}
        arch: x86_64
        script: ./gradlew connectedCheck
{% endraw %}
```

위의 모든 코드를 합하면 다음과 같습니다.

```yaml
{% raw %}
name: Android CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    name: Development build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup JDK 11
        uses: actions/setup-java@v3
        with:
          java-version: "11"
          distribution: "temurin"
          cache: gradle

      - name: Build with Gradle
        run: ./gradlew build

  test:
    name: Tests on Android (API level ${{ matrix.api-level }})
    runs-on: macos-latest
    strategy:
      matrix:
        api-level: [23, 29, 30, 31]
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - name: Setup JDK 11
        uses: actions/setup-java@v3
        with:
          java-version: "11"
          distribution: "temurin"
          cache: gradle
      - name: Setup Android SDK
        uses: android-actions/setup-android@v2

      - name: Run Unit tests
        run: ./gradlew test --stacktrace
      - name: Run UI test
        uses: reactivecircus/android-emulator-runner@v2
        with:
          api-level: ${{ matrix.api-level }}
          arch: x86_64
          script: ./gradlew connectedCheck
{% endraw %}
```

# 삽질

하나의 job에서 테스트 코드까지 수행하려 했으나 UI 테스트만 수행하면 `Error: The process '/bin/sh' failed with exit code 1`를 내뿜으며 실패했습니다.
그래서 Build와 Test를 수행하는 job을 따로 만들어보니 작동했습니다.

따로 구현 후 테스트를 돌려보니 상위 API level에서는 통과하는데 23 level에서는 실패했습니다. 찾아보니 mockk가 API level 28 미만에서는 특정 기능이 작동하지 않는 것을 알게되었습니다.
[이곳](https://mockk.io/ANDROID.html)에서 *Supported features*를 확인하면 됩니다. `@SdkSuppress(minSdkVersion = Build.VERSION_CODES.P)`를 테스트 클래스에 달아 해결했습니다.
