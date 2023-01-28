## Automatic Batching

automatic batching은 리액트 18버전에서 성능 향상을 위해 추가된 새로운 기능입니다. 이전에는 react와 관련된 events에 대해서만 batching 기능이 동작했었습니다. (batching 기능이란 렌더링을 한 번에 모아서 렌더링하는 것을 의미합니다) 이 말인즉, promises, setTimeout, native event handlers, or any other event에서는 batching 기능이 동작하지 않았다는 것입니다. 하지만 18버전으로 넘어오면서 automatic batching 기능이 확장되었습니다.

아래의 코드예시를 통해 설명해보겠습니다.

```jsx
// Before: 예전에는 react events가 아닌 것들에서는 자동으로 batching이 되지 않았습니다. 
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // 때문에 여기선 리액트가 2번 렌더링할 것입니다. 
}, 1000);

// After: 이제는 timeouts, promises,
// native event handlers or any other event 에서 일어나는 변경들에도 batching이 적용됩니다. 
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // 여기선 리액트가 단 한번만 렌더링을 할 것입니다. 
}, 1000);
```

## Transitions

Transition은 긴급한 업데이트와 긴급하지 않은 업데이트를 구분하는 새로운 개념입니다. 아래의 2가지 개념으로 생각해볼 수 있습니다.

-   **Urgent updates :** 이것은 직접적인 상호작용을 반영하는 것들입니다. 예를 들면, 타이핑, 클릭, 입력 등등을 말하는 것입니다.
-   **Transition updates :** 하나의 화면에서 다른 화면으로 변환하는 업데이트를 말합니다.

긴급한 업데이트(타이핑, 클릭, 입력등등)은 즉각적인 반응을 필요로합니다. 그리고 그렇게 하는 것이 우리가 생각하는 직관에 알맞습니다. 그렇지 않으면 아마 뭔가 ‘잘못되었다’라고 느낄 것입니다. 하지만 Transition에 대해서는 우리가 바라보는 관점이 조금 다릅니다. 유저가 무언가가 Transition되고 있다고 생각할 때는 항상 즉각적인 반응을 기대하는 것은 아닙니다.

일반적으로는 최고의 사용자 경험은, 단일한 유저 입력이 일어났을 때, 긴급한 업데이트와 긴급하지 않은 업데이트가 동시에 일어나게 하는 것입니다. 이를 위해서 당신은 startTransition API를 사용해서 긴급한것과 transition이라고 불리는 것을 업데이트 해달라고 리액트에게 알릴 수 있습니다. 아래 코드를 봅시다.

```jsx
import {startTransition} from 'react';

// Urgent: 무엇이 입력되었는지는 긴급한 업데이트 영역입니다. 
setInputValue(input);

// 이제 이 안에 들어간 state 업데이트들은 transition으로 여겨집니다. 
startTransition(() => {
  // Transition: 결과를 보여주는 transition
  setSearchQuery(input);
});
```

startTransition으로 감싸진 업데이트들은 긴급하지 않은 것으로 여겨집니다. 이것이 의미하는 것은 클릭이나, 입력같은 다른 긴급한 이벤트가 들어왔을 때 interrupt 될 수 있다는 말입니다. 만약에 transition이 interrupt 되었다면, 리액트는 아직 완료되지 않은 낡은 렌더링 작업은 던져버리고 가장 최신의 렌더링만 업데이트 할 것입니다.

사실 이 startTransition 외에도 18버전에서 새롭게 추가된 hook이 있는데, 그것은 useTransition이라는 훅입니다. 이 훅은 2가지 튜플 값을 반환합니다. 아래 코드를 통해서 살펴보겠습니다.

```jsx
import { useTransition } from 'react';

function MyComponent() {
    const [startTransition, isPending] = useTransition();

    return (
        <button onClick={() => startTransition(() => setSearchQuery(input))}>
            {isPending ? 'Loading...' : 'Search'}
        </button>
    );
}
```

첫번째 인자는 startTransition이고, 두번째 인자는 isPending입니다. startTransition 안에 들어간 업데이트들은 transition의 개념으로 여기게 됩니다.

button이 클릭되면, setSearchQuery이 실행되지만 다른 긴급한 이벤트가 일어나게되면, 해당 업데이트는 취소될 것입니다.

## Suspense

서스펜스는 아직 화면에 보여줄 준비가 안된 lodingState의 컴포넌트를 표시할 것인지에 대해 선언적으로 작성할 수 있게 해줍니다.

```jsx
<Suspense fallback={<Spinner />}>
  <Comments />
</Suspense>
```

이전에도 서스펜스는 존재하기는 했지만, 제한적인 방법으로 구현되어있었고, 사용되는 케이스는 코드스플리팅을 사용할 때 뿐이었습니다. 하지만 18버전에서는 서버에서 서스펜스 지원을 추가하고, 동시 렌더링 기능을 확장했습니다.

해당 Suspense 컴포넌트는 transition API와 함께 활용하면 좋습니다.

이전에는 어떤 문제가 있었길래 이 Suspense라는 기능을 추가하게 된 것일까요?

-   loading, error를 처리하는 영역에 있어서 명령적인 코드로 처리할 수 밖에 없었다.
-   하지만 Suspense를 통해 loading의 영역에 대해서 선언적으로 코드를 처리할 수 있게 되었다.

```jsx
// before 
if(loading) <div>Loading...</div>

return <div>real ui</div>

// after

<Suspense fallback={<div>Loading...</div>}>
  <div>real ui</div>
</Suspense>
```

```

## Client and Server rendering Api

- createRoot : ReactDom.render 가 없어지고 새롭게 생긴 함수
- hydrateRoot : ReactDom.hydrate 가 없어지고 새롭게 생긴 함수
```

## Strict Mode Behavior

## Hooks

### **useId**

유니크한 Id를 생성해주는 hook이다. 접근성 속성들에도 할당이 가능하다.

근데 해당 id 값을 key Props에 넘겨서는 안된다. key props에 넣는 값은 data로부터 나온 id 값을 넣는 것이 좋다.

사용법 :

```jsx
import { useId } from 'react';

function PasswordField() {
  const passwordHintId = useId();
  return (
    <>
      <label>
        Password:
        <input
          type="password"
          aria-describedby={passwordHintId}
        />
      </label>
      <p id={passwordHintId}>
        The password should contain at least 18 characters
      </p>
    </>
  );
}
```

이는 server rendering에서도 사용할 수 있기 때문에 더욱 유용하다고 할 수 있다.

### **useTransition**

실무에서 사용될 수 있는 예시코드

```jsx
import { useTransition } from 'react';

function Product({ id, name, price }) {
    const [startTransition, isPending] = useTransition({ timeoutMs: 1000 });
    const [cartTotal, setCartTotal] = useState(0);

    const handleAddToCart = async () => {
        startTransition(async () => {
            // Make API call to add item to cart
            await addItemToCart(id);
            setCartTotal(cartTotal + price);
        });
    }

    return (
        <div>
            <h2>{name}</h2>
            <button onClick={handleAddToCart}>
                {isPending ? 'Adding...' : 'Add to cart'}
            </button>
            <p>Cart total: ${cartTotal}</p>
        </div>
    );
}
```

이 코드에서 startTransition이 interrupt 될 수 있는 예시 상황 :

-   Add to cart 버튼을 여러번 누를 경우, 이전에 진행되고 있던 startTransition내부의 코드는 취소될 것이다.
-   startTransition 내부가 실행되는 도중 페이지를 이동했을 때