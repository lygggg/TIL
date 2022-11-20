ing 현재 정리중입니다

## Describe - Context - It 패턴
이 패턴은 코드의 행동을 설명하는 테스트 코드를 작성하는 패턴이다(BDD).

다른 BDD 패턴인 Given - When - Then과 비슷한 철학을 갖고 있지만 미묘하게 다른 점이 있다.

Describe -Context - It은 상황을 설명하기 보다는 테스트 대상을 주인공 삼아 행동을 더 섬세하게 설명하는데 적합하다.


|키워드|설명|
|------|---|---|
|Describe|설명할 테스트 대상을 명시한다.|
|Context|테스트 대상이 놓인 상황을 설명한다.|
|It|테스트 대상의 행동을 설명한다.|

- 영어로 Context 문을 작성할 때에는 반드시 with 또는 when으로 시작하도록 한다.
- It구문은 It return true, It responses 404와 같이 심플하게 설명할수록 좋다.

#### Describe-Context-It 플러그인 설치
```tsx
yarn add -D jest-plugin-context @types/jest-plugin-context
```

#### React Testing Library 설치
```tsx
yarn add -D @testing-library/react @testing-library/user-event @testing-library/dom @testing-library/jest-dom
```


#### 설정 파일
```tsx
jest --init
```

#### jest.config.js
```tsx
const customJestConfig = {
	setupFilesAfterEnv: ["<rootDir>/jest.setup.js"],
}
```

#### jest.setup.js
```js
import "@testing-library/jest-dom/extend-expect";
import "jest-plugin-context/setup";
```


### Example
```tsx
context("isActive가 true면", () => {
	it("portal에 modal이 존재해야 한다.", () => {
	const { getByTestId } = rederModal({ children: "modal", isActive: true });
	const modal = getByTestId("modal");
	expect(modal).toBeInTheDocument();
	});
});

  

context("isActive가 false면", () => {
	it("portal에 modal이 존재하지 않아야 한다.", async () => {
	rederModal({
	children: "modal",
	isActive: false,
	});

const modal = await screen.queryByText("modal");
expect(modal).not.toBeInTheDocument();
	});
});
```

### 용어정리
getBy 메서드는 요소를 찾을수 없을때 오류를 발생시킨다. Dom에 존재하지 않으면 queryBy를 대신 사용할수있다.

`.toBeInTheDocument()`는 요소가 본문에 있는지 여부를 확인하는 matcher