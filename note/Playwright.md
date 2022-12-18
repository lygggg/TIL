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