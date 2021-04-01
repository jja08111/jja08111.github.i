---
title: "[Flutter] Flutter로 알람 앱 기능 구현하기"
date: 2021-04-02 03:00:00 -0400
categories: flutter
tags:
- flutter
- android_alarm_manager
- alarm
- app
---



# Flutter로 알람 앱 기능 구현하기

우리가 흔히 생각하는 알람 앱을 Flutter를 이용하여 만들고 싶었습니다. 알람 시간이 되면 앱이 실행되고 알람 화면이 띄워져 알람이 울리는 것처럼요. 하지만 Flutter로 구현된 알람앱은 흔하지 않았습니다. 다행히 [random-alarm](https://github.com/geisterfurz007/random-alarm)과 같은 좋은 소스를 찾게되어 공유하고자 이 글을 작성했습니다.



## 해당 기능으로 구현한 앱 미리보기

- [random-alarm](https://github.com/geisterfurz007/random-alarm) - by [**geisterfurz007**](https://github.com/geisterfurz007)

  | Main screen                                                  | Edit screen                                                  | Alarm screen                                                 |
  | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | <img src="https://user-images.githubusercontent.com/26303198/78984346-48e28580-7b26-11ea-932f-23d8f3f18ce8.png" style="zoom:50%;" /> | <img src="https://user-images.githubusercontent.com/26303198/78984373-5c8dec00-7b26-11ea-8120-d07890608036.png" style="zoom:50%;" /> | <img src="https://user-images.githubusercontent.com/26303198/78984586-ea69d700-7b26-11ea-8e27-6edb27bc971f.png" style="zoom:50%;" /> |

  

- [꿀밤(Bedtime)](https://play.google.com/store/apps/details?id=io.github.jja08111.good_night_app)

  <img src="https://github.com/jja08111/jja08111.github.io/blob/master/assets/images/bedtime_app_preview.gif?raw=true" style="zoom:75%;" />



## 작동 방식 이해

먼저 앱의 알람 기능은 다음과 같은 순서로 작동합니다.

1. `android_alarm_manager`로 알람을 설정한다.
2. 알람 시간이 되어 `android_alarm_manager`의 `callback`함수가 호출된다.
3. 알람 플래그 파일을 생성한다.(`shared_preference`등 이용)
4. 직접 구현한 `polling_worker`를 이용하여 플래그 파일이 생성되었는지 확인한다.
5. 플래그 파일이 존재한다면 상태를 변경(`mobx, provider`등 이용)하여 알람 화면을 띄운다.



## 사용 방법

### 권한 

안정적으로 알람 앱을 구현하기 위해서는 아래 두 가지의 권한이 필요합니다. 이는 안드로이드 10 이상 버전에는 무조건 허용되어야합니다.

1. 다른 앱 위에 표시(Display over other apps) : 앱이 실행중이 아니거나 백그라운드에서 실행중일때 앱을 최상단에 띄우기 위해 필요합니다.
2. 배터리 최적화 무시(Ignore battery optimization) : 배터리 최적화 기능 때문에 가끔 알람이 제대로 동작하지 않을 때가 있는데 이를 방지합니다.

### 플러그인 

필요한 플러그인은 다음과 같습니다.

1. [android_alarm_manager](https://pub.dev/packages/android_alarm_manager) [![pub package](https://img.shields.io/pub/v/android_alarm_manager.svg)](https://pub.dev/packages/android_alarm_manager)

   - 이 플러그인 수정이 필요합니다. 해당 플러그인의 `AlarmBroadcastReceiver.java`파일을 다음과 같이 수정하세요. (파일 위치 예 : flutter\.pub-cache\hosted\pub.dartlang.org \android_alarm_manager-0.4.1+6\android\src\main\java\io\flutter\plugins\androidalarmmanager\AlarmBroadcastReceiver.java)

     ```java
     // Copyright 2019 The Chromium Authors. All rights reserved.
     // Use of this source code is governed by a BSD-style license that can be
     // found in the LICENSE file.
     
     package io.flutter.plugins.androidalarmmanager;
     
     import android.content.BroadcastReceiver;
     import android.content.Context;
     import android.content.Intent;
     import android.os.PowerManager;
     import android.content.pm.PackageManager;
     import android.view.WindowManager;
     
     public class AlarmBroadcastReceiver extends BroadcastReceiver {
       private static PowerManager.WakeLock wakeLock;
     
       @Override
       public void onReceive(Context context, Intent intent) {
         PowerManager powerManager = (PowerManager)
                 context.getSystemService(Context.POWER_SERVICE);
         wakeLock = powerManager.newWakeLock(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON |
                 PowerManager.ACQUIRE_CAUSES_WAKEUP |
                 PowerManager.ON_AFTER_RELEASE, "My wakelock");
     
         Intent startIntent = context
                 .getPackageManager()
                 .getLaunchIntentForPackage(context.getPackageName());
     
         startIntent.setFlags(
                 Intent.FLAG_ACTIVITY_REORDER_TO_FRONT |
                         Intent.FLAG_ACTIVITY_NEW_TASK |
                         Intent.FLAG_ACTIVITY_RESET_TASK_IF_NEEDED
         );
     
         wakeLock.acquire();
         context.startActivity(startIntent);
         AlarmService.enqueueAlarmProcessing(context, intent);
         wakeLock.release();
       }
     }
     ```

     이는 [이곳](https://github.com/flutter/flutter/issues/30555#issuecomment-501597824)에서 자세히 볼 수 있습니다.

2. [mobx](https://pub.dev/packages/mobx) [![pub package](https://img.shields.io/pub/v/mobx.svg)](https://pub.dev/packages/mobx)

   - 이 플러그인을 이용하려면 다음과 같은 빌드 코드를 터미널 창에 입력해야합니다.

     `flutter packages pub run build_runner build` 

     자세한 정보는 [mobx 페이지](https://mobx.netlify.app/getting-started/)에서 볼수 있습니다.

3. (필수 X) [permission_handler](https://pub.dev/packages/permission_handler) [![pub package](https://img.shields.io/pub/v/permission_handler.svg)](https://pub.dev/packages/permission_handler) 

   - 앱에서 사용자에게 권한을 요구할때 유용한 플러그인입니다. 

   

이와 같은 준비가 되면 알람 앱을 만들 준비가 끝났습니다. 

더 자세한 내용은 곧 작성하도록 하겠습니다.