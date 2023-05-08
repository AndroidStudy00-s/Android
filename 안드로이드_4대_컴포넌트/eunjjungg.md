
## 앱 구성 요소

- Activity
- Service
- Broadcast Receiver
- Content Provider

<br/>

### Activity

- 사용자와 상호작용하기 위한 진입점
- ex. 이메일 앱 안에도 여러 액티비티가 존재. 이메일 리스트 표시 액티비티 + 이메일 작성 액티비티 + 특정 이메일을 읽는 액티비티… 이렇게 있을 수 있음. 이 액티비티들은 각각 독립되어 있고, 이메일 앱이 허용한다면 다른 앱인 카메라 앱에서 액티비티 중 하나를 실행할 수 있음.
- 2개 이상의 activity를 동시에 디스플레이 불가능.
- 한 개 이상의 뷰, 뷰그룹을 포함함.
- Intent를 통해 다른 앱의 액티비티 호출 가능.

<br/>

### Service

- 백그라운드에서 앱을 계속 실행하기 위한 다목적 진입점.
- 백그라운드에서 실행되는 구성 요소로 오랫동안 실행되는 작업을 수행하거나 원격 프로세스를 위한 작업을 수행.
- ex. 백그라운드에서 일부 데이터를 동기화하거나 사용자가 앱에서 나간 후에도 음악을 재생하는 서비스.
- 눈에 보이지 않아도 별도의 스레드가 아닌 메인 스레드에서 동작.
- 애플리케이션이 종료되어도 이미 시작된 서비스는 백그라운드에서 계속 동작함.

<br/>

### Broadcast Receiver

- 앱이 시스템 전체의 Broadcast 알림에 응답할 수 있게 함. → 안드로이드 OS로부터 발생하는 이벤트와 정보를 받아와 핸들링하는 컴포넌트임.
- 현재 실행되지 않은 앱에도 시스템이 브로드캐스트를 전달할 수 있음.
- 각 브로드캐스트는 Intent 객체로 전달됨.

<br/>

### Content Provider

> *콘텐츠 제공자*는 파일 시스템, SQLite 데이터베이스, 웹상이나 앱이 액세스할 수 있는 다른 모든 영구 저장 위치에 저장 가능한 앱 데이터의 공유형 집합을 관리합니다
> 
- 데이터를 관리하고 다른 애플리케이션의 데이터를 제공하는데 사용됨.
- 특정 애플리케이션이 사용하고 있는 데이터베이스를 공유하기 위해 사용.
- 애플리케이션 간의 데이터 공유를 위한 표준화된 인터페이스를 제공함.
- 시스템이 구성요소를 시작할 때 그 앱에 대한 프로세스를 시작 → 해당 구성요소에 필요한 클래스를 인스턴스화 → 구성요소 시작! 이렇게 실행되기 때문에 Android는 단일 진입 지점이 없음.

<br/>

### 각 구성 요소들 활성화 : Intent

- Activity, Service, Broadcast Receiver는 Intent로 실행됨.
- Activity, Service에서 실행되는 Intent는 수행할 작업 정의 + 작업을 수행할 데이터의 URI를 지정함.
- Broadcast Receiver의 Intent는 단순히 브로드캐스트할 알림만 정의함.

<br/>

### Reference

- [https://velog.io/@jojo_devstory/안드로이드-Android-4대-컴포넌트](https://velog.io/@jojo_devstory/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-Android-4%EB%8C%80-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8)
- [https://developer.android.com/guide/components/fundamentals?hl=ko](https://developer.android.com/guide/components/fundamentals?hl=ko)
