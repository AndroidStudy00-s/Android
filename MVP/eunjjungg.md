
## MVP

![image](https://user-images.githubusercontent.com/100047095/236237776-4c2b5339-e86f-4a3b-a0f2-03238f8c8fef.png)
- MVP는 M(Model), V(View), P(Presenter)로 이루어져 있음.
    - Model : MVC 패턴의 모델과 동일. 일반적으로 앱의 데이터 + 상태 + 비즈니스 로직을 묶은 것. 컨트롤러에 묶이지 않으므로 많은 곳에서 재사용할 수 있음.
        - 데이터 : 사용자의 이름, 상품 개수 등과 같은 정보
        - 상태 : UI 상태로 isEnabled, isVisible과 같은 플래그 값으로 나타낼 수도 있고 UI가 갱신되는 경우 이 또한 상태변경에 속함
        - 비즈니스 로직 : 앱에 필요한 동작을 수행하기 위해 데이터를 처리하는 내부 동작들
    - View : MVC 패턴의 View와 동일(안드로이드는 조금 다름). 모델의 **표현**. UI를 그리고 사용자가 앱과 상호작용할 때 컨트롤러와 통신하는 책임을 맡음.
    - Presenter : Model과 상호작용하고 View에 UI 갱신을 요청함. View-Model 사이의 가교 역할을 함. MVC 패턴과는 다르게 View와 직접적으로 연결되는 건 아니지만 그래도 View, Model과의 의존성이 남아있음.
- 흐름은 다음과 같음. 
View에서 user action이 일어남 → Presenter로 전달 → Presenter에서 로직을 처리 → Model로의 접근이 필요한 경우 수행 → Presenter에 View 업데이트 요청 → View 갱신
- Android에서의 MVP?
    - Android에서의 MVC와는 달리 Activity, Fragment가 View에만 속하게 됨.
    - UI 갱신이 필요한 경우 MVC에서는 View, Controller 역할을 하는 Activity가 자체적으로 갱신을 해줬다면 MVP에서는 Presenter가 View에 UI 갱신 요청을 하게 됨.

![image](https://user-images.githubusercontent.com/100047095/236237809-34cfff3c-2b46-4fa6-b23d-05398adbdfcc.png)
- 극단적인 MVP 패턴에서는 Model, Presenter에서는 안드로이드 API를 사용하지 않아야 한다고 함. 따라서 Model, Presenter 둘 다 모듈화되어 Testable해지게 됨.

<br/>

### MVP 장점

- Model과 View 사이의 결합도는 낮출수 있어 새로운 기능 추가 시 필요한 부분만 코드 추가가 가능해져 확장성이 좋음.
- Presenter, Model이 Testable해짐!

<br/>

### MVP 단점

- View, Presenter 사이의 결합도가 높아짐.
- 로직이 많아지면 많아질수록 Presenter의 코드가 비대해짐.

<br/>

### 👩‍💻 MVP Pattern이 뭔가요?

모델, 뷰, 프레젠터로 이루어져 있으며 프레젠터에서 Model과 상호작용을 하고 뷰 갱신이 필요한 경우 뷰에 요청을 하여 뷰가 자체적으로 데이터를 갱신하는 구조입니다. 따라서 프레젠터는 모델과 뷰 사이를 잇는 가교 역할을 한다고도 볼 수 있습니다. MVC 패턴에 비해 View와 Model 사이의 결합도는 줄었지만 View와 Presenter 사이의 결합도는 여전히 높습니다. 

<br/>

### Reference

- [https://github.com/taeiim/Android-Study/blob/master/study/week13/AndroidMVP/Android-MVP.md](https://github.com/taeiim/Android-Study/blob/master/study/week13/AndroidMVP/Android-MVP.md)
- [https://brunch.co.kr/@mystoryg/171](https://brunch.co.kr/@mystoryg/171)
- [https://academy.realm.io/kr/posts/eric-maxwell-mvc-mvp-and-mvvm-on-android/](https://academy.realm.io/kr/posts/eric-maxwell-mvc-mvp-and-mvvm-on-android/)
- [https://www.kodeco.com/books/advanced-android-app-architecture/v1.0/chapters/7-model-view-presenter-theory](https://www.kodeco.com/books/advanced-android-app-architecture/v1.0/chapters/7-model-view-presenter-theory)
