## 이글을 작성하게된 계기

  

Hash 알고리즘 문제를 풀다가 Object와 Map으로 선언했을 때 속도에서 차이가 나는 것을 발견했습니다. 코드는 같지만 다른 한쪽은 Object, 다른 한쪽은 Map을 사용한 코드지만 속도에서 명확한 차이를 보였고, 이 문제를 정확하게 알아보고자 포스팅하게 되었습니다.

  

<br/>

  

## MDN 에서는 다음과 같이 설명합니다.

  

### Map, Object?

  

Map 객체는 간단한 키와 값을 서로 연결시켜 저장하며, 저장된 순서대로 각 요소들을 반복적으로 접근할 수 있도록 한다.

  

Object는 문자열을 값에 매핑하는데 사용한다. Object는 키를 값으로 설정하고, 값을 검색하고, 키를 삭제하고, 키에 저장된 내용을 검색할 수 있게 만들어준다. 그러나 Map 객체는 더 나은 맵이 되도록 하는 몇 가지 장점을 가지고 있다.

  

아래에서 Object와 Map의 차이점을 알아보겠습니다.

  

### Object의 키는 `Strings`이며, Map의 키는 모든 값을 가질 수 있다.

  

이 말은 Object의 키로 배열, 함수, Symbol 등등 어떠한 값을 넣든 String으로 저장됩니다. 실제로 궁금해서 확인해 봤습니다.

  

```jsx

const symbol = Symbol()

const arr = 'arr'

  

const Object = {

string: 'value',

[arr]: 'value',

[symbol]: 'value',

}

```

  

### 결과

  

![image](https://user-images.githubusercontent.com/52567149/183314801-0ebd35d0-e055-40cd-bcd2-930fe585b96e.png)

  

<br/>

  

반면 Map의 키는 모든 값을 가질 수 있습니다. 아래 코드에서 확인해 보겠습니다.

  

```jsx

const symbol = Symbol()

const fun = function() {}

const bigInt = BigInt(1)

const arr = []

const isNaN = 123 / 'a'

const isNull = null

  

const map = new Map()

map.set(symbol, 'value')

map.set(fun, 'value')

map.set(bigInt, 'value')

map.set(arr, 'value')

map.set(isNaN, 'value')

map.set(isNull, 'value')

console.log(map)

```

  

<br/>

  

이렇게 보니 Map 은 Object와 다르게 기능을 굉장히 유연하게 제공하는 것을 알 수 있습니다.

  

### 결과

  

![image](https://user-images.githubusercontent.com/52567149/183314701-6b1427ac-87bb-4b60-af73-6b26bfc2906d.png)

  

<br/>

  

### Object는 크기를 수동으로 추적해야하지만, Map은 크기를 쉽게 얻을 수 있다.

  

Object는 배열이 아니기 때문에 length 프로퍼티로 객체의 길이를 구할 수 없습니다. 대신 map은 size 프로퍼티를 사용해 Map의 길이를 구할 수 있다는 부분에서 차이점을 보입니다. 유사 배열 객체로 만들어서 length 프로퍼티를 생성하는 방법이 있지만, 결론적으로는 없다고 보면 될 것 같습니다.

  

<br/>

  

### Map은 삽입된 순서대로 반복된다.

  

이 말은 Map은 삽입된 순서대로 반복이 가능하다는 말 같습니다. 실제로 Map 속성에 `[Symbol.iterator]` 프로퍼티가 존재하기 때문에 `스프레드 연산자`, `for ... of`, `forEach`를 사용해서 순환하거나, 반복문을 적용할 수 있습니다. 하지만 Object는 요소를 반복하려면 `for ... in`, `Object.keys()`, `Object.values()`, `Object.entries()`를 거쳐서 배열로 변경하는 작업을 거친 후에 반복작업을 실행할 수 있습니다.

  

<br/>

  

![Screenshot from 2022-08-06 10-44-35](https://user-images.githubusercontent.com/52567149/183228815-f6a354be-4790-42a2-a236-5230dac057e6.png)

  

<br/>

  

### 객체(Object)에는 prototype이 있고, Map에도 기본 키들이 있다. (이것은 map = Object.create(null)를 사용하여 우회할 수 있다.)

  

처음에 이 문장을 보고 의아했는데, 결국은 Object, Map 둘 다 프로토타입이 존재하지만, Map의 프로토타입에는 clear, delete, forEach, get, has, set 같은 기본 키가 존재한다는 말 같습니다.

  

<br/>

  

## 성능 비교

  

자 이제 가장 중요한 성능 비교를 할 차례입니다. 모든 것은 상황에 맞게 사용해야 하기 때문에 결국 최선의 선택을 해야 합니다.

  

이전에 저는 Map을 사용해서 풀었던 알고리즘 문제는 Object를 사용한 코드에 비해서 속도가 느린 것을 확인했습니다. 그 후에 둘의 차이에 대해서 공부해 보기로 했습니다.

  

MDN에서는 이렇게 설명하고 있습니다.

  

<br/>

  

- 실행 시까지 키를 알 수 없고, 모든 키가 동일한 type이며, 모든 값들이 동일한 type 일 경우에는 Object를 대신해서 Map을 사용해라.

  

- 각 개별 요소에 대해 적용해야 하는 로직이 있을 경우에는 objects를 사용해라.

  

<br/>

  

음? 개별 요소에 적용해야 하는 로직이 있을 경우 objects를 사용하라고?

  

이 글을 봤을 때는 감이 잘 잡히지 않았습니다. 그래서 구체적으로 어느 상황에 뭐가 더 빠른지 테스트해 보기로 결정했습니다.

  

<br/>

  

먼저 len 길이의 Object와 Map을 만드는 코드를 작성하겠습니다.

  

```jsx

let obj = {},

map = new Map(),

len = 1000000

  

for (let i = 0; i < len; i++) {

obj[i] = i

map.set(i, i)

}

```

  

<br/>

  

아래에서는 console.time()을 사용하여 Map, Object를 가지고 몇 가지를 테스트하면서 특정 상황에서 무엇을 사용하면 좋을지 확인해 보겠습니다.

  

### key를 모르는 상황에서 값이 있는지 확인하기

  

<br/>

  

```jsx

let value

console.time('map test1')

value = map.has(999999) // 0.003ms

console.timeEnd('map test1')

console.time('obj test1')

value = obj.hasOwnProperty('999999') // 0.005ms

console.timeEnd('obj test1')

```

  

### Map : 0.003

  

### Object : 0.005

  

<br/>

  

제가 테스트했을 때는 차이는 미미했고, 여러 번 테스트하면 Object 가 더 빠른 경우도 있었습니다.

  

## key, Index를 아는 상황에서 값을 가져오고 싶을 때

  

```jsx

console.time('map test2')

const value = map.get(999999) // 0.01ms

console.timeEnd('map test2')

console.time('obj test2')

const value = obj[999999] // 0.01ms

console.timeEnd('obj test2')

```

  

### Map : 0.01

  

### Object : 0.01

  

<br/>

key, Index를 알고 있어도 value를 가져오는 속도 또한 비슷합니다.

  

## 데이터를 추가할 때

  

```jsx

console.time('map test3')

map.set(len - 1, len) // 0.001ms

console.timeEnd('map test3')

console.time('obj test3')

obj[len - 1] = len // 0.005ms

console.timeEnd('obj test3')

```

  

### Map : 0.001

  

### Object: 0.005

  

<br/>

  

## 데이터를 삭제할 때

  

### Map : 0.019

  

### Object: 0.019

  

<br/>

  

이렇게 데이터를 추가할 때는 어느 정도 차이가 있는 것으로 보였고, 또한 데이터를 삭제할 때는 비슷한 시간으로 명확한 차이를 발견하지 못했습니다. 여기까지 확인해 봤지만 저는 Map과 Object의 성능에 대한 특별한 차이를 느끼지 못했습니다. 그리고 현재까지 결과로 보면 Object와 Map 중 많은 기능을 제공해 주는 Map을 사용하는 것이 맞다고 생각했습니다.

  

<br/>

  

### 왜 그럼 내가 Map을 사용한 코드는 Object를 사용한 코드보다 느렸을까요?

  

밑에 코드에서 설명드리겠습니다. 새롭게 obj와 map을 선언하고 이번에는 n의 크기를 100000으로 줄이도록 하겠습니다.

  

<br/>

  

```jsx

let obj = {},

map = new Map(),

len = 100000

```

  

<br/>

  

이번에는 반복문을 통해서 Object와 Map에 데이터를 추가하는 코드를 추가하겠습니다.

  

<br/>

  

```jsx

console.time('map test4')

for (let i = 0; i < len; i++) {

map.set(i, i)

} // 7.8ms

console.timeEnd('map test4')

  

console.time('obj test4')

for (let i = 0; i < len; i++) {

obj[i] = i

} // 3.5ms

console.timeEnd('obj test4')

```

  

### Map : 7.8ms

  

### Object : 3.5ms

  

<br/>

  

이렇게 차이를 비교했을 때는 큰 차이를 보이는 것을 확인할 수 있습니다. 순환하는 크기(len)가 1000000이면 무려 4배의 차이를 보였습니다. 이처럼 반복하는 횟수가 많을수록 값을 가져오거나, 추가, 삭제 등 기능을 수행할 때 Object가 Map보다 뛰어난 성능을 보였습니다. 또 여기까지만 봤을 때는 `그럼 Object를 쓰면 되는 거 아니야?`라고 생각하실 수 있겠지만 이것은 key가 Number 타입일 때만 해당하고, 만약 key를 String으로 넣었을 때는 제 예상과는 많이 달랐습니다.

  

<br/>

  

### Key가 string 일 때

  

```jsx

console.time('map test4')

for (let i = 0; i < len; i++) {

map.set(i + 'string', i)

} // 16ms

console.timeEnd('map test4')

  

console.time('obj test4')

for (let i = 0; i < len; i++) {

obj[i + 'string'] = i

} // 32ms

console.timeEnd('obj test4')

```

  

### Map : 16ms

  

### Object : 32ms

  

<br/>

  

이렇게 Key 값을 string으로 넣었을 경우는 Map이 Object보다 뛰어난 성능을 보여주는 것을 확인할 수 있습니다. 여러 가지 테스트를 통해서 이제는 Object, Map 중에 더 나은 선택을 할 수 있도록 고민해 볼 수 있을 것 같습니다.

  

<br/>

  

### But

  

Map과 Object는 둘 다 Key, Value로 이루어져 있지만, Map은 Object를 완벽하게 대체할 수 없습니다. 현재 Map은 JSON을 지원하지 않기 때문에 JSON으로 변환하려면 직접 일일이 변환해 주는 작업을 거쳐야 합니다.

  

</br>

  

## 결론

  

이번 포스팅에서는 Object와 Map의 차이점에 대해서 알아보았습니다. 항상 어떤 기술을 사용할 때는 명확한 이유가 있어야 한다고 생각합니다.

  

Object, Map 두 개 다 장점과 단점이 존재합니다. 예를 들면 우리가 주로 사용하는 JSON 데이터는 Object 형식이기 때문에, Map은 Object를 완벽하게 대체할 수 없습니다. 또한 Map은 Object가 지원하지 않는 forEach, has, set, size 같은 유용한 기능을 제공합니다. 그리고 우리는 테스트를 통해서 어느 것이 어떤 상황에서 더 좋은 성능을 가지고 있는지 확인했습니다. 그리고 이것을 토대로 더 좋은 선택을 할 수 있을 것이라고 생각합니다.

  

만약 저와 같은 고민을 겪고 계신 분들이 있다면 이 글을 통해서 꼭 도움이 되었으면 좋겠습니다.

  

<br/>

  

### Reference

  

[https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Keyed_collections](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Keyed_collections)

  

[https://bretcameron.medium.com/how-javascript-maps-can-make-your-code-faster-90f56bf61d9d](https://bretcameron.medium.com/how-javascript-maps-can-make-your-code-faster-90f56bf61d9d)

  

[https://medium.com/front-end-weekly/es6-map-vs-object-what-and-when-b80621932373](https://medium.com/front-end-weekly/es6-map-vs-object-what-and-when-b80621932373)

  

틀린 내용이 있다면 거침없는 피드백 부탁드립니다