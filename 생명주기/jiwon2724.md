# Activity Lifecycle(생명주기)

유저가 앱을 사용하고, 나가고 다시 들어오면 Activity는 수명주기 안에서 서로 다른 상태를 통해 전환된다. Activity 클래스는 상태 변화를 알아차릴 수 있는 여러 콜백을 제공한다.

### 개념

Activity Class는 6가지 콜백을 제공한다. Activity가 새로운 상태에 들어가면 시스템은 각 콜백을 호출한다. 

![image](https://github.com/AndroidStudy00-s/Android/assets/70135188/6d33a9f4-bfc4-4474-9271-3e2311e3e3e0)


- Activity에 복잡도에 따라 모든 생명 주기 메서드를 구현할 필요가 없다.
- 각각의 생명 주기 메서드를 이해하고, 유저가 예상한 대로 앱이 동작하도록 필요한 생명 주기 메서드를 구현하는 것이 중요하다.

## 생명 주기 콜백

공통 : 해당 콜백 메서드를 받는 생명 주기를 인식하는 구성요소는 콜백 이벤트를 수신한다.

- LifecycleScope, repeatOnLifecycle 등

### onCreate()

- 시스템이 먼저 Activity를 생성할 때 실행되는 메서드로 필수적으로 구현해야함.
- Activity가 생성되면 생성됨(Created)상태가 된다.
- 전체 생명주기 동안 한 번만 발생해야 하는 로직을 실행한다.
    - ex) ViewModel과 연결
- onCreate()는 `savedInstanceState` 매개변수를 수신한다. 이는 Activity 이전 저장 상태가 포함된 `Bundle` 객체이다. 처음 생성된 Activity인 경우 `Bundle` 객체의 값은 null이다.
- 이 메서드가 실행을 완료하면 시작됨(Started)상태가 되고, 시스템이 연달아 `onStart()` 와 `onResume()` 메서드를 호출한다.

### onStart()

- Activity가 시작됨(Started)상태에 들어가면 시스템은 `onStart()` 를 호출한다.
- Activity가 사용자에게 표시되고, 앱은 Activity를 foreground에 보내 상호작용할 수 있도록 준비한다.
    - ex) UI를 관리하는 코드를 초기화
- `onStart()` 메서드는 매우 빠르게 완료되고, 생성됨(Created)상태와 마찬가지로 Activity는 시작됨(Started)상태에 머무르지 않고, 이 콜백이 완료되면 Activity는 재개됨(Resumed)상태에 들어가고, 시스템이 `onResume()` 메서드를 호출한다.

### onResume()

- Activity가 재개됨(Resumed) 상태에 들어가면 foreground에 표시되고, 시스템이 `onResume()` 메서드를 콜백한다.
- 앱이 유저와 상호작용하는 상태이다.
- 이벤트가 발생하여 앱에서 포커스가 떠날 때 까지 해당 이 상태(Resumed)에 머무른다.
    - 이벤트 ex) 다른 화면으로이동, 전화, 화면 꺼짐 등
- 방해되는 이벤트가 발생되면 Activity는 일시중지됨(Paused)상태에 들어가고, 시스템이 `onPause()` 메서를 호출한다.
- 일시중지됨 상태에서 재개됨 상태로 돌아오면 시스템은 `onResume()` 메서드를 다시 한번 호출한다.
- `onResume()` 을 구현하여 `onPause()` 중에 해제하는 구성요소를 초기화하고, 재개됨 상태로 전환될 때마다 필요한 다른 초기화 작업도 수행해야함.

```kotlin
멀티 윈도우 모드에선 Activity가 일시중지됨(Paused)상태에 있더라도 완전히 보일 수 있다.
```

### onPause()

- 시스템은 사용자가 Activity에 포커스를 잃을 때 나타내는 첫 번째 신호로 이 메서드를 호출한다.
    - Activity가 foreground에 있지 않다는 것을 나타냄.
    - 멀티 윈도우 모드에선 표시될 수 있음.
- 일시중지됨 상태일 때 계속 실행되어선 안되지만 잠시 후 다시 시작할 작업을 일시 중지하거나 조정한다.

Activity가  `onPause()` 에 들어가는 이유는 여러가지 이유가 있다.

- 일부 이벤트가 앱 실행 방해
- 새로운 반투명 Activity(ex : Dialog) : Activity는 부분적으로 보이지만 포커스 상태가 아닌 경우엔 일시중지됨 상태로 유지된다.
- `onPause()` 는 아주 잠깐 실행되므로, 저장 작업을 실행하기엔 시간이 부족할 수 있다. 유저의 데이터를 저장하거나 네트워크 호출 등 이러한 작업을 실행해선 안된다.

### onStop()

- Activity가 유저에게 더 이상 표시되지 않으면 중단됨(Stopped)상태에 들어가고, 시스템은 `onStop()` 콜백을 호출한다.
- 새로 시작된 Activity가 화면 전체를 차지할 경우 적용된다.
- 시스템은 Activity의 실행이 완료되어 종료될 시점에 `onStop()` 을 호출할 수도 있다.
- `onStop()` 메서드는 앱이 사용자에게 보이지 않는 동안 필요하지 않은 리소스를 해제하거나 조정해야한다.
    - ex) 애니메이션 일시 중지 등
- CPU를 비교적 많이 소모하는 종료 작업을 실행해야한다.
    - ex) 데이터베이스에 정보 저장
- Activity는 정지됨 상태에서 다시 시작되어 유저와 상호작용 하거나, 실행을 종료하고 사라진다.
    - 다시 시작되는 경우엔 시스템은 `onRestart()` 를 호출한다.
    - Activity가 실행을 종료하면 시스템은 `onDestroy()` 를 호출한다.

```kotlin
Activity가 중단됨 상태에 들어가면 Activity 객체는 메모리 안에 머무르게 된다. Activity가 다시
시작되면, 이 정보를 다시 호출한다. 현재 상태가 재개됨 상태임 콜백 메서드 중 생성된 구성요소는
다시 초기화할 필요 없다. 시스템은 View 객체의 현재 상태도 기록한다.
```

### onRestart()

- `onStop()` 에서 Activity가 유저에게 다시 표시될 때 호출된다.
- `onRestart()` → `onStart()` → `onResume()` 순서로 재개된다.

### onDestroy()

- Activity가 소멸되기 전에 호출된다. 시스템은 다음 중 하나에 해당할 때 `onDestroy` 를 호출함.
    - 유저가 Activity를 완전히 닫거나, Activity 에서 `finish()` 가 호출되어 종료되는 경우
    - 기기 회전이나 멀티 윈도우로 구성 변경이 일어나 시스템이 일시적으로 Activity를 소멸시키는 경우
- Activity가 종료되는 경우 `onDestroy()` 는 Activity가 수신하는 마지막 생명 주기 콜백 메서드이다.
- 구성 변경으로 `onDestroy()` 가 호출되는 경우, 시스템이 즉시 새로운 Activity 인스턴스를 생성한 다음, 새로운 구성이 반영된 Activity에 `onCreate()` 를 호출한다.
- 이전 콜백에서 아직 해제되지 않는 모든 리소스를 해제해야한다.
    - ex) `onStop()` 에 저장된 리소스

# Fragment Lifecycle(생명주기)
![image](https://github.com/AndroidStudy00-s/Android/assets/70135188/53b48cb7-1c56-4553-8237-4580b2aef48f)

옆 그림은 액티비티가 실행중일 때 프래그 먼트의 생명주기이다. 액티비티와 다소 유사하지만, 다른점만 몇가지 살펴보겠다. onCreate() ~ onDestory()는 Activity와 동일하다.

### onAttatch()

- Fragment가 Activity에 부착되었을 때 호출되는 메서드이다.

### onCreateView()

- 시스템은 Fragment가 UI를 처음으로 그릴 때 `onCreateView()` 가 호출된다.
- Fragment에 맞는 UI를 그리려면 메서드에서 View를 반환해야한다.
    - `onCreateView()` 는 Fragment 레이아웃의 루트이다.
    - Fragment가 UI를 제공하지 않는 경우 null을 반환하면 된다.

### onViewCreated()

- `onCreateView()` 에서 UI가 다 그려지면 호출되는 메서드이다.

### onDestroyView()

- Fragment의 UI 요소들이 제거되는 시점에 호출된다.
- Fragment가 다른 프래그먼트로 대체될 때, Activity가 백스택에서 제거될 때 호출됨.
- `onDestoryView()` 가 호출된 이후엔 `onDestroy()` 가 호출되어 Fragment가 완전히 제거된다.
- `onDestoryView()` 가 Fragment가 완전히 종료되는 것이 아니라, UI 요소들만 제거함.
- UI 요소들의 상태를 저장하고, 다시 생성될 때 해당 상태를 복원하는 작업을 수행해야 한다.

### onDetach()

- Fragment가 Activity에 완전히 분리될 때 호출된다.
- Fragment가 참조하고 있는 데이터들에 대한 참조를 모두 해제 해야한다.


# 출처
- https://developer.android.com/guide/components/activities/activity-lifecycle?hl=ko
- https://developer.android.com/guide/components/fragments?hl=ko#Creating

# 정리 포스팅
- [Activity Lifecycle](https://dev-jiwon.notion.site/Activity-Lifecycle-d0fa93cb0c144f58a56642a37540cb98)
- [Fragment Lifecycle](https://dev-jiwon.notion.site/Fragment-Lifecycle-e4bb11a93bd64b52aa1696ac39a164cb)


