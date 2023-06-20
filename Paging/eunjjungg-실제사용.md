### DataSource

```kotlin
// Data

interface GroupDataSource {

    suspend fun searchGroup(
        name: String,
        page: Int,
        pageSize: Int,
    ): ApiResult<GroupSearchApiResponse>
}
```

```kotlin
// Data

class GroupDataSourceImpl @Inject constructor(
    private val groupApi: GroupApi
) : BaseRepository(), GroupDataSource {

    override suspend fun searchGroup(name: String, page: Int, pageSize: Int):
        ApiResult<GroupSearchApiResponse> {
        return safeApiCall {
            groupApi.getGroupSearchResult(
                name = name,
                page = page.toString(),
                pageSize = pageSize.toString()
            )
        }
    }
}
```

<br/>


### Repository

```kotlin
// Domain

interface GroupRepository {

    suspend fun searchGroup(
        name: String,
        page: Int,
        pageSize: Int,
    ): ApiResult<GroupSearchItem>
}
```

```kotlin
// Data

class GroupRepositoryImpl @Inject constructor(
    private val groupDataSource: GroupDataSource,
) : GroupRepository {

    override suspend fun searchGroup(
        name: String,
        page: Int,
        pageSize: Int
    ): ApiResult<GroupSearchItem> {
        return groupDataSource.searchGroup(
            name = name,
            page = page,
            pageSize = pageSize,
        ).toDomain()
    }
}
```

```kotlin
// Data

fun ApiResult<GroupSearchApiResponse>.toDomain(): ApiResult<GroupSearchItem> {
        return when (this) {
            is ApiResult.Success -> ApiResult.Success(
                GroupSearchItem(
                    hasNext = this.data.hasNext,
                    groupItemList = data.toItem(),
                )
            )
            is ApiResult.Error -> ApiResult.Error(exception)
        }
    }

    private fun GroupSearchApiResponse.toItem(): List<GroupItem> {
        return this.content.map { groupItem ->
            GroupItem(
                id = groupItem.id,
                name = groupItem.name,
                description = groupItem.description,
                organization = groupItem.organization,
                profileImageUrl = groupItem.profileImageUrl,
                memberCount = groupItem.memberCount
            )
        }
    }
```

- 처음에는 서버에서 내려주는 다음 페이지의 값이 존재하는가에 대한 여부인 `hasNext`의 값이 없어도 될줄 알았다. 그래서 `hasNext`에 관계없이 일단 데이터를 요청하고, 없다면 페이징 내부에서 처리하도록 했는데,,, 뷰가 복잡해지면서 사이드 이펙트가 생겼다. 그래서 도메인 쪽에서 사용하는 데이터 모델에도 `hasNext`를 넣어주었고, 이 값에 따라 `PagingSource`가 이어서 페이징을 할지 말지를 정해주었다. (`nextKey` 값을 통해)

<br/>

### UseCase

```kotlin
// domain

class SearchGroupByKeywordUseCase @Inject constructor(
    private val groupRepository: GroupRepository,
) {
    operator fun invoke(keyword: String): Flow<PagingData<GroupItem>> {
        val groupPager = Pager(
            config = PagingConfig(
                pageSize = GroupPagingConfig.NETWORK_PAGE_SIZE,
                enablePlaceholders = false,
                initialLoadSize = GROUP_LIST_INIT_LOAD_SIZE
            ),
            pagingSourceFactory = { GroupListPagingSource(groupRepository = groupRepository, keyword = keyword) }
        ).flow

        return groupPager.map { pagingData ->
            pagingData.map { groupSearchItem ->
                groupSearchItem
            }
        }
    }
}
```

- `initialLoadSize`를 따로 정해주지 않는다면 `pageSize`의 배수로 뽑히게 됨. 2배인가.. 3배였던가..

<br/>

### PagingSource

```kotlin
// domain

class GroupListPagingSource @Inject constructor(
    private val groupRepository: GroupRepository,
    private val keyword: String
) : PagingSource<Int, GroupItem>() {
    override fun getRefreshKey(state: PagingState<Int, GroupItem>): Int? {
        return state.anchorPosition?.let { anchorPosition ->
            state.closestPageToPosition(anchorPosition)?.prevKey?.plus(1)
                ?: state.closestPageToPosition(anchorPosition)?.nextKey?.minus(1)
        }
    }

    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, GroupItem> {
        val page = params.key ?: GROUP_LIST_STARTING_PAGE_INDEX
        return try {
            val response = groupRepository.searchGroup(
                name = keyword,
                page = page,
                pageSize = params.loadSize
            )
            val success = response as ApiResult.Success
            LoadResult.Page(
                data = success.data.groupItemList,
                prevKey = if (page > 1) page - 1 else null,
                nextKey = if (response.data.hasNext) page + 1 else null
            )
        } catch (exception: Exception) {
            return LoadResult.Error(exception)
        }
    }
}
```

- `pagingSource`의 `load`에서 맨 처음 페이징할 페이지의 페이지값(인덱스값)을 정해줄 수 있음. 나는 `GROUP_LIST_STARTING_PAGE_INDEX`로 두었는데, 이 값으로 백엔드에 전달하는 paging value의 초기값이 지정되니 잘 생각해서 넣어주어야 함… 백엔드의 페이지네이션이 0부터 시작이라면 0을, 아니라면 1을 넣어주어야 함.

<br/>

### VM

```kotlin
// presentation.vm

fun searchGroup(keyword: String): Flow<PagingData<GroupSearchUiModel>> {
    val groups = searchGroupUseCase(keyword = keyword).cachedIn(viewModelScope)

    return groups
        .map { pagingData ->
            // 데이터가 존재하는 경우 아래와 같이 PagingData 생성
            pagingData.map {
                GroupSearchUiModel.GroupList(it)
            }.insertSeparators { before, after ->
                // 데이터가 존재하지 않는 경우 (before, after 데이터가 없는 경우) DataNotFound를 seperator로 생성
                if (before == null && after == null)
                    return@insertSeparators GroupSearchUiModel.DataNotFound(keyword)
                else
                    return@insertSeparators null
            }
        }
}
```

- 데이터가 없을 때의 상황인 `DataNotFound`를 페이징 되는 `uimodel` 중의 하나로 넣고 싶었음. 왜냐면 그쪽 부분 ui들을 모두 pagingLib으로 사용하고 싶었기 때문임. 따라서 separator처럼 작동하게 하였고 이전 값과 이후 값이 없는 경우인 데이터가 아예 존재하지 않는 경우에 `DataNotFound`를 넣어주었다.

<br/>

### UI

```kotlin
// presentation.activity 

private fun initPagingFlow() {
    val adapter = GroupListAdapter()
    val concatAdapter = adapter.withLoadStateAdapters(
        header = GroupListLoadStateAdapter { adapter.retry() },
        footer = GroupListLoadStateAdapter { adapter.retry() }
    )
    binding.rvGroupList.adapter = concatAdapter

    lifecycleScope.launch() {
        val data = viewModel.searchGroup(GroupSearchConfig.DEFAULT_QUERY).first()
        adapter.submitData(data)
    }

    binding.viewGroupSearch.etGroupSearch.addTextChangedListener {
        lifecycleScope.launch {
            adapter.submitData(PagingData.empty())
            val data = viewModel.searchGroup(it.toString()).first()
            adapter.submitData(data)
        }
    }
}
```

- `withLoadStateAdapters` 함수의 리턴값은 `ConcatAdapter`임. 따라서 이 연결된 어댑터라는 뜻의 `concatAdapter` 타입의 변수를 리사이클러뷰의 어댑터에 지정해주어야 함. 그러나 데이터가 변경될 때마다 실행해주는 `submitData`는 `concatAdapter`가 아닌 `PagingDataAdapter` 타입의 `adapter`로 실행해주어야 함.
- `withLoadStateAdapters`은 커스텀한 함수로 구현체는 아래에 있음. 굳이 커스텀해서 사용한 이유는 기존의 header, footer 형식으로 사용하게 되면 맨 처음 로드가 될 때는 loadState에 따른 adapter 처리가 적용이 안 된다. 그렇기 때문에 아래 커스텀 함수를 사용해주어야 함. 나 같은 경우에는 아래 함수를 사용하기 전까지는 맨 처음 로드할 때 로딩창이 표시가 안 됐음.

```kotlin
/**
 * LoadStateAdapter를 사용하기 위해 ConcatAdapter를 만들어주기 위해
 * + withLoadStateHeaderAndFooter
 * + withLoadStateHeader
 * + withLoadStateFooter
 *
 * 함수를 사용할 때 initial load의 경우에는 LoadState에 따른 UI가 적용되지 않아
 * LoadStateListener를 커스텀해준 함수입니다.
 */

fun <T : Any, V : RecyclerView.ViewHolder> PagingDataAdapter<T, V>.withLoadStateAdapters(
    header: LoadStateAdapter<*>,
    footer: LoadStateAdapter<*>
): ConcatAdapter {
    addLoadStateListener { loadStates ->
        header.loadState = loadStates.refresh
        footer.loadState = loadStates.append
    }

    return ConcatAdapter(header, this, footer)
}
```

<br/>

### Adapter, ViewHolder

- adapter와 vh의 경우에는 기존의 리사이클러뷰로 사용할 때와 크게 다르지 않기에 따로 작성하지 않았음.
