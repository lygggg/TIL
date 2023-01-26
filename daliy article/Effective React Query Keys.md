쿼리키는 리액트 쿼리에서 매우 중요한 개념이다. 쿼리에 대한 종속성이 변경되면 라이브러리가 내부적으로 데이터를 올바르게 캐시하고 자동으로 가져올 수 있도록 하기 위해 필요하다. 마지막으로, 쿼리 캐시와 수동으로 상호작용할 수 있다. 에를 들어 mutation후  데이터를 업데이트하거나 일부 쿼리를 수동으로 무효화해야 하는 경우 처럼

### Caching Data
내부적으로 쿼리 캐시는 단지 JavaScript 개체이자, 키는 직렬화된 쿼리 키, 값은 쿼리 데이터와 메타 정보이다. 키는 결정론적 방식으로 해시되므로 객체들도 사용할 수 있다.

가장 중요한 것은 키가 고유해야한다는 것이다. React Query가 캐시에서 키에 대한 항목을 찾으면 그것을 사용한다. useQuery와 useInfiniteQuery에는 동일한 키를 사용할 수 없다. 결국 쿼리 캐시는 하나뿐이다. 그리고 이 둘 사이에 데이터
를 공유하게 된다.

```tsx
useQuery(['todos'], fetchTodos)

// 🚨 this won't work
useInfiniteQuery(['todos'], fetchInfiniteTodos)

// ✅ choose something else instead
useInfiniteQuery(['infiniteTodos'], fetchInfiniteTodos)
```

### **Automatic Refetching**

> 쿼리는 선언형이다.

이것은 아무리 강조해도 지나치지 않는 매우 중요한 개념이다. 대부분의 사람들은 쿼리, 특히 refetching을 명령형으로 생각하는데, "클릭"하는 데도 시간이 걸릴 수 있다.

어떤 데이터를 fetch하는 쿼리가 하나 있으면, 사용자는 버튼을 클릭하고 refetch가 되길 기대하지만 다른 매개변수가 있어야 한다.

```ts
function Component() {
	const { data, refetch } = useQuery(['todos'],fetchTodos)

	return <Filters onApply={() => refetch(???)} />
}
```
이렇게 하지 않아도 된다.

refetch의 뜻은 그게 아니라, 동일한 매개 변수를 사용하여 refetching을 하는 것이다.

데이터를 변경하는 상태가 있는 경우 키가 변경될 때마다 refetch가 자동적으로 트리거되므로 쿼리 키에 데이터를 넣기만 하면 된다. 따라서 필터를 적용하려는 경우 클라이언트 상태를 변경하기만 하면 된다.

```ts
function Component() {
	const [filters, setFilters] = React.useState()
	const { data } = useQuery(['todos', filters], () => fetchTodos(filters))

	return <Filters onApply={setFilters}/>
}
```

### Manual Interaction
쿼리 캐시와의 수동적 상호 작용은 쿼리 키의 구조가 가장 중요한 부분이다. invalidateQueries, setQueriesData와 같은 많은 상호작용 메서드로 쿼리 키를 얕게 일치시킬 수 있는 Query Filters를 지원한다.

### Colocate
모든 쿼리 키를 /src/utills/queryKey.ts에 글로벌하게 저장한다고 상황이 개선되지는 않을 것이다. 기능 디렉터리에 있는 해당 쿼리 옆에 쿼리 키를 보관해보면 어떨까?

```
- src
	- profile
		- index.ts
		- queries.ts
	- Todos
		- index.tsx
		- queries.ts
```

쿼리 파일에는 React Query와 관련된 모든 내용이 포함된다. 보통 Custom Hook만 내보내므로 실제 쿼리 기능뿐만 아니라 쿼리 키도 지역적으로 유지된다.

### **Always use Array Keys**
쿼리 키도 문자열이 될 수 있지만, 통일성을 유지하기 위해 항상 배열을 사용해라. React Query는 내부적으로 문자열을 배열로 변환한다.

```tsx
// 🚨 will be transformed to ['todos'] anyhow
useQuery('todos')
// ✅
useQuery(['todos'])
```
### Structure
쿼리 키를 가장 일반적인 것부터 세부적인 것까지, 해당 수준을 구체화하여 구성한다. 필터랑 가능한 목록과 세부 보기를 허용하는 todos list를 구성하는 방법은 다음과 같다.
```ts
['todos', 'lits', { filters: 'all' }]
['toods', 'list', { filters: 'done' }]
['todos', 'detail', 1]
['todos', 'detail', 2]
```
이 구조를 사용하면 모든 todos, list, detail 정보를 ['todos']의 관계에서 제외할 수 있을 뿐만 아니라 정확한 키를 알면 하나의 특정 목록을 대상으로 지정할 수 있다. 필요한 경우 모든 목록을 대상으로 지정할 수 있으므로 mutation는 훨씬 유연해진다.

```ts
function useUpdateTitle() {
	return useMutation(updateTitle, {
	onSuccess: () => {
	queryClient.setQueryData(['todos', 'detail', newTodo.id], newTodo)

	queryClient.setQueryData(['todos', 'list'], (previous) =>
		previous.map((todo) => (todo.id === newTodo.id ? newtodo : todo)))
	}
	})
}
```

list와 detail의 구조가 많이 다르면 기능이 작동하지 않을 수도 있으니 이를 대신해 모든 list를 무효화할 수 있다.

```tsx
function useUpdateTitle() {
  return useMutation(updateTitle, {
    onSuccess: (newTodo) => {
      queryClient.setQueryData(['todos', 'detail', newTodo.id], newTodo)

      // ✅ just invalidate all the lists
      queryClient.invalidateQueries(['todos', 'list'])
    },
  })
}
```

예를 들어 url에서 필터를 읽어 현재 어떤 목록에 있는지 알고 있으므로 정확한 쿼리 키를 구성할 수 있다면, 이 두 가지 방법을 결합하여 list에서 setQueryData를 호출하고 다른 모든 것을 무효화할 수도 있다.

```tsx
function useUpdateTitle() {
  // imagine a custom hook that returns the current filters,
  // stored in the url
  const { filters } = useFilterParams()

  return useMutation(updateTitle, {
    onSuccess: (newTodo) => {
      queryClient.setQueryData(['todos', 'detail', newTodo.id], newTodo)

      // ✅ update the list we are currently on instantly
      queryClient.setQueryData(['todos', 'list', { filters }], (previous) =>
        previous.map((todo) => (todo.id === newTodo.id ? newtodo : todo))
      )

      // 🥳 invalidate all the lists, but don't refetch the active one
      queryClient.invalidateQueries({
        queryKey: ['todos', 'list'],
        refetchActive: false,
      })
    },
  })
}
```

v4에서 refetchActive가 refetchType으로 대체되었다.

### **Use Query Key factories**
위에서는 직접 쿼리 키를 선언했으나, 이는 오류가 발생하기 쉬울 뿐만 아니라 키에 다른 수준의 세분화를 추가하려는 경우처럼 향후 변경 작업을 더욱 어렵게 만든다.

Query Key factory를 사용하면 좋다. 오직 entries and functions이 있는 간단한 객체일 뿐이며 쿼리 키를 생성하고, 이를 Custom Hook에서 사용할 수 있기 때문이다. 위의 예제 구조를 바꾼다면 다음과 같다.
```ts
const todoKeys = {
	all: ['todos'] as const,
	lists: () => [...todoKeys.all, 'list'] as const,
	list: (filters: string) => [...todoKeys.lists(), { filters }] as const,
	details: () => [...todoKeys.all, 'detail'] as const,
	detail: (id: number) => [...todoKeys.details(), id] as const,
}
```

각 레벨은 다른 레벨 위에 구축되지만 독립적으로 엑세스할 수 있으므로 유연성이 뛰어나다.

```ts
queryClient.removeQuerires(todoKeys.all)

queryClient.invalidateQueries(todoKeys.list())

queryClient.prefetchQueries(todoKeys.detail(id), () => fetchTodo(id))
```