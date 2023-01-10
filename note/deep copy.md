## 깊은 복사에 대해 얼마나 자세히 알고 있을까?

  

제가 자바스크립트에 관한 책을 읽으면 항상 언급되는 것이 깊은 복사에 대한 문제였습니다. 그만큼 중요하다고 생각하고, 누구나 한 번쯤은 깊은 복사에 대하여 고민해 봤던 적이 있을 것이라고 생각합니다

  

저는 깊은 복사에 대해 제대로 공부하기 전에는 자바스크립트에서 제공하는 JSON.stringify 같은 방법들이 깊은 복사에 대한 문제들을 문제없이 해결해 준다고 생각했습니다. 그리고 잘못 알고 있었던 것을 반성하며 글로 정리해 보려고 합니다.

  

<br/>

<br/>

  

## 복사를 하기 위한 다양한 방법들

  

아래부터는 다양한 복사 방법들을 보고 이 글에서 언급했던 깊은 복사에 대해서 몇 가지 질문들을 생각하면서 글을 읽으시면 더 좋을 것 같습니다.

  

<br/>

  

- 복사된 객체의 참조 주소와 기존의 객체의 참조 주소가 다른가요?

  

- 어떠한 타입도 문제없이 복사할 수 있나요?

  

- 객체 안에 객체가 있는 상황이 반복되어도 복사가 가능하며, 복사된 객체들의 참조 주소도 기존 객체의 참조 주소와 다른가요?

  

<br/>

  

### Spread Operator

  

Spread 연산자는 사용이 간단하고, 실제로 객체를 복사할 때 많이 쓰는 방법입니다.

  

기본적으로 Spread 연산자를 사용하기 위해서는 해당 객체에 `[Symbol.iterator]` 프로퍼티가 존재해야 합니다.

  

![iterator](https://user-images.githubusercontent.com/52567149/177507220-318b8b1a-5493-4e40-ba98-48c338e29439.png)

  

<br/>

  

아래에서 간단하게 Spread 연산자의 사용법에 대해서 알아보겠습니다.

  

<br/>

  

```tsx

const arr = [1, 2, 3, 4]

const copyArr = [...arr]

  

console.log(copyArr) // [1, 2, 3, 4]

```

  

<br/>

  

위 코드를 실행하면 arr 배열을 copyArr에 복사할 수 있습니다.

  

<br/>

  

```tsx

const arr = [1, 2, 3, 4]

const copyArr = [...arr]

  

arr[Symbol.iterator] = null

console.log(copyArr) // [error] arr is not iterable

```

  

<br/>

  

또한 위에서 언급했듯이 `[Symbol.iterator]`를 삭제한다면 위와 같은 에러가 발생하지만 다른 문제가 없다면 안전하게 배열을 복사하는 것을 볼 수 있습니다. 그러나 이러한 복사는 치명적인 문제가 존재합니다.

  

<br/>

  

```tsx

const arr = [1, 2, 3, 4, [1, 2]]

const copyArr = [...arr]

  

copyArr.push(5)

copyArr[4].push(3)

  

console.log(copyArr) // [1, 2, 3, 4, [1, 2, 3], 5]

console.log(arr) // [1, 2, 3, 4, [1, 2, 3]]

```

  

<br/>

  

위 코드를 보면 문제점을 찾을 수 있습니다. copyArr에 5를 push 했을 경우 두 배열의 참조 주소가 다르기 때문에 copyArr에만 5가 들어가지만, arr에는 없는 것을 확인할 수 있습니다. 하지만 copyArr 4번째 인덱스에 있는 배열에 푸시 했을 경우 기존의 arr에도 값이 들어가 있는 것을 확인할 수 있습니다.

  

바로 위에서 언급했던 질문 중에 `객체 안에 객체가 있는 상황이 반복되어도 복사가 가능하며, 복사된 객체들의 참조 주소도 기존 객체의 참조 주소와 다른가요?`의 경우에 맞지 않습니다. 기존 arr 값들을 하나씩 옮겨 담는 작업을 하기 때문에 즉 Spread 연산자를 사용한 복사는 depth가 1이 넘어가는 객체를 깊은 복사할 수 없기 때문에 이 방법은 얇은 복사라고 할 수 있습니다.

  

<br/>

  

### Array.prototype.slice

  

slice 메서드를 사용하는 방법의 문제점도 위에서 언급했던 Spread 연산자의 문제점과 같습니다. 기존의 배열에서 자르고 싶은 부분을 복사할 수 있는 메서드입니다.

  

```tsx

const arr = [1, 2, 3, 4, [1, 2]]

const copyArr = arr.slice() // [1, 2, 3, 4, [1, 2]]

copyArr.push(5)

copyArr[4].push(3)

  

console.log(copyArr) // [1, 2, 3, 4, [1, 2, 3], 5]

console.log(arr) // [1, 2, 3, 4,[1, 2, 3]]

```

  

<br/>

  

위에서 언급했던 문제와 동일하게 depth가 1이 넘어가는 객체를 깊은 복사할 수 없기 때문에 얇은 복사라고 할 수 있습니다.

  

<br/>

  

### Object.assign

  

자바스크립트에서 제공하는 Object, assign()을 사용하면 객체의 모든 데이터들을 복사해 대상 객체에 붙여 넣을 수 있습니다. 하지만 이러한 방법도 위에서 제시한 문제들을 해결할 수 없습니다.

  

```tsx

const object = {

test: 1,

testObject: {

a: 2,

b: 3,

},

}

  

const copyObj = Object.assign({}, object)

copyObj.testObject.a = 4

  

console.log(object.testObject) // {a: 4, b: 3}

console.log(copyObj.testObject) // {a: 4, b: 3}

```

  

<br/>

  

복사한 copyObj.testObject 안의 a를 변경했을 시 기존 object의 a 또한 변경되는 모습을 볼 수 있습니다. 그러므로 이 방법 또한 얇은 복사라고 할 수 있습니다.

  

<br/>

  

### MDN

  

<blockquote>

깊은 클로닝에 대해서, Object.assign() 은 속성의 값을 복사하기 때문에 다른 대안을 사용해야 합니다. 출처 값이 객체에 대한 참조인 경우, 참조 값만을 복사합니다.

</blockquote>

  

<br/>

<br/>

  

### JSON.stringify

  

JSON.stringify 을 사용한 방법은 깊은 복사에 대해 깊게 공부하기 전까지 정말 좋은 방법이라고 생각하고 있었고, 필자도 자주 사용했던 방법입니다.

  

<br/>

  

```tsx

const object = {

test: 1,

testObject: {

a: 2,

b: 3,

},

}

  

const copyObj = JSON.parse(JSON.stringify(object))

  

copyObj.testObject.a = null

  

console.log(copyObj.testObject) // {a: null, b: 3}

console.log(object.testObject) // {a: 2, b: 3}

```

  

<br/>

  

위에 언급했던 얇은 복사 방법들과는 다르게 JSON.stringify를 사용한 복사는 `객체 안에 객체가 있는 상황이 반복되어도 복사가 가능하며, 복사된 객체들의 참조 주소도 기존 객체의 참조 주소와 다른가요?` 에 대한 질문에 만족할 수 있는 방법입니다. 이렇게 될수있는 원리를 알아보면 JSON.stringify는 객체를 string 문자열로 변경해주는 역할을 하고, JSON.parse는 문자열을 다시 객체로 변경해주는 역할을 하기 때문입니다.

  

자바스크립트에서 string, number같은 기본형타입은 불변성(Immutable)을 가진 타입이고, 기본형 타입을 제외한 모든 타입 즉 객체타입은 가변성(mutable)을 가진 타입이기 떄문입니다. 그렇기 떄문에 문자열로 변경하는 작업에서 이전 객체에서의 참조가 없어지는 원리를 이용한 방법입니다.

  

<br/>

  

### 그렇다면 JSON.stringify는 아무런 문제점이 없는 방법인가요?

  

JSON.stringify를 이용한 방법에도 문제점이 존재합니다. 바로 JSON에서 허용하는 데이터 포맷이 아닌 경우에는 정상적으로 데이터를 복사할 수 없습니다. 즉 `어떠한 타입도 문제없이 복사할 수 있나요?` 경우에 맞지 않습니다.

  

<br/>

  

JSON에서 허용하는 데이터 포맷은 아래와 같습니다.

  

- Number

- String

- Boolean

- Array

- Object

- null

  

<br/>

  

예를 들면 객체 안의 함수나 메서드, 정규 표현식, Symbol, Date, Infinity, Map, Set, BigInt 같은 타입은 복사할 수 없습니다.

  

<br/>

  

### 저 위에 언급한 것들도 자바스크립트에서는 객체(Object) 아닌가요?

  

맞습니다 실제로 Map, Set는 타입을 검사해도 Object가 나옵니다. 실제로 저도 같은 생각을 가지고 있었습니다. 그리고 알아본 결과로는 Object.prototype.toString.call를 사용해서 나오는 2번째 값이 Object 일 경우에만 허용하는 것을 확인할 수 있었습니다.

  

<br/>

  

```tsx

console.log(Object.prototype.toString.call(new Set())) // [object Set]

console.log(Object.prototype.toString.call(new Map())) // [object Map]

console.log(Object.prototype.toString.call(function() {})) // [object Function]

console.log(Object.prototype.toString.call(BigInt)) // [object Function]

console.log(Object.prototype.toString.call({})) // [object Object]

```

  

<br/>

  

### 평균적으로 다른 방법들보다 속도 면에서 느리다는 단점이 있습니다.

  

![deepcopytest](https://user-images.githubusercontent.com/52567149/177692595-62767319-5e15-45b7-8704-4090743adecb.png)

  

[성능 테스트 해보기](https://jsben.ch/rmgqP)

  

첫 번째는 재귀를 사용한 복사, 두 번째는 lodash, 세 번째는 JSON.stringify를 사용하고 나온 성능 테스트 결과입니다.

  

<br/>

  

### 직접 함수를 구현하는 방법

  

직접 깊은 복사를 하는 함수를 구현하는 방법이 있습니다. 이 방법은 재귀를 사용한 방법이고, 제가 제시했던 3가지 문제점들을 해결해 줄 수 있습니다.

  

<br/>

  

target이 해당 타입이라면 재귀를 반복하고 아닐 경우 값을 리턴 함으로써 깊은 복사가 되도록 구현해야 합니다.

  

먼저 인자로 들어온 value 값의 타입을 확인해야 합니다. 그리고 타입을 검사하는 방법은 여러 가지가 있습니다. 첫 번째로 `typeof` 방법이 있지만 이 방법은 [], new Date(), null 등을 object로 판별합니다. 그렇기 때문에 현재 상황에서 올바른 방법이 아닙니다.

  

두 번째로 `instanceof`가 있습니다. 해당 연산자를 사용하면 객체가 특정 클래스에 속하는지 아닌지를 확인할 수 있습니다.

  

<br/>

  

```tsx

console.log([] instanceof Arrray) // true

console.log([] instanceof Object) // true

console.log(new Date() instanceof Object) // true

```

  

<br/>

  

이 방법을 사용해서 검사를 할 수는 있겠지만, Array, Date 같은 객체를 따로 타입을 검사하는 코드를 따로 작성해 주어야 하기 때문에 저는 사용하지 않았습니다.

  

세 번째는 `Object.prototype.toString`을 사용하는 방법이 있습니다. 해당 메서드를 사용하면 객체의 종류(일반 객체, RegExp, Fun, Date) 같은 객체까지 식별할 수 있습니다. 필자는 해당 메서드를 사용해서 타입을 검사했습니다.

  

<br/>

  

```tsx

Object.prototype.toString.call() // [object Undefined]

Object.prototype.toString.call(null) // [object Null]

Object.prototype.toString.call([]) // [object Array]

Object.prototype.toString.call({}) // [object Object]

Object.prototype.toString.call(new Date()) // [object Date]

```

  

<br/>

  

먼저 타입을 검사하는 코드를 보겠습니다.

  

<br/>

  

```tsx

const checkType = target => {

const CONSTRUCTOR_TYPE = {

'[object Array]': copyArray,

'[object Object]': copyObject,

'[object Map]': copyMap,

'[object Set]': copySet,

'[object Date]': copyDate,

'[object RegExp]': copyRegExp,

}

const type = Object.prototype.toString.call(target)

return CONSTRUCTOR_TYPE[type]

}

```

  

<br/>

  

checkType 함수는 target이 들어오면 `Object.prototype.toString.call(target)`을 사용해서 타입을 검사하고, 타입(key)에 맞는 타입이 있다면 해당 함수를 리턴합니다.

  

다음은 타입별로 재귀를 호출하는 함수를 살펴보겠습니다. 새로운 객체를 생성하고 재귀적으로 기존 객체의 값을 새로운 배열에 복사한 뒤 리턴합니다.

  

<br/>

  

```tsx

function copyObject(target) {

const result = {}

for (let key of Object.keys(target)) {

result[key] = copyObjectDeep(target[key])

}

return result

}

  

function copyArray(target) {

const result = []

for (let index in target) {

result[index] = copyObjectDeep(target[index])

}

return result

}

  

function copyMap(target) {

const result = new Map()

target.forEach((value, key) => result.set(key, copyObjectDeep(value)))

return result

}

```

  

<br/>

  

그다음은 메인 함수를 보겠습니다. 메인 함수에서 checkType 함수를 통해 맞는 타입이 있다면 해당 함수 결과를 리턴하고 아니면 target을 그대로 리턴해주도록 작성했습니다.

  

<br/>

  

```tsx

const copyObjectDeep = target => {

const isObject = checkType(target)

if (isObject) return isObject(target)

return target

}

```

  

<br/>

  

전체 코드는 아래에서 확인하실 수 있습니다.

  

<br/>

  

```tsx

const checkType = target => {

const CONSTRUCTOR_TYPE = {

'[object Array]': copyArray,

'[object Object]': copyObject,

'[object Map]': copyMap,

'[object Set]': copySet,

'[object Date]': copyDate,

'[object RegExp]': copyRegExp,

}

const type = Object.prototype.toString.call(target)

return CONSTRUCTOR_TYPE[type]

}

  

const copyObjectDeep = target => {

const isObject = checkType(target)

if (isObject) return isObject(target)

return target

}

  

function copyObject(target) {

const result = {}

for (let key of Object.keys(target)) {

result[key] = copyObjectDeep(target[key])

}

return result

}

  

function copyArray(target) {

const result = []

for (let index in target) {

result[index] = copyObjectDeep(target[index])

}

return result

}

  

function copyMap(target) {

const result = new Map()

target.forEach((value, key) => result.set(key, copyObjectDeep(value)))

return result

}

  

function copySet(target) {

const result = new Set()

target.forEach(prop => result.add(copyObjectDeep(prop)))

return result

}

  

function copyRegExp(target) {

return new RegExp(target.source, target.flags)

}

  

function copyDate(target) {

return new Date(target.getTime())

}

```

  

<br/>

  

위 코드가 성공적으로 깊은 복사를 수행할 수 있는지 테스트 코드 또한 작성해 볼 수 있습니다. 재귀로 구현한 함수를 테스트하기 위해서 재귀를 사용하는 방법으로 불변성까지 테스트하도록 작성했습니다.

  

<br/>

  

```tsx

const obj = {

bigInt: BigInt(1),

date: Date(),

string: 'ss',

number: 123,

reg: new RegExp('ab/c'),

isNaN: 123 / 'a',

infinity: 123 / 0,

arr: [1],

undefined: undefined,

null: null,

symbol: Symbol(1),

obj1: {

obj2: {

test: 100,

obj3: [1, 2, 3],

},

},

fun: function() {

return 100

},

}

  

test('객체 안에 객체안에 객체가 있어도 복사할 수 있다.', () => {

const copyObj = copyObjectDeep(obj)

expect(copyObj.obj1.obj2.test).toEqual(100)

})

  

test('Map,Set를 복사할 수 있다.', () => {

const map = new Map()

map.set(1, 'a')

const set = new Set()

set.add('응애')

expect(copyObjectDeep(map)).toEqual(map)

expect(copyObjectDeep(set)).toEqual(set)

})

  

test('객체 안에 배열을 복사할 수 있다.', () => {

const copyObj = copyObjectDeep(obj)

expect(copyObj.obj1.obj2.obj3).toEqual([1, 2, 3])

})

  

test('객체에 함수를 복사할 수 있다.', () => {

const copyObj = copyObjectDeep(obj)

expect(copyObj.fun()).toBe(100)

})

  

test('객체에 Date객체, 정규표현식, Infinity등을 복사할 수 있다.', () => {

const copyObj = copyObjectDeep(obj)

expect(copyObj).toEqual(obj)

})

  

test('객체의 불변성을 테스트한다.', () => {

const copyObj = copyObjectDeep(obj)

const recursiveDeepTest = (obj, copyObj) => {

for (let key in obj) {

if (typeof obj[key] === 'object' && obj[key] !== null) {

expect(obj[key] === copyObj[key]).toBe(false)

recursiveDeepTest(obj[key], copyObj[key])

}

}

}

recursiveDeepTest(obj, copyObj)

})

```

  

![deepjest](https://user-images.githubusercontent.com/52567149/177704550-674f6a27-5a6f-4d79-8945-cc5767e4c413.png)

  

테스트가 성공적으로 수행되는 것을 볼 수 있습니다. 이번에 깊은 복사를 구현하면서 재귀 코드를 테스트하기 위해서 재귀를 사용한다는 게 재미있긴 하지만, 해당 경험을 통해 좋은 인사이트를 얻을 수 있었습니다. 마지막으로 제가 구현한 함수가 완벽한 정답이 아니며. 또한 더 좋은 방법이 있을 수 있습니다.

  

<br/>

  

### Lodash 라이브러리

  

마지막으로는 많은 사람들이 흔히 사용하는 Lodash를 이용하여 깊은 복사를 하는 방법이 있습니다. 팀원들끼리 합의를 했거나, 이미 프로젝트에서 사용 중이라면 사용을 고려해 볼 수 있습니다.

  

Lodash cloneDeep을 사용하면 위에서 언급한 것들을 만족하는 결과를 얻을 수 있습니다. 아래 코드에서 간단하게 사용법만 알아보겠습니다.

  

<br/>

  

```tsx

const _ = require('lodash')

  

const obj = {

string: 'ss',

number: 123,

reg: new RegExp('ab/c'),

isNaN: 123 / 'a',

infinity: 123 / 0,

}

  

const copyObj = _.cloneDeep(obj)

```

  

<br/>

<br/>

  

## 결론

  

자바스크립트에서는 객체를 복사하는 다양한 방법이 존재한다. Spread Operator, Array.prototype.slice, Object.assign는 깊은 복사를 지원하지 못하고, JSON.stringify는 깊은 복사를 지원하지만 JSON 타입에 맞지 않는 유형이 있다면 복사를 못하는 문제점이 있기 때문에 주의해서 사용해야 한다.

  

결국 상황에 맞는 방법을 사용하는 것이 최고라고 생각한다. 특정한 상황에서는 위에 언급한 방법들이 최선의 방법이 될 수도 있다. 예를 들면 알고리즘 문제를 풀 때와 같은 간단한 배열, string, number만 저장하는 객체라면 얇은 복사, JSON.stringify를 사용하는 것도 좋은 방법이 될 수 있다. 하지만 그런 것이 아니라면 직접 구현하거나 lodash 같은 라이브러리를 사용하는 것을 고려해 보자.

  

객체의 불변성은 어디서나 언급될 정도로 중요한 문제지만 무조건은 없다고 생각한다. 객체가 무거운 상황이거나, 생명주기가 짧은 경우, 상태를 공유하는 상황일 때 가변으로 설계를 고려하는 것도 좋은 대안 될 수도 있다.

  

<br/>

<br/>

틀린 내용이 있다면 거침없는 피드백 부탁드립니다