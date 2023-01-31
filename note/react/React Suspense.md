## 이 개념은 무엇인가?
Suspense는 React 16.6부터 추가된 주로 js 번들의 Lazy Loading을 위한 기능이였다. lazy로 동적 임포트하고, Suspense 안에 넣어주면 자동으로 번들이 분리(코드 스플리팅)되고 해당 컴포넌트가 필요할 떈 비동기적으로 번들을 가져왔다.

서스펜스를 사용하면 fallback prop으로 로딩 UI를 넣어주면 동적 임포트 해오는 동안 로딩 UI를 선언적으로 지정할 수 있다.

주로 JS번들을 스플리팅하고 웹 자원중 코드를 Lazy Loading하는데 쓰였던 Suspense는 React 18에서 무엇이든 기다릴 수 있는 기능으로 확장된다. 하지만 공식문서에서는 아직까지 lazy를 제외한 데이터 페칭에 사용되는 것은 권장하지 않는다. 하지만 React Core team에서는 Suspense는 이미지, 스크립트, 그 밖의 비동기 작업을 기다리는데에 모두 사용될 수 있도록 하는 궁극적인 목표를 가지고 있다.

즉 한문장으로 정리하면 Suspense는 컴포넌트의 렌더링을 어떤 작업이 끝날 때 까지 잠시 중단시키고, 다른 컴포넌트를 먼저 렌더링할 수 있도록 도와주는 기능이다.데이터 페칭 등의 응답을 기다리는 작업중 fallback ui를 보여주고 그사이 우선순위가 높은 다른 UI를 먼저 렌더링 할 수 있다.

## 어떤 문제를 해결해주나?
suspense는 React 18의 핵심 기능 중 하나인 Concurrency(동시성)과 깊은 관련이 있으며, 선연형 UI 라이브러리로서의 에러, 비동기, 완료 상태를 조금더 선언적으로 표현할 수 있게 도움을 준다.

아래 코드를 보자
```ts
const resource = fetchProfileData();

function ProfilePage() {
return (
<Suspense fallback={<h1>Loading profile...</h1>}>
<ProfileDetails />
<Suspense fallback={<h1>Loading posts...</h1>}>
<ProfileTimeline />
</Suspense>
</Suspense>
);

}

function ProfileDetails() {
// Try to read user info, although it might not have loaded yet
const user = resource.user.read();

return <h1>{user.name}</h1>;
}

function ProfileTimeline() {
// Try to read posts, although they might not have loaded yet
const posts = resource.posts.read();

return (
<ul>
{posts.map((post) => (
<li key={post.id}>{post.text}</li>
))}
</ul>
);
}
```
만약 user 데이터를 가져오는 api가 1000ms가 걸리고, post를 가져오는 api가 1100ms 딜레이가 걸린다고 했을 때 우리가 서스펜스를 사용한다면 user데이터를 가져오고 난후에 post를 가져오는게 아니라 user, post 데이터를 Concurrent
하게 요청하게 되고, 이에따라 모든 데이터가 로드 되는 시간은 1100ms가 걸리게 된다.


## Declarative React(선언적 리액트)
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
위 코드는 어떤 문제가 있을까? 바로 한 컴포넌트 내에서 성공, 로딩, 에러를 처리하는 로직이 같이 있다는 점이다. 만약 여러 의존성을 가진 비동기 처리들이 동시에 실행된다면 너무 복잡해진다.

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
즉 하나의 컴포넌트가 많은 분기를 가지고, 개발자가 직접 코드를 작성해서 데이터가 로딩중인지, 에러가 발생했는지, 정상적으로 로딩되었는지를 다소 명령적으로 확인해야했다.

하지만 Suspense를 사용하게 되면 `로딩을 처리하는 부분에 대한 책임과 데이터가 로딩되었는지를 판단하는 책임을 suspense에 맞기고` 컴포넌트는 정상적으로 데이터가 로딩되었을 때 의 UI만 선언적으로 구현할 수 있게 된다. 이로써 개발자는 로딩 여부를 판단하고 로딩 중일 때의 UI를 보여주는 책임을 Suspense에게 위임하고, 원래 보여야 하는 UI를 로딩중, 에러 발생과 같은 상태와 분리해서 생각할 수 있게 된다. 이렇게 함으로써 우리는 비즈니스로직과 UI에 더 집중할 수 있게 된다.

## Concurrent React (fix Waterfall Rendering)
위 코드에서 user와 post 데이터를 페칭해오는 컴포넌트에서 suspense를 사용한 경우 걸린 시간은 1100ms지만 suspense를 사용하지 않는다면 1000ms 요청이 끝나고 resolve 된후 1100ms 요청이 실행되기 때문에 2100ms가 소요된다.

당연히 Suepense를 사용한 케이스가 더 나은 사용자 경험을 제공하고, React 공식문서에서는 이를 Render-as-You-Fetch라고 부른다

어떻게 이런게 가능한 것일까?

_With Suspense, we don’t wait for the response to come back before we start rendering. In fact, we start rendering pretty much immediately after kicking off the network request —_ [_React Official Docs_](https://17.reactjs.org/docs/concurrent-mode-suspense.html#approach-3-render-as-you-fetch-using-suspense)

즉, 아직 데이터 로딩이 완료되지 않아도 Fallback UI를 보여주게 되더라도, React는 hidden모드로 자식 컴포넌트를 렌더하게 되며, 이로 인해 아직 부모 컴포넌트가 로딩중이더라도 자식 컴포넌트는 자신의 Data Fetching을 Concurrent 하게 수행할 수 있다.(스케쥴링 [희열])

### 정리
- Suspense를 사용하면 `로딩을 처리하는 부분에 대한 책임과 데이터가 로딩되었는지를 판단하는 책임을 suspense에 맞기고` 컴포넌트는 정상적으로 데이터가 로딩되었을 때 의 UI만 선언적으로 구현할 수 있게 된다. 이로써 우리는 비즈니스 로직에 더욱 집중할 수 있다.
- Suspense를 사용하면 여러 컴포넌트의 로딩 상태를 묶어서 표현할 수 있다.
- Suspense를 활용하면 데이터를 가져오는 시점을 DOM Mount 이전으로 당길 수 있습니다. 즉 Waterfall 문제를 해결한다.

## 이 개념은 어떻게 동작하는가?

## Suspense의 내부 동작
1. Suspense가 Child Component를 렌더할 때, 캐시로부터 데이터를 읽으려고 시도한다.
2. 데이터가 없으면, ChildComponent의 캐시(의 역할을 하는 것)는 promise를 throw한다.
3. Suspense는 이 promise를 받아 fallback을 처리하고 정상적으로 resolve 되었다면 다시 ChildComponent를 보여준다.
```tsx
_// Infrastructure.js_  
let cache = new Map();  
let pending = new Map();

function fetchTextSync(url) {  
  if (cache.has(url)) {  
    return cache.get(url);  
  }  
  if (pending.has(url)) {  
    throw pending.get(url);  
  }  
  let promise = fetch(url).then(  
    response => response.text()  
  ).then(  
    text => {  
      pending.delete(url);  
      cache.set(url, text);  
    }  
  );  
  pending.set(url, promise);  
  throw promise;  
}

async function runPureTask(task) {  
  for (;;) {  
    try {  
      return task();  
    } catch (x) {  
      if (x instanceof Promise) {  
        await x;  
      } else {  
        throw x;  
      }  
    }  
  }  
}
```
위는 Andrew Clark이 설명한 모델의 개념적 구현이다.

위 코드를 보면, 만약 아직 로딩중이라면 아직 resolve되지 않은 promise를 리턴하고, 값이 도착하면 promise가 resolve되면서 Set한 캐시의 value를 리턴한다.

즉 간단하게 말하면 promise를 throw했을때는 suspense가 이를 확인하고 resolve 될 때까지 fallback UI를 보여주다가 promise가 resolve되면, 다시 throw한 곳으로 되돌아가서 정상적인 컴포넌트 로딩을 처리할 수 있게 되는 것이다.