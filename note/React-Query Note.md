### 왜 만들어졌나?
- React 가 데이터를 페칭해오거나 업데이트를 하는 옵션을 제공하지 않기 떄문에 원래 React 개발자들은 각자의 방식으로 http 통신 로직을 작성해야했다.
- Redux 같은 전역 상태관리 라이브러리들이 클라이언트 상태값에 대해서는 잘 작동하지만, 서버 상태에 대해서는 그렇게 잘 작동하지 않는다. Server State와 Client State는 완전 다르기 떄문이다.
- 서버 데이터는 항상 최신 상태임을 보장하지 않는다. 명시적으로 fetching을 수행해야만 최신 데이터로 전환된다.
- 네트워크 통신은 최소한으로 줄이는게 좋은데, 복수의 컴포넌트에서 최신 데이터를 받아오기 위해 fetching을 여러번 수행하는 낭비가 발생할 수 있다.


#### Query
1. fresh: 새롭게 추가된 쿼리 인스턴스 -> active 상태의 시작, 기본 staleTime이 0이기 떄문에 아무것도 설정을 안해주면 호출이 끝나고 바로 stale 상태로 변한다. staleTime을 늘려줄 경우 fresh한 상태가 유지되는데, 이때는 쿼리가 다시 마운트되도 페칭이 발생하지 않고 기존의 fresh한 값을 반환한다.
2. fetching: 요청을 수행하는 중인 쿼리
3. stale: 인스턴스가 존재하지만 이미 패칭이 완료된 쿼리. 특정 쿼리가 stale된 상태에서 같은 쿼리 마운트를 시도한다면 캐싱된 데이터를 반환하면서 리패칭을 시도한다.
4. inactive: active 인스턴스가 하나도 없는 쿼리. inactive된 이후에도 cacheTime 동안 캐시된 데이터가 유지된다. cacheTime이 지나면 GC된다.

- 다음 4가지 경우에 리패칭이 일어난다.
	- 런타임에 stale인 특정 쿼리 인스턴스가 다시 만들어졌을 때
	- window가 다시 포커스가 되었을때
	- 네트워크가 다시 연결되었을 때
	- refetch interval이 있을 떄: 요청 실패한 쿼리는 디폴트로 3번 더 백그라운드 단에서 요청하며, retry, retryDelay 옵션 간격과 횟수를 커스텀 가능

#### Queries
- 쿼리는 server state를 요청하는 프로미스를 리턴하는 함수와 함께 unique key로 맵핑된다.
- 쿼리는 콜백 함수의 요청이 프로미스를 리턴한다면 일단 잘 작동한다. 하지막 서버의 데이터를 바꿀 수 있는 요청이라면 mutation을 쓰는걸 더 추천
- useQuery훅의 인자로 2개가 들어간다. - query의 key, 프로미스를 리턴하는 함수
- unique key : 한 번 fresh가 되었다면 계속 추적이 가능하다. 리패칭, 캐싱, 공유 등을 할때 참조되는 값. 주로 배열을 사용하고, 배열의 요소로 쿼리의 이름을 나타내는 문자열과 프로미스를 리턴하는 함수의 인자로 쓰이는 값을 넣는다.
- useQuery 반환값: 객체, 요청의 상태를 나타내는 몇가지 프로퍼티, 요청의 결과나 에러값을 갖는 프로퍼티도 포함
	- isLoading, isError, isSuccess, isIdle, status
	- error, data, isFetching -> 런타임간 무조건 요청이 한번 이상 발생했다면 값이 존재한다.
- 쿼리 요청 함수의 상태를 표현하는 status값은 4가지다. status 프로퍼티에서는 문자열로, 상태 이름 앞에 is를 붙인 프로퍼티에서는 불리언으로 해당 상태인지 아닌지를 평가 가능하다.
	- idle: 쿼리 data가 하나도 없고 비었을 때. {enabled : false}상태로 쿼리가 호출되었을 때 이 상태로 시작된다.
	- loading: 말그대로 로딩중
	- error: 에러상태
	- success 요청에 성공했을 때
- 주요 쿼리 옵션
	- enabled: true로 설정하면 자동으로 쿼리의 요청 함수가 호출되는 일이 없다.
	- keepPreviousData: success와 loading 사이 널뛰기 방지
	- placeholderData: mock 데이터 설정가능, 대신 캐싱x
	- initialData: 초기값 설정

#### Query Keys
- 문자열: 구별되는 문자열로 키를 줄수있다. 바로 인자가 하나인 배열로 convert된다.
- 배열: 문자열과 함께 숫자를 주면 같은 문자열로 같은 key를 쓰면서 id로 구별이 가능
- 콜백함수에 주는 인자: 배열의 마지막 요소이며, 역시 쿼리를 구별하는데 쓰임 -> 엔드포인트가 같더라도 요청에 넣는 body나 쿼리파람이 다르면 다른 쿼리 인스턴스로 취급됨
- key 배열의 요소 순서도 중요하다. 만약 다르면 다른 다르게 캐싱된다.
- 요청 함수가 특정 변수에 의존할 때, 쿼리 키 배열에 객체로 같이 넣어주면 요청 함수 내에서 인자로 객체를 받을수있음.

#### Parallel
- 몇 가지 상황을 제외하면 쿼리 여러개가 선언되어 있는 일반적인 상황이라면 쿼리 함수들은 그냥 병렬로 요청되서 처리된다. -> 쿼리 처리의 동시성 극대화
```tsx
function App() {
const usersQuery = useQuery('users', fetchUsers);
const teamsQuery = useQuery('teams', fetchTeams);
const projectsQuery = useQuery('projects', fetchProjects)
}
```

- 쿼리 여러 개를 동시에 수행해야 하는데, 렌더링이 거듭되는 사이사이에 계속 쿼리가 수행되어야 한다면 쿼리를 수행하는 로직이 hook룰에 위배될 수도 있다. 그럴 떄 쓰면 좋은게 useQueries다.

```tsx
function App({users}) {
const userQueries = useQueries(
user.map((user) => {
	return {
	return {
		queryKey: ['user', user.id],
		queryFn: () => fetchUserById(user.id),
	};
	}
})
)
}
```

#### Query Retries
```tsx
import { useQuery } from "react-query";

const result = useQuery(["todos", 1], fetchTodoListPage, {
	retry: 10,
})
```

- useQuery의 요청이 fail이 나는 경우, 최대 연속 요청 한계까지 요청을 계속 다시 한다. 디폴트 값은 3이다.
- retry 옵션으로 쿼리의 재요청 횟수를 정한다.
- retryDelay 옵션을 설정하면 요청이 한번 실패했을 때, 설정한 일정 시간이 지난 후 또 요청한다.

#### Mutations
- useQuery와는 다르게 create, update, delete하며 server state에 사이드 이펙트를 일으키는 경우에 사용한다.
- useMutation으로 mutation 객체를 정의하고, mutate 메서드를 사용하면 요청 함수를 호출해 요청이 보내진다.
- useMutation이 반환하는 객체 프로퍼티로 제공되는 상태값은 useQuery와 동일하다.
- mutation.reset: 현재의 error와 data를 모두 지울 수 있다.

#### invalidation
- stale 쿼리 폐기
- 쿼리의 데이터가 요청을 통해 서버에서 바뀌었다면, 백그라운드에 남아 있는 데이터는 과거의 것이 되어 앱에서 쓸모없어지는 상황이 발생할 수 있다.
- invalidateQueries 메소드를 사용하면 개발자가 명시적으로 query가 stale되는 지점을 찝어줄 수 있다. 해당 메소드가 호출되면 쿼리가 바로 stale되고, 리패치가 진행된다.
- 쿼리에 특정 키가 공통적으로 들어가있다면 싸잡아서 Invalidation이 가능하다.

#### QueryCache
react query의 저장소 메커니즘. react query의 모든 데이터를 저장한다. 일반적으로 개발자가 직접 QueryCache에 직접 접근할 일은 거의 없고 특정 cache에 접근하기 위해서는 QueryClient를 사용한다.

#### Caching Process
1. useQuery의 첫번째, 새로운 인스턴스 마운트 -> 만약에 런타임간 최초로 fresh한 해당 쿼리가 호출되었다면, 캐싱하고, 패칭이 끝나면 해당 쿼리를 stale로 바꿈
2. 앱 어딘가에서 useQuery 두번째 인스턴스 마운트 -> 이미 쿼리가 stale이므로 접때 요청때 만들어 놨었던 캐시를 반환하고 리패칭함
3. 쿼리가 언마운트되거나 더이상 사용하지 않을 떄 마지막 인스턴스가 언마운트되어 inacive 상태가 되었을 때 기본 cacheTime이 지나면 자동으로 삭제함.

#### Observer
리액트 쿼리에서 하나의 캐시 entry를 감시하는 객체를 의미한다. 캐시 entry의 무언가가 변경되면 Observer가 알 수 있다.
Observer를 생성하기 가장 쉬운 방법은 useQuery() 함수를 호출하는 것 이다.

useQuery() 함수를 사용하는 컴포넌트가 리렌더링 될 떄마다 useQuery()를 호출하기 떄문에 매번 새로운 Observer를 생성한다. 즉, 여러번 리렌더링되면 하나의 캐시 entry에 대해 여러 개의 Observer를 가질 수 있다.


### 출처
- https://maxkim-j.github.io/posts/react-query-preview/
- 