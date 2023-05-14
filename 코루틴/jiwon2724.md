# Android의 Kotlin 코루틴
안드로이드 코루틴 공식문서를 참고하고 정리한 포스팅 입니다.

# 코루틴이 뭐야?

코루틴은 실행을 `일시 중단` 하고 `재개` 할 수 있는 비동기 프로그래밍을 위한 라이브러리. 서브루틴을 사용하여 어떤 작업을 하기 위해 함수가 호출이 되고 함수의 동작이 끝나면 자신을 호출했던 메인루틴으로 돌아옴. 

- 경량화된 스레드 : 코루틴을 실행 중인 스레드를 차단하지 않는 기능을 제공하고, 단일 스레드에서 많은 코루틴을 실행할 수 있다. `suspend` 키워드를 사용하여 코루틴간 일시 중단 - 재개를 수행하며 실행된다.
- 구조화된 동시성 : `일시 중단 함수`를 사용하여 비동기식 코드를 동기식으로 구현이 가능하다.
- 기본으로 제공되는 취소 지원 : 실행 중인 코루틴 계층 구조를 통해 자동으로 취소가 전달된다. → 예외가 전파된다.
- Jetpack 라이브러리와 통합 : 많은 Jetpack 라이브러리들은 코루틴을 완전히 지원하는 확장 프로그램이 포함되어 있다.

# 안드로이드에서 코루틴을 사용하는 예제

 구글 권장 아키텍처 가이드에 맞춰 코루틴을 사용하는 예제를 살펴보자.

시나리오는 다음과 같다.

```kotlin
📖 네트워크 요청을 보내고, 결과를 기본 스레드로 반환한다.
```

# 백그라운드 스레드에서 실행

UI 스레드에서 네트워크 요청을 할 경우에 응답을 받을 때까지 스레드가 대기하거나 차단된다. 스레드가 차단되는 경우엔 OS는 UI 작업을 하지 못하며, 앱이 정지되고 ANR이 표시될 수 있다.

```kotlin
// 네트워크 요청을 응답을 모델링하기 위한 Result 클래스
sealed class Result<out R> {
    data class Success<out T>(val data: T) : Result<T>()
    data class Error(val exception: Exception) : Result<Nothing>()
}

class LoginRepository(private val responseParser: LoginResponseParser) {
    private const val loginUrl = "https://example.com/login"

    // 동기식이며, 호출 스레드를 차단한다.
    fun makeLoginRequest(
        jsonBody: String
    ): Result<LoginResponse> {
        val url = URL(loginUrl)
        (url.openConnection() as? HttpURLConnection)?.run {
            requestMethod = "POST"
            setRequestProperty("Content-Type", "application/json; utf-8")
            setRequestProperty("Accept", "application/json")
            doOutput = true
            outputStream.write(jsonBody.toByteArray())
            return Result.Success(responseParser.parse(inputStream))
        }
        return Result.Error(Exception("Cannot open HttpURLConnection"))
    }
}
```

위 코드에서 `makeLoginRequest` 함수를 실행하면 비동기식이 아니기 때문에 호출한 스레드를 차단한다. 이는 네트워크 요청이므로, 메인 스레드에선 실행하면 안된다. 안전을 위해 코루틴을 사용해서 코드를 변경해보자.

```kotlin
class LoginRepository(...) {
    ...
    suspend fun makeLoginRequest(
        jsonBody: String
    ): Result<LoginResponse> {

        return withContext(Dispatchers.IO) {
            // Blocking network request code
        }
    }
}
```

`makeLoginRequest` 를 일시중단 함수로 설정해줬다. 코루틴에서 해당 함수를 호출한 경우 코루틴은 이 함수가 재개될 때 까지 일시정지되고, 다른 코루틴을 실행한다. 

`makeLoginRequest` 함수는 `viewModelScope.launch` 에서 사용된다. 이는 설정을 안해줬을 경우 디폴트는 `Dispatcher.Main`이므로 코루틴 스코프를 설정해주거나, `withContext` 를 사용하여 알맞은 스코프로 변환해줘야 한다.

위 예제에선 `withContext` 를 사용하여 코루틴 컨텍스트를 변경해줌으로 네트워크 작업에 용이하다. `withContext` 의 작업이 끝나면, 요청 결과와 함께 기본 스레드에서 실행을 재개한다.

```kotlin
class LoginViewModel(
    private val loginRepository: LoginRepository
): ViewModel() {

    fun login(username: String, token: String) {

        // Create a new coroutine on the UI thread
        viewModelScope.launch {
            val jsonBody = "{ username: \"$username\", token: \"$token\"}"

            // 작업이 끝날 때 까지 일시정지. (값을 받고 재개)
            val result = loginRepository.makeLoginRequest(jsonBody)

            // Display result of the network request to the user
            when (result) {
                is Result.Success<LoginResponse> -> // Happy path
                else -> // Show error in UI
            }
        }
    }
}
```

# 코루틴의 예외처리

```kotlin
class LoginViewModel(
    private val loginRepository: LoginRepository
): ViewModel() {

    fun makeLoginRequest(username: String, token: String) {
        viewModelScope.launch {
            val jsonBody = "{ username: \"$username\", token: \"$token\"}"
            val result = try {
                loginRepository.makeLoginRequest(jsonBody)
            } catch(e: Exception) {
                Result.Error(Exception("Network request failed"))
            }
            when (result) {
                is Result.Success<LoginResponse> -> // Happy path
                else -> // Show error in UI
            }
        }
    }
}
```

기본적으로 `try ~ catch` 를 사용하여 예외를 처리해준다. result에 담긴 데이터로 `when` 식에서 성공, 실패 시 데이터를 핸들링한다.
# Kotlin 코루틴으로 앱 성능 향상


# 장기 실행 작업 관리

코루틴은 장기 실행 작업을 처리하는 두 작업을 추가하여 일반 함수를 기반으로 빌드된다. `invoke` (또는 `call`) 및 `return` 이외에 `suspend` 와 `resume` 을 추가한다.

- suspend : 현재 코루틴 실행을 일시중지하고 모든 로컬 변수를 저장한다.
- resume : 정지된 위치부터 정지된 코루틴을 재개한다.

```kotlin
suspend fun fetchDocs() {                             // Dispatchers.Main
    val result = get("https://developer.android.com") // Dispatchers.IO for `get`
    show(result)                                      // Dispatchers.Main
}

suspend fun get(url: String) = withContext(Dispatchers.IO) { /* ... */ }
```

위 코드의 시나리오를 살펴보자.

1. `fetchDocs()` 함수를 Dispatcher.Main에서 사용한다고 가정하자. 
2. `result` 변수에 `get()` 함수를 대입하면서 `get()` 함수가 끝나기 전 까지 호출된 코루틴을 일시중단한다.
    1. 네트워크 요청을 시작하기 전에 코루틴을 일시정지함.
3. `get()` 함수는 IO Dispatchers에서 실행되고, 네트워크 요청이 완려되면 콜백을 사용하여 다시 Main으로 돌아가 일시정지된 코루틴을 재개한다.

### 어떻게 이런 작업이 가능할까?

```kotlin
💡 코틀린 컴파일러는 `suspend` 함수를 가져와 상태머신을 사용하여 최적화된 버전의 콜백으로 변환한다. 일시중단지점(`suspend`)을 상태로 나타내며 이런 상태는 컴파일러에 의해 `label` 로 표현되어 `when` 식에서 처리된다. 상태 머신에선 `suspend` 키워드가 붙은 함수의 내부를 처리하고 다른 일시중지 함수를 처리하거나 재개된다. 보통은 마지막 상태(마지막 `label`) 때 다시 재개된다.

```

- 코틀린은 스택 프레임을 사용하여 로컬 변수와 함께 실행 중인 함수를 관리한다. 코루틴을 정지하면 현재 스택 프레임이 복사되고 저장된다.
- 재개되면 스택 프레임이 저장된 위치에서 다시 복사되고 함수가 다시 실행된다.

# 기본 안전을 위해 코루틴 사용

`Dispatchers.XX` 는 코루틴 실행에 사용되는 스레드를 확인한다. → Dispatcher.XX가 코루틴을 해당 스레드에 적재한다. Dispatchers는 코루틴의 재개(`resume`)를 담당한다.

- **Dispatchers.Main** : 기본 Android 스레드에서 코루틴을 실행한다. → UI스레드. 이 디스패처는 UI와 상호작용하고, 빠른 작업을 실행하기 위해서만 사용해야한다. ex) LiveData를 사용하여 UI 업데이트
- **Dispatchers.IO** : 기본스레드 외부에서 디스크(로컬DB → Room, 파일) 또는 네트워크 I/O를 실행하도록 최적화 되어있다.
- **Dispatchers.Default** : CPU를 많이 사용하는 작업을 기본 스레드 외부에서 실행하도록 최적화 되어있다. ex) JSON 파싱, 리스트 정렬

코루틴을 사용하면 세부적인 제어를 통해 스레드를 전달할 수 있다. →  `withContext()` 를 사용하면 콜백을 도입하지 않고도 코드 줄의 스레드 풀을 제어할 수있다.

```kotlin
💡 `withContext` 를 사용해서 모든 함수가 기본적으로 안전한지, 즉 기본스레드에서 함수를 호출할 수 있는지 확인하는 것이 좋다.

`suspend` 가 붙은 일시중단 함수는 일반적으론 UI스레드에서 작동한다. 또한 UI 스레드에서 코루틴을 실행하는 것이 일반적이다. 기본적으로 안전이 요구되는 `withContext` 는 항상 `suspend` 함수 내에서 사용해야 한다.

```

# withContext()의 성능

콜백 기반 구현에 비해 오버헤드를 추가하지 않는다. 일부 상황에선 콜백 기반 구현을 능가하게 최적화할 수 있다. 함수가 네트워크를 10회 호출하는 경우, 외부에서 `withContext` 를 사용하여 스레드를 한 번만 전환하도록 할 수 있다. 그러면 `withContext` 를 여러 번 사용하더라도, 동일한 디스패처(`IO`)에 유지되고 스레드가 전환되지 않는다. 

```kotlin
💡 스레드 풀을 사용하는 디스패처(Dispatchers.IO or Dispatchers.Default)를 사용해도 블록이 처음부터 끝까지 동일한 스레드에서 실행된다는 보장이 없다. 일시중단, 재개가 반복되면서 경우에 따라 다른 스레드에서 이동할 수 있다. → `withContext` 블록에 동일한 값을 가리키지 않을 수 있다.

```

# 코루틴 시작

두 가지 방법으로 코루틴을 시작할 수 있다.

- `launch` : 새 코루틴을 시작하고 결과를 반환하지 않는다. 실행 후 삭제로 간주되는 작업은 `launch` 를 사용하여 시작할 수 있다.
- `async` : 새 코루틴을 시작하고 `await` 이라는 일시중단 함수로 결과를 반환한다.

```kotlin
💡 `launch` 와 `async` 는 예외를 서로 다르게 처리한다. 

```

자세한건 [여기를 참고하세요!](https://medium.com/androiddevelopers/cancellation-in-coroutines-aa6b90163629)

### 병렬 분해

```kotlin
suspend fun fetchTwoDocs() =
    coroutineScope {
        val deferredOne = async { fetchDoc(1) }
        val deferredTwo = async { fetchDoc(2) }
        deferredOne.await()
        deferredTwo.await()
    }

suspend fun fetchTwoDocs() =        // called on any Dispatcher (any thread, possibly Main)
    coroutineScope {
        val deferreds = listOf(     // fetch two docs at the same time
            async { fetchDoc(1) },  // async returns a result for the first doc
            async { fetchDoc(2) }   // async returns a result for the second doc
        )
        deferreds.awaitAll()        // use awaitAll to wait for both network requests
    }

// 컬렉션에서 awaitAll()을 사용할 수 있다.
```

구조화된 동시 실행을 사용하여 코루틴을 정의할 수 있다.  단일 코루틴인 경우 `await` 여러 코루틴일 경우 `awaitAll` 을 사용하여 함수가 반환하기 전에 `async` 를 사용한 코루틴의 작업이 미리 완료되도록 보장할 수 있다.

# 코루틴 개념

### CoroutineScope

`launch` , `async` 를 사용하여 만든 코루틴을 추적한다. 진행 중인 작업, 실행 중인 코루틴은 언제든 `scope.cancel()` 을 호출하여 취소가 가능하다.

안드로이드는 특정 수명 주기 클래스에 자체 `CoroutineScope` 를 제공한다.

ex) `viewModelScope` , `lifecycleScope` 

디스패처와 달리 `CoroutineScope` 는 코루틴을 실행하지 않는다.

```kotlin
💡 `lifecycleScope.launch` 뒤에 launch 함수를 호출하여 코루틴을 실행 시킬 수 있다.
```

### Job

코루틴의 핸들이다. `launch` 또는 `async` 로 만드는 각 코루틴은 코루틴을 고유하게 식별하고, 수명 주기를 관리하는 `Job` 인스턴스를 반환한다.

### CoroutineContext

일련의 요소들을 사용하여 코루틴의 동작을 정의한다.

- `Job` : 코루틴의 수명주기를 제어한다.
- `CoroutineDispatcher` : 적절한 스레드에 작업을 전달한다.
- `CoroutineName` : 디버깅에 유용한 코루틴 이름이다.
- `CoroutineExceptionHandler` : 포착되지 않은 예외를 처리한다.


# Android의 코루틴 권장사항
### 디스패처 삽입
새 코루틴을 만들거나 `withContext`를 호출할 때 `Dispatchers`를 하드코딩하지말자.
```kotlin
// DO inject Dispatchers
class NewsRepository(
    private val defaultDispatcher: CoroutineDispatcher = Dispatchers.Default
) {
    suspend fun loadNews() = withContext(defaultDispatcher) { /* ... */ }
}

// DO NOT hardcode Dispatchers
class NewsRepository {
    // DO NOT use Dispatchers.Default directly, inject it instead
    suspend fun loadNews() = withContext(Dispatchers.Default) { /* ... */ }
}
```
이렇게 사용하는경우 테스트에 용이하게 사용할 수 있다. 

### 일시중단 함수는 기본 스레드(UI)에서 호출할 때 안전해야한다
클래스가 코루틴에서 장기적인 실행에 대하여 차단 작업을 실행하는 경우 `withContext`를 사용하여 기본(UI) 스레드에서 실행을 이동하는 역할을 한다.

### ViewModel은 코루틴을 만들어야 한다
```kotlin
class LatestNewsViewModel(
    private val getLatestNewsWithAuthors: GetLatestNewsWithAuthorsUseCase
) : ViewModel() {
    // DO NOT do this. News would probably need to be refreshed as well.
    // Instead of exposing a single value with a suspend function, news should
    // be exposed using a stream of data as in the code snippet above.
    suspend fun loadNews() = getLatestNewsWithAuthors()
}
```
ViewModel을 사용하면 비즈니스 로직을 실행하기 위해 일시 중단 함수를 노출하는 대신 코루틴을 만들게 된다. View에서 직접 코루틴을 트리거하여 비즈니스 로직을 실행하면 안된다.
해당 작업은 ViewModel에게 맡기자.

### 변경 가능한 타입 노출하지 않기
```kotlin
// DO expose immutable types
class LatestNewsViewModel : ViewModel() {

    private val _uiState = MutableStateFlow(LatestNewsUiState.Loading)
    val uiState: StateFlow<LatestNewsUiState> = _uiState

    /* ... */
}

class LatestNewsViewModel : ViewModel() {

    // DO NOT expose mutable types
    val uiState = MutableStateFlow(LatestNewsUiState.Loading)

    /* ... */
}
```
변경할 수 없는 유형을 다른 클래스에 노출하는 것을 지양하자. 이러한 방식으로 변경 가능한 유형에 대한 모든 변경 사항은 하나의 클래스에 집중되어 문제가 발생했을 때 디버깅하기가 더 쉽다.

### 데이터, 비즈니스 레이어는 일시중단함수와 Flow를 노출하자
구글 권장 아키텍처와 클린 아키텍처 기반 관점에서 차이점 확인해보기.

### 비즈니스 및 데이터 레이어에서 코루틴 만들기 
구글 권장 아키텍처와 클린 아키텍처 기반 관점에서 차이점 확인해보기.

### 테스트에 TestDispatcher 삽입하기

### GlobalScope 피하기 
- 제어가 되지 않는 범위에서 코드가 실행되므로 어려워지고, 실행을 제어할 수 없음.
- GlobalScope는 싱글톤으로 만들어져있다. 잘못 사용한다면 프로그램 전체에 영향을 미칠 수 있음.

### 코루틴을 취소 가능하게 만들기

### 예외에 주의
코루틴에서 발생하는 예외를 처리하지 않으면 앱이 비정상 종료될 수 있다. 예외가 발생할 가능성이 있다면 `viewModelScope` 또는 `lifecycleScope`를 사용하여
만든 코루틴의 본문에서 예외를 포착하자.


# 출처
- https://developer.android.com/kotlin/coroutines?hl=eng
- https://developer.android.com/kotlin/coroutines-adv?hl=eng

# 정리 포스팅
- [Android의 Kotlin 코루틴](https://dev-jiwon.notion.site/Android-Kotlin-3faa1c1298ec477b95d9f56a63b72b20)
- [Kotlin 코루틴으로 앱 성능 향상](https://dev-jiwon.notion.site/Kotlin-250f33532e214793b23d1c52becb3838)



