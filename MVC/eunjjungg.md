## MVC
![image](https://user-images.githubusercontent.com/100047095/235994680-13dcaed3-e2b9-44d7-8467-bf79ea372773.png)

- 지금 나와있는 MVP, MVVM 패턴은 MVC를 초석으로 모듈화되고 테스터블하게 나온 패턴.
- MVC는 M(Model), V(View), C(Controller)로 이루어져 있음.
    - Model : 일반적으로 앱의 데이터 + 상태 + 비즈니스 로직을 묶은 것. 컨트롤러에 묶이지 않으므로 많은 곳에서 재사용할 수 있음.
        - 데이터 : 사용자의 이름, 상품 개수 등과 같은 정보
        - 상태 : UI 상태로 isEnabled, isVisible과 같은 플래그 값으로 나타낼 수도 있고 UI가 갱신되는 경우 이 또한 상태변경에 속함
        - 비즈니스 로직 : 앱에 필요한 동작을 수행하기 위해 데이터를 처리하는 내부 동작들
    - View : 모델의 **표현**. UI를 그리고 사용자가 앱과 상호작용할 때 컨트롤러와 통신하는 책임을 맡음. MVC 구조에서의 뷰는 해당 뷰와 연관된 모델에 대한 이해가 전혀 없음. 따라서 사용자의 입력이 발생하더라도 어떻게 처리해야 하는지 모름.
    - Controller : 사용자가 버튼을 눌렀다고 알리면 컨트롤러는 그에 따라 어떻게 모델과 상호작용할지 결정함. 모델에서 데이터가 변화되면 그에 따라 컨트롤러는 뷰의 상태를 업데이트할 수 있음. 안드로이드 앱에서는 주로 activity, fragment가 controller 역할.
- Android에서의 MVC?
    - View, Control이 Activity, Fragment와 같은 뷰들이 수행함. 웹에서의 MVC 패턴은 Model, View, Controller가 모두 구분되어 있음. 그러나 안드로이드는 View, Controller가 합쳐져 있음. (그러나 xml을 View로 보는 사람들도 있음)
    - 사용자 이벤트가 발생해서 UI 갱신을 한다고 했을 때 Android에서의 흐름은 다음과 같음. Activity(View, Controller)가 사용자 이벤트 감지 → Model로부터 데이터 갱신이 필요한지 확인 → Controller는 Model로부터 전달 받은 데이터를 통해 View의 업데이트 필요한지 확인 → View 갱신

<br/>

### MVC 내가 만든 코드에서 살펴본다면?

링크 : [https://github.com/AndroidStudy00-s/AS-eunjjungg/tree/W1-MVC](https://github.com/AndroidStudy00-s/AS-eunjjungg/tree/W1-MVC)

Model : 초기값 저장하고 +1, -1에 대한 로직 처리

View, Controller : 값이 바뀔 때마다 UI 갱신 

<br/>

### MVC 장점

- 코드 파악이 쉬워짐

<br/>

### MVC 단점

- 테스트 용이성이 떨어짐. 컨트롤러가 안드로이드 API와 깊게 종속됨. 따라서 유닛테스트가 어려움.
- 모듈화 및 유연성의 한계. 컨트롤러가 뷰와 강력하게 결합되어 뷰를 변경하면 컨트롤러로 돌아가서 변경도 해주어야 함.
- 유지보수 시 어려움. 시간이 지날수록 컨트롤러가 비대해짐.

<br/>

### 👩‍💻 MVC Pattern이 뭔가요?

모델, 뷰, 컨트롤러로 이루어져 있으며 일반적으로 안드로이드에서는 액티비티와 프래그먼트가 View, Controller의 역할을 수행합니다. 모델에서는 데이터, 비지니스 로직, 상태가 묶여있으며 컨트롤러는 모델 값이 바뀌면 뷰의 상태를 업데이트할 수 있습니다. 

<br/>

### Reference

- [https://academy.realm.io/kr/posts/eric-maxwell-mvc-mvp-and-mvvm-on-android/](https://academy.realm.io/kr/posts/eric-maxwell-mvc-mvp-and-mvvm-on-android/)
- [https://thdev.tech/androiddev/2016/10/23/Android-MVC-Architecture/](https://thdev.tech/androiddev/2016/10/23/Android-MVC-Architecture/)
- [https://brunch.co.kr/@mystoryg/170](https://brunch.co.kr/@mystoryg/170)
- [https://www.kodeco.com/books/advanced-android-app-architecture/v1.0/chapters/2-model-view-controller-theory](https://www.kodeco.com/books/advanced-android-app-architecture/v1.0/chapters/2-model-view-controller-theory)
