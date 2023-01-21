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

