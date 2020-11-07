---
title: "[Flutter] 'Execution failed for task ':app:compileDebugKotlin'.' after update"
date: 2020-11-08 08:10:00 -0400
categories: flutter
tags:
- flutter
- error
- update
- android_studio
- gradle
---

안드로이드 스튜디오를 업데이트를 한 뒤 무언가 잘못 건드린것 같다.  
빌드 후 실행을 해보려 하지만 아래와 같은 에러문구를 쏟아내기만 했다. 

```
~~~ 생략 

* What went wrong:
Execution failed for task ':app:compileDebugKotlin'.
> Kotlin could not find the required JDK tools in the Java installation 'C:\Program Files (x86)\Java\jre1.8.0_171' used by Gradle. Make sure Gradle is running on a JDK, not JRE.

~~~ 생략 
```


# 1차 접근 

위 에러코드를 자세히 보면 jdk가 아니라 jre가 폴더로 지정된 것을 볼 수 있다. 
그래서 JAVA_HOME 환경변수 설정을 JDK가 설치된 폴더로 바꿔봤다. 

하지만 또다시 아래와 같은 에러를 내뿜는다.
```
Launching lib\main.dart on sdk gphone x86 arm in debug mode...
Running Gradle task 'assembleDebug'...

ERROR: JAVA_HOME is set to an invalid directory: C:\jdk-15\bin

Please set the JAVA_HOME variable in your environment to match the
location of your Java installation.
Exception: Gradle task assembleDebug failed with exit code 1
```
자세히 보니 잘못된 장소에 접근하고 있는 것 같다. 역시 [그렇다](https://stackoverflow.com/questions/45182717/java-home-is-set-to-an-invalid-directory).  
bin을 떼어내고 다시 빌드해보았다. 
그러나 또다시 실패... 
```
Gradle: Could not initialize class org.codehaus.groovy.runtime.InvokerHelper
```

# 2차 접근 
gradle이 문제가 있는 듯 하다. [버전이 잘못되었다고 한다.](https://stackoverflow.com/questions/62315496/gradle-could-not-initialize-class-org-codehaus-groovy-runtime-invokerhelper)  
수정하여 접근해보니 에러는 나오나 빌드가 되긴 한다. 
```
warning: [options] source value 7 is obsolete and will be removed in a future release
warning: [options] target value 7 is obsolete and will be removed in a future release
warning: [options] To suppress warnings about obsolete options, use -Xlint:-options.
3 warnings
```

# 3차 접근중...
