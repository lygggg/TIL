

### setTimeout()
보통 어떤 코드를 바로 실행하지 않고 일정시간 기다린후 실행해야하는 경우 사용한다.

```tsx
setTimeout(() => console.log("2초 후에 실행됨"), 2000); // 2초 후에 실행됨
```

```tsx
function add(x, y) {
  console.log(x + y);
}
setTimeout(add, 2000, 3, 4); // 7

```

`setTimeOut()` 함수는 타임아웃 아이디를 반환한다. 타임아웃 아이디는 `setTimeout()` 함수를 호출할 때마다 내부적으로 생성되는 타이머 객체를 가리킨다. 이 값을 인자로 `clearTimeout()` 함수를 호출하면 기다렸다가 실행될 코드를 취소할 수 있다.

```tsx
const timeoutId = setTimeout(() => console.log("5초 후에 실행됨"), 5000); // 출력x
```

### 지연시간
아래 사진을 보면 `setTimeout`은 지연시간(100ms)을 보장한다. 그렇기 때문에 지연시간을 정확하게 설정할 수 있다.
![[Pasted image 20221120154837.png]]


### setInterval()
웹페이지의 특정 부분을 주기적으로 업데이트해줘야 하거나, 어떤 API로 부터 변경된 데이터를 주기적으로 받아와야 하는 경우에 사용한다.

```tsx
setInterval(() => console.log(new Date()), 2000);
```

```tsx
Sun Dec 12 2021 12:29:06 GMT-0500 (Eastern Standard Time)
Sun Dec 12 2021 12:29:08 GMT-0500 (Eastern Standard Time)
Sun Dec 12 2021 12:29:10 GMT-0500 (Eastern Standard Time)
Sun Dec 12 2021 12:29:12 GMT-0500 (Eastern Standard Time)
Sun Dec 12 2021 12:29:14 GMT-0500 (Eastern Standard Time)
Sun Dec 12 2021 12:29:16 GMT-0500 (Eastern Standard Time)
Sun Dec 12 2021 12:29:18 GMT-0500 (Eastern Standard Time)
```

`setInterval()` 함수도 인터벌 아이디를 반환한다. 인터벌 아이디는 함수를 호출할 때마다 내부적으로 생성되는 타이머 객체를 가리킨다. 이값을 인자로 `clearInterval(`함수를 호출하면 코드가 주기적으로 실행되는 것을 중단시킬 수 있다.
```tsx
> const intervalId = setInterval(() => console.log(new Date()), 2000);
< Sun Dec 12 2021 12:45:31 GMT-0500 (Eastern Standard Time)
< Sun Dec 12 2021 12:45:33 GMT-0500 (Eastern Standard Time)
< Sun Dec 12 2021 12:45:35 GMT-0500 (Eastern Standard Time)
> clearInterval(intervalId);
```
### 지연시간
`setInterval()`은 실제 지연시간이 설정한 것보다 적다. 이유는 함수가 실행에 걸리는 시간이 간격의 일부를 소비하기 때문에 그렇다고한다. 결론은 정상인것..
![[Pasted image 20221120154740.png]]

### 결론
- `setTimeout`시간 간격 후에 함수를 한 번 실행할 수 있다.
- `setInterval`시간 간격 이후에 시작하여 해당 간격으로 계속 반복하여 함수를 반복적으로 실행할 수 있다.
- mdn에서는 setTimeout을 사용하는게 setInertval을 사용하는 것보다 실행 사이의 지연을 보다 정확하게 설정할 수 있다고 한다.

### 출처
- https://www.daleseo.com/js-timer/
- https://medium.com/@monica1109/scheduling-settimeout-and-setinterval-ca2ee50cd99f
- https://javascript.info/settimeout-setinterval