# #3: React Query Render Optimizations
React Query는 이미 매우 우수한 최적화 및 기본값이 제공되며, 대부분 최적화가 필요하지 않다.

## **isFetching transition**

```tsx
export const useTodosQuery = (select) =>
  useQuery(['todos'], fetchTodos, { select })
export const useTodosCount = () => useTodosQuery((data) => data.length)

function TodosCount() {
  const todosCount = useTodosCount()

  return <div>{todosCount.data}</div>
}
```

background-fetch를 다시 실행할 때마다 이 구성 요소는 다음 쿼리 정보로 두번 re-render 된다.

```ts
{ status: 'success', data: 2, isFetching: true }
{ status: 'success', data: 2, isFetching: false }
```

React Quert는 각 쿼리에 대해 많은 메타 정보를 노출하는데, isFetching이 그 중 하나이기 때문이다. 요청이 진행 중일때 이 플래그는 항상 ture이다. 이 기능은 background loading indicator를 화면에 표시하려는 경우 매우 유용하다. 물론 그렇게 하지 않을거라면 딱히 필요없다.

## **notifyOnChangeProps**
React Query에는 notifyOnChangeProps 옵션이 있다. props 중 하나가 변경될 경우에만 해당 observer에게 변경 사항을 알려주도록 observer 수준에서 설정할 수 있다. 이 옵션을 "data"로 설정하면 원하는 최적화된 버전을 찾을 수 있다.

```ts
export const useTodosQuery = (select, notifyOnChangeProps) =>
	useQuery(['todos'], fetchTodos, { select, notifyOnChangeProps })
export const useTodosCount = () =>
	useTodosQuery((data) => data.length, ['data'])
```

위 코드는 잘 작동하지만, 쉽게 동기화 되지 않는다. 만약 동기화되지 않은 오류에도 대응하고 싶다면 notifyOnChangeProps 목록을 컴포넌트에서 실제로 사용하고 있는 필드와 동기화하면 된다. 이 작업을 건너 뛰고 데이터 속성만 관찰했을 때 오류가 표시된다면, 컴포넌트가 다시 렌더링 되지 않으므로 오래된 것이다. 특히 Custom hook에서 하드 코딩할 경우 컴포넌트가 실제로 무엇을 사용할지 모르기 떄문에 문제가 발생한다.

```ts
export const useTodosCount = () =>
	useTodosQuery((data) => data.length, ['data'])

function TodosCount() {
	const { error, data } = useTodosCount()

	<div>
		{error ? error : null}
		{data ? data : null}
	</div>
}
```
이것은 불필요한 re-render를 하는 것보다 나쁘다. 물론 커스텀훅 옵션을 전달할 수 있지만, 이것은 여전히 매우 수동적이고 보일러 플레이트처럼 느껴진다. 자동으로 할수 있는 방법이 없을까?

## **Tracked Queries**
notifyOnChangeProps를 "tracked"로 설정하면 React Query는 렌더링 중에 사용중인 필드를 추적하고 이 필드를 사용하여 목록을 계산한다. 이렇게 하면 목록을 수동으로 지정하는 것과 정확히 동일한 방식으로 최적화되지만, 사용자가 고려할 필요가 없다. 해당 옵션은 모든 쿼리에 대해 전역적으로 설정할 수 있다.

```tsx
// 🚨 will track all fields
const { isLoading, ...queryInfo } = useQuery(...)

// ✅ this is totally fine
const { isLoading, data } = useQuery(...)
```


```tsx
const queryInfo = useQuery(...)

// 🚨 will not corectly track data
React.useEffect(() => {
    console.log(queryInfo.data)
})

// ✅ fine because the dependency array is accessed during render
React.useEffect(() => {
    console.log(queryInfo.data)
}, [queryInfo.data])
```

```tsx
const queryInfo = useQuery(...)

if (someCondition()) {
    // 🟡 we will track the data field if someCondition was true in any previous render cycle
    return <div>{queryInfo.data}</div>
}
```


## **Structural sharing**
React Query가 즉시 설정한 다른 중요한 렌더 최적화는 구조적 공유다. 이 기능을 사용하면 모든 수준에서 데이터의 참조 ID를 유지할 수 있다. 예를 들어 다음과 같은 데이터 구조가 있다고 가정하면

```tsx
[
  { "id": 1, "name": "Learn React", "status": "active" },
  { "id": 2, "name": "Learn React Query", "status": "todo" }
]
```

이제 첫번째 todo를 done 상태로 전환하고, background re-fetch를 한다고 가정하면, 백엔드에서 완전히 새로운 json을 얻을 수 있다.

```tsx
[
-  { "id": 1, "name": "Learn React", "status": "active" },
+  { "id": 1, "name": "Learn React", "status": "done" },
  { "id": 2, "name": "Learn React Query", "status": "todo" }
]
```

이제 React Query는 이전 상태와 새 상태를 비교하고 가능한 많은 이전 상태를 유지한다. 이 예에서는 todo를 업데이트 했기 떄문에 todos 배열이 새로워진다. id가 1인 개체도 새 개체이지만 id가 2인 개체는 이전 상태의 개체와 동일한 참조가 된다. React Query는 새 결과에 아무것도 변경되지 않았기 떄문에 새 결과를 복사하기만 하면 된다.

이 기능은 부분 구독에 선택자를 사용할 때 매우 유용하다

```tsx
// ✅ will only re-render if _something_ within todo with id:2 changes
// thanks to structural sharing
const { data } = useTodo(2)
```

셀렉터의 경우 구조적 공유가 두번 수행된다. 결과가 한 번 변경되었는지 확인하기 위해 queryFn에서 반환된 후 선택자 함수의 결과를 다시 한 번 확인한다. 단 매우 큰 데이터 세트를 가지고 있는 경우 구조적 공유는 병목 현상이 될 수 있으며 json 직렬화가 가능한 데이터에서만 작동한다. 이 최적화가 필요하지 않은 경우 모든 쿼리에서 _structuralSharing: false_를 설정하여 최적화를 해제할 수 있다.

## 결론
- background-fetch를 다시 실행할 때마다 두번 re-render 된다. 요청이 진행중일때는 isFetching이 true 완료되면  false
- notifyOnChangeProps 를 tracked로 설정하면 렌더링 중에 사용중인 필드를 추적하고 이 필드를 사용하여 목록을 계산한다.  하지만 v4부터 기본값으로 설정되었다.
- React Query는 ## **Structural sharing**를 사용하여 모든 수준에서 데이터의 참조 ID를 유지할 수 있다.