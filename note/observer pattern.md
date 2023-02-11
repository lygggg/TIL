
## 옵저버 패턴

  

옵저버 패턴은 주체가 어떤 객체의 상태 변화를 관찰하다가 상태 변화가 있을 때마다 메서드 등을 통해 옵저버 목록에 있는 옵저버들에게 변화를 알려주는 디자인 패턴입니다.

  

여기서 주체란 객체의 상태 변화를 보고 있는 관찰자입니다. 옵저버들이란 객체의 상태 변화에 따라 전달되는 메서드 등을 기반으로 추가 변화사항이 생기는 객체들을 의미합니다.

  

![image](https://user-images.githubusercontent.com/52567149/181366547-0bb6c6e2-6380-4a5c-a982-be5ea8cae5e0.png)

  

옵저버 패턴은 주로 이벤트 핸들링 시스템에 사용하며 MVC(Model-View-Controller) 패턴에도 사용됩니다. 예를 들어 주체라고 볼 수 있는 모델에서 변경 사항이 생겨 update() 메서드로 옵저버 뷰에게 알려주고 이를 기반으로 컨트롤러가 작동합니다.

  

### 옵저버 패턴을 사용하여 Todolist 구현해 보기

  

```jsx

<body>

<ul></ul>

<form>

<input required type="text" />

<button type="submit">추가</button>

</form>

</body>

```

  

먼저 todolist를 출력할 ul 태그, todo를 입력할 input, 추가 버튼 요소를 생성해 줍니다. 이후에 자바스크립트 코드를 작성하겠습니다.

  

할 일을 담을 배열과 옵저버 배열을 생성합니다.

  

```jsx

let todolist = []

let observers = []

```

  

할 일 추가 기능을 구현한 코드입니다. 여기까지는 특별한 것은 없습니다.

  

```jsx

function addTodo(item) {

todolist.push(item)

}

  

const form = document.querySelector('form')

form.addEventListener('submit', event => {

event.preventDefault()

const input = form.elements[0]

const item = {

description: input.value,

}

addTodo(item)

input.value = ''

})

```

  

추가한 데이터가 있는 할 일 목록을 화면에 보여주어야 합니다. displayTodolist() 함수를 사용해서 화면에 할 일 목록을 표시하도록 합니다.

  

```jsx

function displayTodolist() {

const ul = document.querySelector('ul')

ul.innerHTML = ''

todolist.forEach(todo => {

const li = document.createElement('li')

li.innerText = todo.description

ul.appendChild(li)

})

}

```

  

해당 displayTodolist 함수를 옵저버 배열에 등록합니다. 코드가 실행되면 registerObserver 함수를 통해 함수를 옵저버 배열에 등록합니다.

  

```jsx

function registerObserver(observer) {

observers.push(observer)

}

  

registerObserver(displayTodolist)

```

  

자 이제 옵저버들에게 알려주는 함수를 만들어야 합니다. 아래 함수는 옵저버 안에 있는 함수들을 실행합니다.

  

```jsx

function notifyObservers() {

observers.forEach(observer => observer())

}

```

  

그다음 notifyObservers 함수를 todo가 추가되는 함수인에 넣으면 addTodo 함수가 실행될 때마다 todoList가 업데이트됩니다.

  

```jsx

function addTodo(item) {

todolist.push(item)

notifyObservers()

}

```

  

전체적인 코드는 이렇습니다.

  

```jsx

// 전체적인 코드

function TodoObject() {

let todolist = []

let observers = []

  

function addTodo(item) {

todolist.push(item)

}

  

function registerObserver(observer) {

observers.push(observer)

}

  

function notifyObservers() {

console.log(observers)

observers.forEach(observer => {

observer()

})

}

  

function addTodo(item) {

todolist.push(item)

notifyObservers() // Add this line

}

  

return {

addTodo,

registerObserver,

todolist,

}

}

  

const todoOdj = TodoObject()

  

const form = document.querySelector('form')

  

form.addEventListener('submit', event => {

event.preventDefault()

const input = form.elements[0]

const item = {

description: input.value,

}

todoOdj.addTodo(item)

input.value = ''

})

  

todoOdj.registerObserver(displayTodolist)

  

function displayTodolist() {

const ul = document.querySelector('ul')

ul.innerHTML = ''

todoOdj.todolist.forEach(todo => {

const li = document.createElement('li')

li.innerText = todo.description

ul.appendChild(li)

})

}

```

  

## 어떤 문제점이 있을까?

  

첫 번째로 알람 순서를 정할 수 없다는 것이다. 만약 옵저버가 바라보는 것들끼리 의존성이 존재한다면 당연히 문제가 생길 수밖에 없다. 이런 부분을 충분히 검토하고 사용해야 한다. 두 번째로 옵저버 체인으로 인해서 복잡해질 수 있다. 예를 들어 한 옵저버가 알림을 받고 또 자신이 주체가 되어서 다른 옵저버에게 알림을 계속 이어나가게 된다면 코드가 복잡해지고 디버깅 또한 힘들어진다. 이런 상황에 Mediator 패턴을 사용하면 도움이 될 수 있다.

세 번째로 메모리 누수 문제가 있다. 옵저버 객체를 계속 참조하고 있기 때문에 가비지 컬렉터가 안되는 현상이다. 자바스크립트 가비지 컬렉터는 하나라도 참조되는 게 있다면 수집 대상에서 제외한다. 그래서 사용하지 않는 것이 있다면 구독을 해제해 주는 방법으로 메모리 누수를 피할 수 있다.

  

### 마무리

  

옵저버 패턴을 사용하면 객체의 변경사항을 다른 객체에 전파할 수 있다. 옵저버 패턴은 프런트엔드 개발자라면 꼭 알고 있어야 하는 디자인 패턴 중 하나이다. 실제로 상태 관리 라이브러리 Redux 같은 경우도 옵저버 패턴을 사용하고 있다. 다음에는 옵저버 패턴을 사용한 상태 관리를 직접 구현해 보는 시간을 가질 예정입니다. 원리를 알고 있다면 라이브러리를 사용하는 것도 수월하지 않을까..?