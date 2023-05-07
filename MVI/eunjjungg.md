## MVI

- MVI는 M(Model), V(View), I(Intent)로 이루어져 있음.
    - Model : 다른 패턴들과 다르게 상태만 의미함. 변경이 불가능한 데이터.
        - 상태 : UI 상태로 isEnabled, isVisible과 같은 플래그 값으로 나타낼 수도 있고 UI가 갱신되는 경우 이 또한 상태변경에 속함
    - View : UI 자체. Activity, Fragment, Compose 등이 됨.
    - Intent : 사용자의 액션 및 시스템 이벤트에 따른 결과. 이벤트라고 생각!
- reactive programming을 위한 도구로 reactive programming은 변수 값의 변화에 따라 앱이 반응하여 변경되는 것을 말함.
- 기존 패턴들의 Model은 API로부터 받아오는 데이터들이 주로 담겼는데 MVI에서의 모델은 데이터보다는 State를 나타냄. 여러 클래스에서 모델을 관리할 필요가 없어지고 Model이 다 관리하고 상태를 V에 전달할것임.
- MVI의 Model은 불변성을 가짐. 따라서 여러 클래스에서 모델이 수정되지 않고 모델 객체가 새로 생성되게 됨. → 앱의 전체 라이프사이클동안 단일 상태를 유지할 수 있음.
- MVI는 View는 화면 렌더링 목적인 render() 함수를, 유저의 행동을 반응하기 위한 intent() 함수를 가지고 있음. 따라서 state에 따라 render()를 통해 화면을 렌더링해줌. 그리고 intent 함수도 유저의 액션에 따라 수행해주어야 하는 함수를 연결해주면 되는 것 같음.
- state reducer는 과거 상태에 기반하여 새로운 상태를 만들 때 과거 상태의 변경사항을 유지하며 새로운 상태를 만드는 것.

<br/>

### MVI 장점

- 데이터의 플로우가 단방향으로 이루어짐.
- 따라서 → View의 생명주기 동안 일관성 있는 상태를 가짐.

<br/>

### MVI 단점

- RX에 대한 지식이 필요함.
- 작은 변경도 intent로 처리해주어야 하고 보일러플레이트 코드의 양산이 일어날 수 있음.

<br/>

### 👩‍💻 MVI Pattern이 뭔가요?

![image](https://user-images.githubusercontent.com/100047095/236664694-09e9a85c-0cee-4606-a568-3022ea9a557e.png)

Model, View, Intent로 이루어져 있으며 다른 패턴들과 다르게 모델은 UI의 상태 값만 의미합니다. 이 Model의 상태값으로 View가 갱신되고 Model의 상태 값을 새로 만들기 위해서는 Intent가 필요합니다. Intent는 따라서 유저의 액션 혹은 시스템 이벤트에 따른 결과를 의미합니다. MVI의 장점은 데이터의 플로우가 단방향으로 이루어지고 View의 생명주기 동안 일관성있는 상태를 가진다는 점입니다. 

<br/>

### Reference

- [https://jaehochoe.medium.com/번역-안드로이드를-위한-mvi-model-view-intent-아키텍쳐-튜토리얼-시작하기-165bda9dfbe7](https://jaehochoe.medium.com/%EB%B2%88%EC%97%AD-%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C%EB%A5%BC-%EC%9C%84%ED%95%9C-mvi-model-view-intent-%EC%95%84%ED%82%A4%ED%85%8D%EC%B3%90-%ED%8A%9C%ED%86%A0%EB%A6%AC%EC%96%BC-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-165bda9dfbe7)
- [https://velog.io/@jshme/MVI-Architecture-for-Android](https://velog.io/@jshme/MVI-Architecture-for-Android)
