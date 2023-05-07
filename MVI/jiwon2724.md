# MVI


MVI란?

`Model`, `View`, `Intent`로 MVVM에서 다수의 입출력으로 인해 `상태 문제`와 `부수효과`를 해결하기 위해 나온 디자인패턴이다. UDF(단방향 데이터 흐름)
를 사용하며, 기존 디자인 패턴과는 다른방식으로 작동한다.

```kotlin
여기서 Intent는 안드로이드의 Intent가 아니다. MVI의 Intent는 밑에서 설명하겠다.
```



### 상태 문제
화면에 나타나는 모든 정보, 프로그레스바 상태, 버튼 활성화 등 상태를 관리하기 힘들어지고, 의도하지 않은 방향으로 제어가 된다면 이를 상태 문제라고 부른다.
ex) 서버의 응답으로 리스트를 성공적으로 출력했지만, 프로그레스바가 계속 보이는 상황

### 부수 효과
안드로이드는 무수한 부수 효과들로 이루어져 있다. 서버 호출, DB접근 등 어떤 결과를 얻을지 예상할 수 없다. 그에 따라 상태 변경에 어려움을 느낌.
위 문제점들을 해결하기 위해 MVI는 어떤 방식으로 동작할까?

# MVI의 동작방식
- Model : UI에 반영될 단일 상태(State)를 의미하고, 데이터 플로우가 단방향이라 불변성을 보장해야한다.
- View : Activity, Fragment를 나타낸다. UI 그 자체이다.
- Intent : 사용자의 Action과, 그에 따른 결과이다. 즉, 사용자가 취하는 행위이다.

![image](https://user-images.githubusercontent.com/70135188/236680934-ea4d23a9-cff4-4265-a66e-e26e6bc03cbb.png)

예를 들어 유저가 새로고침 버튼을 통해서 화면 목록 갱신을 한다면, 유저의 Action(새로고침 변경)이 model(State)의 변경을 알리고, 새롭게 도출된
model(State)이 view에 렌더링 되는 구조이다. 즉, 유저의 Action에 따라 상태가 변경된다.


![image](https://user-images.githubusercontent.com/70135188/236681140-247820a7-030f-438b-9f8a-ad12cf4e810f.png)


위에서 설명했듯이 MVI는 UDF(Uni-directional-flow)이므로 위 그림처럼 표현된다. Intent로 부터 Model을 새로 생성하는 구조이므로 예측 가능한
상태를 설정할 수 있고, 이로 인해 디버깅이 쉬워진다.

# 어떻게 구현해야 할까?
다른 MVXX 디자인 패턴과는 다르게, MVI의 I(Intent)는 앱의 상태를 바꾸려는 의도(intent)를 의미한다. 다른 MVXX 패턴들 과는 다르게
구조적인 컴포넌트에 할당하지 않았다. -> 즉, 다른 아키텍처와는 다른 시각을 갖고 있고 이는 앱을 어떤 구조로 구성하는 것 보단, 앱의 상태(State)와 
데이터의 흐름을 어떻게 다룰지에 대한 기술이다.
```kotlin
실제로 MVI패턴은 MVP, MVVM 기반으로 작성한 예제들이 많았다. 그만큼 MVI 패턴을 구현하기 위한 방법은 열려있다. -> 유연하다
```

![image](https://user-images.githubusercontent.com/70135188/236682744-d963602d-f527-4124-a550-8f678e2f8819.png)

위 사진은 MVVM 패턴에서 MVI가 어느 계층에 속하는지 보여준다.


# Side Effects
![image](https://user-images.githubusercontent.com/70135188/236684887-0a9cbc49-188e-47a1-abb4-382a778d109a.png)

`View(Model(Intent()))` 구조로 잘 순환하길 기대하지만 실제론 다양한 부수적인 효과가 있다. 상태를 변경할 필요가 없는 이벤트가 필요
할수 도 있기 때문. ex) Activity/Fragment의 이동, 토스트 노출 등이 부수적인 효과에 해당한다.

그렇기 때문에 MVI를 언급할 땐 일반적으로 Side Effects라는 개념을 사용해서 이를 처리한다.



# MVI 패턴을 구현하는 안드로이드 코틀린 코드
```kotlin
코드 구현해보고 작성할 것. Side Effects 포함
```


# MVI의 장단점
장점
 - 상태 관리가 쉽다
 - 데이터가 단방향으로 흐른다
 - 디버깅 및 테스트가 쉽다

단점
- 러닝커브가 가파르다 (공부하면서 느꼈다.)
- 보일러 플레이트 코드가 양산된다
- 작은 변경도 Intent로 처리해야한다.










# 출처 
https://www.charlezz.com/?p=46365

https://velog.io/@jshme/MVI-Architecture-for-Android

https://medium.com/myrealtrip-product/android-mvi-79809c5c14f0

https://medium.com/myrealtrip-product/android-mvi-79809c5c14f0
