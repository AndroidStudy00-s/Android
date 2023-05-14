## Coroutine 공식 문서 읽기

> *코루틴*은 비동기적으로 실행되는 코드를 간소화하기 위해 Android에서 사용할 수 있는 동시 실행 설계 패턴입니다.
> 

코루틴의 기능

- 경량 : 코루틴을 실행 중인 스레드를 차단하지 않는 suspension을 지원하므로 단일 스레드에서 많은 코루틴을 실행할 수 있음.
- memory leak 감소 : 구조화된 동시 실행을 사용하여 범위 내에서 작업을 실행함. (구조화된 동시성 : 새로운 코루틴은 코루틴의 수명을 결정하는 특정 CoroutineScope에서만 시작할 수 있음 → 하나의 스레드에서 여러개의 코루틴 실행 가능.)
    - 용어 설명 - CoroutineScope : 모든 코루틴은 범위 내에서 실행해야 합니다. `CoroutineScope`는 하나 이상의 관련 코루틴을 관리합니다.
- cancel 기본 지원 : job.cancle()로 취소가 가능함

<br/>

### 백그라운드 스레드에서 실행

메인 스레드에서 네트워크 request를 보내면 response를 받을 때까지 차단됨. → ANR이 발생할 수 있음. 따라서 백그라운드에서 네트워크 관련 요청을 보내는 것이 맞음. 

아래 코드는 네트워크 관련 요청을 작성한 코드임. 안드로이드 기본 웹 네트워크 라이브러리인 HttpURLConnection을 사용하고 있음. 

```kotlin
// response 타입 정의
sealed class Result<out R> {
    data class Success<out T>(val data: T) : Result<T>()
    data class Error(val exception: Exception) : Result<Nothing>()
}

class LoginRepository(private val responseParser: LoginResponseParser) {
    private const val loginUrl = "https://example.com/login"

		// 아래 함수가 동기식이며 호출한 스레드를 차단하게 됨. 
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

```kotlin
class LoginViewModel(
    private val loginRepository: LoginRepository
): ViewModel() {

		// 네트워크 요청을 보낼 때 UI 스레드(= 메인 스레드)를 차단하게 됨
		// -> 새로운 코루틴을 만들고 I/O 스레드에서 작업해야 함
    fun login(username: String, token: String) {
        val jsonBody = "{ username: \"$username\", token: \"$token\"}"
        loginRepository.makeLoginRequest(jsonBody)
    }
}
```

```kotlin
class LoginViewModel(
    private val loginRepository: LoginRepository
): ViewModel() {

    fun login(username: String, token: String) {
				// 수정해서 I/O 스레드에서 작업
        viewModelScope.launch(Dispatchers.IO) {
            val jsonBody = "{ username: \"$username\", token: \"$token\"}"
            loginRepository.makeLoginRequest(jsonBody)
        }
    }
}
```

결국 마지막 코드의 `login` 함수가 핵심으로, 코루틴을 사용해서 네트워크 작업을 처리하고 있음. 

<br/>

**코드 설명**

- `viewModelScope`는 ViewModel KTX extension에 포함된 사전 정의된 코루틴 스코프임.
- `launch` is a function that creates a coroutine and dispatches the execution of its function body to the corresponding dispatcher. → `launch`는 코루틴을 만들고 이를 dispatcher에 전달하는 역할을 함

> **Dispatcher chatGPT에게 물어보기**
> 
> 
> 안드로이드에서 `Dispatcher`는 Kotlin 코루틴 라이브러리에서 제공하는 클래스로, 백그라운드 스레드에서 실행되는 코루틴을 관리하고 실행할 수 있도록 합니다.
> 
> `Dispatcher`는 코루틴이 실행되는 스레드를 제어할 수 있는 방법을 제공합니다. 예를 들어, `Dispatchers.Main`을 사용하면 UI 스레드에서 코루틴을 실행할 수 있습니다. `Dispatchers.IO`를 사용하면 네트워크 요청이나 파일 입출력과 같은 I/O 작업을 처리하기에 적합한 백그라운드 스레드에서 코루틴을 실행할 수 있습니다.
> 
> 안드로이드에서는 `Dispatcher`를 사용하여 백그라운드 스레드에서 실행되는 작업을 쉽게 관리할 수 있습니다. 이를 통해 UI 스레드에서 블로킹되는 작업을 백그라운드 스레드에서 처리하고, 애플리케이션의 반응성을 향상시킬 수 있습니다.
> 
> 또한, `Dispatcher`는 기본적으로 코루틴이 실행되는 스레드를 관리하지만, 개발자가 직접 `CoroutineContext`를 지정하여 다른 스레드에서 코루틴을 실행할 수도 있습니다. 이러한 방식으로 `Dispatcher`는 안드로이드 애플리케이션에서 코루틴을 효율적으로 사용할 수 있도록 도와줍니다.
> 
- **Dispatchers.IO**는 코루틴이 I/O 작업으로 예약된 스레드에서 실행해야 함을 나타냄


<br/>

**코드 흐름**

1. 앱이 main 스레드의 View layer에서 login 함수를 호출함.
2. login 함수 내부의 launch가 새 코루틴을 만들고 해당 코루틴은 I/O용 스레드에서 독립적으로 네트워크 요청이 이루어짐.
3. 코루틴이 실행되는 동안 네트워크 요청이 완료되기 전에 login 함수가 계속 실행되어 결과를 반환함. 
    1. 코루틴 내에서 리턴 잘못 쓰면 초기값 반환되는 것의 이유인듯. 

<br/>


### Use coroutines for main-safety

main-safe function : main 스레드에서 UI 업데이트를 block하지 않는 함수

메인 스레드에서 `LoginRepository().makeLoginRequest()`를 호출하면 UI가 block 됨. 즉 저 함수는 main safety하지 않는 함수. 따라서 메인 스레드에서 다른 스레드로 이동할 필요가 있음. 이때 `withContext()` 함수를 사용하게 됨. 

```kotlin
class LoginRepository(...) {
    ...
    suspend fun makeLoginRequest(
        jsonBody: String
    ): Result<LoginResponse> {

        // Move the execution of the coroutine to the I/O dispatcher
        return withContext(Dispatchers.IO) {
            // Blocking network request code
        }
    }
}
```

위 코드에서 `withContext(Dispatchers.IO)`는 코루틴 실행을 I/O 스레드로 이동하여 호출 함수를 main safe 함수로 만듦. 또한 `suspend` 키워드가 fun 앞에 붙었는데 이는 코루틴 내에서 함수가 호출되도록 강제하는 Kotlin 언어의 기능임. 


<br/>

> **withContext()의 필요성 부분이 잘 이해가 가지 않아 chatGPT에게 질문**
> 
> 
> Kotlin 코루틴을 사용하여 백그라운드 스레드에서 비동기 작업을 수행하면, 결과를 처리하기 위해 UI 스레드에서 실행되는 코드를 작성해야 할 수도 있습니다. 이 경우, 메인 스레드 안전성을 고려해야 합니다. 이를 위해 안드로이드에서는 `withContext(Dispatchers.Main)` 함수를 사용하여 메인 스레드에서 실행되는 코드 블록을 만들 수 있습니다.
> 
> **makeLoginRequest 함수는 main 스레드에서 실행되어도 안전한가요?**
> 
> `makeLoginRequest` 함수 내에서 수행되는 블로킹 네트워크 요청 코드가 I/O 디스패처에서 실행되므로, `makeLoginRequest` 함수 자체는 메인 스레드에서 실행되어도 안전합니다.
> 
> 하지만, `makeLoginRequest` 함수가 반환하는 `Result<LoginResponse>` 객체를 처리하는 코드는 다르게 동작할 수 있습니다. 예를 들어, `Result` 객체를 처리하는 코드가 UI 업데이트를 수행하거나, 긴 작업을 수행하는 경우에는 메인 스레드에서 실행될 경우 ANR(Application Not Responding) 오류가 발생할 수 있습니다.
> 
> 따라서, `makeLoginRequest` 함수에서 반환하는 `Result<LoginResponse>` 객체를 처리하는 코드는 메인 스레드에서 실행되지 않도록 주의해야 합니다. 일반적으로, UI 업데이트와 같은 작업은 메인 스레드에서만 수행되므로, `withContext(Dispatchers.Main)` 함수를 사용하여 적절한 디스패처에서 실행하도록 해야 합니다.
> 


<br/>

**ViewModel 개선**

아래 코드는 LoginViewModel에 코루틴을 넣어서 response를 처리하고 있음. 

```kotlin
class LoginViewModel(
    private val loginRepository: LoginRepository
): ViewModel() {

    fun login(username: String, token: String) {

        // Create a new coroutine on the UI thread
        viewModelScope.launch {
            val jsonBody = "{ username: \"$username\", token: \"$token\"}"

            // Make the network call and suspend execution until it finishes
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

login 함수를 보면 `LoginRepository().makeLoginRequest()`가 `suspend` 키워드가 붙은, 코루틴을 강제하는 함수이므로 코루틴이 필요함. (이 부분은 기존과 동일) 하지만 다른 점은 아래와 같음.

1. `launch(Dispatchers.IO) {…} → launch {…}`
    1. `Dispatchers.IO`로 전달해주지 않으면 `viewModelScope`에서 실행된 코루틴은 main 스레드에서 실행되게 됨. 
    2. 위와 같이 변경이 가능했던 건 `makeLoginRequest` 함수에서 네트워크 관련 작업을 `withContext(Dispatchers.IO)`로 처리해주었기 때문. 따라서 결과를 받으면 결과에 따라 UI 작업을 스레드를 변경하지 않고 UI 스레드인 메인 스레드에서 처리해줄 수 있음. 
2. 네트워크 요청의 결과를 sealed class Result로 처리하여 UI 변화가 일어남. 

<br/>

**ViewModel 개선 코드 설명**

1. main 스레드의 View layer에서 login() 함수를 호출.
2. viewModel의 login()에서는 launch가 메인 스레드에서 네트워크 요청을 보낼 새 스레드를 만들고 실행.
3. ⭐️ 코루틴 내부에서 `loginRepository.makeLoginRequest()` 호출은 `makeLoginRequest()`의 `withContext` 블록 실행이 끝날 때까지 코루틴의 추가 실행을 suspend함!
4. `loginRepository.makeLoginRequest()` 내부의 `withContext` 블록이 완료되면 `login()`의 코루틴이 네트워크 요청의 결과를 메인 스레드에서 다시 처리하게 됨. 

<br/>


### 예외 처리

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

network 통신 부분의 error를 잡는 try-catch와 result 결과가 원하는 Success가 아닐 때의 에러는 다름. 그러나 같이 처리하도록 result에 Result.Error를 리턴해주고 있음. 그러면 when 절에서 같이 처리가 가능함. 

<br/>

### 장기 실행 작업 관리

suspend : 현재 코루틴 실행을 일시중지하고 모든 로컬 변수를 저장함.

resume : 정지된 위치부터 정지되었던 코루틴을 계속 실행함.

`suspend fun`은 다른 `suspend` 함수에서 호출하거나 코루틴 빌더(ex. `launch`)를 사용하여 새 코루틴을 시작하는 방법으로만 호출할 수 있음. 

```kotlin
suspend fun fetchDocs() {                             // Dispatchers.Main
    val result = get("https://developer.android.com") // Dispatchers.IO for `get`
    show(result)                                      // Dispatchers.Main
}

suspend fun get(url: String) = withContext(Dispatchers.IO) { /* ... */ }
```

위 코드를 내 식대로 해석해보면 fetchDocs 함수는 별다른 콘텍스트를 외부에서 주지 않는 이상 메인스레드에서 실행됨. 그런데 get 함수는 withContext(Dispatchers.IO)에서 실행되므로 IO 작업용 스레드에서 실행되게 됨. 

공식 문서를 다시 보면 아래와 같음.

> 이 예에서 `get()`은 여전히 기본 스레드에서 실행되지만 네트워크 요청을 시작하기 전에 코루틴을 정지합니다. 네트워크 요청이 완료되면 `get`은 콜백을 사용하여 기본 스레드에 알리는 대신 정지된 코루틴을 재개합니다.
> 

⭐️ 네트워크 요청이 완료되면 get 함수는 콜백을 사용해서 기본 스레드에 알리는 대신 정지된 코루틴을 재개함. 왜 그럴까? withContext 키워드에 대해 알아보면 알 수 있음.

withContext의 기능

- 코루틴의 실행 컨텍스트를 바꿀 때 사용.
- 현재 코루틴이 실행중인 컨텍스트에서 다른 컨텍스트로 전환하여 { … } 안을 실행할 수 있음.

<br/>

### Main-safety를 위한 코루틴 사용

코루틴은 dispatcher를 사용하여 코루틴 실행에 사용되는 스레드를 확인함. 또한 dispatcher는 자체적으로 suspend할 수 있는 기능을 가진 코루틴을 재개하는 기능도 있음. 

<br/>

**dispatcher 종류**

- Dispatchers.Main : 이 디스패처로 기본 안드로이드 스레드에서 코루틴을 실행할 수 있음. UI와 상호작용하거나 빠른 작업을 실행하기 위해서만 사용해야 함.
- Dispatchers.IO : 이 디스패처는 기본 스레드 외부에서 디스크 or 네트워크 I/O를 실행하도록 됨.
- Dispatchers.Default : 이 디스패처는 CPU를 많이 사용하는 작업을 기본 스레드 외부에서 실행하도록 **최적화**되어 있음. e.g. 목록 정렬 or JSON 파싱

<br/>

**fecthDocs - get 개선 코드**

```kotlin
suspend fun fetchDocs() {                      // Dispatchers.Main
    val result = get("developer.android.com")  // Dispatchers.Main
    show(result)                               // Dispatchers.Main
}

suspend fun get(url: String) =                 // Dispatchers.Main
    withContext(Dispatchers.IO) {              // Dispatchers.IO (main-safety block)
        /* perform network IO here */          // Dispatchers.IO (main-safety block)
    }                                          // Dispatchers.Main
}
```

<br/>

**test** 

```kotlin
suspend fun main() { // main
    get() //IO
    print("main last line") // main
}

suspend fun get() { // main
    withContext(Dispatchers.IO) { // IO
        delay(2000L) // IO
        println("withContext") // IO
    }
    println("get last line") // main
}
```

실행 흐름은 다음과 같음. main() → get() → delay(200L) → println(”withContext”) → println(”get last line”) → print(”main last line”)

![image](https://github.com/AndroidStudy00-s/Android/assets/100047095/9d8573f7-d9b4-4210-b363-d920a0a7736d)
→ 그렇다고 한다! suspend는 별다른 명시를 하지 않는 이상 main 스레드에서 작동하고 main에서 하는게 일반적임. 하지만 main safe가 요구되는 withContext는 다른 스레드 풀에서 실행되도록 해야하며 suspend 함수 내부에서 사용해야 함. 

<br/>

**withContext()의 성능**

![image](https://github.com/AndroidStudy00-s/Android/assets/100047095/7616b4d3-2af2-4865-be30-34b27ff84b22)

콜백 기반 구현(콜백 헬로 이어지는…)에 비해 오버헤드를 추가하지 않음. → 이해 잘 안 됨. 나중에 다시 정리. 

<br/>

### 코루틴 시작

- `launch` : 새 코루틴을 시작하고 호출자에게 결과를 반환하지 않음. 실행 후 삭제로 간주되는 모든 작업을 이 키워드로 시작하면 됨.
- `async` : 새 코루틴을 시작하고 awiat이라는 정지 함수로 결과를 반환하도록 허용함. 다른 코루틴 내부에서만 사용하거나 suspend fun 내부에서 병렬 분해를 실행할 때 사용함. (여러 suspend 함수 async로 걸어놓는것 말하는듯)

<br/>

**병렬 분해**

suspend function에 의해 시작되는 모든 코루틴은 함수가 반환되면 중지되어야 함. → 따라서 그 전에 코루틴이 완료되도록 보장해야 함. how? 코루틴을 `coroutineScope`를 통해 시작하고 `await()` or `awaitAll()`을 사용하여 함수를 반환하기 전에 이런 코루틴이 완료됨을 보장할 수 있음.  

```kotlin
suspend fun fetchTwoDocs() =        // called on any Dispatcher (any thread, possibly Main)
    coroutineScope {
        val deferreds = listOf(     // fetch two docs at the same time
            async { fetchDoc(1) },  // async returns a result for the first doc
            async { fetchDoc(2) }   // async returns a result for the second doc
        )
        deferreds.awaitAll()        // use awaitAll to wait for both network requests
    }
```

위 코드에서 awaitAll()은 deferreds에 넣어놓은 모든 deffered들이 완료되기 전까지 기다림. 하지만 coroutineScope 빌더는 모든 새 코루틴이 완료될 때까지 재개하지 않음. 

<br/>

### CoroutineScope

launch or async를 사용해서 만든 코루틴을 추적할 수 있음. 실행중 코루틴은 scope.cancle()로 취소할 수 있음. android에는 viewModelScope, lifecycleSope가 있음. 하지만 디스패처와 달리 CoroutineScope는 코루틴을 실행하지 않음. 

<br/>

### Job

> A Job is a handle to a coroutine.
> 

launch, async로 만든 각 코루틴은 이를 고유하게 식별하고 수명주기를 관리할 수 있는 Job 인스턴스를 반환함. 

<br/>


### CoroutineContext

CoroutineContext는 아래 요소를 사용해 코루틴을 정의함

- Job : 코루틴의 수명주기 제어
- CoroutineDispatcher : 적절한 스레드에 작업을 전달
- CoroutineName : 디버깅에 유용한 코루틴의 이름
- CoroutineExceptionHandler : 포착되지 않은 코루틴 예외 처리

<br/>


### Reference

- [https://developer.android.com/kotlin/coroutines?hl=ko](https://developer.android.com/kotlin/coroutines?hl=ko)
- [https://developer.android.com/kotlin/coroutines-adv?hl=ko](https://developer.android.com/kotlin/coroutines-adv?hl=ko)
