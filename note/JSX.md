JSX는 JSX는 바벨과 같은 트랜스파일러를 통해 JSX코드를 브라우저나 JavaScript 엔진에서 이해할 수 있는 일반 JavaScript코드로 변환하는 프로세스입니다. React에서는 JSX 코드를 `React.createElement` 메서드로 변환합니다.

첫번째 인자로는 태그 이름이 들어가고, 두번째 인자로는 클래스명과 같은 프로퍼티명, 세번째 인자에는 자식 태그가 있다면 `React.createElement`를 다시 호출하고, 없다면 해당 문자열이 들어가거나 아무것도 들어가지 않습니다.

### 예시 코드

```ts
// 변환 전
function hello() {
  return
  <div className="wrapper" property property2={false}>
  	<span className="span1">1</span>
    <span className="span1">2</span>
    <span className="span1"></span>
  </div>;
}
// 변환 후
function hello() {
  return;

  /*#__PURE__*/
  React.createElement("div", {
    className: "wrapper",
    property: true,
    property2: false
  }, /*#__PURE__*/React.createElement("span", {
    className: "span1"
  }, "1"), /*#__PURE__*/React.createElement("span", {
    className: "span1"
  }, "2"), /*#__PURE__*/React.createElement("span", {
    className: "span1"
  }));
}
```
