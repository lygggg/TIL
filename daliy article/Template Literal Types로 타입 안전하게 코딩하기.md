

## Template Literal Type이란?
간단히 말해, Template Literal Type이란 기존 TypeScript의 String Literal Type을 기반으로 새로운 타입을 만드는 도구입니다.

### 예시 1: 가장 간단한 형태
```ts
type Toss = "toss";

// type TossPayments = "toss payments"
type TossPayments = `${Toss} payments`;
```

### 예시 2: 하나의 Union Type
```ts
type Toss = "toss";
type Companies = 'core' | 'bank' | 'securities' | 'payments' | 'insurance';

// type TossCompanies = 'toss core' | 'toss bank' | 'toss securities' | ...;
type TossCompanies = `${Toss} ${Companies}`
```

Emplate Literal Type을 Union type(합 타입)과 함께하면, 결과물도 Union Type이 된다.

예를들어, 위 예시에서 `toss` 타입과 `'core' | 'bank' | 'securities' | ...` 타입을 Template Literal Type으로 연결하면 `'toss core' | 'toss bank' | 'toss securities' ...`와 같이 확장되는 것을 확인할 수 있다.

### 예시 3: 여러 개의 Union Type
```ts
type VerticalAlignment = "top" | "middle" | "bottom";
type HorizontalAlignment = "left" | "center" | "right";

// type Alignment = // | "top-left" | "top-center" | "top-right" // | "middle-left" | "middle-center" | "middle-right" // | "bottom-left" | "bottom-center" | "bottom-right"
type Alignment = `${VerticalAlignment} - ${HorizontalAlignment}`;
```
여러 개의 Union Type을 연결할 수도 있다.
예를 들어, 위에서는 `VerticalAlignment` 타입과 `HorizontalAlignment` 타입을 연결하여, `${VerticalAlignment} - ${HorizontalAlignment}`타입을 만들었다.

원래하면 중복해서 Alignment 타입을 다시 정의해야 했겠지만, Template Literal Type을 사용함으로써 중복 없이 더욱 간결하게 타입을 표현할 수 있게 되었다.

### 예시 4: 반복되는 타입 정의 없애기
```ts
// 이벤트 이름이 하나 추가될때 마다...
type EventNames = 'click' | 'doubleClick' | 'mouseDown' | 'mouseUp';

type MyElement = {
	addEventListener(eventName: EventNames, handler: (e: Event) => void): void;

	onClick(e: Event): void;
	onDoubleClick(e: Event): void;
	onMouseDown(e: Event): void;
	onMouse(e: Event): void;
};
```

이벤트에 대한 핸들러를 등록할 때, `addEventListener('event', handler)`와 `onEvent = handler`의 두 가지 형식을 모두 사용할 수 있는 `MyElement`타입을 생각해보자.
```ts
// 두 가지 방법 모두 사용할 수 있는 경우
element.addEventListener('click', () => alert('I am clicked!')); 
element.onClick = () => alert('I am clicked!');
```

예를 들어, `click`이벤트를 구독할 떄, 위의 두 가지 방법을 모두 사용할 수 있는 것이다.

요소에 추가할 수 있는 이벤트의 종류는 자주 변경된다. 예를들어, 브라우저 API가 바뀌면서 `'pointerDown'`과 같은 이벤트가 새로 추가될 수 있다.

이런 경우, TypeScript 4.1 이전에는 매번 수동으로 여러 곳의 타입을 수정해야 했다.
우선 addEventListener의 인자로 사용되는 이벤트
이름 `EventNames`타입에 `pointerDown`을 넣어야 했다.
또 `onPointerDown`메서드를 명시해야 했다.
하지만 Template Literal Type을 이용하면 한곳만 수정해도 모두 반영되도록 할 수 있다.

```ts
type EventNames = 'click' | 'doubleClick' | 'mouseDown' | 'mouseUp';

type CapitalizedEventNames = Capitalize<EventNames>;

type HandlerNames = `on${CapitalizedEventNames}`;

type Handlers = {
	[H in HandlerNames] : (event: Event) => void
};

// 원래 MyElement 그대로 작동
type MyElement = Hanlders & {
	addEventListener: (eventName: EventNames, handler: (event: Event) => void) => void;
}
```

1. `CapitalizedEventNames` 타입을 정의할 때, TypeScript 4.1에서 추가된 `Capitalize<T>`타입을 사용하여 `EventNames`의 첫 글자를 대문자로 만들었다.
2. `HandlerNames`타입을 만들 때, Template Literal Type으로 onClick과 같이 on 접두사를 붙였다.
3. `Hanlders`타입에서는 기존의 `onClick`, `onMouseDown`과 같은 이벤트 핸들러를 메서드로 가지도록 했고,
4. 마지막으로 `MyElement`에서는 `addEventListener`메서드를 가지는 객체와 연결하여 원래와 동일한 동작을 하는 타입을 만들 수 있었다.
이제 `EventNames`만 수정하면 `MyElemnt`에서 이벤트를 구독하는 양쪽 모두 대응이 되므로, 코드가 깔끔해지고 실수의 여지가 적어졌다.

## Conditional Type과 더 강력한 추론하기
Template Literal Type은 Conditinal Type과 함께 더 강력하게 사용할 수 있다.

### Conditional Type 되짚어보기
Conditional Type은 JavaScript의 삼항 연산자와 비슷하게 분기를 수행하면서, 타입을 추론하는 방법이다. 고급 TypeScript 사용에서 강력한 타입 연산을 하기 위해서 빠지지 않는다.

#### 예시 1: 제네릭 타입 인자 꺼내오기
Conditional Type을 가장 자주 사용하는 경우로, `Promise<number>`와 같은 타입에서 `number`를 꺼내오고 싶은 상황을 생각해보자.

```ts
type PromiseType<T> = T extends Promise<infer U> ? U :never;

// type A = number
type A = PromiseType<Promise<number>>;

type B = PromiseType<Promise<string | boolean>>;

type C = PromiseType<number>;
```
위 코드를 살펴보면, `PromiseType<T>` 타입에 `Promise<number`타입을 인자로 넘기면 `number`타입을 얻고 있다.

삼항 연산자처럼 생긴 부분 가운데 `X extends Y`와 같이 생긴 조건 부분은 `X`타입의 변수가 `Y`타입에 할당될 수 있는지에 따라 참값이 평가된다.

##### 예시
- `true extends boolean`: `true`는 `boolean`에 할당될 수 있으므로 참
-  `toss extends string`: `toss` 는 `string` 에 할당될 수 있으므로 참
- `string extends number`: 문자열은 숫자 타입에 할당될 수 없으므로 거짓

조건식이 참으로 평가될 때에는 `infer`키워드를 사용할 수 있다. 예를 들어, `Promise<number> extends Promise<infer U>`와 같은 타입을 작성하면, `U`타입은 `number`타입으로 추론된다. 이후 참인 경우에 대응되는 식에서 추론된 `U`타입을 사용할 수 있다.

예를 들어, `Promise<number> extends Promise<infer U> ? : never`에서는 조건식이 참이고 `U`타입이 number로 추론되므로, 이를 평가한 타입의 결가는 number가 된다.

반대로 `number extends Promise<infer U> ? U : never`에서는 조건식이 거짓이므로 이를 평가한 결과는 never가 된다.

#### 예시 2: Tuple 다루기
`[string, number, boolean]`과 같은 TypeScript의 Tuple Type에서 그 꼬리 부분인 `[number, boolean]`과 같은 부분만 가져오고 싶은 상황을 생각해보자.

Conditional Type과 Variadic Tuple Type을 활용함으로써 이를 간단히 구현할 수 있다.

```ts
type TailOf<T> = T extends [unknown, ...infer U] ? U : [];

// type A = [boolean, number];
type A = TailOf<[string, boolean, number]>;
```

간단한 형태로 튜플이 비어있는지 검사하기 위해, 아래와 같은 `IsEmpty<T>`타입을 정의할 수 있다.

```ts
type IsEmpty<T extends any[]> = T extends [] ? true : false;

type B = IsEmpty<[]>;

type C = IsEmpty<[number, string]>;
```

#### 초급 예시 1: 간단한 추론
```ts
type InOrOut<T> = T extends `fade${infer R}` ? R : never;

// type I = "In"
type I = InOrOut<"fadeIn">;
type O = InOrOut<"fadeOut">;
```
`'fadeIn' | 'fadeOut`과 같은 타입에서 앞의 `fade`접두사를 버리고 `'In' | 'Out'`만 가져오고 싶은 상황을 생각해보면

`Promise<number>`에서 `number`를 가져오는 것과 유사하게, Conditional Type을 이용하여 접두사를 제외할 수 있다.

#### 중급 예시: 문자열에서 공백 없애기
위의 예시를 응용하면 문자열의 공백을 없애는 타입을 정의할 수 있다. 예를 들어, 아래와 같이 오른쪽의 공백을 모두 제거한 타입을 만들 수 있다.

```ts
// type T = "Toss"
type T = TrimRight<"Toss       ">;
```

`TrimRight<T>`타입은 재귀적 타입 선언을 활용한다.
```ts
type TrimRight<T extends string> = 
	T extends `${infer R}`
		? TrimRight<R>
		: T;
```

위 코드를 살펴보면, `infer R`문 뒤에 하나의 공백이 있다. 즉, `T`타입의 오른쪽에 공백이 하나 있다면, 공백을 하나 빠뜨린 것을 `R`타입으로 추론하고, 다시 `TrmRight<R>`을 호출한다.

만약 공백이 더 이상 존재하지 않는다면, 원래 주어진 타입 그대로 반환한다.

TypeScript에는 `if`문이 존재하지 않지만, 만약 존재한다고 가정했을 때 아래와 같이 작성할 수 있다.

```ts
type TrimRight<T extends string> = 
	if(T extends `${infer R} `) {
	return TrimRight<R>;
	} else {
	return T;
	}
```

#### 중급 예시 2: 점으로 연결된 문자열 Split하기
재귀적 타입정의를 활용하면 `'foo.bar.baz'`와 같은 타입을 `['foo','bar','baz']`로 나누는 타입을 정의할 수 있다.

```ts
type Split<S extends string> = 
	S extends `${infer T}.${infer U}`
		? [T, ...Split<U>]
		: [S];

// type S = ["foo", "bar", "baz"];
type S = Split<"foo.bar.baz">;
```

주어진 `S`타입에서 첫번째 (`.`)을 찾고, 그 앞 부분을 `T`, 뒷 부분을 `U`로 추론한다. 이후 이를 `[T, ...Split<U>]`와 같이 재귀적으로 하나씩 값을 이어 나가면서 원하는 결과 타입을 만들어 나간다.

이 경우에도 `if`문이 있다는 가정 하에 pseudo-code로 정리해볼 수 있다.

```ts
type Split<S extends string> =
	if (S extends `${infer T}.${infer U}`) {
	return [T, ...Split<infer I>];
	} else {
	return [S];
	}
``` 


#### 고급 예시: lodash.sst()함수 타입 추론하기

`lodash.set()`는 아래와 같이 문자열로 된 접근자를 이용하여 객체의 깊은 프로퍼티까지 수정할 수 있는 함수입니다.

```ts
const someObject= {
	toss: {
		core: {
			client: {
				platform: "foo"
			}
		}
	}
};

lodashSet(someObject, "toss.core.client", { platform: "bar" });

// Error : "bar" is not assignable to type '{ platform: string }'
lodashSet(someObject, "toss.core.client", "bar");
```

Template Literal Type이 있기 전, 이런 함수는 타입 안전하게 사용할 수 없어 세번째 인자를 any로 규정해야 했습니다. 그러나 위에서 살펴본 타입 정의를 조합하면 `lodash.set()`를 더욱 안전하게 타이핑할 수 있습니다.

`lodash.set()`함수를 정확하게 타이핑하기 위해서는 아래의 `ValueOf<T, P>`타입이 필요합니다. `ValueOf<T, P>`타입을 객체의 `T`와 접근 경로 `P`가 주어졌을 때, `T`를 `P`경로로 순서대로 접근했을 때 결과로 나오는 타입을 나타냅니다.

```ts
interface Foo {
	foo: {
		bar: {
			baz: string;
		}
	}
}

// type A = { bar: { baz: string }};
type A = ValueOf<Foo, ["foo"]>;

// type B = { baz: string }
type B = ValueOf<Foo, ["foo", "bar"]>;

// type C = string;
type C = ValueOf<Foo, ["foo", "bar", "baz"]>;
```

만약 위와 같은 `ValueOf<T, P>`이 있다면, 위에서 만들었던 `Split<S>`과 조합하여 쉽게 lodash.set()함수에 타입을 부여할 수 있을 것입니다.

```ts
function lodashSet<Type, Path>(
 obj: Type,
 path: Path,
 value: ValueOf<Type, Split<Path>>
): void;
```

이제 `ValueOf<T, P>`타입을 만들어보자. `if`문과 내부 타입 선언이 있는 pseudo-code로 나타내면, 아래와 같이 코드를 작성할 수 있다.

```ts
type ValueOf<Type, Paths> =
	type Head = Paths[0];
	type Tail = TailOf<paths>;
	if(/*Tail의 길이가 0이다*/) {
		return type[Head];
	} else {
	return ValueOf<Type[Head], Tail>;
	}
```

`ValueOf<T, P>`타입이 그렇게 동작한다면, 위의 `Foo`예시에서는 아래와 같이 차례대로 값이 계산될 것입니다.
```ts
ValueOf<Foo, ["foo", "bar"]>
== ValueOf<Foo["foo"], ["bar"]>
== ValueOf<Foo["foo"]["bar"], []>
== Foo["foo"]["bar"]
```

작성했던 의사 코드를 유효한 TypeScript 코드로 나타내면 다음과 같습니다.

```ts
type ValueOf<Type, Paths extends any[]> =
/*
 IsEmpty<TailOf<Paths>>가 참이면 * == TailOf<Paths>가 빈 Tuple이면
*/
IsEmpty<TailOf<Paths>> extends true
	? Type[HeadOf<Paths>]
	: ValueOf<Type[HeadOf<Paths>], TailOf<Paths>>;
```
위 내용을 모두 조합하면 lodash.set()을 안전하게 다룰 수 있다.

### Template Literal Type의 응용
위에서 살펴본 바와 같이, Template Literal Type을 Conditional Type과 사용하면 더욱 많은 코드를 안전하게 사용할 수 있다. [awesome-template-literal-types](https://github.com/ghoullier/awesome-template-literal-types) 를참고해라

대표적으로 화제가 되었던 예시들에 대해서 살펴보자.
1. TypeScript로 JSON 파서 만들기

```ts
// type Json = { key1: ['value1', null]; key2: 'value2' };
type Json = ParseJson<'{ "key1": ["value1", null], "key2": "value2" }'>;
```

코드와 같이 JSON 문자열을 바로 TypeScript 타입으로 옮길 수 있다는 Proof-of-concept로 화제가 되었다.

2. document.querySelector를 타입 안전하게 사용하기
```ts
const a = querySelector('div.banner > a.call-to-action'); //-> HTMLAnchorElement 
const b = querySelector('input, div'); //-> HTMLInputElement | HTMLDivElement 
const c = querySelector('circle[cx="150"]') //-> SVGCircleElement 
const d = querySelector('button#buy-now'); //-> HTMLButtonElement 
const e = querySelector('section p:first-of-type'); //-> HTMLParagraphElement
```

a태그를 선택했을 떄 결과값이 `HTMLAnchorElement`가 되는 것을 확인할 수 있다.

3. Express의 Route Parameter로부터 타입 추론하기

![image](https://user-images.githubusercontent.com/52567149/210721465-1760aea5-681f-4367-81e5-7dc38c908ec7.png)


https://toss.tech/article/template-literal-types