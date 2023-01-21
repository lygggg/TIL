# #5: Testing React Query
컨테이너 컴포넌트를 테스트하는 것은 쉽지 않다. 

## **Mocking network requests**
React Query는 비동기 서버 상태 관리 라이브러리이므로 컴포넌트가 백엔드에 요청할 가능성이 높다. 테스트시 데이터를 제공하는 백엔드를 실제로 사용할 수 없으며, 백엔드는 테스트에 의존하지 않을 수도 있다.


jest로 데이터를 mocking하는 방법에 대한 블로그가 많이 있다. 당신만의 API 클라이언트가 있다면 mock할 수 있다. fetch 또는 axios를 직접 mock할 수 있지만 Kent C. Dodds가 쓴 [”fetch를 그만 mock 해”](https://kentcdodds.com/blog/stop-mocking-fetch) 를 참고해보자.

## **QueryClientProvider**
React Query를 사용할 때마다 QueryClientProvider가 필요하며 QueryCache를 보관하는 QueryClient를 제공해야 한다. 쿼리 데이터는 캐시를 담고 있다.

각 테스트에 고유한 QueryClientProvider를 지정하고 각 테스트에 대해 새 QueryClient를 만드는 것을 선호하는데, 테스트는 서로 격리되기 때문이다. 다른 접근 방식은 각 테스트 후 캐시를 지우는 것이지만, 그래도 테스트 간의 상태는 가능한 최소로 유지하는 것이 좋다. 그렇지 않으면 테스트를 병렬로 실행할 경우 예기치 않은 결과가 발생할 수 있다.

## **For custom hooks**
react-hooks-testing-library를 사용하여 Custom Hook을 테스트하는 것이 확실하다. 이 라이브러리를 사용하면 wrpper로 Hook을 감쌀 수 있다.wrapper는 렌더링 시 테스트 구성 요소를 감쌀 수 있는 React 컴포넌트다.
QueryClient는 테스트 당 한 번 씩 실행되므로 이 컴포넌트가 QueryClient를 만들기에 완벽한 장소다.

```tsx
const createWrapper = () => {
  // ✅ creates a new QueryClient for each test
  const queryClient = new QueryClient()
  return ({ children }) => (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  )
}

test("my first test", async () => {
  const { result } = renderHook(() => useCustomHook(), {
    wrapper: createWrapper()
  })
}
```

useQuery Hook을 사용하는 컴포넌트를 테스트하려면 해당 컴포넌트를 QueryClientProvider에서 랩핑 해야한다. [react-testing-library](https://testing-library.com/docs/react-testing-library/intro/) 의 render 주변의 작은 wrapper는 좋은 선택이다.