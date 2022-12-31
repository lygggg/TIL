React testing library의 9가지 팁과 요령

### React Testing Library는 무엇인가?
Kent C. Dodds는 한가지 문제를 해결하기 위해 React Testing Library를 만들었다. 다른 테스트 라이브러리는 개발자가 테스트 중인 구성 요소의 구현 세부 정보와 너무 밀접하게 연결된 테스트를 작동하도록 권장했다. Dodds는 테스트가 구성 요소의 최종 결과(동작)만 확인해야 한다고 말했다.

사용자 관점에서(구성 요소가 DOM에서 어떻게 보이고 작동하는지)에 초점을 맞춘 테스트가 궁극적으로 중요하다. 

Redux 스토어에 무언가를 저장하거나 데이터를 가져오기 위해 특정 API를 호출하는지 여부는 신경쓰지 않는다. 특히 사용자는 이를 모르고, 신경을 쓰지도 않는다. 화면에 자신이 클릭한 정보가 표시되는지 또는 주문한 항목이 배송되는지에 관심이 있다. 또한 개발자로서 우리도 그것에 관심을 가져야한다.

이것을 위해 Dodds는 다음과 같은 방식으로 테스트에 접근할 것을 권장한다.

- 구성 요소의 구현 세부 정보에 대한 종속성을 피해라
- 테스트를 유지 관리 하기 쉽게 만들어라(즉, 구성 요소 구현을 리팩토링 해도 테스트가 중단되지 않음)
- 구성 요소가 통합되어 사용자 문제를 해결하는지 테스트 하려면 shallow mounting를 피해야한다.

다음은 React Testing Library의 재미있는 몇가지를 소개하겠다.
- 렌더링된 React 구성 요소의 인스턴스를 확인하는 대신 실제 DOM 노드를 확인하는 데 중점을 둔다.
- 사용자와 동일한 방식으로 DOM을 쿼리하기 위한 유틸리티를 제공한다.
- 레이블 텍스트를 요소로 찾는다.
- 사용자가 원하는 것처럼 텍스트에서 링크와 버튼을 찾는다.
- `data-testid` 텍스트 콘텐츠 및 레이블이 의미가 없거나 실용적이지 않거나 사용할 수 없는 요소에 대한 탈출구로, 요소를 찾을 수 있는 방법을 제공한다.

### #팁1 구성 요소 대신에 screen 사용
```ts
import React from 'react';

import { screen } from '@testing-library/dom';

import { render } from '@testing-library/react';

import { act } from 'react-dom/test-utils';

import App from './App';

describe('Product Browser App', () => {

it('renders a welcome message', () => {

act(() => {

render(<App />);

});

screen.findByText('Hello world');

});

});
```

### #팁2 waitForElement 대신 FindByText...getByText

```ts
const orderButton = await waitForElement(() => container.getByTestId('order-submit-button'));
```

```ts
const orderButton = await container.findByTestId('order-submit-button');
```
`findByText`, `findByTestId`, `findByPlaceholder`등 을 사용하면 검색할 요소가 렌더링 되기를 기다리는 것이 자동으로 래핑되므로 더 깔끔하다.

### #팁3 비동기 작업을 위한 UserEvent
```ts
await fireEvent.click(orderButton);
```

```ts
import userEvent from '@testing-library/user-event';

userEvent.click(orderButton);
```
굳이 `fireEvent()`를 수동으로 래핑할 필요가 없다. user-event를 사용해라

### #팁4 debug() 대신 PrettyDOM을 사용해라

```ts
console.log(component.debug());
```

```ts
console.log(prettyDOM(component.container, 99999));
```

debug를 사용하면 매우 큰 dom을 출력할 경우 텍스트가 잘릴경우가 있다. 그렇기 때무에 PrettyDoM을 사용해라


### #팁5 구성 요소의 class를 확인하는 Document.querySelector
만약 Ant Design과 같은 라이브러리를 사용하고 있다면 RTL에서 사용하기가 힘들 수 도 있다.
```ts
expect(document.querySelector('.hidden')).toBeInTheDocument();
```

### #팁6 Target 위의 요소에 접근하기 위한 ParentElement
```ts
const input = component.queryByPlaceholderText(componentText);
const enter = input.parentElement.querySelector('.anticon-plus');
```

### #팁7 Target 위의 요소에 접근하기 위한 ParentElement
```ts
const keyToSelect = await component.findByText('key to select');

userEvent.click(keyToSelect.previousSibling.firstChild.firstChild);
```

### #팁8 DOM에서 Fuzzy Text Matches에 { exact: false } 사용
경우에 따라 RTL에서 `getByText` 또는 `findByText` 유형 검색 중 하나를 사용한느 것이 정확한 일치로 기본 설정되어 있기 떄문에 사용하기 어렵다. 즉 `getByText('Hi there')`, 문자열이 DOM에서 정확히 동일한 경우에만 작동한다. 전체 문자열, 대소문자 구분 개행(`\n`) 또는 `"Hi"`와 `"there"` 사이에 다른것이 있으면 getByText() 쿼리는 텍스트를 찾지 못한다.

대소문자를 구분하지 않고 하위 문자열을 포함하는 Fuzzy Text Matches를 수행하려면 `{ exact: false }`를 사용해야한다.

```ts
<div>Hi there!</div>

getByText(container, 'i the', { exact: false });

getByText(container, 'hi there', { exact: false });
```
하위 문자열을 일치시켜야 하는 경우 정규식을 사용하는것을 더 권장하지만 DOM에서 간단한 텍스트 선택을 위해 이것을 사용하는 것은 훌륭한 대안이라고 생각한다.

### #팁9 명령줄에서 브라우저의 모든 파일 열기
선택한 JavaScript IDE로 VS Code를 사용 중이고 VS Code 내장 터미널 명령줄에서 브라우저의 파일을 열려면 간단하다.

명령줄에서 `open <file path>`를 입력하면 된다. 브라우저에서 새 창이 열린다.

예를 들어 프로젝트의 코드 커버리지를 열려면 다음과 같이 작성한다.

```ts
Coverage/lcov-report/index.html 열기
```