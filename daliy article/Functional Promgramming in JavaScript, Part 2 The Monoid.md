TLDR: 모노이드는 2인자 함수를 다인자 함수로 변환(reduce)

모노이드 reduce. 결합법칙과 항등원. 분산 컴퓨팅. 비동기 합성

숫자 더하기, 문자열 연결, 배열 연결 및 함수 합성의 유사점은 무엇입니까?
그들은 모두 모노이드입니다. 그리고 모노이드는 매우 유용합니다.
monoid라는 용어는 범주 이론에서 유래했습니다.

`이것은 종종 concat이라고 하는 특정 이항 연산(binary operation)과 결합될 때 3가지 특성을 갖는 요소 집합(a set of elements)을 설명합니다.`

- 마그마
	- 연산은 집합의 두 값을 동일한 집합의 세 번째 값으로 결합해야 합니다. a와 b가 집합의 일부이면 concat(a,b)도 집합의 일부여야 합니다. 범주 이론에서는 이것을 마그마라고 합니다.
- 결합 법칙
	- 연산은 `결합 법칙`이 성립해야 합니다. concat(x, concat(y,fz))는 concat(concat(x, y), z)와 같아야 합니다. 여기서 x, y, z는 집합의 값입니다. 연산을 그룹화하는 방법에 관계 없이 순서가 존중되는 한 결과는 동일해야 합니다.
		- 수학에서 결합법칙은 이항연산이 가질 수 있는 성질이다. 한 식에서 연산이 두 번 이상 연속될 때, 앞쪽의 연산을 먼저 계산한 값과 뒤쪽의 연산을 먼저 계산한 결과가 항상 같을경우 그 연산은 결합 법칙을 만족한다고 한다.
- 항등원
	- 집합은 연산과 관련하여 중립적인 요소(항등원)를 가지고 있어야 합니다. 그 중립 요소가 다른 값과 결합되어 있으면 변경해서는 안됩니다.
		- `concat(element, neutral) == concat(neutral, element) == element`

### 숫자 덧셈은 모노이드입니다.
두 개의 숫자를 더하면 JavaScript Number 인스턴스 세트를 조작하게 됩니다.
더하기 연산은 두 개의 숫자를 사용하며 항상 숫자를 반환합니다. 따라서 마그마입니다.

덧셈은 결합 법칙이 성립합니다.
```ts
(1 + 2) + 3 == 1 + (2 + 3); // true
```

마지막으로 숫자 0은 덧셈 연산의 중립 요소(항등원)입니다.
따라서 정의에 따르면 숫자는 더하기 연산에 대해 모노이드를 형성합니다.
#### TIP
팁: concat 함수의 이름을 concat으로 지정하지 않아도 모노이드가 존재한다는 것을 알 수 있습니다. 여기서는 +입니다.

### String Concatenation Is a Monoid
JavaScript에서 concat은 String 인스턴스의 메서드입니다. 그러나 순수 함수를 추출하는 것은 쉽습니다.

```ts
const concat = (a, b) => a.concat(b);
```

이 함수는 두 문자열에 대해 작동하고 문자열을 반환합니다. 그래서 마그마입니다.
또한 결합 법칙이 성립합니다.

```ts
concat("hello", concat(' ', "world")); // "Hello world"
concat(concat("hello", " "), "world"); // "Hello world"
```

중립 요소(항등원)이 존재합니다.

```ts
concat("hello", ""); // "hello"
concat("", "hello"); // "hello"
```
따라서 정의에 따르면 문자열은 결합 연산에 대해 모노이드를 형성합니다.

### Function Composition Is A Monoid. (함수 합성도 모노이드입니다.)

```ts
const compose = (func1, fun2) => arg => func1(fuc2(arg));
```

Compose는 두 개의 함수를 인수로 사용하고 함수를 반환합니다. 그래서 마그마입니다.
이제 결합성과 중립성을 살펴보겠습니다.

```ts
const compose = (f1, f2) => arg => f1(f2(arg));

const add5 = a => a + 5;
const double = a a=> a * 2;
const resultIs = a => `result: ${a}`;

const doubleThenAdd5 = compose(
	add5,
	double
);
// a=> a*2+5
doubleThenAdd5(3); // 11

compose(
compose(
resultIs,
add5
),
double
)(3); // result 11

conpose(
resultIs,
compose(
add5,
double
)
)(3);  // 11  위와 동일, 결합 법칙

const neutral = v => v;

compose(
	add5,
	neutral
)(3);
compose(
neutral,
add5
)(3);
```

### Reducing Function Arguments (함수 인자 변환 : 2인자 함수를 다인자 함수로)
함수를 조작할 때 두 개의 인수를 취하는 함수를 임의의 수의 인수로 취하는 함수로 바꾸는 방법이 필요하다고 설명했습니다. 숫자 덧셈부터 시작하겠습니다. 더하기 연산에는 두 개의 인수가 필요합니다. 임의의 크기의 배열에 어떻게 적용합니까? Array.reduce를 사용합니다.

```ts
const add = (a, b) => a + b;
const addArray = arr => arr.reduce(add, 0);
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
addArray(numbers); // 55
```

이것은 다른 모노이드에서도 동작합니다.

```ts
const concat = (a,b) => a.concat(b);
const concatArray = arr => arr.reduce(concat, "");
const strings = ["hello", " ", "world"];
const concatArray(strings); // hello world;

const compose = (f1, f2) => arg => f1(f2(arg));
const composeArray => arr => arr.reduce(compose, x => x);
const resultIs = a => `result: ${a}`;
const add5 = a => a + 5;
const double = a => a * 2;
const functions = [resultIs, add5, double];
const myOperation = composeArray(functions);
myOperation(2);
```

일반화: 모노이드가 있을 떄 다음을 호출하여 두 개의 인수를 취하는 함수를 인수의 배열을 취하는 함수로 변환할 수 있습니다.
```ts
[value1, value2, value3, ...].reduce(concat, neutral);
```

## Splitting Computation In Chunks (덩어리로 나누기).
모노이드 연산은 결합 법칙이 성립하므로 값 배열을 더 작은 청크로 자르고 각 배열에 concat 연산을 적용한 다음 concat 연산을 사용하여 결과를 다시 결합할 수 있습니다.

```ts
const concat = (a,b) => a + b;
const bigArray = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
bigArray.reduce(concat, 0);
const result1 = bigArray.slice(0,5).reduce(concat, 0); // 15
const result2 = bigArray.slice(5).reduce(concat, 0); // 40
concat(result1, result2); // 55
```

이 방법의 이점은 계산 단위(코어)에 대규모 연산을 분산하는 것입니다.
이것은 모든 모노이드에서 가능하므로 병렬 프로그래밍이 함수형 패러다임에 의해 쉬워지는 이유를 설명합니다.

### Async Composition(비동기 합성)
동일한 논리를 사용하여 비동기 함수(또는 promise를 반환하는 모든 함수)를 합성하는 함수를 만들 수 있습니다. 모노이드 구조를 형성하기만 하면 됩니다.
```ts
const fetchJoke = async number => fetch(`http://api.icndb.com/jokes/${number}`);
const toJson = async response => response.json(); 
const parseJoke = json => json.value.joke;

const getJoke = async number => parseJoke(await toJson(await fetchJoke(number)))
getJoke(23).then(console.log);

const asyncCompose = (func1, func2) => async x => func1(await func2(x));

asyncCompose(parseJoke, asyncCompose(toJson, fetchJoke))(23).then(console.log); // "Time waits for no man. Unless that man is Chuck Norris."

asyncCompose(asyncCompose(parseJoke, toJson), fetchJoke)(23).then(console.log); // "Time waits for no man. Unless that man is Chuck Norris."

const neutralAsyncFunc = x => x; asyncCompose(a => Promise.resolve(a + 1), neutralAsyncFunc)(5) // Promise(6)
asyncCompose(neutralAsyncFunc, a => Promise.resolve(a + 1))(5) // Promise(6)

const asyncComposeArray = functions => functions.reduce(asyncCompose, x => x);
const asyncComposeArgs = (...args) => args.reduce(asyncCompose, x => x);

const getJoke2 = asyncComposeArgs(parseJoke, toJson, fetchJoke); getJoke2(23).then(console.log); // "Time waits for no man. Unless that man is Chuck Norris."
```

### Async Flow (앞에 있는 함수부터 실행)
한 가지 문제가 있습니다. 이것은 거꾸로 읽습니다.
따라서 compose()를 사용하는 대신, 정확히 동일한 작업을 수행하지만 그 함수의 순서가 반대인 flow()를 사용하겠습니다.

```ts
const fetchJoke = async number => fetch(`http://api.icndb.com/jokes/${number}`);
const toJson = async response => response.json();
const parse = json => json.value.joke;

const flow = (func1, func2) => async x => func2(await func1(x));

const flowArray = functions => functions.reduce(flow, x => x);
const flowArgs = (...args) => args.reduce(flow, x => x);

const getJoke2 = flowArgs(fetchJoke, toJson, parseJoke);

```

훨씬 읽기 쉽습니다. 나는 flowArgs()에 인수로 전달된 함수의 순차적 실행 모델을 사용할 수 있습니다.
저는 이 flowArgs() 패턴을 좋아합니다. 테스트 가능성, 캡슐화 및 지연된 실행 측면에서 많은 이점이 있습니다.

### 결론
모노이드는 매우 일반적이며 Array.reduce와의 링크로 인해 매우 유용합니다.
실제보다 더 어려워 보이게 만드는 모노이드에 대한 학술 기사를 보고 겁먹지마세요

임의의 길이의 배열의 요소를 합성해야 하는 경우 모노이드를 찾으세요.
이 시리즈의 다음 게시물에서는 Array.map에 숨겨진 초능력에 대해 이야기하겠습니다.
그것은 Functor의 시간이 될 것이며, 이것은 Power Rangers의 악당의 이름이 아닙니다.