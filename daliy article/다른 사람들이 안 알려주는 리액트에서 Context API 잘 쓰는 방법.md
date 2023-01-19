Context는 리액트 컴포넌트 간에 어떠한 값을 공유할 수 있게 해주는 기능이다. 주로 Context는 전역적(global)으로 필요한 값을 다룰 때 사용한다. 하지만 꼭 전역적일 필요는 없다. Context를 단순히 리액트 컴포넌트에서 Props가 아닌 또 다른 방식으로 컴포넌트간에 값을 전달하는 방법이다 라고 접근하는 게 좋다.

createContext 를 불러와서 만들고
```js
import { createContext } from 'react';
const MyContext = createContext();
```

Context 객체 안에는 Provider라는 컴포넌트가 들어 있다. 그리고, 그 컴포넌트 간에 공유하고자 하는 값을 `value`라는 Props로 설정하면 자식 컴포넌트들에서 해당 값에 바로 접근할 수 있다.

```js
function App() {
  return (
    <MyContext.Provider value="Hello World">
      <GrandParent />
    </MyContext.Provider>
  );
}
```

이렇게 하면, 원하는 컴포넌트에서 useContext를 사용하여 Context에 넣은 값에 바로 접근이 가능하다.

```js
import { createContext, useContext } from 'react';

function Message() {
  const value = useContext(MyContext);
  return <div>Received: {value}</div>;
}
```


만약 Context가 여러 컴포넌트에서 사용되고 있다면 커스텀 훅을 만들어서 사용하는 것도 좋은 방법

```js
import { createContext, useContext } from 'react';
const MyContext = createContext();

function useMyContext() {
  return useContext(MyContext);
}

function App() {
  return (
    <MyContext.Provider value="Hello World">
      <AwesomeComponent />
    </MyContext.Provider>
  );
}

function AwesomeComponent() {
  return (
    <div>
      <FirstComponent />
      <SecondComponent />
      <ThirdComponent />
    </div>
  );
}

function FirstComponent() {
  const value = useMyContext();
  return <div>First Component says: "{value}"</div>;
}

function SecondComponent() {
  const value = useMyContext();
  return <div>Second Component says: "{value}"</div>;
}

function ThirdComponent() {
  const value = useMyContext();
  return <div>Third Component says: "{value}"</div>;
}

export default App;
```

만약 Provider로 감싸지 않았다면 undefined가 조회된다. 이럴경우엔 기본 값을 설정하는 방법이 있다.
```js
const MyContext = createContext('default value');
```

기본값을 보여주지 않고 아예 오류를 띄워서 개발자가 고치도록 명시하고 싶다면 커스텀훅을 사용하자

```js
const MyContext = createContext();

function useMyContext() {
  const value = useContext(MyContext);
  if (value === undefined) {
    throw new Error('useMyContext should be used within MyContext.Provider');
  }
}
```

Context에서 상태 관리가 필요한 경우
이번에는 Context에서 유동적인 값을 다뤄야 할 때를 살펴보자

Context에서 유동적인 값을 관리하는 경우엔 Provider를 새로 만들어 주는 것이 좋다. 


```js
import { createContext } from 'react';

const CounterContext = createContext();

function CounterProvider({ children }) {
	return <CounterContext.Provider>{ch}lidren}</CounterContext.Provider>;
}

function App() {
	return (
		<CounterProvider>
			<div>
				<Value/>
				<Buttons/>
			</div>
		</CounterProvider>
	)
}
```


그다음 필요한 기능들을 CounterProvider 컴포넌트 안에서 구현해준다.
```js
import { createContext, useState } from 'react';

const CounterContext = createContext();

function CounterProvider({ children }) {
  const counterState = useState(1);
  return (
    <CounterContext.Provider value={counterState}>
      {children}
    </CounterContext.Provider>
  );
}

// (...)
```

그 다음에는 useCounterState라는 커스텀 Hook을 만들고
```js
import { createContext, useContext, useState } from 'react';

// (...)

function useCounterState() {
  const value = useContext(CounterContext);
  if (value === undefined) {
    throw new Error('useCounterState should be used within CounterProvider');
  }
  return value;
}
```

이렇게 hook으로 사용하면 CounterProvider의 자식 컴포넌트 어디서든지 useCounterState를 사용하여 값을 조회하거나 변경할 수 있다.

```js
// (...)

function Value() {
  const [counter] = useCounterState();
  return <h1>{counter}</h1>;
}
function Buttons() {
  const [, setCounter] = useCounterState();
  const increase = () => setCounter((prev) => prev + 1);
  const decrease = () => setCounter((prev) => prev - 1);

  return (
    <div>
      <button onClick={increase}>+</button>
      <button onClick={decrease}>-</button>
    </div>
  );
}

export default App;
```

여기서 데이터를 어떻게 업데이트할지에 대한 로직을 컴포넌트가 아니라 Provider단에서 구현하고 싶다면
```js
import { createContext, useContext, useMemo, useState } from 'react';

const CounterContext = createContext();

function CounterProvider({ children }) {
  const [counter, setCounter] = useState(1);
  const actions = useMemo(
    () => ({
      increase() {
        setCounter((prev) => prev + 1);
      },
      decrease() {
        setCounter((prev) => prev - 1);
      }
    }),
    []
  );

  const value = useMemo(() => [counter, actions], [counter, actions]);

  return (
    <CounterContext.Provider value={value}>{children}</CounterContext.Provider>
  );
}


function useCounter() {
  const value = useContext(CounterContext);
  if (value === undefined) {
    throw new Error('useCounterState should be used within CounterProvider');
  }
  return value;
}

function App() {
  return (
    <CounterProvider>
      <div>
        <Value />
        <Buttons />
      </div>
    </CounterProvider>
  );
}

function Value() {
  const [counter] = useCounter();
  return <h1>{counter}</h1>;
}

function Buttons() {
  const [, actions] = useCounter();

  return (
    <div>
      <button onClick={actions.increase}>+</button>
      <button onClick={actions.decrease}>-</button>
    </div>
  );
}

export default App;
```
actions라는 객체를 만들어서 그 안에 변화를 일으키는 함수를 넣고, 컴포넌트가 리렌더링 될 때마다 함수를 새로 만드는게 아니라 처음에 한번 만들고 그 이후에 재사용할 수 있도록 useMemo로 감쌌다.

그리고 value에 값을 넣어주기 전에 `[counter, action]`을 useMemo로 한번 감싸주었다. 감싸지 않으면 CounterProvider가 리렌더링 될 때마다 새로운 배열을 만들기 때문에 `useContext`를 사용하는 컴포넌트 쪽에서 Context의 값이 바뀐것으로 간주하게 되어 낭비 렌더링이 발생하게 된다.

만약 Context에서 관리하는 상태가 빈번하게 업데이트 되지 않는다면 방금 작성한 코드만으로도 충분하나, 만약 상태가 빈번하게 업데이트 된다면, 성능적으로 좋지 않다. 실제로 변화가 반영되는 곳은 value 컴포넌트뿐인데, Buttons 컴포넌트도 리렌더링 되기 떄문이다.

우리가 비록 value를 만드는 과정에서 useMemo로 감싸주긴 했지만, 어쨌든 counter가 바뀔 때 마다 새로운 배열을 만들어서 반환하고 있고 useContext에선 이를 감지하여 리렌더링을 하기 때문이다.

이를 고치는 방법은 Context를 하나 더 만드는 것이다.

```js
import { createContext, useContext, useMemo, useState } from 'react';

const CounterValueContext = createContext();
const CounterActionsContext = createContext();

function CounterProvider({ children }) {
  const [counter, setCounter] = useState(1);
  const actions = useMemo(
    () => ({
      increase() {
        setCounter((prev) => prev + 1);
      },
      decrease() {
        setCounter((prev) => prev - 1);
      }
    }),
    []
  );

  return (
    <CounterActionsContext.Provider value={actions}>
      <CounterValueContext.Provider value={counter}>
        {children}
      </CounterValueContext.Provider>
    </CounterActionsContext.Provider>
  );
}

function useCounterValue() {
  const value = useContext(CounterValueContext);
  if (value === undefined) {
    throw new Error('useCounterValue should be used within CounterProvider');
  }
  return value;
}

function useCounterActions() {
  const value = useContext(CounterActionsContext);
  if (value === undefined) {
    throw new Error('useCounterActions should be used within CounterProvider');
  }
  return value;
}

function App() {
  return (
    <CounterProvider>
      <div>
        <Value />
        <Buttons />
      </div>
    </CounterProvider>
  );
}

function Value() {
  console.log('Value');
  const counter = useCounterValue();
  return <h1>{counter}</h1>;
}
function Buttons() {
  console.log('Buttons');
  const actions = useCounterActions();

  return (
    <div>
      <button onClick={actions.increase}>+</button>
      <button onClick={actions.decrease}>-</button>
    </div>
  );
}

export default App;
```

기존의 CounterContext를 CounterValueContext와 CountextActionsContext로 분리했다. 그리고 모두 Provider를 사용해주었다. 커스텀훅 또한 두개로 분리했다.

이렇게하면 버튼을 눌러서 상태변화가 일어날 때, Value 컴포넌트에서만 리렌더링이 발생한다.

이방식 외에도 useReducer를 통해서 상태 업데이트를 하도록 구현하고 dispatch를 별도의 Context로 넣어주는 방식도 있다. [Kent C Dodds](https://kentcdodds.com/blog/how-to-use-react-context-effectively)

## Context의 상태에서 배열이나 객체를 다루는 경우
화면의 중앙에 문구를 띄우는 모달의 상태를 Context로 작성한다면, 다음과 같이 구현 가능하다.
```ts
const ModalValueContext = createContext();
const ModalActionsContext = createContext();

function ModalProvider({ chlidren }) {
	const [modal, setModal] = useState({
		visible: false,
		message: ""
	});

	const actions = useMemo(
		() => ({
		open(message) {
			setModal({
				message,
				visible: true
			});
		},
		close() {
			setModal((prev) => ({
			...prev,
			visible: false
			}));
		}
		}),
		[]
	);

	return (
		<ModalActionsContext.Provider value={actions}
			<ModalValueContext.Provider value={modal}>
				{chlidren}
			</ModalValueContext.Provider>
		</ModalActionsContext.Provider>
	)
}

function useModalValue() {
  const value = useContext(ModalValueContext);
  if (value === undefined) {
    throw new Error('useModalValue should be used within ModalProvider');
  }
  return value;
}

function useModalActions() {
  const value = useContext(ModalActionsContext);
  if (value === undefined) {
    throw new Error('useModalActions should be used within ModalProvider');
  }
  return value;
}
```

이렇게 하면 원하는 곳 어디서든 다음과 같이 모달을 띄울 수 있다.
```js
const { open } = useModalActions();

const handleSomething = () => {
  open('안녕하세요!');
};
```

만약 할 일 목록같은 배열을 다룬다면?
```js
import { createContext, useContext, useMemo, useRef, useState } from 'react';

const TodosValueContext = createContext();
const TodosActionsContext = createContext();

function TodosProvider({ children }) {
  const idRef = useRef(3);
  const [todos, setTodos] = useState([
    {
      id: 1,
      text: '밥먹기',
      done: true
    },
    {
      id: 2,
      text: '잠자기',
      done: false
    }
  ]);

  const actions = useMemo(
    () => ({
      add(text) {
        const id = idRef.current;
        idRef.current += 1;
        setTodos((prev) => [
          ...prev,
          {
            id,
            text,
            done: false
          }
        ]);
      },
      toggle(id) {
        setTodos((prev) =>
          prev.map((item) =>
            item.id === id
              ? {
                  ...item,
                  done: !item.done
                }
              : item
          )
        );
      },
      remove(id) {
        setTodos((prev) => prev.filter((item) => item.id !== id));
      }
    }),
    []
  );

  return (
    <TodosActionsContext.Provider value={actions}>
      <TodosValueContext.Provider value={todos}>
        {children}
      </TodosValueContext.Provider>
    </TodosActionsContext.Provider>
  );
}

function useTodosValue() {
  const value = useContext(TodosValueContext);
  if (value === undefined) {
    throw new Error('useTodosValue should be used within TodosProvider');
  }
  return value;
}

function useTodosActions() {
  const value = useContext(TodosActionsContext);
  if (value === undefined) {
    throw new Error('useTodosActions should be used within TodosProvider');
  }
  return value;
}
```

할일을 추가할 떄는
```js
const { add } = useTodosActions();

const handleSubmit = () => {
  add(text);
}
```

항목을 보여줄때는
```js
const { toggle, remove } = useTodosActions()

const handleToggle = () => {
  toggle(id);
};

const handleRemove = () => {
  remove(id);
};
```

### Context가 꼭 전역적이어야 한다는 생각을 버리자
Context에서 다루는 값을 꼭 전역적일 필요가 없다.

### 전역 상태 관리 라이브러리는 언제 써야할까?
과거에는 리액트의 Context가 굉장히 불편해서 전역 상태 관리 라이브러리를 사용하는 것이 당연시 여겨졌던 시절이 있었지만, 이제는 사용하기 편해져서 단순히 전역적인 상태를 관리하기 위함이라면 더이상 사용해야할 이유가 없다.

단 "상태 관리 라이브러리"와 Context는 완전히 별개의 개념이다. Context는 전역 상태 관리를 할 수 있는 수단일 뿐이고, 상태 관리 라이브러리는 상태 관리를 더욱 편하고, 효율적으로 할 수 있게 해주는 기능들을 제공해주는 도구다.

예를들어, Redux의 경우에는 액션과 리듀서라는 개념을 사용하여 상태 업데이트 로직을 컴포넌트 밖으로 분리할 수 있게 해주며, 상태가 업데이트 될 때 실제로 의존하는 값이 바뀔 때만 컴포넌트가 리렌더링 되도록 최적화해준다. 만약 Context를 쓴다면, 각기 다른 상태마다 새롭게 Context를 만들어주어야 하는데, 이 과정을 생략할 수 있다.

Mobx의 경우엔 Redux와 마찬가지로 상태 업데이트 로직을 컴포넌트 밖으로 분리할 수 있게 해주고, 함수형 리액티브 프로그래밍 방식을 도입하여 mutable한 상태가 리액트에서도 잘 보여지게 해주고, 상태 업데이트 로직을 더욱 편하게 작성할 수 있게 해준다. 덤으로 최적화도 해준다.

Recoil, Jotai, Zustand의 경우엔 Context를 일일이 만드는 과정을 생략하고 Hook 기반으로 아주 편하게 전역 상태를 관리할 수 있도록 해준다. 최적화도 해준다.

전역 상태 라이브러리는 결국 상태 관리를 조금 더 쉽게 하기 위해서 사용하는 것이며, 취향에 따라 선택하면된다.