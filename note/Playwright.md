하나의 api몰 모든 최신 브라우저(크롬, 파이어 폭스, 웹킷)에서 빠르고, 안정적인 자동화를 지원하는 MS에서 만든 자동화 도구다. 안타깝게도 레거시 Edge나 IE11은 지원하지 않지만 다음과 같은 다양한 장점이 있다.

- 여러 페이지, 도메인, iframe에 걸친 시나리오
- 클릭 같은 액션을 실행하기 전에 엘리먼트가 준비될때 까지 자동으로 기다림
- 네트워크 요청을 모킹하기 위한 네트워크 활동 가로채기
- 마우스, 키보드의 기본 입력 이벤트

## 주요 개념

### 브라우저(Browser)
브라우저(크롬, 파이어 폭스, 웹킷)의 인스턴스를 나타낸다. `Playwright`는 브라우저 인스턴스를 시작하고 브라우저 인스턴스를 닫는 것으로 끝난다.f
```tsx
const { chromium } = require('playwright');

const browser = await chromium.launch();

await browser.close();
```

브라우저 인스턴스를 만드는데 비용이 많이 들기 떄문에 `Playwright`는 단일 인스턴스가 여러 브라우저 컨텍스트를 통해 수행할 수 있는 작업을 극대화하도록 설계되었다.

### 컨텍스트(Browser context)
브라우저 컨텍스트는 브라우저 인스턴스 내에서 분리된 유사 세션이다. 만드는데 빠르고 비용이 적다. 각 테스트를 새로운 브라우저 컨텍스트에서 실행하여 브라우저 상태를 테스트 간에 분리하는 것 이 좋다.

```tsx
const browser = await chromium.lanch();
const conext = await browser.newContext();
```

컨텍스트별로 다른 기기를 시뮬레이션 할 수 있다. 예제에서는 아이폰과 아이패드에 대한 컨텍스트를 생성하고 있다.
```tsx
import { devices } from "playwright";

const iPhone = devices["iPhone i1 Pro"];
```

컨텍스트별로 다른 기기를 시뮬레이션 할 수 있다. 예제에서는 아이폰과 아이패드에 대한 컨텍스트를 생성하고 있다.
```js
import { devices } from "playwright";

const iPhone = devices['iPhone 11 Pro'];
const iPad = devices['iPad Pro 11'];

const iPhoneContext = await browser.newContext({
  ...iPhone,
  permissions: ['geolocation'],
  geolocation: { latitude: 52.52, longitude: 13.39 },
  colorScheme: 'dark',
  locale: 'de-DE'
});
const iPadContext = await browser.newContext({
  ...iPad,
  permissions: ['geolocation'],
  geolocation: { latitude: 37.4, longitude: 127.1 },
  // ...
});
```

### 페이지와 프레임
컨텍스트는 컨텍스트 내의 단일 탭 또는 팝업 창을 나타내는 페이지를 가질 수 있다. URL로 이동해서 페이지의 콘텐츠와 상호작용할 때 사용하면 된다.

페이지를 이용해서 각기 다른 탭에서 작업하는 것처럼 할 수 있다. 예제에서는 각기 다른 페이지를 탐색한다.

```js
// ...
// 페이지1 생성
const page1 = await context.newPage();

// 브라우저에 URL을 입력하는 것처럼 탐색
await page1.goto('http://example.com');
// 인풋 채우기
await page1.fill('#search', 'query');

// 페이지2 생성
const page2 = await context.newPage();

await page2.goto('https://www.nhn.com');
// 링크를 클릭하여 탐색
await page.click('.nav_locale');
// 새로운 url 출력
console.log(page.url());
```

페이지는 다시 1개 이상의 프레임 객체를 가질 수 있다. 각 페이지에는 메인 프레임이 있고 페이지 레벨 상호작용이 메인 프렝미에서 작동하는 것으로 가정한다. `iframe` 태그와 함께 추가 프레임을 가질 수 있다.

프레임을 이용해서 프레임 내부에 있는 엘리먼트를 가져오거나 조작할 수 있다. 예제에서는 프레임을 가져오는 방법들과 프레임 내부의 엘리먼트와 상호작용 하는 방법을 보여준다.

```js
// 프레임의 name 속성으로 프레임 가져오기
const frame = page.frame('frame-login');

// 프레임의 URL로 가져오기
const frame = page.frame({ url: /.*domain.*/ });

// 선택자로 프레임 가져오기
const frameElementHandle = await page.$('.frame-class');
const frame = await frameElementHandle.contentFrame();

// 프레임과 상호작용
await frame.fill('#username-input', 'John');
```

### 선택자
CSS 선택자, XPath 선택자, `id`나 `data-test-id`같은 HTML 속성을 이용해 엘리먼트를 검색할 수 있다.

사용하기 쉬운 CSS 선택자는 물론이고 HTML 속성을 통해서도 엘리멘트를 찾을 수 있다는 장점이 있다. 예제에서는 엘리먼트를 검색하는 다양한 선택자들을 보여준다.

```js
// data-test-id 선택자 사용
await page.click('data-test-id=foo');

// CSS, XPath 선택자가 자동으로 탐지된다.
await page.click('div');
await.page.click('//html/body/div');

// 부분 텍스트로 노드 검색
await page.click('text=Hello w');

// 명시적인 CSS, XPath 표기법
await page.click('css=div');
await page.click('xpath=//html/body/div');

// 쉐도우 DOM이 아닌 light DOM만 검색
await page.click('css:light=div');
```

`>>`구분자를 이용해서 선택자들을 체이닝해서 사용할 수도 있다. 선택자로 찾아온 엘리먼트에서 다시 검색하지 않아도 `>>` 구분자를 통해서 선택자를 묶어서 사용 가능하다는 장점이 있다.

예를 들어, `css=article >> css=.bar > .baz >> css=span[attr=value]` 선택자는 `document.querySelector`를 체이닝해서 사용한 것과 같다.

```js
// 동일한 엘리먼트를 가져오는 2가지 방법
await page.$('css=article >> css=.bar > .baz >> css=span[attr=value]');

document
  .querySelector('article')
  .querySelector('.bar > .baz')
  .querySelector('span[attr=value]')
```
실제로 다음과 같이 사용할 수 있다.
```js
// #free-month-promo 안의 'Sign Up' 텍스트를 가진 엘리먼트 클릭
await page.click('#free-month-promo >> text=Sign Up');
```

### 자동 대기
작업들은 엘리먼트가 나타나고 실행 가능해질 때까지 자동으로 기다린다. 예를 들어, 클릭을 하면
- 주어진 선택자가 DOM에 나타날 때 까지 기다린다.
- 나타날(visible) 때까지 기다린다.
- 애니메이션을 멈출 때까지 기다린다. (예를 들어 css 트랜잭션이 끝날 떄 까지)

위와 같이 엘리먼트가 나타날 때까지 자동으로 기다린다.

이 기능 덕분에 엘리먼트를 가져오기 위해 엘리먼트가 화면에 나타날 때까지 대기하는 코드가 필요하지 않고 그냥 엘리먼트를 가져오거나 클릭을 하는 등의 동작을 수행하면 된다.
```js
// DOM에 #search 엘리먼트가 나타날 때까지 기다린다.
await page.fill('#search', 'query');

// 에니메이션이 멈출 때까지 기다린 후 클릭한다.
await page.click('#search');
```
```js
// #search 엘리먼트가 DOM에 나타날 때까지 기다린다.
await page.waitForSelector('#search', { state: 'attached' });

// #promo가 보일 때까지 기다린다. (예를 들어 visibility: visible)
await page.waitForSelector('#promo');
```
```js
// #details가 보이지 않을 때까지 기다린다. (예를 들어 display: none)
await page.waitForSelector('#details', { state: 'hidden' });

// #promo가 DOM에서 제거 될 때까지 기다린다.
await page.waitForSelector('#promo', { state: 'detached' });
```