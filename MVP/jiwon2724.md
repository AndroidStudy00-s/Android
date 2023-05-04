
# MVP 패턴

**MVP란?**

MVP 패턴은 MVC 패턴의 의존적인 부분(`View와 Model간`)을 해결하고자 나온 디자인 패턴이다. 구성하는 요소를 `Model`, `View`, `Presenter` 세 가지 요소로 나눠서
서로 분리되어 동작하는 방식이다. 

## Model
- 비즈니스 로직을 담당하고 `View`에 보여질 데이터를 나타낸다.
- 데이터에 대한 로직을 정의(DB, Api Call 등)하고 해당 데이터를 핸들링 하는 메서드(함수)를 제공한다.
- 이는 `MVC 패턴의 Model`과 동일하다.

## View
- 사용자 인터페이스(UI)를 나타내고, 유저의 이벤트가 발생한다.(클릭, 스와이프 등)
- 이벤트 처리를 담당하는 `Presenter`로 전달한다.

## Presenter
- `View`에서 전달받은 이벤트를 감지하여 데이터가 필요한 경우에 `Model`에 요청한다.
- `Model`에서 받은 데이터를 `View`에게 전달한다.

# 안드로이드에서는 어떻게 사용할까?

![image](https://user-images.githubusercontent.com/70135188/236233356-8529f83a-08cc-44ef-9597-7f458389dbc2.png)

1. `View`에서 터치 이벤트 발생
2. `View` -> `Presenter`에 이벤트 전달
3. `Presenter`는 `Model`에 데이터 요청
4. `Model`은 데이터를 local 혹은 remote에서 가져옴
5. 가져온 데이터를 `Presenter`에 전달
6. `Presenter`는 받은 데이터를 정제해서 `View`에 전달
7. `View`는 받은 데이터를 UI에 갱신


다음은 안드로이드에서 어떻게 MVP패턴을 구현하는지 살펴보자.
## MVP 패턴을 구현하는 안드로이드 코틀린 코드
```kotlin
// 프로필 조회 상황 가정.
// Activity (View)
class MainActivity : AppCompatActivity(), ProfileView {

    private lateinit var mainPresenter: Presenter

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        binding.loadProfileButton.setOnclickLisetener {
            Presenter(this).apply {
                getProfile(binding.id.text.toString())
            }
        }
    }
    
    override fun loadProfile(url: String) {
        Glide.with(this).load(url).load(binding.profileImaeView)url
    }
    
    override fun loadNickname(nickname: Stirng) {
        binding.profileTextView.text = nickname
    }
}
```

```kotlin
    // Presenter
    class Presenter(private val view: ProfileView) {
        fun getProfile(id: String) {
            // api 호출
            val response = UserModel().getProfile(id)
            
            view.loadProfile(response.profileUrl)
            view.loadNickname(response.profileNickname)
        }
    }
```


```kotlin
    // interface
    interface ProfileView {
        fun loadProfile(url: String)
        fun loadNickname(nickname: String)
    }
```


MVP 패턴을 살펴봤다. 확실히 MVC 패턴에서의 문제점(View와 Model간 종속성)이 해결된 것으로 보인다.

## 장, 단점

**장점**
- View가 Model에 종속적이지 않아 테스트가 수월하다.
- Presenter를 통해 요소를 분리하여 결합도가 약하다.
- 결합도가 약해 졌으므로 유지보수가 용이하다.


**단점**
- View, Presenter, Model을 각각 구현해야하므로 코드량이 늘어난다.
- View와 Presenter가 1:1 관계이므로, 강한 의존성을 갖는다.
- MVC 패턴에 비해 설계가 복잡하다.





# 1분 답변
MVP 디자인 패턴은 MVC 디자인 패턴의 단점을 보완하기 위해 탄생한 디자인 패턴입니다.. `Model`, `View`, `Presenter` 세 가지 요소로 나눠 분리되어 동작하는 방식입니다. 유저의 이벤트를 `View`에서 받고
이를 `Presenter`에 알려 `Presenter`는 `Model`에 업데이트 요청을 하여 값을 받고, 다시 `View`에 UI를 갱신합니다.




# 출처
https://thdev.tech/androiddev/2016/10/12/Android-MVP-Intro/

https://jminie.tistory.com/168





