계획
배열을 하나 만들고
만약 배열의 길이가 1이상이고 popped의 맨앞의 값과 같다면 해당 배열 pop해준다.
만든 배열에 pushed 의 값들을 하나하나씩 push한다.
만약 만들었던 배열의 길이가 0이 아니면 false를 리턴
0이라면 true를 리턴한다.

```ts
function ValidateStackSequences(pushed, popped) {

const reversedPopped = popped.slice().reverse();

return validateSequences(pushed, reversedPopped);

}

  

function validateSequences(pushed, popped) {

const stack = [];

  

for (let i = 0; i < pushed.length; i++) {

stack.push(pushed[i]);

  

while (stack.length > 0 && stack[stack.length - 1] === popped[0]) {

stack.pop();

popped.shift();

}

}

  

return stack.length === 0;

}

  

test("ValidateStackSequences", () => {

expect(ValidateStackSequences([1, 2, 3, 4, 5], [4, 5, 3, 2, 1])).toEqual(

true

);

expect(ValidateStackSequences([1, 2, 3, 4, 5], [4, 3, 5, 1, 2])).toEqual(

false

);

});
```