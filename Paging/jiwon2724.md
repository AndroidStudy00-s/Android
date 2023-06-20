# Paging3

Paging3 라이브러리를 사용하면 로컬DB(Room, SQLite)나 네트워크를 통해 대규모 데이터 페이지를 로드하고 표시할 수 있다!

즉, 시스템 리소스를 모두 더 효율적으로 사용이 가능.

```
💡 데이터의 전체를 불러오기 < 페이징으로 불러오기
```

### Paging3를 사용하면 얻을 수 있는 이점

- 페이징된 데이터의 메모리 내 캐싱이 가능하다. → 앱이 페이징 데이터로 작업하는 동안 시스템 리소스를 효율적으로 사용이 가능하다.
- 중복 제거 기능이 기본적으로 제공된다.
- RecyclerView를 끝까지 스크롤할 때 어댑터가 자동으로 데이터를 요청한다.
- Coroutine, Flow, LiveData들과 호환성이 좋다.
- 새로고침 및 재시도 기능을 포함하며 오류 처리를 기본으로 지원한다.

---

<aside>
💡 기존엔 RecyclerView 스크롤 끝을 감지해서 네트워크 호출하여 처리를 해줬는데, 다방면으로 좋은 기능들이 많다.

</aside>

### Paging3 Library 추가하기

```kotlin
dependencies {
  def paging_version = "3.1.1"

  implementation "androidx.paging:paging-runtime:$paging_version"

  // alternatively - without Android dependencies for tests
  testImplementation "androidx.paging:paging-common:$paging_version"

  // optional - RxJava2 support
  implementation "androidx.paging:paging-rxjava2:$paging_version"

  // optional - RxJava3 support
  implementation "androidx.paging:paging-rxjava3:$paging_version"

  // optional - Guava ListenableFuture support
  implementation "androidx.paging:paging-guava:$paging_version"

  // optional - Jetpack Compose integration
  implementation "androidx.paging:paging-compose:1.0.0-alpha17"
}
```

`build.gradle` 에 종속항목을 추가해주고 라이브러리를 사용하면 된다.

안드로이드 권장 아키텍처에서 사용되니 AAC와 MVVM 패턴 숙지해야함.

## Paging 라이브러리의 핵심 구성요소

- `PagingSource` : 특정 페이지 쿼리의 Data Chunk(한번에 하나씩 데이터를 읽어 Chunk라는 덩어리를 만든후 Chunk단위로 트랜잭션. 커밋 사이에 처리되는 row의 수)를 로드하는 기본 클래스이다. 데이터 레이어(Data Layer)의 일부이고, 일반적으로 `DataSource` 클래스에서 노출되고 이후에 `ViewModel` 에서 사용하기 위해 `Repository` 에 노출된다.
- `PagingConfig` : 페이징 동작을 결정하는 클래스. 페이지의 크기, placeholder 등등의 사용여부 설정
- `Pager` : `PagingData` 스트림을 생성하는 클래스. `PagingSource` 에 따라 다르게 실행되며 `ViewModel` 에서 만들어야 한다.
- `PagingData` : 페이지로 나눈 데이터의 컨테이너, 데이터를 새로고침할 때마다 자체 `PagingSource` 로 상응하는 `PagingData` 내보내기가 별도로 생성된다. → 페이지로 나눈 데이터의 스냅샷(데이터를 포착하여 보관)을 보유하고 있음
- `PagingDataAdapter` : RecyclerView에 PagingData를 표시하는 Apdapter의 서브클래스. `PagingDataAdapter` 는 내부 `PagingData` 로드 이벤트를 수신 대기하고, 페이지가 로드될 때 UI를 효율적으로 업데이트한다.

```
메모리에 로드된 모든 항목을 StateFlow에 유지한다. 이는 데이터가 너무 커지면 성능에영향을 미친다.

데이터가 변경됐을 때 리스트에서 기사 하나 이상을 업데이트 하는 작업은 리스트가 클수록 비용이 더 많이 든다.

Paging3 라이브러리는 이 모든 문제를 해결하는 동시에 점진적으로 데이터를 가져오는 일관된 API를 제공한다.
```

### 페이징을 구현할 때 확인해야하는 조건

- UI의 데이터 요청을 올바르게 처리하여 동일한 쿼리에 여러요청 금지.
- 관리가 가능한 양의 데이터를 가져오고 메모리에 유지.
- 이미 가져온 데이터를 보완하기 위해 추가 데이터를 가지고오는 요청을 트리거.

→ `PagingSouce` 를 사용하면 위 작업을 모두 수행이 가능하다!

### PagingSource 빌드시 정의해야하는 항목

1. 페이징 키의 유형 : 특정 아이템의 ID. ID가 정렬되고, 증가한다고 보장된다. DB에서 PK느낌
2. 로드된 데이터의 유형 : 해당 아이템의 리스트를 반환한다.
3. 데이터를 가지고오는 위치 : 로컬DB, 네트워크 리소스 → Data Layer

### 어떤 메커니즘인지?

![image](https://github.com/AndroidStudy00-s/Android/assets/70135188/53db6c52-5585-4c84-a100-6478f335c297)


Repository(PagingSource, RemoteMediator(학습예정))와 PagingConfig 정보를 토대로 Pager를 통해 PagingData를 만들고, 해당 인스턴스를 PagingDataAdapter가 활용하여 UI를 그리는 메커니즘이다.

### PagingSource 필수 재정의 함수

```kotlin
override suspend fun load(params: LoadParams<Int>): LoadResult<Int, Dataclass>
override fun getRefreshKey(state: PagingState<Int, Dataclass>): Int?
```

1. `load()`

```kotlin
override suspend fun load(params: LoadParams<Int>): LoadResult<Int, Article> {
    valstart=params.key ?:STARTING_KEY
    valrange=start.until(start+params.loadSize) 

    if(start!=STARTING_KEY) delay(LOAD_DELAY_MILLIS)

    return LoadResult.Page(
        data=range.map { number->
            Article(
                id =number,
                title = "Article $number",
                description = "This des article $number",
                created =firstArticleCreateTime.minusDays(number.toLong())
            )
        },
        prevKey= when(start) {
            STARTING_KEY-> null
            else -> ensureValidKey(key=range.first -params.loadSize)
        },
        nextKey=range.last + 1
    )

    // 사용자가 스크롤할 떼 표시할 더 많은 데이터를 비동기식으로 가져오는 함수
    // LoadParams 객체에는 다음 항목을 불러오는 정보가 저장된다.

    // 로드할 페이지의 키 : load() 함수가 처음 호출되는 경우 params.key는 null이다.
    // params.key가 null일 경우에 초기 페이지의 키를 정의해야한다. 보통 0으로 설정하는듯.

    // Load size : 요청된 항목의 수

    // 이 함수는 LoadResult를 반환한다. LoadResult는 다음 유형 중 하나이다.
    // 1. LoadResult.Page : load를 성공한 경우
    // 2. LoadResult.Error : 오류가 발생한 경우
    // 3. LoadResult.Invalid : PagingSource가 더 이상 결과의 무결성을 보장안하므로 무효화되어야 하는 경우

    // LoadResult.Page에는 세 가지 필수 인수가(arguments) 있다.
    // 1. data : 가져온 항목의 List
    // 2. prevKey : 현재 페이지에서 이전항목을 가져와야 하는 경우에 이 함수(load()) 에서 사용되는 key
    // 3. nextKey : 현재 페이지에서 다음항목을 가져와야 하는 경우에 이 함수(load()) 에서 사용되는 key
    // 상응하는 방향으로 로드할 데이터가 더 이상 없는경우 nextKey, prevKey가 null이다.

    // 로드되는 key는 Article.id 필드이다. -> ID는 1씩 increase
}
```

1. `getRefreshKey()` 

```kotlin
override fun getRefreshKey(state: PagingState<Int, Article>): Int? {
    // Paging 라이브러리가 UI 관련 항목을 새로고침해야 할 때 호출됨. -> PagingSoruce의 데이터가 변경 됐으니까!
    // Paging Source의 기본 데이터가 변경되었으며 UI에서 업데이트해야 하는 상황을 invalidation(무효화)라고 부름
    // Paging 라이브러리가 데이터를 새로고침할 때 새로운 PagingSource를 만들고 이에 응하는
    // 새로운 PagingData를 내보내 UI에 알린다.
    // 새로운 PagingSource에서 로드할 땐 새로고침 후 리스트에서 현재 위치(position?)을 잃지 않도록
    // 해당 키를 제공하기위해 이 함수(getFreshKey)가 호출된다.

    // 무효화가 발생하는 이유는 두 가지중 하나이다.
    // 1. PagingAdapter에서 refresh() 호출 시
    // 2. PagingSouce에서 invalidate() 호출 시 -? LoadResult.Invalid?
    // 무효화 후 리스트가 이동하지 않도록 하려면 반환된 키가 화면을 채울 만큼 충분한 항목을 로드해야한다.
    // -> 스크롤 유지!

    val anchorPosition = state.anchorPosition ?: return null
    // anchorPosition : PagingData는 아이템을 read할 때 특정 색인에서 읽으려고 함
    // 데이터를 읽은 경우에 UI에 표시되는 방식이다. anchorPosition은 데이터를 성공적으로 읽었을 때
    // 마지막 색인의 인덱스이다. 새로고침 시에는 anchorPosition에 가장 가까운 key를 사용
    val article = state.closestItemToPosition(anchorPosition) ?: return null
    return ensureValidKey(key = article.id - (state.config.pageSize / 2))
}
```

### PagingConfig 정의

```kotlin
// ViewModel
val items: Flow<PagingData<Article>> = Pager(
    config = PagingConfig(pageSize = ITEMS_PER_PAGE, enablePlaceholders = false),
    pagingSourceFactory = { repository.articlePagingSource()}
	  // pagingSourceFactory 람다는 PagingSource 인스턴스를 재사용할 수 없으므로 호출되는 경우 항상
    // 완전히 새로운 PagingSource를 반환해야 한다.
).flow
```

pagingSourceFactory에 구현해놓은 PagingSource를 람다로 전달하여 Pager를 통해 PagingData를 만든다.
