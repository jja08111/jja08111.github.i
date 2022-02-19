---
title: "[Flutter] Isolate에서 TfLite 구동하기"
date: 2022-02-18 00:20:00 -0400
categories: flutter
tags:
  - flutter
  - tflite
  - isolate
---

Flutter에는 tflite를 이용할 수 있는 [플러그인](https://pub.dev/packages/tflite_flutter)이 존재한다. 이 플러그인에는 다양한 예제, 블로그 글 등 쉽게 따라 할 수 있는 요소가 많다.

이 글은 isolate를 이용하여 메인 쓰레드의 성능 저하 없이 tflite 프로세싱을 동시에 여러번 처리하는 방법에 대해서 중점적으로 다룰 것이다. 본문 전에 isolate가 무엇인지 간단히 요약하자면 독립된 메모리 속에서 이벤트 루프를 처리하는 쓰레드라고 볼 수 있다.

Flutter에는 isolate를 아주 간단히 이용할 수 있는 `compute` 함수가 존재한다. 이 함수는 완전히 독립된 새로운 isoalate를 만들고 전달된 함수를 실행한 뒤 해당 값을 반환한다. 처리할 항목이 하나인 경우 이 함수를 이용하면 된다.

하지만 나의 경우 15개정도의 이미지를 동시에 처리해야 했다. 때문에 `compute` 함수를 이용했더니 계속해서 isolate가 생성되었고 오히려 성능 저하가 발생했다. 이를 해결할 수 있는 방법은 하나의 isolate만 생성하고 이것과 통신하여 메시지를 주고 받는 것이었다.

# Isolate 생성

하나의 isolate를 생성하는 법은 `Isolate.spawn` 함수를 이용하면 된다. 나는 이 함수를 `FoodPredictor`라는 상태 관리자를 생성 할 때 호출하였다. 또한 여기서 tflite에 필요한 `interpreterBuffer`를 미리 불러와 메시지로 전달하고 있다. 그리고 `ReceivePort`를 만들어 이것의 `sendPort`를 전달하고 있다. 이렇게 하는 이유는 isolate를 향해 메시지를 전달할 수 있는 포트를 받기 위해서이다.

```dart
// Getx를 사용하고 있어서 GetxController를 상속했다.
class FoodPredictor extends GetxController {
  SendPort? _mainToIsolateSendPort;

  String get _modelFilePath => 'assets/tf_model/model.tflite';

  FoodPredictor() {
    _init();
  }

  void _init() async {
    try {
      final rawAssetFile = await rootBundle.load(_modelFilePath);
      final interpreterBuffer = rawAssetFile.buffer.asUint8List();
      final port = ReceivePort();

      port.listen((message) {
        _mainToIsolateSendPort = message as SendPort;
        port.close();
      });
      await Isolate.spawn(_entryPoint, [port.sendPort, interpreterBuffer]);
    } catch (e) {
      debugPrint('AI interpreter 파일 읽기 실패: $e');
    }
  }

  // ...
```

# Isolate로 향하는 포트를 메인 쓰레드로 전달

Isolate가 생성되면 `_entryPoint` 함수가 실행된다. 전달받은 메시지를 변수에 넣어주고 `Classifier` 객체를 만들고 있다. 이 객체는 다음에 자세히 설명할 것이다. 그 후 isolate로 향하는 포트를 만든 뒤 리스너를 달아준다. 그리고 메시지로 전달받은 `sendPort`를 이용하여 메인 쓰레드에 isolate로 향하는 포트를 전달한다. 이렇게 하면 메인 쓰레드에서 isolate로 메시지를 전달할 준비는 완료되었다.

```dart
// ...

static void _entryPoint(List<Object> data) {
  final sendPort = data[0] as SendPort;
  final interpreterBuffer = data[1] as Uint8List;

  final classifier = Classifier(Interpreter.fromBuffer(interpreterBuffer));
  final mainToIsolatePort = ReceivePort()
    ..listen((message) {
      final sendPort = message[0] as SendPort;
      final bytes = message[1] as Uint8List;

      final img = decodeImage(bytes);
      if (img == null) {
        sendPort.send(false);
        return;
      }
      sendPort.send(classifier.predictFood(img));
    });

  sendPort.send(mainToIsolatePort.sendPort);
}

// ...
```

이제 실제로 `FoodPredictor` 객체 외부에서 사용하는 함수를 어떤식으로 작성하는지 보겠다. 먼저 `_mainToIsolateSendPort`가 `null`인 경우 `true`를 반환하도록 예외처리했다. 경우에 따라 `false`로 반환하도록 하는 것이 좋을 수도 있으나 나의 앱은 그렇지 않았다. 그 후 새로운 포트를 만든다. 그리고 음식이 있는지 확인할 이미지의 `Uint8List`를 포트와 함께 isolate에 전달한다. 위의 `_entryPoint`에서 `mainToIsolatePort`의 리스너를 보면 포트와 이미지를 받아 실제 prediction을 하고 있다. predicted된 결과물은 다시 아래의 포트에 등록된 리스너에 전달되어 `Completer`를 종료한다. 이렇게 `isFoodImage`를 호출한 곳에서 비동기로 예측된 값을 받을 수 있다.

```dart
// ...

Future<bool> isFoodImage(Uint8List bytes) async {
  if (_mainToIsolateSendPort == null) return true;

  final completer = Completer<bool>();
  final port = ReceivePort();

  port.listen((message) {
    completer.complete(message as bool);
    port.close();
  });
  _mainToIsolateSendPort!.send([port.sendPort, bytes]);

  return completer.future;
}
```

# Classifier

`Classifier`는 플러그인 예제의 것과 큰 차이는 없다. 특징이라면 `predictFood` 함수에서 가장 높게 예측된 레이블이 음식인 경우에 참을 반환한다는 것이 있다.

```dart
import 'dart:math';

import 'package:image/image.dart';
import 'package:flutter/foundation.dart';
import 'package:tflite_flutter_helper/tflite_flutter_helper.dart';
import 'package:tflite_flutter/tflite_flutter.dart';

class Classifier {
  final Interpreter _interpreter;

  final List<int> _inputShape;
  final List<int> _outputShape;

  TensorImage? _inputImage;
  TensorBuffer? _outputBuffer;

  final TfLiteType _inputType;
  final TfLiteType _outputType;

  SequentialProcessor? _probabilityProcessor;

  static const List<String> _labels = ['Food', 'Non-food'];

  NormalizeOp get preProcessNormalizeOp => NormalizeOp(0, 1);

  NormalizeOp get postProcessNormalizeOp => NormalizeOp(0, 255);

  Classifier(this._interpreter)
      : _inputShape = _interpreter.getInputTensor(0).shape,
        _outputShape = _interpreter.getOutputTensor(0).shape,
        _inputType = _interpreter.getInputTensor(0).type,
        _outputType = _interpreter.getOutputTensor(0).type {
    _outputBuffer = TensorBuffer.createFixedSize(_outputShape, _outputType);
    _probabilityProcessor =
        TensorProcessorBuilder().add(postProcessNormalizeOp).build();
  }

  bool _isInitialized() {
    return _outputBuffer != null && _probabilityProcessor != null;
  }

  TensorImage _preProcess() {
    int cropSize = min(_inputImage!.height, _inputImage!.width);
    return ImageProcessorBuilder()
        .add(ResizeWithCropOrPadOp(cropSize, cropSize))
        .add(ResizeOp(
            _inputShape[1], _inputShape[2], ResizeMethod.NEAREST_NEIGHBOUR))
        .add(preProcessNormalizeOp)
        .build()
        .process(_inputImage!);
  }

  bool predictFood(Image image) {
    // 초기화 되지 않았으면 무조건 `true`를 반환한다.
    if (!_isInitialized()) return true;

    try {
      _inputImage = TensorImage(_inputType);
      _inputImage!.loadImage(image);
      _inputImage = _preProcess();
      _interpreter.run(_inputImage!.buffer, _outputBuffer!.getBuffer());

      Map<String, double> labeledProb = TensorLabel.fromList(
              _labels, _probabilityProcessor!.process(_outputBuffer))
          .getMapWithFloatValue();
      final prediction = _getTopProbability(labeledProb);
      return prediction.key == 'Food';
    } catch (e) {
      debugPrint('predict 오류 발생: $e');
    }
    return true;
  }

  void close() {
    _interpreter.close();
  }
}

MapEntry<String, double> _getTopProbability(Map<String, double> labeledProb) {
  MapEntry<String, double> result = const MapEntry<String, double>('', -1.0);
  labeledProb.forEach((key, value) {
    if (result.value < value) {
      result = MapEntry<String, double>(key, value);
    }
  });
  return result;
}
```

# 마치며

이 방법을 통해 기존 20프레임 이하로 곤두박질 치던 성능을 50프레임 이상으로 향상할 수 있었다.
