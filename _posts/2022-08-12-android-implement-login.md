---
title: "[Android] 로그인 기능 구현"
date: 2022-08-12 09:12:00 +0900
categories: android
tags:
  - android
  - compose
  - test
  - flaky
---

Firebase Auth가 아닌 **벡엔드 팀이 직접 구축한 로그인 기능**을 이용한다. 이전에 Firebase Auth를 사용해본 것과 달라 정리하고자 글을 쓴다.
참고로 블로그 앱의 구조는 Clean architecture로 되어있으며 Hilt를 통해 의존성 주입을 하고 있다.

2022년 8월 20일 기준 로그인 기능은 다음과 같이 작동한다.

- 로그인을 성공하면 서버에서 세션 토큰을 응답에 포함하여 준다.
- 이전에 로그인한 상태로 앱을 다시 실행하면 로컬에 저장된 세션 토큰의 유효성 검사 후 유효하다면 그대로 이용한다.
- 토큰이 유효하지 않다면 로컬에 저장된 세션 토큰을 지우고 로그인 화면을 보인다.

전체적인 구조는 아래 그림과 같다.

![architecture](https://user-images.githubusercontent.com/57604817/185727748-bf4b1e85-a43d-4a11-9c85-ccfc509a80b3.png)

로컬 저장을 위해 SharedPreferences를 이용하고 있다. 이는 내부적으로 암호화를 위해 EncryptedSharedPreferences를 쓸것이다.
그리고 로그인한 현재 유저의 정보를 얻기 위해 UserRepository를 참조하고 있다. 가장 밑의 레이어부터 차근차근 알아보자.

# API

API는 다음과 같다. 로그인 성공시에 서버는 유저의 ID와 세션 토큰을 응답에 포함하여 준다.

```kotlin
interface AuthApi {
    @POST("auth/login")
    @Headers("x-pocs-device-type: android")
    suspend fun login(
        @Body loginRequestBody: LoginRequestBody
    ): Response<ResponseBody<LoginResponseData>>

    @POST("auth/logout")
    suspend fun logout(
        @Header("x-pocs-session-token") token: String
    ): Response<ResponseBody<Unit>>

    @POST("auth/validation")
    suspend fun isSessionValid(
        @Header("x-pocs-session-token") token: String
    ): Response<ResponseBody<Unit>>
}
```

# DataSource

리모트와 로컬로 나누어진 data source가 존재한다. 리모트에서는 API 호출을 진행한다.

**Remote**

```kotlin
class AuthRemoteDataSourceImpl @Inject constructor(
    private val api: AuthApi
) : AuthRemoteDataSource {

    override suspend fun login(loginRequestBody: LoginRequestBody) = api.login(loginRequestBody)

    override suspend fun logout(token: String) = api.logout(token)

    override suspend fun isSessionValid(token: String) = api.isSessionValid(token)
}
```

로컬에서는 리모트를 통해 얻은 세션 토큰과 유저 아이디를 AuthLocalData 객체로 묶어 아래와 같이 다루고 있다.

**Local**

```kotlin
class AuthLocalDataSourceImpl @Inject constructor(
    @Named("auth") private val sharedPreferences: SharedPreferences
) : AuthLocalDataSource {

    override fun getData(): AuthLocalData? {
        val json = sharedPreferences.getString(AUTH_LOCAL_DATA_PREFS_KEY, null) ?: return null
        return Gson().fromJson(json, AuthLocalData::class.java)
    }

    override fun setData(authLocalData: AuthLocalData) {
        val json = Gson().toJson(authLocalData)
        sharedPreferences.edit().putString(AUTH_LOCAL_DATA_PREFS_KEY, json).apply()
    }

    override fun clear() {
        sharedPreferences.edit().remove(AUTH_LOCAL_DATA_PREFS_KEY).apply()
    }

    companion object {
        private const val AUTH_LOCAL_DATA_PREFS_KEY = "authLocalData"
    }
}
```

그런데 위를 보면 SharedPreferences를 의존성 주입받고 있다. 이는 아래와 같이 모듈을 만들어 주입할 수 있다.
보통의 SharedPreferences는 루팅된 기기에서 값을 알아낼 수 있기 때문에 EncryptedSharedPreferences을 사용하여 암호화 했다.

```kotlin
@Module
@InstallIn(SingletonComponent::class)
class LocalModule {

    @Singleton
    @Provides
    @Named("auth")
    fun provideEncryptedSharedPreferences(
        @ApplicationContext context: Context
    ): SharedPreferences {
        val masterKeyAlias = MasterKey.Builder(context, MasterKey.DEFAULT_MASTER_KEY_ALIAS)
            .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
            .build()

        return EncryptedSharedPreferences.create(
            context,
            "auth",
            masterKeyAlias,
            EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
            EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
        )
    }

    ...
}
```

# Repository

먼저 인터페이스를 살펴보며 repository에서 어떤 작업을 하는지 파악해보자. `isReady` 함수는 auth가 준비되었는지에 대한 흐름을 반환한다.
이는 세션 토큰을 확인하는 작업보다 로그인 화면이 먼저 띄워지는 것을 막는 방법이다. `getCurrentUser` 함수는 현재 로그인한 유저의 정보의 흐름을 반환한다.
로그인하지 않았다면 `null`이다.

```kotlin
interface AuthRepository {
    /**
     * 세션 토큰 검증과 유저의 자세한 정보를 얻는 작업이 끝난 경우 `true`를 방출한다.
     */
    fun isReady(): Flow<Boolean>

    suspend fun login(userName: String, password: String): Result<Unit>

    suspend fun logout(): Result<Unit>

    fun getCurrentUser(): StateFlow<UserDetail?>
}
```

이제 인터페이스를 구현한 클래스의 초기화 부분을 보자. 초기에는 이전에 로그인하여 저장된 local 데이터가 있는지 확인한다. 있다면 local 세션 토큰의 유효성 검증을 실행한다.
세션 토큰이 유효하다면 응답에 포함된 유저 정보를 이용하여 현재 유저 상태를 갱신하고 토큰값을 변수에 담아둔다. 유효성 검증 결과 유효하지 않다면 로컬 데이터를 지운다.

유효성 검증 등의 작업이 끝나면 `isReady` 흐름에 `true`를 방출하여 Auth 준비가 끝났음을 UI에 알린다.

```kotlin
class AuthRepositoryImpl @Inject constructor(
    private val remoteDataSource: AuthRemoteDataSource,
    private val localDataSource: AuthLocalDataSource,
    private val userRepository: UserRepository
) : AuthRepository {

    private val isReady: MutableStateFlow<Boolean> = MutableStateFlow(false)

    private val currentUserState: MutableStateFlow<UserDetail?> = MutableStateFlow(null)

    private var token: String? = null

    init {
        val localData = localDataSource.getData()
        MainScope().launch {
            if (localData != null) {
                try {
                    val response = remoteDataSource.isSessionValid(localData.sessionToken)
                    val isSessionValid = response.isSuccessful

                    if (isSessionValid) {
                        val userDto = response.body()!!.data.user
                        currentUserState.value = userDto.toDetailEntity()
                        token = localData.sessionToken
                    } else {
                        localDataSource.clear()
                    }
                } catch (e: Exception) {
                    // 인터넷 연결이 끊김 등의 예외는 무시한다.
                }
            }
            isReady.emit(true)
        }
    }

    override fun isReady(): Flow<Boolean> = isReady

    ...
}
```

이번에는 로그인 함수의 구현을 살펴보자. RemoteDataSource를 통해 로그인 API 요청을 보낸다.
응답으로 성공을 받은 경우 현재 유저의 상태 업데이트, 토큰 값 갱신 그리고 로컬 스토리지에 데이터를 저장한다.

```kotlin
override suspend fun login(userName: String, password: String): Result<Unit> {
    assert(token == null) { "이미 로그인한 상태에서 로그인을 시도했습니다." }
    try {
        val response = remoteDataSource.login(
            LoginRequestBody(username = userName, password = password)
        )

        if (response.isSuccessful) {
            val loginResponseData = response.body()!!.data

            currentUserState.value = loginResponseData.user.toDetailEntity()
            token = loginResponseData.sessionToken
            localDataSource.setData(AuthLocalData(sessionToken = token!!))

            return Result.success(Unit)
        } else {
            throw Exception(response.errorMessage)
        }
    } catch (e: Exception) {
        return Result.failure(e)
    }
}
```

이번에는 로그아웃 함수이다. 로그아웃에 성공하면 현재 유저 상태 흐름을 `null`로 바꾸고 토큰 값도 `null`로 바꾼다. 그리고 로컬 스토리지에 저장된 세션 정보를 지운다.

```kotlin
override suspend fun logout(): Result<Unit> {
    assert(token != null) { "로그인하지 않은 상태로 로그아웃을 시도했습니다." }
    return try {
        val response = remoteDataSource.logout(token!!)

        if (response.isSuccessful) {
            currentUserState.value = null
            token = null
            localDataSource.clear()

            Result.success(Unit)
        } else {
            throw Exception(response.errorMessage)
        }
    } catch (e: Exception) {
        Result.failure(e)
    }
}
```

<br>

# UI

나는 시작 activity를 LoginActivity로 지정하였다. 그리고 이미 로그인된 경우 곧바로 HomeActivity로 전환하였다. 자세한 내용을 살펴보자.

## 스플래시 스크린 지연

LoginActivity가 보이기 전에 **이미 로그인 된 경우** HomeActivity로 전환을 해야한다. 이는 Auth가 준비될 때 까지 스플래시 스크린을 지연하는 방법으로 구현할 수 있다.
Auth가 준비된다는 것의 의미는 세션 토큰의 유효성 검증이 완료되었으며, 로그인한 유저의 정보를 성공적으로 얻어왔다는 의미이다.

스플래시 스크린을 지연하는 방법은 [구글의 splash screen 공식 문서](https://developer.android.com/guide/topics/ui/splash-screen#suspend-drawing)에 잘 설명되어있다.

아래는 이를 구현한 것이다. [뷰모델의 UiState가 변경되면 화면을 갱신하는 패턴](https://developer.android.com/jetpack/guide/ui-layer?hl=ko)을 사용하고 있다. Auth가 준비되면 viewModel 내부에서 `hideSplashScreen`가 `false`로 수정되고 스플래시 화면은 종료된다.
만약 uiState가 업데이트되었을 때 로그인 되어있는 상태라면 `navigateToHomeActivity`를 호출하여 홈화면으로 전환한다.

**LoginActivity.kt**

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    ...

    showSplashUntilAuthIsReady()

    lifecycleScope.launch {
        repeatOnLifecycle(Lifecycle.State.STARTED) {
            viewModel.uiState.collect(::updateUi)
        }
    }

    ...
}

private fun showSplashUntilAuthIsReady() {
    val content: View = findViewById(android.R.id.content)
    content.viewTreeObserver.addOnPreDrawListener(
        object : ViewTreeObserver.OnPreDrawListener {
            override fun onPreDraw(): Boolean {
                val hideSplashScreen = viewModel.uiState.value.hideSplashScreen

                return if (hideSplashScreen) {
                    content.viewTreeObserver.removeOnPreDrawListener(this)
                    true
                } else {
                    false
                }
            }
        }
    )
}

private fun updateUi(uiState: LoginUiState) {
    if (uiState.isLoggedIn) {
        navigateToHomeActivity()
    }
    ...
}

private fun navigateToHomeActivity() {
    val intent = HomeActivity.getIntent(this)
    startActivity(intent)
    finish()
}
```

참고로 Auth가 준비되었다고 해서 자동 로그인이 확인된 상태까지 `hideSplashScreen`을 `true`로 바꾸면 안된다. 만약 바꾸게 되면 로그인 화면이 잠깐 보였다가 홈 화면으로 이동하기 때문에 보기 나쁘기 때문이다.
이는 아래처럼 뷰모델 코드를 작성하면 된다.

**LoginViewModel.kt**

```kotlin
init {
    viewModelScope.launch {
        isAuthReadyUseCase().collectLatest { isAuthReady ->
            if (isAuthReady) {
                val isLoggedIn = getCurrentUserUseCase() != null

                _uiState.update {
                    if (isLoggedIn) {
                        // 이미 로그인 되어있을 때는 `hideSplashScreen`를 `true`로 갱신하지 않는다.
                        // 그 이유는 앱을 실행하고 곧바로 홈 화면으로 이동시 짧은 시간동안 로그인 화면이 등장하는
                        // 버그를 막기 위해서이다.
                        // https://github.com/hansung-pocs/blog-android/issues/150
                        it.copy(isLoggedIn = true)
                    } else {
                        it.copy(hideSplashScreen = true)
                    }
                }
            }
        }
    }
}
```

## 로그인 처리

로그인은 다음과 같이 뷰모델에서 처리하고 있다. 로그인에 성공하는 경우 `isLoggedIn`을 `true`로 업데이트하여 홈 화면으로 전환한다.
실패한 경우 `errorMessage`를 전달하여 뷰에서 스낵바를 보이도록 한다.

**LoginViewModel.kt**

```kotlin
fun login() {
    viewModelScope.launch {
        val result = loginUseCase(
            userName = uiState.value.userName,
            password = uiState.value.password
        )
        if (result.isSuccess) {
            _uiState.update {
                it.copy(isLoggedIn = true)
            }
        } else {
            val errorMessage = result.exceptionOrNull()!!.message
            _uiState.update {
                it.copy(errorMessage = errorMessage)
            }
        }
    }
}
```

<br>

# 테스트 작성

설명에 앞서 Hilt 테스트에 관한 내용은 [안드로이드 Hilt 테스트 문서](https://developer.android.com/training/dependency-injection/hilt-testing?hl=ko#replace-binding)를 참고하면 된다.

## AuthRepository 테스트

레파지토리는 data source들의 Fake를 만들어 아래와 같이 테스트하면된다.

```kotlin
@HiltAndroidTest
class AuthRepositoryTest {

    private val testDispatcher = UnconfinedTestDispatcher()

    private val userRepository = FakeUserRepositoryImpl()
    private val remoteDataSource = FakeAuthRemoteDataSource()
    private val localDataSource = FakeAuthLocalDataSource()

    private lateinit var repository: AuthRepositoryImpl

    @Before
    fun setUp() {
        Dispatchers.setMain(testDispatcher)
    }

    @After
    fun tearDown() {
        Dispatchers.resetMain()
    }

    @Test
    fun currentUserIsNull_WhenRunningAppFirstTime() = runTest {
        localDataSource.authLocalData = null

        initRepository()

        assertNull(repository.getCurrentUser().value)
    }

    ...
}
```

주요 repository 테스트 케이스는 아래와 같다.

- 초기화 작업 테스트 케이스

  - 1
    - Given: 로컬 데이터가 없을 때
    - When: 앱 실행시
    - Then: 현재 유저는 `null`이다.
  - 2
    - Given: 로컬 데이터가 있으며 세션이 유효하고 유저 정보 얻기에 성공했을 때
    - When: 앱 실행시
    - Then: 현재 유저는 존재한다.
  - 3
    - Given: 로컬 데이터가 있으나 세션이 유효하지 않을 때
    - When: 앱 실행시
    - Then: 로컬에 세션 정보가 지워진다.

- 로그인 테스트 케이스

  - Given: 로그인에 성공하는 상황에서
  - When: 로그인 시도시
  - Then: 로컬에 세션 토큰 등이 저장된다.

- 로그아웃 테스트 케이스
  - Given: 로그아웃에 성공하는 상황에서
  - When: 로그아웃 시도시
  - Then: 로컬에 세션 정보가 지워진다.

이외에도 더 다양한 케이스가 존재하지만 글에는 생략하겠다. 실제 코드는 [AuthRepositoryTest.kt](https://github.com/hansung-pocs/blog-android/blob/main/data/src/test/java/com.pocs/data/AuthRepositoryTest.kt)에서 확인할 수 있다.

## LoginViewModel 테스트

뷰모델은 FakeAuthRepository를 만들어 테스트하면 된다. 간단한 예로 아래와 같은 테스트 케이스가 존재한다.

- Given: Repository에 현재 유저가 존재하는 상황에서
- When: 뷰모델에서 현재 유저를 갱신했을 때
- Then: `isAuthReady`가 참이다.

실제 코드는 [LoginViewModelTest.kt](https://github.com/hansung-pocs/blog-android/blob/main/presentation/src/test/java/com/pocs/presentation/LoginViewModelTest.kt)에서 확인 가능하다.
