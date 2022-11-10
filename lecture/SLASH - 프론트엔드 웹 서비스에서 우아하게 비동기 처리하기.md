


### 이 코드의 문제점은 무엇일까?
```tsx
function fetchAccounts(callback) {
	fetchUserEntity((err, user) => {
		if(err != null) {
			callback(err, null);
			return;
		}

		fetchUserAccounts(user.no, (err, accounts) => {
			if(err != null) {
				callback(err, null);
				return;
			}

			callback(null, accounts);
		});
	});
}
```
위 코드는 데이터를 가져올때 에러가 있으면 에러를 Emit하고 에러가 없으면 실제값을 Emit한다. 이 코드의 문제점은 "성공하는 경우"와 "실패하는 경우"가 섞여있다. 함수의 진짜 역할이 무엇인지 가려져있다. 그리고 코드를 작성하는 입장에서 매번 에러 유무를 확인해야한다.

그럼 async await를 사용한 코드는 어떤점이 다른가? 
```tsx
async function fetchAccounts() {
const user = await fetchUserEntity();
const accounts = await fetchUserAccounts(user.no);
return accounts;
}
```
"성공하는 경우"만 다루고, "실패하는 경우"는 catch절에서 분리해서 처리한다. "실패하는 경우"에 대한 처리를 외부에 위임할 수 있다. 

즉 위 코드에서 좋은 코드의 핵심은 
- 성공, 실패의 경우를 분리해서 처리할수 있다.
- 비즈니스 로직을 한눈에 파악할 수 있다.


### 보통의 프론트엔드 웹서비스에서 비동기 처리

#### ex) React Query, SWR
보통 우리는 컴포넌트에서 로딩과 에러 처리를 동시에 수행한다.
```tsx
const foo = useAsyncValue(() => {
return fetchSomething();
});

if (foo.error) return <div>로딩에 실패했습니다.</div>
if (!foo.data) return <div>로딩 중입니다.</div>
return <div>{foo.data.name}님 안녕하세요!</div>
```
심플해보이지만 위에서 언급했던 안좋은 코드처럼 "성공하는 경우"와 "실패하는 경우가" 섞여서 처리되고있다. 여러개의 비동기 작업이 동시에 실행되면 문제는 더 심각해진다.

만약 아래처럼 의존성있는 비동기처리가 여러개 늘어난다면 코드는 더욱 복잡해질 것이다.

```tsx
function Profile() {
const foo = useAsyncValue(() => {
return fetchSomething();
});

const bar = useAsyncValue(() => {
	if(foo.error || !foo.data) {
	return undefined;
	}
})
if (foo.error || bar.error) return <div>로딩에 실패했습니다.</div>
if (!foo.data || !bar.data) return <div>로딩 중입니다.</div>
return <div>{foo.data.name}님 안녕하세요!</div>
};
```

### 그럼 우리는 에러 상태나 로딩 상태는 어떻게 처리해야하나? 
함수의 에러 처리를 감싸는 catch문에서 하는 것처럼, 로딩처리와 에러처리도 컴포넌트를 쓰는곳에서 해주면 된다. 로딩상태는 서스펜스의 fallback으로 그려주고, 에러상태도 ErrorBoundary가 처리하게 하면된다.

```tsx
<ErrorBoundary fallback={<MyErrorPage />}>
	<Suspense fallback={<Loader />}>
		<FooBar />
	</Suspense>
</ErrorBoundary>
```

### 어떻게 사용할 수 있는가?
- Recoil에는 Async Selector로 사용할 수 있다.
- SWR, React-Query에서는 suspense: true로 사용할 수 있다.


아래처럼 비동기처리를 하는 컴포넌트를 suspense로 감싸주기만 하면 된다.
```tsx
funtcion TemplateSetDetails({ templateSetNo }: Props) {
const tempplateSet = useRecoilValue(templateSetSelector(templateSetNo));
/* 이 아래에서는 temlplateSet이 존재하는 것이 보장됨*/
}

<Suspense fallback={<Skeleton/>}>
	<TemplateSetDetails />
</Suspense>
```

우리는 데이터가 준비되는 대로 하나씩 자연스럽게 보여줄수 있다.


### 웹 서비스의 코드 복잡도를 낮춘 방법: Hooks

hooks 은 선언적으로 사용할수 있어 복잡도를 낮출수 있다.

- useState는 상태 사용을 선언
- useMemo는 메모이제이션 사용을 선언
- useCallback은 콜백 레퍼런스 보존을 선언
- useEffect는 부수 효과 발생을 선언



### 웹 서비스의 코드 복잡도를 낮춘 방법: Suspense

서스펜스도 비슷하다. 컴포넌트에서는 비동기적인 리소스를 선언하고, 그 값을 읽어온다고 선언한것이다. 그러면 실제 로딩 상태나 에러처리는 컴포넌트를 감싸는 부모컴포넌트가 해준다.

이렇게 어떤 코드 조각을 감싸는 맥락으로 책임을 분리하는 방식을 대수적 효과라고한다. 또, 의존성 역전의 원칙과 유사하다. 

### 의존성 역전의 원칙이란 무엇?

> 오브젝트 책에서는 의존성 역전의 원칙을 상위 모듈은 하위모듈에 의존하지 않고, 상위모듈 하위모듈 둘다 추상화에 의존해야한다고 나와있다.

### 만약 사용자 경험 향상에 관심이 있다면 요것을  찾아보자

- React Concurrent Mode
- useTransition, useDeferredValue

## 정리

- 비동기 처리에서 성공과 실패하는 경우를 분리하면 코드의 복잡도를 낮출수있다.
- 함수의 에러 처리를 감싸는 catch문에서 해주는 것 처럼로딩과 에러처리도 컴포넌트를 쓰는곳에서 해주면 된다. ex) suspense, ErrorBoundary가 처리하도록
- 컴포넌트는 성공한 상태만 다루고 에러처리, 로딩처리는 더 잘 할수있는 외부(suspense, ErrorBoundary)에 위임해야한다. `객체지향에서는 전문가에게 시켜라 라는것이 존재한다.`

## 내 생각

비동기처리에 대해서 고민이 많았는데, 재미있게 봤다. 내용중에 의존성 역전의 원칙과 유사하다고 말씀해주신 부분에 대해서 생각을 해보면, 컴포넌트에서 로딩 처리를 모든 각자 컴포넌트안에 의존 하는게 아니고,  로딩처리를 하는 추상화된 suspense에 의존하도록 만든다는 점에서 유사하다고 말씀하신건가? 이부분은 좀 더 생각을 해보자.