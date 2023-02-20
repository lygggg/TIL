Commonjs는 왜 만들어졌나? js에 문제점이 있기 때문에 만들어졌을 것이다.

브라우저가 성장함에따라 js는 인기를 얻고있었고,  js는 브라우저는 위해 만들어진 간편한 언어였지만, 이것을 서버사이드에서도 사용하고자 하는 수요는 지속적으로 있었다. 이런 수요로 제공되고 있던 기술도 있었으며,  js라는 언어로 클라이언트와 서버가 코드를 주고받은 것은 소통의 측면에서 엄청난 이점이였다. 하지만 이렇게 사용되던 라이브러리들 또한 표준이라고 할 수 없었으며 이 시기에 목소를 내던 사람이 있었다.

kevin Dangoor는 js에도 서버사이드를 위한 제대로된 표준이 필요하다는 initiative를 발표한다. 그리고 파이썬에는 있지만 js에는 없은 아쉬운 부분들을 지적한다.

그 내용은 다음과 같다.

1. standard library가 없다.
	- 파일이나 디렉터리를 읽을 수 없다.
2. standard interface가 없다.
	- 웹 서버, 데이터베이스 등에 연결하거나 쿼리하기 위한 표준 인터페이스가 없다.
3. package manager가 없다.
	- 패키지를 배포하고, 관리하고, ,일제히 설치하는 그런 기능이 존재하지 않는다.
4. module system이 없다.
	- 다양한 모듈들을 쉽게 load할 수 없고, 네임스페이스를 구분하지 않는다.
	- 
### 전역 변수 관리 문제
개발을 하다보면 캡슐화와 의존성에대한 이야기를 들어본적이 있을 것이다. 각각 다른 조각의 소프트웨어는 고립된 영역에서 개발되어야 한다. 그렇게 만들어진 조각이 서로 협력하면서 더 큰 소프트웨어를 만들게 된다. 이때 중요한 것은 모듈간에 충돌이 생겨서는 안된다는 것이다. 캡슐화가 잘되어있지 않다면 앱의 크기가 커질수록 충돌을 피하기는 어려워질 것이다. 그러므로 각 모듈을 잘 캡슐화하는 것은 관리포인트를 줄이고, 안정적인 앱을 관리하는데 필수적인 일이였다.

기존의 모듈은 의존성을 지키기 위해 순서대로 위치하게 만들었어야했다.

```javascript
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <title>useTodo.js Todos</title>
        <link rel="stylesheet" href="todos.css"/>
    </head>

    <body>
        <script src="../../test/vendor/json2.js"></script>
        <script src="../../test/vendor/jquery.js"></script>
        <script src="../../test/vendor/underscore.js"></script>
        <script src="../../useTodo.js"></script>
        <script src="../useTodo.sessionStorage.js"></script>
        <script src="todos.js"></script>
    </body>

    <!-- (...) -->

</html>
```

만약 script를 가져오는 순서를 섬세하게 관리해주지 않으면, 애플리케이션은 망가질 것이다.

## CommonJs :
commons에 의해 module systems이 만들어졌다. 이를 통해 브라우저 뿐만 아니라, 서버, 데스크탑 어플리케이션, 보안 샌드박스 등 (common)에도 이용 가능하게 되었다고 해서 common이라는 이름이 붙여졌다.

모든 곳(common)에서 자바스크립트를 사용하겠다는 의지가 보인다.

### 무엇이 달라졌나?
commonjs로 넘어오면서 많은 기능을 표준화하려고 했다.

-   Modules
-   Binary strings and buffers
-   Charset encodings
-   Binary, buffered, and textual input and output (io) streams
-   System process arguments, environment, and streams
-   File system interface
-   Socket streams
-   Unit test assertions, running, and reporting
-   Web server gateway interface, JSGI
-   Local and remote packages and package management


## 모듈화
commonjs로 넘어오면서 생긴 변화중 우리가 주목할만한 부분은 모듈화다. 이 모듈화는 아래와 같이 세 부분으로 이루어 진다.
- 스코프 : 모든 모듈은 자신만의 독립적인 실행 영역이 있어야 한다.
- 정의 : 모듈 정의는 exports 객체를 이용한다.
- 사용 : 모듈 사용은 require 함수를 이용한다.

이제 자바스크립트도 독립적인 실행영역이 생기고, 그 실행영역을 밖으로 보내고(exports), 사용하는 방법(require)이 생겼다.

```ts
const add = (a,b) => a + b;
const subtract = (a,b) => a - b;

exports.add = add
exports.subtract = subtract
```

```javascript
//app.js
const math = require('./math.js')

math.add(1,2) // 3
math.subtract(2,1) // 1
```

여기서 require는 함수다. 그렇기 떄문에 require는 if안에도 들어가 조건부로 다른 모듈의 내용을 가져올 수 있다.

```javascript
if(user.length > 0){
	const math = require('./math.js')
    //... Do something.. 
}
```

초창기에도 nodejs는 commonjs를 표준으로 채택해서 사용했으나 일부만 사용하고 노션을 변경했다.

commonjs와 nodejs는 약간의 차이점이 있다.

예를들면 module.export이다. 노드에서 module.export는 하나의 객체다. module이라는 객체 안에 default로 exports라는 변수를 가지고있는 그런 객체다. 반면 commonjs에서는 moduel.exports라는 객체를 가지고 있지 않다. 그래서 곧장 exports를 사용한다.

```javascript
exports =  (width) => {
  return {
    area: () => width * width
  };
}

//nodejs
module.exports = (width) => {
  return {
    area: () => width * width
  };
}
```

commonjs에도 문제점은 있었다.
1. ECMA standards의 지원 없이 독립적으로 개발되었기 때문에 언어 자체로는 commonjs를 포함하지 않는 다는 것이였다. 그러므로 브라우저 차원에서 이것을 지원할리도 없었다.
2. 정적으로 모듈을 분석하고, 트리쉐이킹 할 방법이 없었다.
3. 구현의 세부사항들이 로컬 환경에 기반해 있었던 commonjs는 브라우저의 맥락에서는 문제점이 있었다. 대표적으로 모듈을 동기적으로 로드한다는 것. 순차적으로 하나씩 모듈을 로드하는 브라우저 환경에서는 좋지 못한 선택이였다.

## AMD(Asynchronous Module Definition)
commonjs는 모듈을 동기적으로 로드한다고했는데, 이런 문제점을 개선하고자 하는 워킹 그룹이 AMD이다. commons에서 합의점을 찾다가 분리되어 나왔다. AMD가 목표로 하는 것은 브라우저 환경에서 모듈을 로드해야 할 때도 정상적으로 모듈을 사용할 수 있도록 정의하는 것이다.

commonjs에서 분리되어 나온 그룹이기 때문에 require이나 exports같은 문법을 그대로 사용할 수 있었다. 특징이라면 define함수가 있다.

브라우저에서는 모듈간 스코프가 존재하지 않는다. 떄문에 파일간 모듈로 구분하기 위해 define 함수의 클로저를 활용한다.
```javascript
// example.js
define(['lodash', 'yumyum'], function(_, Y) {
  console.log(_); // lodash
  console.log(Y); // yumyum
  return {
    a: _,
    b: Y,
  }
});
```

첫번째 인자에는 사용할 다른 사람들의 코드를 배열로 넣어준다. 그리고 두번째 인자에는 첫번째 인자에 배열로 넣어준 의존성 코드들을 인자로 받아온다.

```javascript
require(['example'], function (example) {
  console.log(example.a); // lodash
  console.log(example.b); // yumyum
  console.log(lodash); // undefined 또는 에러
});
```

## ESM(ECMAScript Modules)
이렇게 표준을 찾아다니다가 ECMAScript에서 모듈시스템에 대한 표준을 발표했다. 이 표준은 현재 모든 브라우저에서 지원한다. 이는 자바스크립트 자체로 모듈에 대한 문법을 가지게 되었다는 것을 의미한다.

### import와 export 
```javascript
// math.mjs
export const add = (a,b) => a + b
export function subtract (a,b) { a - b }
```

```javascript
// app.mjs
import {add, subtract} from "./math.mjs"

add(1,2) // 3 
subtract(2,1) // 1
```

### 스코프 구분 :
모듈을 사용하면 더 이상 변수들이 전역에 할당되지 않는다. 이전에는
```javascript
const 철수 = 20
```
와 같이 작성하고, window로 접근하면 값을 얻을 수 있었지만, 모듈로 사용하는 경우는 undefined를 할당한다.
```javascript
// ..js
console.log(window.철수) // 20

// ..mjs 
console.log(window.철수) // undefined
```

this도 마찬가지로 class script에서는 전역 공간에 this를 사용하는 경우 window를 가리키지만, mjs에서 this를 전역 공간에서 사용하면 undefined를 내뱉는다.

### 사용법 :

#### type = "module" :
브라우저에서 모듈을 사용하기 위해서는 해당 스크립트를 모듈로 여겨달라고 브라우저에게 말해야 한다. type="module"을 명시해주면 된다.
```javascript
<script type="module" src="main.mjs"></script>
<script nomodule src="fallback.js"></script>
```

만약 브라우저가 module을 이해한다면 해당 script를 사용하게 될 것이고, 지원하지 않는 브라우저의 경우 nomodule이 명시된 script를 사용한다.

nodejs 프로젝트에서 사용하는 또 다른 방법은 pakage.json에 type:module를 명시해주는 것
```javascript
{
  "name": "my-library",
  "version": "1.0.0",
  "type": "module",
  // ...
}
```
이렇게 작성하면 nodejs는 프로젝트 내에 존재하는 파일들을 esm으로 여길 것이고, 덕분에 파일명을 .mjs로 변환해주지 않아도 된다.

nodejs 프로젝트에서 사용하는 또 다른 방법은 바벨같은 트랜스파일러를 사용하는 방법으로 내부적으로 import문을 require문으로 바꿔주고 있기 때문이다.

class script는 매번 다시 평가하지만, module은 단 한번만 평가한다.

```javascript
<script src="classic.js"></script>
<script src="classic.js"></script>
<!-- 2번 실행 -->

<script type="module" src="module.mjs"></script>
<script type="module" src="module.mjs"></script>
<script type="module">import './module.mjs';</script>
<!-- 1번만 실행 -->
```

### ESM은 static하다:
commons를 사용하거나, 리액트에서 webpack을 사용하는 경우 파일 확장자를 생략하는 경우가 있었다. 하지만, esm은 static하다. import와 export를 컴파일 시점에 결정한다. nodejs나 webpack해서 사용하던 것처럼 실행중에 그것을 결정하는게 아니라 esm의 경우엔 문법적으로 static하게 작성하도록 강제한다. 때문에 파일 확장자와 경로를 꼭 명시해야한다.

```javascript
import {shout} from './lib.mjs';

// 지원안함:
import {shout} from 'jquery';
import {shout} from 'lib.mjs';
import {shout} from 'modules/lib.mjs';

// 지원:
import {shout} from './lib.mjs';
import {shout} from '../lib.mjs';
import {shout} from '/modules/lib.mjs';
import {shout} from 'https://simple.example/modules/lib.mjs';
```

또 다른 의미는 반드시 최상단에서 사용해야 한다. commonjs 같은 경우는 require문이 함수였기 떄문에 if문 안에서도 사용할 수 있었다.

하지만 esm에서는 import가 함수가 아닌 하나의 키워드로 사용되고 있기 떄문에 변수에 할당한다던지, if문과 같이 중첩된 구조 안에서 사용되어서는 안된다.

### dynamic import :
static import 같은 경우엔 중첩문 안에서는 import를 사용할 수 없었지만, dynamic import는 사용가능하다.
```javascript
button.addEventListener('click', event => {
    import('./dialogBox.js')
    .then(dialogBox => {
        dialogBox.open();
    })
    .catch(error => {
        /* Error handling */
    })
});
```
