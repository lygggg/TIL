# #4: Status Checks in React Query

- success: 쿼리가 성공, 데이터가 있음
- error: 쿼리가 작동하지x 오류
- lodaing: 쿼리에 데이터가 없으며 현재 처음으로 로드중
- idle: 쿼리가 활성화되지 않았기 때문에 실행된적이 없음, v4에서 제거됨

isFetching 플래그는 내부 상태 시스템의 일부가 아니며 요청이 진행중일 때마다 true가 되는 추가 플래그다. loading과 success가 동시에 수행될 수는 없다. 상태머신이 확실하게 정해준다.

```tsx
const todos = useTodos()

if (todos.isLoading) {
  return 'Loading...'
}
if (todos.error) {
  return 'An error has occurred: ' + todos.error.message
}

return <div>{todos.data.map(renderTodo)}</div>
```
여기서 먼저 로딩과 오류를 확인하고 데이터를 표시한다. 많은 Data-fetch 솔류션, 특히 자체적으로 만든 솔루션은 re-fetch mechanism이 없거나 명시적인 사용자 상호작용(user interactions)에 대해서만 re-fetch한다. 대부분 좋을수는 있지만, 일부 사용 사례에서는 적합하지 않다.

하지만 리액트 쿼리는 괜찮다.

기본적으로 React Query는 적극적으로 re-fetch를 하고, 사용자가 요청하지 않아도 re-fetch한다. _refetchOnMount_, _refetchOnWindowFocus_, _refetchOnReconnect_의 개념은 데이터를 정확하게 유지하는데 매우 유용하다.

- _refetchOnMount_
	- 데이터가 stale 상태일 경우 마운트 시 마다 refetch를 실행한다.
- refetchOnWindowFocus
	- 데이터가 stale 상태일 경우 윈도우 포커싱 될 때마다 refetch를 실행하는 옵션이다.
- refetchOnReconnect
	- 데이터가 stale 상태일 경우 재 연결될 때 refetch를 실행하는 옵션이다.

다만 background re-fetch가 실패할 경우 혼동을 일으킨다.

## Background errors
많은 경우 background re-fetch가 실패하면 무시될 수 있다. 하지만 위 코드느 그렇지 않다.

- 사용자가 페이지를 열고 초기 쿼리를 성공적으로 로드한다. 한동안 페이지에서 작업하다가, 이메일을 확인하기 위해 브라우저 탭을 바꾼다. 몇 분 후에 돌아오고, React Query는 background re-fetch를 한다. 그리고 이 fetch는 실패한다.
- 사용자는 목록을 볼 수 있는 페이지에 있으며, 세부 보기로 드롭 다운하기 위해 한 항목을 클릭한다. 당연히 잘 작동하므로 목록 보기로 돌아간다. 다시 세부 보기로 이동하는 경우 캐시에서 데이터를 볼 수 있다. background re-fetch가 실패하는 경우를 제외하고는 이 방법이 제일 좋습니다.

두경우 모두 다음과 같은 상태가 된다.
```tsx
{
  "status": "error",
  "error": { "message": "Something went wrong" },
  "data": [{ ... }]
}
```
보다시피, 기존 데이터와 Error를 모두 사용할 수 있다. 이것이 바로 React Query의 장점이다. 즉, 데이터가 존재하는 경우 항상 제공한다.

오류를 보여주는게 중요한가요? 오래된 데이터가 있으면 되나요? 둘다 보여야할까요? 이것에는 명확한 답이 없다. 상황에 따라 다르다.

React Query가 지수 백오프를 통해 실패한 쿼리를 default로 세번 재시도 하기 때문에 기존 데이터가 오류 화면을 교체되는 데 몇 초가 걸릴 수 있다는 점을 고려할 때 더욱 중요하다. background re-fetch indicator가 없는 경우 이작업은 매우 복잡하다.

이게 바로 데이터 가용성을 먼저 확인하는 이유다.

```tsx
const todos = useTodos()

if (todos.data) {
  return <div>{todos.data.map(renderTodo)}</div>
}
if (todos.error) {
  return 'An error has occurred: ' + todos.error.message
}

return 'Loading...'
```

결론은 무엇이 옳은지에 대한 원칙은 없다. 왜냐면 상황에 크게 의존하기 떄문이다. 