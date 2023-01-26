# #6: React Query and TypeScript

## **Generics**
React Query는 제네릭을 많이 사용한다. 이것은 라이브러리가 실제로 데이터를 가져오지 않고, 당신의 api가 반환하는 데이터의 타입을 알 수 없기 때문에 필요합니다.

호출할 때 쿼리가 기대하는 Generic을 명시적으로 지정하도록 지시한다.

```ts
function useGroups() {
	return useQuery<Group[], Error>('groups', fetchGroups)
}
```

시간이 지남에 따라 React Query는 useQuery hook에 더 많은 제네릭을 추가했는데, 이는 더 많은 기능이 확장 되었기 때문이다. 위 코드는 잘 작동한다. Custom Hook의 데이터 속성이 Group[] | undefined로 올바르게 입력되었는지 확인하고 오류 타입이 Error | undefind인지 확인한다.

그러나 많은 상황에서는 잘 쓰지 않는데, 다른 제네릭이 두개 더 필요하기 때문이다.

## **The four Generics**

```ts
export function useQuery<
	TQueryFnData = unknown,
	TError = unknown,
	TData = TQueryFnData,
	TQueryKey extends QueryKey = QueryKey
>
```
- TQueryFnData: queryFn에서 반환된 타입이다. 위의 예제에서는 Group[]이다.
- TError: queryFn에서 예상되는 오류 타입이다. 위의 예제에서는 Error이다.
- TData: 데이터 프로퍼티가 최종적으로 보유하게 될 타입이다. 데이터 프로퍼티가 queryFn이 반환하는 것과 다를 수 있으므로 선택 옵션을 사용하는 경우에만 해당된다. 그렇지 않으면 queryFn이 반환하는 것으로 기본설정 된다.
- TQueryKey: queryFn에 전달된 queryKey를 사용하는 경우에만 해당 queryKey으이 타입이다.

모든 Generic에는 기본값이 있다. 즉 이 값을 제공하지 않으면 TypeScript가 해당 타입으로 되돌아간다. 이것은 자바스크립트의 기본 매개 변수와 거의 비슷하게 동작한다.

```ts
function multiply(a, b =2) {
	return a * b
}

	multiply(19)
	multiply(10, 3)
```

## **Type Inference**
TypeScript는 어떤 타입이어야 하는지 추론(또는 파악)할 때 동작한다. 코드를 쉽게 작성할 수 있을 뿐만 아니라 읽기 쉬워진다. 많은 경우, 타입 추론은 코드를 자바스크립트와 똑같이 보이게할 수 있다.

```ts
const num = Math.random() + 5 // number

function greet(greeting = 'ciao') {
	return `${greeting}, ${getName()}`
}
```

제네릭인한, 일반적으로 어떻게 사용되었으냐에 따라 추론될 수 있는데, 매우 훌륭하게 동작한다. 수동으로 제공할 수도 있지만, 대부분 생략된다.

```tsx
function identity<T>(value: T): T {
  return value
}

// 🚨 no need to provide the generic
let result = identity<number>(23)

// ⚠️ or to annotate the result
let result: number = identity(23)

// 😎 infers correctly to `string`
let result = identity('react-query')
```

## **Partial Type Argument Inference**
TypeScript에 아직 없다. 이것은 기본적으로 하나의 Generic을 제공하는 경우 모든 Generic을 제공해야 함을 의미한다. 그러나 React Query 에는 Generics에 대한 기본ㄱ밧이 있으므로 이러한 값이 사용된다는 사실을 바로 알 수 는 없다. 결과적으로 발생하는 오류 메시지는 복잡할 수 있다.


예시를 보자

```tsx
function useGroupCount() {
  return useQuery<Group[], Error>('groups', fetchGroups, {
    select: (groups) => groups.length,
    // 🚨 Type '(groups: Group[]) => number' is not assignable to type '(data: Group[]) => Group[]'.
    // Type 'number' is not assignable to type 'Group[]'.ts(2322)
  })
}
```
3번쨰 Generic을 제공하지 않았기 때문에 기본값은 Group[]이지만 선택자(Selector function)에서 number을 반환한다. 이를 해결할 단순한 방법  하나는 세번째 제네릭을 추가하는 것이다.

```ts
function useGroupCount() {
	return useQuery<Group[], Error, number>("group", fetchGroups, {
		select: (groups) => groups.length,	
	})
}
```
Partial 타입 인수 추론이 없는 한, 우리는 여태 얻은 것으로 작업해야한다.

## **Infer all the things**
Generic을 전혀 전달하지 않는 것으로 시작하고 TypeScript가 무엇을 해야할지 결정하도록 해보자. 그렇게 동작하기 위해서는 queryFn이 좋은 반환 타입을 가져야 한다. 물론 명시적으로 반환 타입 없이 해당 함수를 인라인 하면 임의의 값을 가질 수 있다. 왜냐하면 이것이 axios 또는 fetch가 제공하는 것이다.

```ts
function useGroups() {
	return useQuery('groups', () =>
		axios.get('groups').then((res) => res.data)
	)
}
```

api 계층을 쿼리에서 분리하고싶다면, 암묵적인 것을 피하기 위해 타입 정의를 추가해야한다. 그러면 React Query가 나머지를 유추할 수 있다.
```tsx
function fetchGroups(): Promise<Group[]> {
  return axios.get('groups').then((response) => response.data)
}

// ✅ data will be `Group[] | undefined` here
function useGroups() {
  return useQuery('groups', fetchGroups)
}

// ✅ data will be `number | undefined` here
function useGroupCount() {
  return useQuery('groups', fetchGroups, {
    select: (groups) => groups.length,
  })
}
```

이 접근 방식의 장점은 다음과 같다.

- Generics를 더 이상 수동으로 지정하지 않는다.
- 세번째 및 네번째 제네릭이 필요한 경우에 동작한다.
- 더 많은 제네릭이 추가되어도 계속 작동한다.
- 코드가 덜 혼잡스럽고, 자바스크립트에 더 가깝다.

## **What about error?**
제네릭을 사용하지 않으면 오류가 unknown으로 추론된다. 왜 Error가 아닐까? 이것은 의도된 것이다. 왜냐하면 자바스크립트에서는 무엇이든 throw할 수 있다. Error 타입이 아니어도 된다.

```tsx
throw 5
throw undefined
throw Symbol('foo')
```

React Query는 Promise를 반환하는 기능을 담당하지 않기 때문에 어떤 타입의 오류가 발생하는지 알 수 없다. 그래서 unknown이 맞다. 일단 TypeScript를 사용하면 여러 Generic이 있는 함수를 호출할 때 일부 Generic을 건너뛸 수 있으므로 이 문제를 잘 처리할 수 있지만, 현재로서는 Generic을 전달해야 하고 Generic을 전달하지 않으려면 다음과 같은 검사 인스턴스로 타입을 좁힐 수 있다.

```tsx
const groups = useGroups()

if (groups.error) {
  // 🚨 this doesn't work because: Object is of type 'unknown'.ts(2571)
  return <div>An error occurred: {groups.error.message}</div>
}

// ✅ the instanceOf check narrows to type `Error`
if (groups.error instanceof Error) {
  return <div>An error occurred: {groups.error.message}</div>
}
```

오류가 있는지 확인하기 위해 무언가 확인해야 하므로 검사 인스턴스는 전혀 나쁜 생각처럼 보이지 않고, 실행 시 오류가 실제로 프로퍼티 메시지를 가지고 있는지 점검할 수 있다.

