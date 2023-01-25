# useReducer에 대해서

## 이 개념은 무엇인가?

useState의 대체함수다. (state, action) ⇒ newState의 형태로 reducer를 받고 dispatch 메서드와 짝이 형태로 현재 state를 반환한다. **한 컴포넌트내에서 State를 업데이트하는 로직 부분을 그 컴포넌트로부터 분리시키는 것을 가능하게 해준다.**

## 언제 사용해야하나

-   관리해야할 State가 1개 이상, 복수일 경우
-   혹은 현재는 단일 State 값만 관리하지만, 추후 유동적일 가능성이 있는
-   스케일이 큰 프로젝트의 경우
-   State의 구조가 복잡해질 것으로 보이는 경우

다수의 하윗값을 포함하는 복잡한 정적 로직을 만드는 경우나 다음 state가 이전 state에 의존적인 경우 보통 useState보다 useReducer를 선호한다.

또한 useReducer는 자세한 업데이트를 트리거하는 컴포넌트의 성능을 최적화할 수 있게 하는데, 이것은 콜백 대신 dispatch를 전달할 수 있기 때문이다.

### 이 개념은 어떻게 사용하는가? (이 개념의 예시코드는 무엇인가?)

```jsx
const initialState = {count: 0};

function reducer(state, action) {
	switch (action type) {
	case 'increment':
		return {count: state.count + 1};
	case 'decrement':
		return {count: state.count - 1};
	dafault:
		throw new Error();
	}
}

function Counter() {
	const [state, dispatch] = useReducer(reducer, initalState);
return(
		<>
      Count: {state.count}
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
    </>
)
}
```

### 이 개념의 장/단점은 무엇인가?

### 장점

useReducer는 콜백대신 디스패치를 전달하여 업데이트를 보다 효율적으로 처리할 수 있다. useReducer를 사용한 디스패치 작업은 구성 요소로 인해 발생하는 리렌더의 양을 줄임으로써 업데이트를 보다 최적화된 방식으로 처리할 수 있기 때문에 성능이 향상될 수 있다.

useReducer를 사용하는 것의 주요 이점중 하나는 상태 업데이트와 실제 상태 업데이트를 처리하기 위한 논리의 명확한 분리를 허용한다는 것이다. 이러한 관심사의 분리는 상태 업데이트에 대해 추론하기를 더 쉽게 만들고 테스트를 더 쉽게 만든다. 또한 useReducer를 사용하면 미들웨어를 사용하여 데이터 가져오기 및 비동기 작업처리 같은 부작용을 처리할 수 있다.

또 다른 이점은 상태 기록을 통해 시간여행할 수 있는 기능을 지원한다. 이를 통해 상태 기록을 왔다갔다 하면서 응용 프로그램을 디버깅하고 테스트할 수 있다.

위 코드를 다시보면 상태 업데이트를 처리하기 위한 논리가 실제 상태 업데이트와 분리된다는 것이다. Counter 구성 요소는 상태가 업데이트되는 방식을 알 필요가 없으며, 어떤 작업은 디스패치할지만 알면 된다. 이러한 관심사의 분리는 상태 업데이트에 대해 추론하기를 더 쉽게 만들고 테스트를 더 쉽게 만든다.

### 단점

코드를 더 복잡하게 만든다. 리듀서 기능과 디스패치 작업을 통해 업데이트를 처리하기 때문에 코드가 더 상세하고 이해하기 어려워진다.

setState를 사용하면 상태가 어떻게 변화하는지 쉽게 알 수 있지만, useReducer는 상태 업데이트의 흐름을 추적하는데 더 많은 노력이 필요하다. 그래서 정말 간단하다면 setState를 쓰는게 좋아보인다.

### useState와 useReducer의 주요 차이점

useReducer를 사용하면 업데이트를 보다 제어되고 최적화된 방식으로 처리할 수 있다. useState를 사용할 때 React는 상태가 변경될 때마다 전체 구성요소와 모든 하위 구성요소를 다시 렌더링한다. 그러나 useReducer를 사용할 때 발송된 작업에 따라 변경된 상태의 특정 부분만 업데이트하여 재랜더 수를 최소화할 수 있다. 이는 특히 업데이트가 많거나 구성요소가 복잡한 상황에서 더욱 성능이 향상될 수 있다.