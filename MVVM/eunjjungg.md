
## MVVM

- MVP는 M(Model), V(View), VM(ViewModel)로 이루어져 있음.
    - Model : MVC, MVP 패턴의 모델과 동일. 일반적으로 앱의 데이터 + 상태 + 비즈니스 로직을 묶은 것. 컨트롤러에 묶이지 않으므로 많은 곳에서 재사용할 수 있음.
        - 데이터 : 사용자의 이름, 상품 개수 등과 같은 정보
        - 상태 : UI 상태로 isEnabled, isVisible과 같은 플래그 값으로 나타낼 수도 있고 UI가 갱신되는 경우 이 또한 상태변경에 속함
        - 비즈니스 로직 : 앱에 필요한 동작을 수행하기 위해 데이터를 처리하는 내부 동작들
    - View : 기본적으로 데이터를 보여주는 역할을 하므로 비즈니스 로직이 포함되지 않지만, UI 변경과 같은 일부 로직은 포함될 수 있음. View는 ViewModel을 관찰하다가 상태 변화가 전달되면 화면 갱신을 해줌.
    - ViewModel : View에 필요한 데이터를 Model로부터 가져와서 가공하고 View에 보여주는 View, Model 사이의 가교 역할을 함. 또한 presenter : view가 1 : 1 관계였던 것에 반해 viewmodel : view는 1 : n의 관계임.

![image](https://user-images.githubusercontent.com/100047095/236493395-3d7ae8b7-12c3-445f-9ed3-cab03fb2bad3.png)

- Android에서의 MVVM?
    - MVP pattern에서 model, view 사이의 결합도는 낮췄지만 view, presenter 사이의 결합도는 여전히 높았음. 하지만 MVVM은 Data Binding을 통해 viewmodel, view 사이의 결합도를 낮췄음.

<br/>

### MVVM 장점

- Model ↔ View, ViewModel ↔ View 사이의 의존성이 없어져 유닛테스트가 더 쉬워지게 됨.

<br/>

### MVVM 단점

- 뷰가 변수와 표현식 모두에 바인딩 될 수 있어 규모가 커지면 관련 없은 프레젠테이션 로직이 늘어나게 됨. 그러면 XML에 직접 코드를 넣게 되는데 프로젝트 이해에 악영향을 미칠 수 있음.

<br/>

### MVVM 구현 시 주의점

- ViewModel은 Android Framework가 최대한 없도록 구현되어야 함
- ViewModel은 View의 내부 요소를 참조하거나 View의 context도 사용하면 안됨. 왜냐하면 Viewmodel의 라이프사이클이 activity/fragment의 라이프사이클보다 더 길기 때문임. 메모리 누수가 발생할 수 있음. 따라서 LiveData를 활용해서 observer pattern으로 viewmodel에서 view와 통신하면 됨.
- ViewModel의 mutablelivedata의 접근제한자를 public으로 선언하면 안 됨. 왜냐하면 view에서 이 값을 변경할 수 있기 때문임.
- Activity, Fragment에는 복잡한 로직이 존재해서는 안 됨. (비즈니스 로직이 존재하면 MVVM 설계 패턴에 위배됨)

<br/>

### MVVM ViewModel vs AAC ViewModel

MVVM의 ViweModel은 View, Model 사이에서 데이터를 관리해주고 바인딩해주는 역할임. AAC ViewModel은 Android Lifecycle을 알고 있고 Activity/Fragment가 `onDestroy()` 될 때 데이터를 해제해주는 역할을 함. 따라서 AAC ViewModel은 MVVM ViewModel의 역할을 하기 위해 나온 건 아님. 

하지만 AAC ViewModel을 사용해서 MVVM의 ViewModel 역할을 하도록 할 수 있음. ObservableField, LiveData 등을 사용해서 **데이터 바인딩**을 해준다면 ViewModel로 사용이 가능함.

<br/>

### 👩‍💻 MVVM Pattern이 뭔가요?

Model, View, ViewModel의 줄임말로 Model은 데이터, 상태를 나타내고 View는 ViewModel을 관찰하고 있으며 데이터 갱신을 하고 ViewModel은 View와 관련된 모든 비즈니스 로직을 담당하고 있는 구조입니다. MVP 패턴과 마찬가지로 M-V 사이의 의존성은 없고 MVP 패턴과는 다르게 V-VM 사이의 의존성도 없어 테스트에 용이하다는 장점이 있습니다. 

<br/>

### Reference

- [https://velog.io/@jojo_devstory/안드로이드-아키텍처-패턴-MVVM이-뭘까](https://velog.io/@jojo_devstory/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98-%ED%8C%A8%ED%84%B4-MVVM%EC%9D%B4-%EB%AD%98%EA%B9%8C)
- [https://academy.realm.io/kr/posts/eric-maxwell-mvc-mvp-and-mvvm-on-android/](https://academy.realm.io/kr/posts/eric-maxwell-mvc-mvp-and-mvvm-on-android/)
- [https://blog.gangnamunni.com/post/mvvm_anti_pattern/](https://blog.gangnamunni.com/post/mvvm_anti_pattern/)
- [https://medium.com/@dheerubhadoria/android-mvvm-how-to-use-mvvm-in-android-example-7dec84a1fb73](https://medium.com/@dheerubhadoria/android-mvvm-how-to-use-mvvm-in-android-example-7dec84a1fb73)
- [https://brunch.co.kr/@mystoryg/175](https://brunch.co.kr/@mystoryg/175)
- [https://academy.realm.io/kr/posts/eric-maxwell-mvc-mvp-and-mvvm-on-android/](https://academy.realm.io/kr/posts/eric-maxwell-mvc-mvp-and-mvvm-on-android/)
- [https://medium.com/kenneth-android/android-mvvm-viewmodel과-aac-viewmodel의-차이-8c0d54922e07](https://medium.com/kenneth-android/android-mvvm-viewmodel%EA%B3%BC-aac-viewmodel%EC%9D%98-%EC%B0%A8%EC%9D%B4-8c0d54922e07)
