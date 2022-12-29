
### 렌더 이슈 해결 가능
- Cypress는 Automatic waiting을 지원하기 떄문에 코드 작성에 비동기 처리가 거의 필요 없다.
- 즉, 자동으로 같은 동작을 성공할 떄까지 여러 번 반복해 결과를 얻는 방식으로 처리를 하고 있는데, 이렇게 되면 기존에 간헐적으로 렌더가 되지 않았던 문제를 해결할 수 있어 테스트 결과가 간헐적으로 실패하는 일이 줄어들고, 테스트 결과를 더 신뢰할 수 있게 된다.

### 빠른 실행 시간
Cypress에서도 Protractor와 같이 Headless 모드를 지원한다. Headless 모드는 직접 브라우저를 띄우지 않고 Node 엔진 위에서만 실행하여 실행 시간 단축이 가능합니다.

### Commands 기능
- 첫 번째는 Commands 기능으로, 마치 메소드처럼 테스트 코드를 묶어서 재사용할 수 있게 해주는 기능이다.
- Commands로 선언하면 다른 Cypress에서 제공하는 메소드들과 같이 따로 import하지 않아도 cy 객체로부터 호출하여 사용할 수 있다.

```javascript
Cypress.Commands.add('logoutAndLogin', (account) => {
  cy.logout();
  cy.visit('/');
  cy.login(account);
});

// 사용
cy.logoutAndLogin(account);
```

### Intercept 기능
- Intercept 기능은 서비스에서 호출된 API를 중간에서 조작할 수 있는 기능입니다. 카카오에서는 특정 API 호출 시 응답이 올떄까지 대기했다가 테스트를 재개하거나, 응답을 Mocking 하는데 주로 사용했다고한다.
- Intercept API 사용으로 임의의 시간 동안 대기하는 코드를 거의 없앨 수 있었고, Mocking도 간단하게 할 수 있었다고 한다.

```ts
it("API 응답을 받을 때까지 기다렸다가 진행하기", () => {
cy.intercept("GET", `${apiUrl}/hello`).as('hello');
cy.get(button).click();
cy.wait('@hello');
})
```

## 무엇을 테스트하는지를 명확하게 하기
- 프론트엔드 E2E 테스트에는 기능적 테스트와 시각적 테스트 두 종류가 있다.
- 기능적 테스트는 이미지 비교를 통해 테스트 성공 여부를 결정하는 Visual Regression 방식 사용
- 시각적 테스트는 DOM 트리 탐색을 통해 화면을 조작하고 검증
- Visual Regression 방식의 테스팅은 기대했던 이미지와 실행 결과를 캡처한 이미지를 비교하여 픽셀의 어떤 부분이 달라졌는지를 검증한다. 카카오에서는는 기능적 테스트를 지향하기로 했다고 한다. 이유는 시각적 테스트를 하게되면 마크업 변경이 있을 때마다 테스트 코드가 변경되어야 하는데, 테스트 코드가 너무 자주 수정되면 유지 보수 비용이 증가할 것 같다고 생각했다고 하는데... 음...
- 기존의 테스트에서 코드가 중복되는 것을 방지하고, HTML 구조 변경과 테스트 코드 변경을 분리하기 위해 Selector 들을 하나의 객체로 모아 관리하는 Page Obect를 사용하고있다. Page Object를 사용하면 Element를 가져오는 코드가 중복되는 것을 방지하고, Selector를 테스트 코드와 분리할 수 있다는 장점이 있다고 한다.

```javascript
class PageObject {
  getConfirmButton = elementByCss('.submit_form > div.confirm_button > button');
  getCancelButton = elementByCss('.submit_form > div.cancel_button > button');
  getTitle = elementByCss('.title');
}
```
위의 코드와 같은 동작을 HTML Tag에 Data Attribute를 추가하는 방식으로 변경한 것이 아래의 코드다. 위 코드에서는 상위 DOM 구조가 변경되면 해당 Element 탐색까지의 경로가 변경되며 코드 변경이 필요한데, 아래에서는 Element의 속성만 유지되면 되기 떄문이다.
```javascript
{
  confirmButton: '[data-cy="confirm-button"]',
  cancelButton: '[data-cy="cancel-button"]',
  title: '[data-cy="title"]'
}
```
대신 Production 용으로 빌드된 코드에 테스트 관련 코드가 들어가면 개발에만 사용하는 속성을 외부 사용자도 Inspector나 빌드 된 파일에서 볼 수 있고, 불필요한 코드가 포함되면 빌드 파일의 용량도 늘어나기 떄문에 Webpack 플러그인을 추가하여 빌드 단계에서 Attribute를 제거했다고한다.

### 테스트 잘 작성하기
- 좋은 테스트는 정해진 형식을 가지고 있다.
- 준비 Context : 검증할 로직이 실행될 환경에 해당하는 테스트의 콘텍스트를 마련한다.
- 실행 Execute : 검증할 로직을 실행한다.
- 검증 Verify : 예상할 수 있는 가시적인 효과를 검사한다.
- 정리 Teardown : 다른 테스트에 손상을 입힐 수 있는 잔종 상태를 정리한다.
- 테스트의 핵심 부분은 검증이므로, 적극적으로 부수 기능을 다루는 객체를 만들고 준비와 정리 부분을 테스트 코드와 분리하자.
- 검증의 단점과 예상 구문은 대상 코드의 행위에서 중요한 바를 정확히 전달해야 한다.
- 테스트가 너무 많은 세부 사항을 단정하는 코드는 읽기 어렵고 상황이 바뀌었을 때 불안정해지므로, 테스트와 무관한 나머지 세부사항은 무시한다.
- 내가 작성한 테스트가 신뢸르 주는지 항상 의심하자.
- 테스트는 비용이다. 불필요한 테스트를 최소화하자.
- 시각적 테스트와 기능적 테스트를 분리하자.

https://tech.kakao.com/2022/02/25/angular-e2e-testing-2/