functor가 객체와 같은 복잡한 데이터 구조의 유효성을 검사하기 위해, 다른 유효성 검사 함수(Validator)를 조합하는 방법을 알아봅시다.

### 함수 인자 Reducing
2장에서 reduce 호출을 초기화하기 위한 중립적 모노이드가 있는 경우, concat 연산을 통해 단일 모노이드로 줄이는 방법을 설명했습니다.

예를들어 , 부울 세트(값은 true 또는 false)와 concat으로 구성된 연산(객체)으로 구성된 모노이드의 경우, 다음과 같이 모노이드 배열을 줄일 수 있습니다.

```ts
const and = (a, b) => a && b;
const neutral = true;
[true, false, true].reduce(and, neutral);
```

또한 이전 기사에서 입력의 유효성을 검사할 수 있는 "Validator" 모노이드를 소개했습니다.

```ts
const isGreaterThan = max =>
	validator(value => {
		if(value > max) {
		return;
		}
		return `value must be greater than ${max}`;
	});

const isSmallerThan = min =>
	validator(value => {
		if(value < min) {
			return;
		}
		return `value must be less than ${min}`;
	});

const isBetweenThreeAndTen = isGreaterThan(3).and(isSmallerThan(10));
```
따라서 리듀스 기술을 사용하여 Validator 배열을 단일 Validator로 변환할 수 있습니다. 왜 그렇게 할까요?

### The Object Validator (객체 유효성 검사기)
제가 원하는 것은 다음과 같이 개체의 속성을 확인하는 것입니다.
```ts
{
	firstName: 'John', // should not be empty
	lastName: 'Doe', // should not be empty
	age: 35, // should be over 18
}
```

개별 속성에 대한 유효성 검사기를 작성하는 방법을 이미 알고있습니다.
필요한 것은 전체 개체의 유효성 검사하기 위해 단일 유효성 검사기 함수로 결합하는 방법입니다.
익숙하게 들리나요? 예, 우리는 reduce 기술을 사용할 것입니다.

우리가 원하는 것은 제가 specification(사양)이라고 부르는 객체를 취하는 것입니다.
그리고 이것을 단일 Validator로 변환합니다.

```ts
{
	firstName: isNotNull, // should not be empty
	lastName: isNotNull, // should not be empty
	age: isGreaterThan(18), // should be over 18
}
```

다음은 첫 번째 구현입니다.

```ts
// 먼저 우리는 중립적인 값(항등원)이 필요합니다.
// Valid concat Valid === Valid며, Valid concat Invalid는 Invalid입니다. 즉 자신은 결과에 영향을 미치지 않습니다.
const neutral = Validator(value => {
	return Validation.Valid(value);
})

const objectValidator = spec => {
	const validators = Object.keys(spec)
		.map
}
```