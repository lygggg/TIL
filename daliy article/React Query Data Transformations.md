# #2: React Query Data Transformations
## **Data Transformation**

## **1. In the queryFn**
queryFn은 Query를 사용하기 위해 전달되는 함수다. 사용자 Promise를 반환할 것으로 예상하고, 결과 데이터는 쿼리 캐시로 전환된다. 그러나 백엔드가 제공하는 구조의 데이터를 완전히 반환해야 한다는 뜻은 아니다.

```tsx
const fetchTodos = async (): Promise<Todos> => {
  const response = await axios.get('todos')
  const data: Todos = response.data

  return data.map((todo) => todo.name.toUpperCase())
}

export const useTodosQuery = () => useQuery(['todos'], fetchTodos)
```

### **2. In the render function**
커스텀 훅으로 작성하면 쉽게 변환이 가능하다.
```tsx
const fetchTodos = async (): Promise<Todos> => {
  const response = await axios.get('todos')
  return response.data
}

export const useTodosQuery = () => {
  const queryInfo = useQuery(['todos'], fetchTodos)

  return {
    ...queryInfo,
    data: queryInfo.data?.map((todo) => todo.name.toUpperCase()),
  }
}
```
현재 상태로는 fetch 함수마다 실행될 뿐만 아니라 실제로 모든 렌더(data-tetch)를 포함하지 않는 렌더에서도 살행된다. 만약 문제가 생긴다면 useMemo로 최적화할 수 있다. 주의를 기울여 의존성을 가능한 한 좁게 정의해야한다. queryInfo 내부의 데이터는 실제로 변경되지 않는 한 관계적으로 안정적이지만, queryInfo 자체는 안정적이지 않다. queryInfo를 종속성으로 추가하면 변환이 모든 렌더에서 다시 실행된다.
```ts
export const useTodosQuery = () => {
	const queryInfo = useQuery(['todos'], fetchTodos)

	return {
		...queryInfo,
	// 🚨 don't do this - the useMemo does nothing at all here!
		data: React.useMemo(
			() => queryInfo.data?.map((todo) => todo.name.toUpperCase()),
			[queryInfo]
		),
    // ✅ correctly memoizes by queryInfo.data
		data: React.useMemo(
		() => queryInfo.data?.map((todo) => todo.name.toUpperCase()),
		[queryInfo.data]
		),
	}
}
```

### **3. using the select option**
v3에는 데이터 변환에도 사용할 수 있는 built-in 선택자가 도입되었다.
```ts
export const useTodosQuery = () =>
	useQuery(['todos'], fetchTodos, {
	select: (data) => data.map((todo) => todo.name.toUpperCase()),
	})
```
선택자는 데이터가 존재하는 경우에만 호출되므로 정의되지 않는 것에 신경쓸 필요가 없다. 위와 같은 선택자도 함수 ID가 변경되기 때문에 모든 렌더에서 실행된다. 변환 비용이 많이 드는 경우 useCallback을 사용하거나 안정적인 함수 참조로 분리할 수 있다.

```ts
const transformTodoNames = (data: Todos) =>
	data.map((todo) => todo.name.toUpperCase())

export const useTodosQuery = () =>
	useQuery(['todos'], fetchTodos, {
	select: transformTodoNames,
	})

export const useTodosQuery = () =>
	useQuery(['todos'], fetchTodos, {
	select: React.useCallback(
		(data: Todos) => data.map((todo) => todo.name.toUpperCase()),
		[]
	)
	})
```

또한 select 옵션을 사용하여 데이터의 일부만 구독할 수도있다.
```ts
export const useTodosQuery = (select) =>
	useQuery(['todos'], fetchTodos, { select })

export const useTodosCount = () => useTodosQuery((data) => data.length)
export const useTodo = (id) =>
	useTodosQuery((data) => data.find((todo) => todo.id === id))
```