## Activity Lifecycle

<img width="762" alt="image" src="https://github.com/AndroidStudy00-s/Android/assets/100047095/9c1de143-9cdb-4264-a083-94a1aec81507">

**activity class에서 제공하는 6가지 콜백**

- onCreate()
- onStart()
- onResume()
- onPause()
- onStop()
- onDestroy()

<br/>

### onCreate()

- 이 콜백은 시스템이 먼저 activity를 생성할 때 실행됨.
- 필수적으로 구현해주어야 함.
- activity의 생명주기 동안 한 번만 발생해야 하는 로직 여기서 실행.
    - e.g. viewmodel과 관련된 데이터를 list에 binding
    - e.g. class-scope 변수들 instantiate
- parameter로 `savedInstanceState`를 받음. 이는 activity의 이전 저장 상태가 포함된 Bundle 객체로 처음 생성된 activity의 경우 `null` 값이 들어감.
- 주로 setContentView()의 param으로 해당하는 layout의 xml 파일을 전달하기도 하지만 View 객체를 사용해서 새로운 View를 ViewGroup에 넣어서 뷰 계층 구조를 빌드할 수 있음.
- create 상태가 되면 lifecycle components들은 `ON_CREATE` event를 받음.
- `onCreate()` 메소드 이후에는 시스템이 이어서 `onStart()`, `onResume()`을 호출함.

<br/>

### onStart()

- activity가 start 상태에 들어가면 이 콜백이 호출됨.
- onStart()가 호출되면 사용자에게 activity가 표시되고 앱은 activity를 foregorund로 보내 상호작용할 수 있도록 준비함.
- start 상태가 되면 lifecycle components들은 `ON_START` event를 받음.
- onStart() 메소드는 빠르게 완료되고 onResume() 메소드를 호출함.

<br/>

### onResume()

- activity가 resume 상태에 들어가면 foreground에 표시되고 이 콜백이 호출됨.
- resume 상태에서 사용자가 앱과 상호작용함.
- 다른 event가 발생하여 앱에서 포커스가 떠날 때가지 resume 상태에 머무르게 됨. event가 발생하면 `onPause()` 콜백을 호출함.
    - 다른 event의 예시 : 전화 or 사용자가 다른 activity로 이동 or 기기 화면이 꺼짐
- resume 상태가 되면 lifecycle components들은 `ON_RESUME` event를 받음.

아래 코드는 ON_RESUME 이벤트를 수신하는 lifecycle component의 예시임.

```kotlin
class CameraComponent : LifecycleObserver {

    ...

    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    fun initializeCamera() {
        if (camera == null) {
            getCamera()
        }
    }

    ...
}
```

<br/>

### onPause()

- 사용자가 activity를 떠나는 첫 번째 신호로 이 메소드가 호출됨
- `onPause()`가 실행되었다고 해도 `onDestroy()`로 무조건 이어지는 건 아님.
- 멀티윈도우를 사용할 때 열려있는 앱 중 하나의 앱만 포커스를 가질 수 있기 때문에 그 외에는 모두 pause 상태에 들어가게 됨.
- 그외에도 Dialog가 열려있을 때도 이전 Activity가 부분적으로 보이지만 pause 상태가 유지됨.
- pause 상태가 되면 lifecycle components들은 `ON_PAUSE` event를 받음.
- resume에 초기화가 되면 pause에서 해제해주는 것이 좋음.
- pause에서 activity가 다시 시작되면 `onResume()`을 호출함.
- `onPuase()`는 아주 잠깐 실행되므로 부하가 많은 종료 작업은 onPause()에서 수행되면 안 됨. 대신 `onStop()`에서 실행해주어야 함.
- activity가 pause → resume 상태로 돌아가면 시스템은 activity의 인스턴스를 activity 메모리에 넣어두고 시스템이 onResume() 호출할 때 인스턴스를 다시 호출함.
- activity가 사용자에게 완전히 보이지 않게 되면 `onStop()` 호출

<br/>

### onStop()

- activity가 사용자에게 더 이상 보이지 않으면 onStop()으로 가게 됨.
- stop 상태가 되면 lifecycle components들은 `ON_STOP` event를 받음.
- `onStop()`을 사용해서 CPU를 비교적 많이 소모하는 종료 작업을 수행해야 함.
- stop 상태가 되면 activity 객체는 메모리 안에 머무르게 됨. 활동이 다시 시작되면 이 객체를 다시 호출함.
- stop 상태에서 다시 시작되거나 실행을 종료하고 사라지게 됨. 다시 시작되면 `onRestart()`를 호출하고 종료되면 `onDestroy()`를 호출하게 됨.

<br/>

### onDestroy()

- activity가 소멸되기 전에 호출됨.
- 호출 조건
    - 사용자가 activity를 완전히 닫거나 activity에서 finish()가 호출되어 activity가 종료되는 경우
    - 구성 변경(e.g. 기기 회전 or 멀티 윈도우 모드)으로 인해 activity가 일시적으로 소멸되는 경우
- destroy 상태가 되면 lifecycle components들은 `ON_DESTROY` event를 받음.
- 기기 회전과 같은 구성 변경으로 `onDestroy()`가 호출되면 시스템은 즉시 새 activity 인스턴스를 생성하고 이 새로운 인스턴스에 대해 `onCreate()`을 호출함.

---

## Fragment Lifecycle

![image](https://github.com/AndroidStudy00-s/Android/assets/100047095/da1727e3-d4ee-459e-827b-a816dbe636ef)

<br/>

### onAttach()

- fragment가 액티비티에 붙을 때 호출

<br/>

### onCreate()

- fragment를 생성할 때 시스템에서 호출함.
- fragment가 일시정지되거나 중단 되었다가 다시 실행할 때 유지하고자 하는 것을 초기화해주어야 함.
- Bundle로 액티비티로부터 데이터가 넘어옴
- UI 초기화 불가능

<br/>

### onCreateView()

- 시스템이 프래그먼트가 자신의 UI를 처음으로 그릴 때 호출함. 즉, layout inflate 담당.
- param의 savedInstanceState로 이전 상태에 대한 데이터 제공 가능함.
- return 값이 View?인데 UI를 그리면 view를 반환하고 없다면 null 값을 반환하면 됨.

![image](https://github.com/AndroidStudy00-s/Android/assets/100047095/ed81c03f-66e1-4d77-8ca9-5cb3602b740b)

<br/>

### onViewCreated()

- onCreateView()의 리턴 값인 View 객체는 이 함수의 param으로 전달됨.
- 이 콜백에서 lifecycle의 상태가 initialized로 업데이트가 되므로 view의 초기값 설정, livedata 옵저빙, recyclerview 어댑터 세팅 등은 이 메소드에서 해주는 것이 좋음.

<br/>

### onViewStateRestored()

- 저장해둔 모든 state 값이 fragment의 view 계층 구조에 복원되었을 때 호출
- lifecycle이 Initialized → Created로 변경

<br/>

### onStart()

- 사용자에게 보여질 수 있을 때 호출
- lifecycle이 Created → Started로 변경

<br/>

### onResume()

- 사용자가 fragment와 상호작용할 수 있을 때 호출
- lifecycle이 Started → Resume으로 변경

<br/>

### onPause()

- user가 fragment를 떠난다는 것을 나타내는 신호. 하지만 여전히 fragment는 visible
- 사용자 세션을 넘어서 지속되어야 하는 변경 사항을 커밋함. 왜냐하면 이 콜백이 호출되었을 때 사용자가 다시 돌아온다는 것을 보장할 수 없기 때문.
- lifecycle이 Resume → Started로 변경

<br/>

### onStop()

- fragment가 더 이상 화면에 보이지 않게 되면 호출
- lifecycle이 Started → Created로 변경
- fragment transition을 안전하게 수행할 수 있는 마지막 지점

<br/>

### onDestroyView()

- 모든 exit animation, transaction이 완료되고 Fragment가 화면으로부터 벗어났을 때 호출
- lifecycle이 Created → Destroyed로 변경
- GC에 의해 수거될 수 있도록 Fragment에 대한 참조가 제거되어야 함.

<br/>

### onDestroy()

- Fragment가 제거되거나 FragmentManager가 destroy 되면 호출

### Reference

- [https://velog.io/@evergreen_tree/Android-프래그먼트-생명주기](https://velog.io/@evergreen_tree/Android-%ED%94%84%EB%9E%98%EA%B7%B8%EB%A8%BC%ED%8A%B8-%EC%83%9D%EB%AA%85%EC%A3%BC%EA%B8%B0)
- [https://developer.android.com/guide/components/activities/activity-lifecycle?hl=ko](https://developer.android.com/guide/components/activities/activity-lifecycle?hl=ko)
- [https://developer.android.com/guide/components/fragments?hl=ko](https://developer.android.com/guide/components/fragments?hl=ko)
