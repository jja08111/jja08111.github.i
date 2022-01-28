---
title: "[Flutter] google 로그인 테스트하기"
date: 2022-01-28 00:40:00 -0400
categories: flutter
tags:
- flutter
- test
- firebase
---

구글 로그인 기능을 구현하고 테스트를 추가하려 했으나 생각처럼 쉽게 되지 않았다. 
테스트 하려는 것은 구글로 로그인이 된 경우 홈페이지를 띄우고 아닌 경우는 로그인 페이지를 띄우는지 검증하는 굉장히 간단한 것이었다.

바로 결론을 말하겠다. 방법은 pub 사이트에 제공된 Mock들(MockFirebaseAuth, MockGoogleSignIn)을 이용하고 추가로 FireBaseExtended에서 제공하는 [mock 파일](https://github.com/FirebaseExtended/flutterfire/blob/master/packages/firebase_auth/firebase_auth/test/mock.dart)을 
이용하는 것이다.

나의 파이어베이스를 담당하는 provider는 아래와 같다.
```dart
class FirebaseProvider extends GetxController {
  final FirebaseAuth _auth;
  final GoogleSignIn _googleSignIn;

  FirebaseProvider({
    @visibleForTesting FirebaseAuth? auth,
    @visibleForTesting GoogleSignIn? googleSignIn,
  })  : _auth = auth ?? FirebaseAuth.instance,
        _googleSignIn = googleSignIn ??
            GoogleSignIn(
              scopes: <String>[
                'email',
                'https://www.googleapis.com/auth/contacts.readonly',
              ],
            );

  bool isLoggedIn() {
    return _auth.currentUser != null;
  }

  Future<void> signInWithGoogle() async {
    try {
      final GoogleSignInAccount? googleSignInAccount =
          await _googleSignIn.signIn();
      if (googleSignInAccount == null) return;

      final GoogleSignInAuthentication googleSignInAuthentication =
          await googleSignInAccount.authentication;
      final AuthCredential credential = GoogleAuthProvider.credential(
        accessToken: googleSignInAuthentication.accessToken,
        idToken: googleSignInAuthentication.idToken,
      );
      await _auth.signInWithCredential(credential);
    } on FirebaseAuthException catch (e) {
      debugPrint(e.message);
      rethrow;
    }
  }

  Future<void> signOutFromGoogle() async {
    await _googleSignIn.signOut();
    await _auth.signOut();
  }
}
```

# 테스트 코드

완성된 테스트 코드는 다음과 같다.

```dart
void main() async {
  setupFirebaseAuthMocks();

  setUpAll(() async {
    await Firebase.initializeApp();
  });

  group('Login 테스트', () {
    tearDown(() {
      Get.deleteAll();
    });

    testWidgets('로그인이 되어 있지 않다면 로그인 페이지를 띄운다.', (tester) async {
      Get.put(FirebaseProvider(
        googleSignIn: MockGoogleSignIn(),
        auth: MockFirebaseAuth(),
      ));

      await tester.pumpWidget(const MyApp());

      expect(find.byType(LoginPage), findsOneWidget);
    });

    testWidgets('로그인이 되어 있다면 바로 홈 페이지를 띄운다.', (tester) async {
      Get.put(FirebaseProvider(
        googleSignIn: MockGoogleSignIn(),
        auth: MockFirebaseAuth(mockUser: MockUser(), signedIn: true),
      ));
      Get.put(HomePageProvider());

      await tester.pumpWidget(const MyApp());

      expect(find.byType(LoginPage), findsNothing);
      expect(find.byType(HomePage), findsOneWidget);
    });
  });
}

typedef Callback = void Function(MethodCall call);

void setupFirebaseAuthMocks([Callback? customHandlers]) {
  TestWidgetsFlutterBinding.ensureInitialized();

  MethodChannelFirebase.channel.setMockMethodCallHandler((call) async {
    if (call.method == 'Firebase#initializeCore') {
      return [
        {
          'name': defaultFirebaseAppName,
          'options': {
            'apiKey': '123',
            'appId': '123',
            'messagingSenderId': '123',
            'projectId': '123',
          },
          'pluginConstants': {},
        }
      ];
    }
    if (call.method == 'Firebase#initializeApp') {
      return {
        'name': call.arguments['appName'],
        'options': call.arguments['options'],
        'pluginConstants': {},
      };
    }
    if (customHandlers != null) {
      customHandlers(call);
    }
    return null;
  });
}
```
