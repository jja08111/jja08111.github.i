---
title: "[Flutter] Flutter로 알람 앱 기능 구현하기"
date: 2021-04-02 03:00:00 -0400
categories: flutter
tags:
  - flutter
  - android_alarm_manager
  - alarm
---

흔히 생각하는 알람 앱을 Flutter를 이용하여 만들고 싶었습니다. 알람 시간이 되면 앱이 실행되고 알람 화면이 띄워져 알람이 울리는 것처럼요. 하지만 Flutter로 구현된 예제를 쉽게 찾을 수 없었습니다. 다행히 [random-alarm](https://github.com/geisterfurz007/random-alarm)를 찾게 되었고 많은 수정을 거친 코드를 공유하고자 이 글을 작성했습니다.

**참고**

- _**안드로이드에서만 작동**합니다._
- _**Flutter 3.0.1**, **Dart 2.17** 버전을 기준으로 작성되었습니다._

# 샘플

- [flutter_alarm_app](https://github.com/jja08111/flutter_alarm_app) (**이 레파지토리를 기준으로 설명합니다.**)

<img src="assets/images/flutter_alarm_app.png" style="zoom:75%;" />

- [random-alarm](https://github.com/geisterfurz007/random-alarm) - by [**geisterfurz007**](https://github.com/geisterfurz007) (**게시글의 코드와는 차이가 있습니다.**)

  | Main screen                                                                                                                          | Edit screen                                                                                                                          | Alarm screen                                                                                                                         |
  | ------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------ |
  | <img src="https://user-images.githubusercontent.com/26303198/78984346-48e28580-7b26-11ea-932f-23d8f3f18ce8.png" style="zoom:50%;" /> | <img src="https://user-images.githubusercontent.com/26303198/78984373-5c8dec00-7b26-11ea-8120-d07890608036.png" style="zoom:50%;" /> | <img src="https://user-images.githubusercontent.com/26303198/78984586-ea69d700-7b26-11ea-8e27-6edb27bc971f.png" style="zoom:50%;" /> |

- [꿀밤(Bedtime)](https://play.google.com/store/apps/details?id=io.github.jja08111.good_night_app)

  <img src="https://github.com/jja08111/jja08111.github.io/blob/master/assets/images/bedtime_app_preview.gif?raw=true" style="zoom:75%;" />

# 작동 순서

먼저 앱의 알람 기능은 다음과 같은 순서로 작동합니다.

1. `android_alarm_manager`로 알람을 설정한다.
2. 알람 시간이 되어 `android_alarm_manager`의 `AlarmBroadcastReceiver.onCreate`함수가 호출된다.

   이 함수에서 `SharedPreference`를 이용하여 알람 ID 플래그를 생성하고, 앱을 실행한다.

3. 앱이 실행되고 `polling_worker`를 이용하여 플래그가 생성되었는지 확인한다.
4. 플래그가 존재한다면 상태를 변경하여 알람 화면을 띄운다.

# 요구사항

## 권한

안정적으로 알람 앱을 구현하기 위해서는 아래 두 가지의 권한이 필요합니다. 이는 **안드로이드 10 이상에는 반드시 허용**되어야합니다.

1. 다른 앱 위에 표시(Display over other apps): 앱이 실행중이 아니거나 백그라운드에서 실행중일때 앱을 최상단에 띄우기 위해 필요합니다.
2. 배터리 최적화 무시(Ignore battery optimization): 배터리 최적화 기능(Doze mode) 때문에 가끔 알람이 제대로 동작하지 않을 때가 있는데 이를 방지합니다.

## Android에서의 MainActivity.kt 수정

MainActivity에서 `onCreate`함수에 다음과 같이 추가해주세요. 알람 화면이 잠금화면에서 올바르게 나올 수 있도록 코드를 추가하는 것 입니다. 아래의 코드는 코틀린을 기준으로 작성되었습니다. (참고로 안드로이드 코드를 수정하실 때는 프로젝트의 안드로이드 폴더를 따로 열어서 수정하시면 편리합니다.)

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O_MR1) {
        setShowWhenLocked(true)
        setTurnScreenOn(true)
        window.addFlags(WindowManager.LayoutParams.FLAG_ALLOW_LOCK_WHILE_SCREEN_ON)
    } else {
        window.addFlags(
            WindowManager.LayoutParams.FLAG_SHOW_WHEN_LOCKED    // deprecated api 27
            or WindowManager.LayoutParams.FLAG_DISMISS_KEYGUARD     // deprecated api 26
            or WindowManager.LayoutParams.FLAG_TURN_SCREEN_ON   // deprecated api 27
            or WindowManager.LayoutParams.FLAG_ALLOW_LOCK_WHILE_SCREEN_ON
        )
    }
    val keyguardMgr = getSystemService(Context.KEYGUARD_SERVICE) as KeyguardManager
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        keyguardMgr.requestDismissKeyguard(this, null)
    }
}
```

## 패키지

필요한 패키지들은 다음과 같으며 **해당 패키지의 `README.md`를 꼭 확인**하세요.

1. [android_alarm_manager_plus](https://pub.dev/packages/android_alarm_manager_plus) [![pub package](https://img.shields.io/pub/v/android_alarm_manager_plus.svg)](https://pub.dev/packages/android_alarm_manager_plus)

   알람을 설정할때 이용할 플러그인입니다.

   알람이 작동할때, 즉 목표한 시간이 되었을 때 **플래그 형성 및 앱을 실행**하기 위해서는 **플러그인 수정**이 필요합니다. 제가 folk 해놓은 깃허브를 이용하시거나 해당 플러그인을 아래처럼 수정하세요.

   ### 이미 수정된 플러그인 이용하기

   ```yaml
   dependencies:
     android_alarm_manager_plus:
       git:
         url: https://github.com/jja08111/plus_plugins.git
         path: packages/android_alarm_manager_plus
         ref: alarm_app
   ```

   ### 직접 수정하기

   먼저 `AlarmFlagManager.java` 파일을 플러그인의 안드로이드 부분에 다음과 같이 생성하세요. 이때 `SharedPreferences`의 put 유형을 `Long`으로 해야 플러터 코드 `int`로 읽어올 수 있는 것에 주의하세요.

   `android_alarm_manager_plus` 플러그인을 통해 설정된 알람의 ID를 SharedPreferences를 이용하여 저장하는 것을 볼 수 있습니다. 이 플래그는 Flutter 코드에서 알람을 작동시킬때 사용 할 것입니다.

   #### AlarmFlagManager.java

   ```java
   package dev.fluttercommunity.plus.androidalarmmanager;

   import android.content.Context;
   import android.content.Intent;
   import android.content.SharedPreferences;

   public class AlarmFlagManager {

     private static final String FLUTTER_SHARED_PREFERENCE_KEY = "FlutterSharedPreferences";
     private static final String ALARM_FLAG_KEY = "flutter.alarmFlagKey";

     static public void set(Context context, Intent intent) {
       int alarmId = intent.getIntExtra("id", -1);

       SharedPreferences prefs = context.getSharedPreferences(FLUTTER_SHARED_PREFERENCE_KEY, 0);
       prefs.edit().putLong(ALARM_FLAG_KEY, alarmId).apply();
     }
   }
   ```

   그 다음 `AlarmBroadcastReceiver.java`파일을 다음과 같이 수정하세요.

   코드에서 플래그를 설정하고 화면을 키고, 앱을 실행하여 화면의 최상단으로 띄우는 것을 볼 수 있습니다.

   #### AlarmBroadcastReceiver.java

   ```java
   package dev.fluttercommunity.plus.androidalarmmanager;

   import android.content.BroadcastReceiver;
   import android.content.Context;
   import android.content.Intent;
   import android.os.Build;
   import android.os.PowerManager;

   public class AlarmBroadcastReceiver extends BroadcastReceiver {
     @Override
     public void onReceive(Context context, Intent intent) {
       AlarmFlagManager.set(context, intent);

       PowerManager powerManager = (PowerManager)
         context.getSystemService(Context.POWER_SERVICE);
       PowerManager.WakeLock wakeLock = powerManager.newWakeLock(PowerManager.FULL_WAKE_LOCK |
         PowerManager.ACQUIRE_CAUSES_WAKEUP |
         PowerManager.ON_AFTER_RELEASE, "AlarmBroadcastReceiver:My wakelock");

       Intent startIntent = context
         .getPackageManager()
         .getLaunchIntentForPackage(context.getPackageName());

       if (startIntent != null)
         startIntent.setFlags(
           Intent.FLAG_ACTIVITY_REORDER_TO_FRONT |
             Intent.FLAG_ACTIVITY_NEW_TASK |
             Intent.FLAG_ACTIVITY_RESET_TASK_IF_NEEDED
         );

       wakeLock.acquire(3 * 60 * 1000L /*3 minutes*/);
       context.startActivity(startIntent);
       AlarmService.enqueueAlarmProcessing(context, intent);
       wakeLock.release();

       if (Build.VERSION.SDK_INT < 31)
         context.sendBroadcast(new Intent(Intent.ACTION_CLOSE_SYSTEM_DIALOGS));
     }
   }
   ```

2. [provider](https://pub.dev/packages/provider) [![pub package](https://img.shields.io/pub/v/provider.svg)](https://pub.dev/packages/provider)

   상태관리를 위해 이용합니다.

3. [shared_preferences](https://pub.dev/packages/shared_preferences) [![pub package](https://img.shields.io/pub/v/shared_preferences.svg)](https://pub.dev/packages/shared_preferences)

   알람 플래그를 형성할 때 이용할 것입니다.

4. [permission_handler](https://pub.dev/packages/permission_handler) [![pub package](https://img.shields.io/pub/v/permission_handler.svg)](https://pub.dev/packages/permission_handler)

   앱에서 사용자에게 권한을 요구할 때 필요한 플러그인입니다.

이와 같은 준비가 되면 알람 앱을 만들 준비가 끝났습니다.

# 기능 구현

알람 객체와 리스트를 구현하여 관리한다고 가정하고 이 글에서는 기능적인 부분 위주로 다루도록 하겠습니다.

## 1. 알람 설정

먼저 `android_alarm_manager`를 이용하여 목표하는 시간에 알람을 설정합니다. 이때 `alarmClock`을 true로 하여 안드로이드 내부에서 정확한 알람 시계로 작동할 수 있도록 합니다. 그리고 알람 작동시 스마트폰의 화면을 켜기 위해 `wakeup` 또한 true로 설정합니다. 재부팅시 알람이 작동할 수 있도록 `rescheduleOnReboot`도 true로 설정합니다.

```dart
void emptyCallback() {} // 이 함수는 최상단에 위치해야 한다. 즉 클래스 내부이면 static으로 정의해야 한다.

AndroidAlarmManager.oneShotAt(
  dateTime,
  id,
  emptyCallback,
  alarmClock: true,
  wakeup: true,
  rescheduleOnReboot: true,
);
```

위의 `emptyCallback` 함수를 보면 내부에 내용이 없는데요. 이용하지 않을 것이기 때문에 알람 설정을 위해 형식적으로 둔 것입니다.

플러그인의 콜백함수를 이용하지 않고 `AlarmBroadcastReceiver.java`를 이용하여 알람을 작동시키는 이유는 시스템에 의해 Dart 코드가 지연되어 작동할 수 있기 때문입니다. 지연되어 작동하면 알람이 제대로 동작하지 않을 위험성이 커집니다.

## 2. AlarmBroadcastReceiver.onCreate 호출

알람 시간이 되면 `AlarmBroadcastReceiver.onCreate`가 실행됩니다. 이때 위에서 설명한대로 플래그가 형성되고 앱이 실행됩니다.

저는 플래그를 확인 및 삭제하는 클래스를 아래와 같이 구현했습니다. 아래에서 `alarmFlagKey`가 위에서 설명한 `flutter.alarmFlagKey`의 뒷부분과 일치하는 것을 볼 수 있습니다. 이 클래스의 사용은 뒤에 설명하겠습니다.

### alarm_flag_manager.dart

```dart
import 'package:shared_preferences/shared_preferences.dart';

class AlarmFlagManager {
  static final AlarmFlagManager _instance = AlarmFlagManager._();

  factory AlarmFlagManager() => _instance;

  AlarmFlagManager._();

  static const String _alarmFlagKey = "alarmFlagKey";

  Future<int?> getFiredId() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.reload();
    return prefs.getInt(_alarmFlagKey);
  }

  Future<void> clear() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.remove(_alarmFlagKey);
  }
}
```

## 3. Flag 탐색 후 상태변경

알람 플래그를 형성했으니 `AlarmPollingWorker`로 탐색을 하여 플래그가 있다면 **알람이 울린 상태**로 변경해야합니다. 이때 탐색기는 다음 두 가지 경우에 실행됩니다.

- 앱 실행시 -> 앱의 매인 클래스 진입시
- 앱을 종료하지 않고 다시 진입한 후 -> 앱 메인 루트 화면에서 `WidgetsBindingObserver` 이용

두 번째 경우에 앱이 실행 중일때 알람이 울린 경우도 포함됩니다. 그렇다면 어떻게 위와 같은 사항을 탐색할까요? 바로 앱의 메인 UI에 해당하는 곳에 `WidgetsBindingObserver`를 이용하면 됩니다. 이는 앱이 종료되는지 혹은 다시 진입했는지 등 앱의 라이프 사이클을 관찰하는 믹스인이라고 생각하시면 됩니다.

탐색이 된 경우 아래의 `AlarmState`의 상태를 변경하면 됩니다.

### 앱 실행시

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  await AndroidAlarmManager.initialize();

  final AlarmState alarmState = AlarmState();

  // 앱 진입시 알람 탐색을 시작한다.
  AlarmPollingWorker().createPollingWorker(alarmState);

  ...

  runApp(MultiProvider(
    providers: [
      ChangeNotifierProvider(create: (context) => alarmState),
    ],
    child: MyApp(),
  ));
}
```

### 앱 재진입시

```dart
class SomeScreen extends StatefulWidget {
  @override
  _SomeScreenState createState() => _SomeScreenState();
}

class _SomeScreenState extends State<ObserveAlarm>
    with WidgetsBindingObserver {
  @override
  void initState() {
    super.initState();
    WidgetsBinding.instance!.addObserver(this);
  }

  @override
  void dispose() {
    WidgetsBinding.instance!.removeObserver(this);
    super.dispose();
  }

  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    switch (state) {

      ...생략

      case AppLifecycleState.resumed:
        AlarmPollingWorker().createPollingWorker(context.read<AlarmState>());
        break;
    }
  }

  ...생략
}
```

### alarm_polling_worker.dart

여기서 알람 플래그를 탐색하고 삭제하는 것을 볼 수 있습니다.

```dart
class AlarmPollingWorker {
  static final AlarmPollingWorker _instance = AlarmPollingWorker._();

  factory AlarmPollingWorker() {
    return _instance;
  }

  AlarmPollingWorker._();

  bool _running = false;

  /// 알람 플래그 탐색을 시작한다.
  void createPollingWorker(AlarmState alarmState) async {
    if (_running) return;

    debugPrint('Starts polling worker');
    _running = true;
    final int? callbackAlarmId = await _poller(10);
    _running = false;

    if (callbackAlarmId != null) {
      if (!alarmState.isFired) {
        alarmState.fire(callbackAlarmId);
      }
      await AlarmFlagManager().clear();
    }
  }

  /// 알람 플래그를 찾은 경우 해당 알람의 Id를 반환하고, 플래그가 없는 경우 `null`을 반환한다.
  Future<int?> _poller(int iterations) async {
    int? alarmId;
    int iterator = 0;

    await Future.doWhile(() async {
      alarmId = await AlarmFlagManager().getFiredId();
      if (alarmId != null || iterator++ >= iterations) return false;
      await Future.delayed(const Duration(milliseconds: 25));
      return true;
    });
    return alarmId;
  }
}
```

### alarm_status.dart

```dart
class AlarmState extends ChangeNotifier {
  int? get callbackAlarmId => _callbackAlarmId;
  int? _callbackAlarmId;

  bool get isFired => _callbackAlarmId != null;

  void fire(int alarmId) {
    _callbackAlarmId = alarmId;
    notifyListeners();
    debugPrint('Alarm has fired #$alarmId');
  }

  void dismiss() {
    _callbackAlarmId = null;
    notifyListeners();
  }
}
```

## 4. 메인 화면에서 알람 화면으로 변경

이제 상태를 변경하였으니 알람 화면을 보여주면 됩니다. 저는 `AlarmObserver`라는 클래스를 만들어 화면을 분기했습니다. `AlarmState.isFired`가 true이면 알람 화면을, 아니면 홈 화면을 보여줍니다. 저는 앞서 소개한 `WidgetsBindingObserver`를 이곳에 적용했습니다.

### alarm_observer.dart

```dart
class AlarmObserver extends StatefulWidget {
  final Widget child;

  AlarmObserver({
    Key? key,
    required this.child,
  }) : super(key: key);

  @override
  _AlarmObserverState createState() => _AlarmObserverState();
}

class _AlarmObserverState extends State<AlarmObserver>
    with WidgetsBindingObserver {

  ...생략

  @override
  Widget build(BuildContext context) {
    return Consumer<AlarmState>(builder: (context, state, child) {
      Widget? alarmScreen;

      if (state.isFired) {
        final callbackId = state.callbackAlarmId!;
        Alarm? alarm = context.read<AlarmListProvider>().getAlarmBy(callbackId);
        if (alarm != null) {
          alarmScreen = AlarmScreen(
            alarm: alarm,
          );
        }
      }
      return IndexedStack(
        index: alarmScreen != null ? 0 : 1,
        children: [
          alarmScreen ?? Container(),
          widget.child,
        ],
      );
    });
  }
}
```

위에서 보이는 `Alarm`, `AlarmScreen`, `AlarmListProvider` 클래스는 [`flutter_alarm_app`](https://github.com/jja08111/flutter_alarm_app)을 참고하시면 됩니다.

# 마치며

추가적으로 알람을 주 단위로 반복하여 설정하는 기능, 알람 다시 울림 기능(스누즈) 등을 구현할 수도 있습니다.

Flutter로 알람 앱의 기본적인 기능을 구현해봤습니다. 알람 기능만 네이티브로 따로 구현하는 것도 좋은 방법이라고 생각합니다. 이상 긴 글 읽어 주셔서 감사합니다!

# 수정

- 2021-04-02 초기 업로드
- 2021-07-09 내용 추가, 보완 및 Null-safety 적용
- 2021-08-03 `callback`함수 삭제 및 플래그 설정 위치 변경
- 2022-05-30 상태관리를 `provider`로 하도록 수정 및 예제 레파지토리 추가
