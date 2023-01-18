## useCallback 이란

메모제이션된 콜백을 반환하는 React Hook이다.

### useCallback를 왜 사용하나요?

기본적으로 구성 요소가 다시 렌더링되면 React는모든 자식을 재귀적으로 다시 렌더링한다. 아래 컴포넌트에서 theme를 토글하면 앱이 잠시 정지되지만 ShippingForm를 제거하면 빠르게 느껴진다. 이는 구성요소를 최적화할 가치가 있는 것이다.

```jsx
function ProductPage({ productId, referrer, theme }) {
  // Every time the theme changes, this will be a different function...
  function handleSubmit(orderDetails) {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }
  
  return (
    <div className={theme}>
      {/* ... so ShippingForm's props will never be the same, and it will re-render every time */}
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}

```

리렌더링이 느리다는 것을 확인한 경우 다음과 같이 memo로 래핑해서 props가 동일할 때 리렌더링을 건너뛰도록 할 수 있다.

```jsx
import { memo } from 'react';

const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  // ...
});
```

위 변경으로 인해 props가 마지막 렌더링과 동일한 경우 ShippingForm는 리렌더링을 건너 뛴다. 여기서 함수 캐싱이 중요해진다.

JavaScript에서 `function () {} or () => {} {}` 는 객체 리터럴이 항상 새 객체를 생성하는 것과 유사하게 항상 다른 함수를 생성한다. 일반적으로 이것은 문제되지 않지만 ShippingForm의 props와 동일하지 않으며, 최적화가 작동하지 않는다는 것을 의미한다. 이곳에 `useCallback` 을 사용할수 있다.

```jsx
function ProductPage({ productId, referrer, theme }) {
  // Tell React to cache your function between re-renders...
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]); // ...so as long as these dependencies don't change...

  return (
    <div className={theme}>
      {/* ...ShippingForm will receive the same props and can skip re-rendering */}
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

handleSubmit를 래핑하여 리렌더링간에 동일한 기능을 수행하도록 한다. 위에서 useCallback을 사용한 이유는 메모라이즈된 구성요소를 전달하여 리렌더링을 건너뛸 수 있기 때문이다.

메모제이션은 useMemo편에서 많이 디뤘기 떄문에 자세한건 usememo를 참고하자.