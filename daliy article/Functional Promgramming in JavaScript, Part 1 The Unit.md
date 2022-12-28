### Monad, Gonad, Mossad?

> 모나드는 주어진 타입 시스템 안에 다른 타입시스템을 집어넣는 구조다. 모나딕 타입 시스템은 내부 요소의 타입의 모든 중요한 측면을 유지하면서 모나드에 특정한 기능을 추가한다.

> 모나드는 endofunctors 범주에 있는 모노이드 일뿐이다.
> (A monad is just a monoid in the category of endofunctors.)

### Introduction To Functional Programming
함수형 프로그래밍은 선언적 프로그래밍 패러다임이다.
명령형 프로그래밍과 달리 컴퓨터가 동작하는 방식을 말하지 않고(how to do) 해주길 원하는 것(what to do)을 설명합니다.

보다 정확하게 진정한 함수형 프로그래밍에는 제어 흐름이 없다.
즉 for, while, if, try, throw 또는 catch 문이 없다.

함수형 프로그래밍 언어에는 이에 상응하는 기능을 제공한다.
- for, while 대신 map, reduce
- if 대신 3항 조건 연산자
- 오류를 throw 하는 대신 함수의 결과로 반환

함수형 프로그래밍은 부수 효과를 피한다. 더 정확하게 말하면, 함수형 프로그래밍에는 부수 효과가 있을 수 있지만 나머지 코드와 분리된다. 그리고 부수효과는 HTTP 요청 보내기, 화면에 표시되는 정보 변경, 로그 기록 또는 이러한 부수 효과가 있는 함수 호출과 같이 함수 외부의 상태를 수정하는 모든 것이다.

마지막으로, 함수형 프로그래밍은 불변성을 선호한다. 즉, 함수는 인수를 수정해서는 안된다.
예를 들어, `[1, 2, 3].push(4)`는 초기 배열을 변경하고 새 길이를 반환한다.
반면에 `[1, 2, 3].concat(4)`는 값이 `[1, 2, 3, 4]`인 새 배열을 반환한다. 즉 초기 객체를 변경하지 않는다.

### Why Bother With Functional Programming?
왜 부수효과를 사용하고 우리가 원하는 모든 것을 변형하지 않습니까?
우리는 수십 년 동안 명령형 프로그래밍을 해왔습니다. `부수효과와 뮤테이션 없는` 프로그래밍은 코드를 더 예측 가능하게 만들고 오류를 덜 발생시킵니다. 또한 병렬로 코드를 쉽게 실행할 수 있으며 오늘날의 컴퓨터에 장착된 수많은 CPU의 이점을 누릴 수 있습니다. FP 코드는 실행이 컨텍스트에 의존하지 않기 떄문에 더 예측이 가능합니다. 함수를 오류를 발생시키지 않으며 동일한 입력이 주어지면 항상 동일한 결과를 반환합니다.

##### 부수 효과가 있는 코드 예시
```ts
var global = 0;
const square = x => {
	global++;
	return x * x;
};
const main = () => {
	var res = square(5);
	res += global;
	return res;
}

main();
main();
```

##### 부수효과와 뮤테이션 없는 코드 예시
```ts
const square = x => x * x;
const main = g => {
	return square(5) + g;
};

const global = 1;
main(global);
main(global + 1);
```

모든 함수 호출을 try/catch에 포함하는 것이 지겹지 않습니까?

다른 함수에 의해 유발되는 부작용으로 인한 버그 사냥에 지겹지 않은가요?

### Pure Functions (순수 함수)
나는 말 그대로 함수형 프로그래밍의 코드 단위, 즉 함수에 대해 이야기함으로써 기본부터 시작할 것입니다.

FP에서 함수는 순수해야 합니다. 순수 함수에는 다음과 같은 특성이 있습니다.

`그 결과는 입력에만 의존합니다`

```ts
// impure
var changeResult = true;
const a => changeResult ? a + a : a;

// pure
const fn = (a, changeResult) => changeResult ? a + a : a;
```

`본체 외부의 어떤 변수도 변경하지 않습니다.`

```ts
//impure
var nbCall = 0;
const fn = () => {
	nbCall++;
};

// pure
const fn = (arg, nbCall) => {
 return {
 result,
 nbCall: nbCall++,
 }
}
```

`파라미터를 변경하지 않습니다.`
```ts
// impure
const inc = i => i++; // i = i + 1

// pure
const inc = i => i + 1; // return i + 1
```

`오류를 던지지 않습니다.`
오류를 던지면 실행 흐름이 중단되고 함수는 오류가 실제로 catch되는 경우 오류가 catch될 위치를 제어할 수 없습니다. 오류를 던지는 것은 다른 사람들이 처리할 수 있기를 바라는 마음으로 폭탄을 던지는 것과 같습니다. 그리고 함수 사용자가 try/catch로 감싸도록 합니다. 대신 부드럽게 오류를 반환할 수 있습니다.

```ts
// impure
const add = (a, b) => {
	if(typeof a !== "number" || typeof b !== "number") {
	throw new Error("add must take two numbers");
	}
	return a + b;
};

// pure
const add = (a, b) => {
	if(typeof a !== "numer" || typeof b !== "number") {
	return { error: new Error("add must take numbers") };
	}
	return { result: a + b };
}
```

당신을 알지 못하는 사이에 이미 순수 함수를 사용하고 있을지도 모릅니다.
여기서 배워야 할 중요한 것은 순수 함수와 순수 함수를 구별하는 방법입니다.

### Currying
순수 함수보다 더 나은 것이 무엇인지 아시나요? 하나의 인수만 취하는 함수입니다.
값에서 다른 값으로의 간단한 매핑이 되기 때문에 더 좋습니다. 이러한 함수는 객체로 대체될 수 있습니다.

```ts
const increment = v => v + 1;
increment(3); // 4

const increment = {
	1: 2,
	2: 3,
	4: 4,
};
increment[3]; // 4
```

#### 파라미터가 하나인 함수론 덧셈을 못하는데?
할 수 있다. 함수가 다른 함수를 반환하도록 합니다.

```ts
function add(a) {
	return function(b) {
	return a + b;
	};
}

// or, using arrow functions
const add = a => b => a + b;

add(3)(5);
```

이런 종류의 함수를 커리 함수라고 한다.
이름은 개념을 개발한 수학자 Haskell Curry에서 따왔습니다. 커리 함수를 사용하면 다른 함수를 조작하는 코드에서 예상되는 내용을 알 수 있습니다. 함수를 실행하려면 항상 단일 인수가 필요합니다.

이것은 또한 보다 일반적인 코드를 허용합니다.

많은 사람들에게 인사 메시지를 보내야 한다고 가정해 봅시다.

```ts
const getMessage = greet => name => `${greet} ${name}`;
const sayHelloTo = getMessage("Hello");

sayHelloTo("world"); // "Hello world"
sayHelloTo("marmelab") // "Hello marmelab"
```

##### TIP
add(a)(b) 대신 add(a,b)를 호출할 수 있도록 하려면 다음과 같은 uncurry 도우미 함수를 사용할 수 있습니다.

```ts
const uncurry = fn => (...args) => {
	const result = args.reduce((prevResult, arg) => {
	if(typeof prevResult === "function") {
		return prevResult(arg);
		}
		
	return prevResult;
	}, fn);

return typeof result === "function" ? uncurry(result) : result;
};

const add = unCurry(a => b => c => a + b + c);

add(1)(2)(3); // 6
add(1, 2, 3); // 6
add(1, 2)(3); // 6
add(1)(2, 3); // 6
```

또한 여러 인수가 있는 함수를 커리 함수로 변환할 수 있습니다.
```ts
const curry = (fn, length = fn.length) => (...args) => {
	if(args.length >= length) {
		return fn(...args);
	}

	return curry(
		(...nextArgs) => fn(...args, ...nextArgs),
		fn.length - args.length
	);
}

const add = curry((a, b, c) => a + b + c);
add(1)(2)(3); // 6
add(1, 2, 3); // 6
add(1, 2)(3); // 6
add(1)(2, 3); // 6
```

이 커리 구현은 얼마나 많은 인수를 받을지 미리 알지 못한다면 가변 인자 인수를 취하는 함수에서 작동하지 않습니다.
 그런데 이런 경우에는 애초에 왜 이런 기능을 가지고 있는 걸까요?
 결국 다양한 인수로 함수를 커리하는 것은 불가능합니다.
 또한 디폴트 파라미터를 사용하는 함수에서도 오작동합니다.

```ts
const mul = (a, b =2) => a * b;

const badlyCurriedMul = curry(nul);
badlyCurriedMul(1);

console.log(mul.length);

const curriedMul = curry(mul, 2);
curriedMul(1);
curriedMul(1, 5);
```

lodash 또는 ramda와 같은 라이브러리는 더 나은 커리 구현을 제공합니다.

### Function Composition

하나의 매개변수만 사용하는 함수의 가장 큰 이점 중 하나는 합성입니다.

```ts
const add5 = a => a + 5;
const double = a => a * 2;
const add5ThenMultiplyBy2 = a => double(add5(a));

add5ThenMultiplyBy2(1); // 12
```

함수가 하나의 인수만 취하면 호출이 연결될 수 있습니다.
하나는 다른 하나의 결과를 가져옵니다.
그러나 f1(f2(f3(...))) 를 작성하는 것은 빨리 지루해집니다. 고맙게도 함수 호출을 합성하는 도우미 함수를 만들 수 있습니다. 합성이라고 합시다.

```ts
const compose = (f1, f2) = > arg => f1(f2(arg));

const add5ThenMultiplyBy2 = compose(
	double,
	add5
);
add5ThenMultiplyBy2(1);
```

##### TIP
compose를 사용하면 실행의 역순으로 함수를 작성하게 됩니다.
이 순서의 이유는 수학에서 나오며 논리입니다.
간단히 말해서, f1을 f2와 합성하면 "f1 of f2 of x"라고 읽는 f1(f2(x))이 됩니다.

자 이제 더 복잡한 것을 얻기 위해 합성할 수 있는 단위 함수가 생겼습니다. 레고같은 느낌입니다.

그러나 compose는 두 개의 함수 인수만 사용합니다.
여러 함수와 함께 작동하면 더 좋을 것 입니다.
우리는 어떤 수의 인수로도 작동하도록 함수를 일반화하는 방법이 필요합니다.
모노이드 패턴이 필요합니다. 이 시리즈의 다음 게시물에서 그것에 대해 이야기하겠습니다.

## 결론
- 함수형 프로그래밍은 선언적 프로그래밍 패러다임이다.
- 함수형 프로그래밍에서 함수는 순수해야한다. 부수효과와 뮤테이션이 없어야 한다는 것이다. 우리는 더 예측가능한 코드를 작성해야한다. 함수는 오류를 발생시키지 않으며 동일한 입력이 주어지면 항상 같은 결과를 반환한다.
- 순수 함수에는 다음과 같은 특성이 있다. 첫번쨰 그 결과는 입력에만 의존한다. 두번째 본체 외부의 어떤 변수도 변경하지 않는다. 세번쨰 파라미터를 변경하지 않는다. 네번째 오류를 던지지 않는다.
- 숨수 함수보다 더 나은 것은 하나의 인수만 취하는 함수다.
- 하나의 매개변수만 사용하는 함수의 가장 큰 이점 중 하나는 합성이다.