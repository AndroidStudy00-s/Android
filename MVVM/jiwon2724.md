# MVVM 패턴

**MVVM란?**

MVP 패턴의 단점(View와 Presenter의 1:1 관계로 강한 의존성을 가짐.)을 해결하기 위해 나온 디자인 패턴.

## Model
- 비즈니스 로직을 담당하고 `View`에 보여질 데이터를 나타낸다.
- 데이터에 대한 로직을 정의(DB, Api Call 등)하고 해당 데이터를 핸들링 하는 메서드(함수)를 제공한다.
- 이는 `MVC, MVP 패턴의 Model`과 동일하다.

## View
- 사용자 인터페이스(UI)를 나타내고, 유저의 이벤트가 발생한다.(클릭, 스와이프 등)
- 데이터 변화를 위한 observer를 가지고 있다.
- `ViewModel`로 이벤트를 전송한다.

## ViewModel
- `View`의 추상화된 형태이다. 
- `View`에 보여져야하는 데이터와 메서드(함수)를 가지고 있다.
- `View`가 `ViewModel`을 관찰(observe)하는 형태로 바인딩 되어있어 데이터의 갱신을 자동으로 받을 수 있다.
- 1:1 방식이 아닌 1:N 방식이다.
```kotlin
💡 구글에서 제공하는 안드로이드의 AAC ViewModel과 위에서 설명하는 ViewModel은 완전히 다르다.
```

# 안드로이드에서는 어떻게 사용할까?

![image](https://user-images.githubusercontent.com/70135188/236458355-13d09660-f0e8-4217-8101-7c6b3dd3274b.png)

1. `View`에서 이벤트 발생
2. 이벤트에 따라서 `ViewModel`은 `Model`에 데이터 요청
3. `Model`은 `ViewModel`에 받은 요청을 처리.
4. 처리한 데이터를 `ViewModel`에 전송
5. `ViewModel`는 `View`에게 상태 변화를 알림.
6. `View`는 상태변화에 맞게 UI 갱신


다음은 안드로이드에서 어떻게 MVVM 패턴을 구현하는지 살펴보자.
## MVVM 패턴을 구현하는 안드로이드 코틀린 코드
```kotlin
// 프로필 이름 변경 상황 가정.
// Activity (View)
class MainActivity : AppCompatActivity(), ProfileView {

    private lateinit var mainPresenter: Presenter

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        observe()

        binding.updateNicknameEditTextView.setOnClickListener {
            nicknameViewModel.updateUser(updateNicknameEditTextView.text.toString())
        }
    }
    
    private fun observe() {
        nicknameViewModel.userInfo.observe(this) {
            updateNicknameEditTextView.setText(it.nickname)
        }
    }
}
```

```kotlin
    // ViewModel
class NicknameViewModel() {
    private var _userInfo: MutableLiveData<UserInfo> = MutableLiveData(UserInfo(""))

    val userInfo: LiveData<UserInfo>
        get() = _userInfo

    fun updateUser(name: String) {
        val response = UserModel().update(name) // api 호출
        _userInfo.value = UserInfo(response.name) 
    }
}
```

## 장, 단점

**장점**
- View가 Model사이 의존성이 없다.
- View와 ViewModel이 바인딩되어 코드의 양이 줄어든다.
- 데이터 전달을 간단하게 할 수 있다.


**단점**
- 설계가 어렵고 복잡하다.
- 특정 기술(observe)의 종속되며, 기술에 대한 이해도가 필요하다.




# 1분 답변
MVVM 디자인 패턴은 MVP 디자인 패턴의 단점(View와 Presenter의 1:1관계 -> 의존성이 강함)을 보완하기 위해 탄생한 디자인 패턴입니다. `Model`, `View`, `ViewModel` 세 가지 요소로 나눠 분리되어 동작하는 방식이고, 유저의 이벤트를 `ViewModel`에서 받고
이를 `Model`에 알려 `ViewModel`는 `Model`이 처리한 데이터를 가지고 `View`에 상태 변화를 알립니다. `View`는 변화된 상태에 맞게 UI를 갱신합니다.



