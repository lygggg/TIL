모든 state 공유 문제를 해결하기 위해 context를 쓰지 마라. 또한 context는 앱 전체에서 전역으로 사용될 필요가 없다는 것을 알아둬라. 하지만 context는 트리의 한 부분에 적용될 수 있으며 앱 안에서 논리적으로 분리된 여러개의 context들을 가질 수 있다.

```ts
import * as React from 'react'

const CountContext = React.createContext()
```

우선, `CountContext`는 초기값에 initial value를 가지고 있지 않다. 만약 초기값을 원했다면 React.createContext({count: 0})이렇게 호출하면 된다. 이것은 의도한 것이고, 기본값은 아래와 같은 상황에서만 유용하다.
```ts
 function CountDisplay() {
    const {count} = React.useContext(CountContext)
    return <div>{count}</div>
  }
  ReactDOM.render(<CountDisplay />, document.getElementById('⚛️'))
```

CountContext에 기본값이 없기 때문에 useContext에 구조 분해 할당을 사용해서 값을 리턴한 부분에 에러줄이 뜬다. 왜냐하면 기본값이 undefined인데 undefined는 구조 분해 할당이 될수 없으니까.

기본값이 유용한 상황들이 있긴하지만 대부분의 경우 필요허거나 유용하지는 않다.

리액트 공식문서에서는 기본값을 제공하는건 "컴포넌트가 감싸인 형태가 아닌 분리된 상황에서 테스트하는데에는 유욯할 수 있다"라고 제안한다. 이렇게 하는 것이 가능하다는건 맞는 말이지만, 필요한 컨텍스트로 컴포넌트를 감싸는것보다 낫다는 부분에 있어서는 동의하지 않는다.

# The Custom Provider Component
해당 context 모듈을 유용하게 만들기 위해서는 Provider를 사용하고 값을 제공해주는 컴포넌트를 노출시킬 필요가 있다.

```ts
  function App() {
    return (
      <CountProvider>
        <CountDisplay />
        <Counter />
      </CountProvider>
    )
  }

  ReactDOM.render(<App />, document.getElementById('⚛️'))
```

그렇다면 위와 같이 사용할 수 있는 컴포넌트들을 만들어 보자.

```ts
import * as React from "react"

const CountContext = React.createContext()

functin countReducer(state, action) {
	switch (action.type) {
		case 'increment': {
			return {count: state.count +1}
		}
		case 'decrement': {
			return {count: state.count -1}
		}
		default: {
			throw new Error(`Unhandled action type: ${action.type}`)
		}
	}
}

function CountProvider({chlidren}) {
	const [state, dispatch] = React.useReducer(countReducer, {count:0})
	
	const value = {state, dispatch}

	return <CountContext.Provider value={value}>{children}</CountContext.Provider>
}

export {CountProvider}
```

# TypeScript
타입스크립트를 사용할 때 기본값을 사용하지 않음으로써 발생하는 문제를 피해보자
```ts
  // src/count-context.tsx
  import * as React from 'react'
  type Action = {type: 'increment'} | {type: 'decrement'}
  type Dispatch = (action: Action) => void
  type State = {count: number}
  type CountProviderProps = {children: React.ReactNode}

  const CountStateContext = React.createContext<
    {state: State; dispatch: Dispatch} | undefined
  >(undefined)

  function countReducer(state: State, action: Action) {
    switch (action.type) {
      case 'increment': {
        return {count: state.count + 1}
      }
      default: {
        throw new Error(`Unhandled action type: ${action.type}`)
      }
    }
  }

  function CountProvider({children}: CountProviderProps) {
    const [state, dispatch] = React.useReducer(countReducer, {count: 0})
    // NOTE: you *might* need to memoize this value
    // Learn more in http://kcd.im/optimize-context
    const value = {state, dispatch}
    return (
      <CountStateContext.Provider value={value}>
        {children}
      </CountStateContext.Provider>
    )
  }
function useCount() {
	const context = React.useContext(CountStateContext)

	if(context === undefined) {
		throw new Error('useCount must be used within a CountProvider')
	}
	return context
}

export {CountProvider, useCount}
```

# What about dispatch type types?
이 시점에서, 리덕스는 소리를 지를것이다. "이봐, 액션 생성자(action creators)는 어디 있는거야?? 만약 위가 액션 생성자를 사용한다고 해도 괜찮다. 하지만 kent는 액셩 생성자가 불필요한 요약이라고 느꼈다. 또한 만약 우리가 타입스크립트를 잘 사용하고 있으며, 액션이 잘 정의되어 있다면, 굳이 필요하지 않다.

`dispatch`함수를 위의 방식처럼 넘기는 것을 선호한다. 이런 방식의 또 다른 장점은, `dispatch`함수는 생성된 컴포넌트의 생명주기동안 안전하기에 우리는 `dispatch`함수를 useEffect의 종속성 배열에 넘길 필요가 없다.

만약 우리가 자바스크립트를 입력하지 않을 경우를 생갹해야한다. 만약 그런적이 없다면 말이다. 놓친 액션 타입 때문에 나오는 에러는 안전장치다.

# What about async actions?
만약 우리가 비동기 요청을 해야하고 이 요청 과정에서 여러가지들을 dispatch 해야하는 상황에 처하게 된다면 어떻게 해야할까? 물론 컴포넌트가 호출되는 시점에서 할 수 있긴 하지만, 이런 일을 해야하는 컴포넌트들에게 모든 구성 요소들을 수동적으로 연결하는 것은 굉장히 귀찮은 일이다

글쓴이가 제안하는 것은 `dispatch`함수와 함께 우리가 필요한 데이터를 받는 context 모듈 안에 도우미 함수(helper function)를 만드는 것이다. 그리고 그 도우미 함수가 모든것을 책임지게 만드는 것이다.

```ts
// user-context.js
async function updateUser(dispatch, user, updates) {
	dispatch({type: 'start update', updates})
	
	try {
	const updatedUser = awaitt userClient.updateUser(user, updates)
	dispatch({type: 'finish update', updateUser})
	} catch (error) {
	dispatch({type: 'fail update', error})
	}
}
export {UserProvider, useUser, updateUser}
```
그리고 이렇게 사용할 수 있다.

```ts
// user-profile.js
import {useUser, updateUser} from './user-context'

function UserSettings() {
	const [{uset, status, error}, userDispatch] = useUser()
	function handleSubmit(event) {
		event.preventDefault()
		updateUser(userDispatch, user, formState)
	}
}
```

아래는 최종 코드다
```ts
 // src/count-context.js
  import * as React from 'react'

  const CountContext = React.createContext()
  function countReducer(state, action) {
    switch (action.type) {
      case 'increment': {
        return {count: state.count + 1}
      }
      case 'decrement': {
        return {count: state.count - 1}
      }
      default: {
        throw new Error(`Unhandled action type: ${action.type}`)
      }
    }
  }

  function CountProvider({children}) {
    const [state, dispatch] = React.useReducer(countReducer, {count: 0})
    // NOTE: you *might* need to memoize this value
    // Learn more in http://kcd.im/optimize-context
    const value = {state, dispatch}
    return <CountContext.Provider value={value}>{children}</CountContext.Provider>
  }

  function useCount() {
    const context = React.useContext(CountContext)
    if (context === undefined) {
      throw new Error('useCount must be used within a CountProvider')
    }
    return context
  }

  export {CountProvider, useCount}
```

1. 마주치는 모든 state 공유 문제를 해결하기 위해 context를 사용하지 말자
2. Context는 꼭 앱 전역에 사용되는 전역상태일 필요가 없다. 하지만 여러분이 만든 트리 한 부분에서 적용될 수 있다.
3. 앱에서 논리적으로 분리된 여러가지 context를 가질 수 있다.