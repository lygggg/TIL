## Client State vs Server State
### GraphQL과 Apollo Client는 왜 Redux를 완전히 대체한다는 소리가 나왔던 것일까?
Apollo는 원하는 데이터를 정의하고 가져오는 것 뿐만 아니라 해당 서버 데이터에 대한 캐시도 함께 제공하기 때문

즉 우리는 항상 서버 상태를 다른 클라이언트 상태처럼 취급해온 것 같다. 우리는 단순히 사용자를 위해 가져온 최신 데이터를 화면에  표시하기 위해 잠시 빌렸을 뿐이다. 데이터를 소유하는 것은 서버다.

이것은 새로운 패러다임이였다. 캐시를 활용하여 소윻지 않은 데이터를 표시할수 있다면 실제 클라이언트 상태도 거의 남아있지 않기 때문이다.

The Defaults explained
먼저 React Query는 모든 re-render에 대해 queryFn을 호출하지 않는다. 매번 가져오는 짓은 미친짓이다.

refetchOnWindowFocus는 매우 유용한 기능이다. 만약 사용자가 다른 브라우저 탭으로 이동한 후 앱으로 돌아오면 백그라운드 re-fetch가 자동으로 트리거되며, 그 사이에 서버에서 변경된 내용이 있으면 화면의 데이터가 업데이트 된다. 이 모든 작업은 loading 스피너를 표시하지 않고 수행되며, 데이터가 현재 캐시에 있는 것과 동일하다면 DOM은 다시 렌더링 되지 않는다.

- StaleTime: 쿼리가 fresh에서 stale로 전활될 때까지의 유효기간이다. 최신 쿼리라면 데이터는 항상 캐시에서만 가져오고 네트워크 요청은 발생하지 않는다. 그러나 쿼리가 오래된 경우 캐시에서 데이터를 가져오긴 하지만 특정 상황에서는 백그라운드 re-fetch가 발생할 수 있다.
- CacheTime: 비활성 쿼리가 캐시에서 제거될 떄까지의 기간이다. 기본값은 5분이다. 쿼리는 등록된 관찰자가 없는 즉시 비활성 상태로 전환되므로 해당 쿼리를 사용하는 모든 구성은 언마운트 된다.

게다가 리액트 쿼리는 데브툴스를 지원한다! 조아조아~

## **Treat the query key like a dependency array**
Query Key와 useEffect는 비슷하다!
바로 Query Key가 변경될 때마다 re-fetch를 트리거하기 떄문이다.따라서 매개변수를 queryFn에 전달하면 값이 변경될때마다 항상 데이터를 가져온다. 수동으로 re-fetch를 트리거하도록 복잡한 effects를 조정하는 대신 다음과 같은 Query key를 사용할 수 있다.

```ts
type State = "all" | "open" | "done"
type Todo = {
	id: number
	state: State
}
type Todos = ReadonlyArray<Todo>

const fetchTodos = async (state: State): Promise<Todos> => {
	const response = await axios.get(`todos/${state}`)
	return response.data
}

export const useTodosQuery = (state: State) => 
	useQuery(["todos", state], () => fetchTodos(state))
```

## **Treat the query key like a dependency array**
쿼리 키가 캐시의 키로 사용되므로 "all"에서 "done"으로 전환할 때 캐시 항목이 표시되며, 처음 전환할 때 로딩상태가 발생한다. 이러한 경우 keepPreciousData 옵션을 사용하거나 가능하면 새로 생성된 캐시 항목을 initalData로 미리 채울 수 있다.

_keepPreviousData_는 데이터가 리패치 될때 이전 데이터를 유지하는지 정하는 옵션이다.

```tsx
type State = 'all' | 'open' | 'done'
type Todo = {
  id: number
  state: State
}
type Todos = ReadonlyArray<Todo>

const fetchTodos = async (state: State): Promise<Todos> => {
  const response = await axios.get(`todos/${state}`)
  return response.data
}

export const useTodosQuery = (state: State) =>
  useQuery(['todos', state], () => fetchTodos(state), {
    initialData: () => {
      const allTodos = queryClient.getQueryData<Todos>(['todos', 'all'])
      const filteredData =
        allTodos?.filter((todo) => todo.state === state) ?? []

      return filteredData.length > 0 ? filteredData : undefined
    },
  })
```
이제 사용자가 상태를 전환할 때마다 데이터가 없으면 "all todos" 캐시의 데이터로 채운다.

## **Keep server and client state separate**
useQuery에서 데이터를 가져왔다면 해당 데이터를 지역 상태로 전환하지 마라. "복사"상태는 업데이트 되지 않기 때문에 리액트 쿼리에서 수행하는 모든 백그라운드 업데이트를 암묵적으로 수행하기 때문이다.
지역 "복사"기 없기 때문에 항상 최신 데이터를 볼 수 있다.

## **The enabled option is very powerful**
useQuery Hook에는 사용자 정의를 위한 많은 옵션이 있으며 매우 강력하다. 이런 옵션들 덕분에 달성할수 있었던 것들은 다음과 같다.
- 종속 쿼리: 첫번 째 쿼리에서 데이터를 성공적으로 얻은 후에만 두 번째 쿼리를 실행하도록 한다.
- 쿼리 설정 및 해제: refetchInterval 덕분에 정기적으로 데이터를 폴링하는 쿼리가 하나있지만, 화면 뒤쪽에서 업데이트를 피하기 위해 모달이 열려있으면 일시 중지 할 수있다.
- 사용자 입력 대기: Query key에 필터 기준을 일부 가지고 있지만 사용자가 필터를 적용하지 않은 동안에는 비활성화 하도록 설정한다.
- 사용자 입력 후 일부 쿼리 사용 안함: 예를 들어 서버 데이터보다 우선해야 하는 초기 값이 있는 경우 쿼리를 실행하지 않는다.

## **Don’t use the queryCache as a local state manager**
queryCache를 수정하는 경우는 낙관적 업데이트 또는 변환 후 백엔드로부터 수신한 쓰기 전용의 데이터여야 한다. 모든 background fetch는 해당 데이터를 재정의 할 수 있으므로 지역 상태에는 사용하지 마라.

## **Create custom hooks**
하나의 useQuery 호출을 래핑하기 위한 경우 커스텀 훅을 만들면 다음과 같은 이점이 있다.
- UI에서 실제 데이터를 가져오지만 useQuery 호출과 같은 위치에 배치할 수 있다.
- 하나의 Query Key를 하나의 파일에 유지할 수 있다.
- 일부 설정을 변경하거나 데이터 변환을 추가해야 하는 경우 한곳에서 수행할 수 있다.

## 결론
- React Query는 화면에 데이터를 보여주기위해 서버에서 빌려온 데이터를 캐싱한다.
- 예상치 못한 re-fetch가 나타나는 경우 refetchOnWindowFocus를 실행하기 때문일 수 있다.
- refetchOnWindowFocus는 사용자가 다른 브라우저 탭으로 이동한 후 앱으로 돌아오면 백그라운드 re-fetch가 필어나는 것
- StaleTime는 쿼리가 fresh에서 stale로 전환될 때까지의 유효기간
- CacheTime은 비활성 쿼리가 캐시에서 제거될 때까지의 기간
- DevTools를 사용하라! 캐시에 있는 데이터를 알려준다. 디버깅 하기 좋음, 개발 서버는 상당히 빠르기 때문에 백그라운드 re-fetch를 잘 인식하려면 브라우저 개발자 도구의 Network 탭에 들어가 throttle을 걸면 좋다.
- `keepPreviousData` - 쿼리 키(ex.페이지 번호)가 변경되어서 새로운 데이터를 요청하는 동안에도 마지막 `data`값을 유지한다.
- 아니면  [initialData](https://react-query.tanstack.com/docs/guides/initial-query-data#initial-data-from-cache)로 미리 채울 수 있다.
- 모든 백그라운드 데이터는 지역상태에 사용하지 마라
- 커스텀훅을 사용하면 많은 이점이 있다. 위 참고