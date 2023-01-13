### useMemo가 무엇인가요?

메모제이션 된 값을 반환한다. usememo는 의존성이 변경되었을 때에만 메모제이션된 값만 다시 계산한다. 만약 의존성 값이 변경되지 않은 경우 캐시된 결과를 반환한다. 이렇게 하면 모든 불필요한 재랜더링을 방지하여 프로그램의 성능을 향상시킨다.

### useMemo를 왜 사용하나요?

useMemo는 컴포넌트의 불필요한 재 렌더를 방지하여 React의 성능을 최적화하는데 사용된다. 컴포넌트가 렌더링되면 해당 컴포넌트의 모든 자식도 다시 렌더링된다. 만약 하위 컴포넌트의 출력을 계산하는데 비용이 많이 드는 경우 상위 컴포넌트가 업데이트될때마다 다시 렌더링하면 성능 문제가 발생한다. usememo를 사용하여 하위 컴포넌트의 출력을 메모하면 입력이 변경될때맏 다시 계산되도록 할 수 있다. 이렇게 하면 불필요한 재렌더 수를 줄이고 프로그램의 성능을 향상시킬수 있다.

### 이 개념은 어떻게 사용하는가?

```jsx
export type VideoGameSearchProps = {
  allGames: VideoGameProps[],
}

export const VideoGameSearch: React.FC<VideoGameSearchProps> = ({ allGames }) => {
  const [searchTerm, setSearchTerm] = React.useState('')
  const [count, setCount] = React.useState < number > 1

  // NOTE useMemo here!!
  const results = useMemo(() => {
    console.log('Filtering games')
    return allGames.filter((game) => game.name.includes(searchTerm))
  }, [searchTerm, allGames])

  const onChangeHandler = (event: React.ChangeEvent<HTMLInputElement>) => {
    setSearchTerm(event.target.value)
  }

  const onClickHandler = () => {
    setCount((prevCount) => prevCount + 1)
  }

  return (
    <>
      <input type="text" value={searchTerm} onChange={onChangeHandler} />
      {results.map((game) => (
        <VideoGame key={game.name} rating={game.rating} name={game.name} releaseDate={game.releaseDate} />
      ))}
      <br />
      <br />
      <p>Count: {count}</p>
      <button onClick={onClickHandler}>Increment count</button>
    </>
  )
}
```

위 예시에서 보면 results는 계산된 값이 메모화되어 저장되고, searchTerm, allGames 배열이 변경될 때만 다시 계산된다. 만약 useMemo를 사용하지 않았다면 직접적인 영향을 미치지 않더라도 count 상태를 증가시키기 위해 버튼을 클릭할떄마다 함수가 계속해서 다시 계산되었을 것이다. 이는 상태 변경으로 인해 상위 구성요소가 다시 렌더링되어 강제로 다시 계산시키기 떄문이다.

### 다른 대안은 없나?

두 컴포넌트 모두 자체 상탤르 관리하게 하면 각자의 컴포넌트의 상태가 변경될 때만 다시 렌더링되고 다른 구성 요소의 상태가 변경될 떄는 렌더링 되지 않는다.

```jsx
export const VideoGameSearch: React.FC<VideoGameSearchProps> = ({ allGames }) => {
  return (
    <>
      <VideoGameSearcher allGames={allGames} />
      <Counter />
    </>
  )
}
```

### 자식 컴포넌트로 나눌수 없는 경우에는 어떻게 해야할까요?

상태를 자식 컴포넌트 구성 요소로 완전히 밀어넣을수 없는 상황에 처했을 경우에는 어떻게 해야할까? 어떤 이유로든 count의 상태가 `VideoGameSearch` 컴포넌트에 저장되어야 한다면? 이경우 자식을 React.memo로 래핑할수있다.

```jsx
export const VideoGameSearcher: React.FC<VideoGameSearchProps> = React.memo(({ allGames }) => {
  // etc.
})  
```

참조 동일성을 유지해야 하므로 useMemo를 사용하여 `VideoGameSearcher` 에 내려주는 값도 래핑해야한다.

```jsx
export const VideoGameSearch: React.FC<VideoGameSearchProps> = ({ allGames }) => {
  const [count, setCount] = React.useState < number > 1

  const allGamesMemo = useMemo(() => allGames, [allGames]) 

  return (
    <>
      <VideoGameSearcher allGames={allGamesMemo} /> // Wrapped in React.memo
      <p>Count: {count}</p>
      <button onClick={onClickHandler}>Increment count</button>
    </>
  )
}
```

참조: [](https://bionicjulia.com/blog/react-memo-and-usememo-whats-the-difference)[https://bionicjulia.com/blog/react-memo-and-usememo-whats-the-difference](https://bionicjulia.com/blog/react-memo-and-usememo-whats-the-difference)

### useMemo 주의사항은 무엇인가요?

### 참조 동일성 위에거 참고하세요

-   useMemo는 처리량이 많을 떄 사용해야한다. 처리량이 적을경우 오히려 성능이 떨어진다는 실험 결과가 있음. useMemo를 사용하면 초기 렌더링은 성능 면에서 상당한 비용을 지불한다. for문으로 1~100까지인 함수에 useMemo를 사용할 경우 초기 렌더링이 62% 느려짐
-   즉 결론은 **처리량이 많아** 렌더링에 문제가 되는 경우 리렌더시에 비용 절감을 위해 사용해야한다.

### useMemo는 어떤 경우에 사용하면 좋을까요?

1.  하위 트리에 많은 Consumer가 있는 값을 Context Provider에 전달해야 하는 경우 `useMemo` 를 사용하는 것이 좋다. **`<ProductContext.Provider value={{id, name}} >`** 의 경우 어떤 넌트가 해당 컴포넌트가 리렌더링 된다면 `id` , `name` 이 동일하더라도 매번 새로운 참조를 만들어 죄다 리렌더링 된다.
2.  계산 비용이 많이 들고, 사용자의 입력 값이 `map` 과 `filer` 를 사용했을 때와 같이 렌더링 이후로도 참조적으로 동일할 가능성이 높은 경우, useMemo를 사용하는 것이 좋다.
3.  자식 컴포넌트에서 useEffect가 반복적으로 트리거되는 것을 막고 싶을 때 사용하자.
4.  매우 큰 리액트 트리구조 내에서, 부모가 리렌더링 되었을 때 이에 따른 렌더링 전파를 막고 싶을 때 사용하자. 자식 컴포넌트가 React.memo, React.pureComponent일 경우, 메모제이션된 props를 사용하게 되면 딱 필요한 부분만 리렌더링 될것이다.
5.  React Devtools Profiler를 사용하면 컴포넌트의 리렌더링 속도가 느린경우, 상태 변경이 일어났을 떄 얼마나 랜더링 시간이 걸렸는지 조사할 수 있다. 이렇게하면 거대한 계단식 리렌더링을 방지하기 위해 React.memo를 사용할 위치를 찾을 수 있고, 필요한 경우 useCallback, useMemo를 사용하여 상태 변경을 효율적으로 만들 수 있다.

### useMemo는 어떤 원리인가요?

useMemo 함수에 알아보기전에 메모제이션에 대해서 알아보면, 메모제이션이란 기존에 수행한 연산의 결과값을 저장해두고 동일한 입력이 들어오면 재활용하는 프로그래밍 기법을 말한다. 메모제이션을 적절하게 사용하면 불필요한 중복 연산을 피할수 있기 떄문에 메모리를 조금더 사용하더라도 애플리케이션의 성능을 최적화할 수 있다.

[](https://medium.com/swlh/should-you-use-usememo-in-react-a-benchmarked-analysis-159faf6609b7)[https://medium.com/swlh/should-you-use-usememo-in-react-a-benchmarked-analysis-159faf6609b7](https://medium.com/swlh/should-you-use-usememo-in-react-a-benchmarked-analysis-159faf6609b7) useMemo 성능