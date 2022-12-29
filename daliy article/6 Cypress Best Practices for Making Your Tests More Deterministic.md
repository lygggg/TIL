좋은 E2E 테스트를 작성하는 것은 생각보다 어렵다. 올바른 구성, 테스트 도구 모음을 사용하는 올바른 방법, 테스트할 항목과 하지 말아야할 항목, 테스트가 필요한 항목을 테스트하는 방법을 결정해야 한다. 따라서 Cypress를 최대한 활용하기 위해 테스트를 작성할 때 따를 수 있는 6가지 모범 사례를 소개한다. 실제로 Cypress 뿐만 아니라 모든 테스트 프레임워크에 아래 방법을 적용할 수 있다.

## 1. Use data Attributes When Selecting Elements(요소 선택 시 데이터 속성 사용)
E2E 테스트를 생성할 떄 따를 수 있는 가장 중요한 모범 사례중 하나는 CSS 또는 JavaScript와 완전히 분리된 선택기를 작성하는 것이다. css, JavaScript 업데이트가 테스트를 손상시키지 않도록 해야한다. 여기서 가장 좋은 옵션은 사용자 정의 `data` 속성을 사용하는 것이다.

```js
// ✅ Do
cy.get('[data-cy="link"]');
cy.get('[data-test-id="link"]');

// ❌ Don't
cy.get('button');          // Too generic
cy.get('.button');         // Coupled with CSS
cy.get('#button');         // Coupled with JS
cy.get('[type="submit"]'); // Coupled with HTML
```

이것들은 목적을 명확하게 설명하며, 그것들을 변경하면 테스트 케이스도 업데이트해야 한다는 것을 알게 된다.



## 2. Set a Base Url(기본 URL 설정)
기본 URL 을 전역적으로 설정하는 것도 좋은 방법이다. 테스트를 더 깔끔하게 만들 수 있을 뿐만 아니라 localhost 및 프로덕션 사이트와 같은 서로 다른 환경 간에 테스트를 쉽게 전환할 수 있다.

```js
// ✅ Do set a base URL in your cypress.json config
cy.visit('webtips/cypress');

// ❌ Don't
cy.visit('https://webtips.dev/webtips/cypress');
cy.visit('http://localhost/webtips/cypress');
```

Cypress의 경우 성능상의 이점도 있다. 기본적으로 전역 기본 URL을 정의하지 않으면 Cypress는 `cy.visit`명령을 만나면 최종 위치로 전환하기 전에 로컬호스트에서 로드를 시도한다.

## 3. Avoid Using cy.wait with a Number(cy.wait를 숫자와 함께 사용하지 마세요.)
Cypress의 일반적임 함정은 `cy.wait` 고정된 숫자로 명령을 사용하는 것이다. 진행하기 전에 요소가 나타나거나 네트워크 요청이 완료되기를 기다리기 떄문에 이런일이 발생할 수 있다. 무작위 실패를 방지하기 위해 `cy.wait`  임의의 숫자를 도입하여 명령 실행이 완료되었는지 확인한다.

이것의 문제는 결국 필요 이상으로 기다리게 된다는 것이다. 5000밀리초  `cy.wait`를 사용하여 요청이 500밀리초 안에 완료되면 아무 이유 없이 테스트 도구 모음의 실행 시간을 4500밀리초 늘린 것이다.

```js
// ✅ Do
cy.intercept('POST', '/login').as('login');
cy.wait('@login'); // Waiting for the request explicitly

// ❌ Don't
cy.wait(5000);
```
`cy.wait` 를 사용하여 기다리고 있는 조건이 충족되었는지 확인하여 안전하게 진행할 수 있다. `cy.wait`를 계속 진행하기 전에 특정 조건이 충족되었는지 확인하기 위해 `cy.wait` 대신 인수를 사용하여 계속하기전에 특정 조건이 충족되는지 확인할 수 있다.

## 4. Tests Should be Able to Pass Independently(테스트는 독립적으로 통과할 수 있어야 한다.)
당신이 할 수 있는 또 다른 일반적인 실수는 서로 결합되고 의존하는 테스트를 만드는 것이다. 이전 테스트의 상태에 의존하면 초기 조건이 충족되지 않으면 테스트 사례의 나머지 부분을 깨드릴 수 있는 불안정한 테스트가 된다.

```js
// ❌ Don't
it('Should log the user in', () => { ... });
it('Should be able to change settings', () => {
    cy.get('[data-cy="email"]').type('email@updated.com');
});

it('Should show updated settings', () => {
    cy.contains('[data-cy="profile"]', 'email@updated.com');
});
```
위의 코드 예제에서 각 테스트는 이전 테스트에 의존하므로 하나가 실패하면 다른 테스트도 마찬가지다. 첫 번째 항목을 변경하면 나머지 항목을 업데이트 해야 할 수도 있다. 테스트를 분리하고 서로 의존하는 여러 단계를 하나로 결합하거나 재사용할 수 있는 공유 코드를 만들어야 한다.

## 5. Control State Programmatically(프로그래밍 방식으로 상태 제어)
올바른 조건에서 테스트할 수 있ㄷ록 애플리케이션의 상태를 설정해야 할 떄마다 항상 UI를 사용하는 대신 프로그래밍 방식으로 상태를 설정하려고 한다. 이는 상태가 UI에서 분리됨을 의미한다. 또한 상태를 프로그래밍 방식으로 설정하는 것이 애플리케이션의 UI를 사용하는 것보다 빠르기 떄문에 성능이 향상 되는 것을 볼 수 있다.

```js
// ✅ Do
cy.request('POST', '/login', {
    email: 'test@email.com',
    pass: 'testPass'
});

// ❌ Don't
cy.get('[data-cy="email"]').type('test@email.com');
cy.get('[data-cy="pass"]').type('test@email.com');
cy.get('[data-cy="submit"]').click();
```

## 6. Avoid Single Assertions(단일 인수를 피해라)
마지막으로 단일 인수를 사용하지 마세요. 단일 인수는 단위 테스트에 적합할 수 있지만 여기서는 E2E 테스트를 작성한다. 인수를 다른 테스트 단계로 분리하지 않더라도 어떤 인수가 실패했는지 정확히 알 수 있다.

```js
// ✅ Do
it('Should have an external link pointing to the right domain', () => {
    cy.get('.link')
      .should('have.length', 1)
      .find('a') 
      .should('contain', 'webtips.dev');
      .and('have.attr', 'target', '_blank');
});

// ❌ Don't
it('Should have a link', () => {
    cy.get('.link')
      .should('have.length', 1)
      .find('a');
});

it('Should contain the right text', () => {
    cy.get('.link').find('a').should('contain', 'webtips.dev');
});

it('Should be external', () => {
    cy.get('.link').find('a').should('have.attr', 'target', '_blank');
});
```
가장 중요한 것은 Cypress가 상태를 재설정하는 테스트 간에 수명 주기 이벤트를 실행한다는 것이다. 이는 단일 테스트 인수를 추가하는 것보다 더 많은 계산이 필요하다. 따라서 단일 인수를 작성하면 테스트 의 성능에 부정적인 영향을 미칠 수 있다.

https://www.webtips.dev/webtips/cypress/cypress-best-practices