### 웹사이트 속도는 고객 획득에 영향을 준다
페이지가 2초이내에 로딩되면 9%의 이탈율을 보이지만, 5초를 넘어갔을 때는 38%를 넘어간다. 이처럼 웹사이드 로딩속도는 고객 획득에 영향을 준다.

자바스크립트는 같은 크기의 이미지나 웹폰트보다 브라우저가 처리하는 비용이 더 많이 든다. 그렇기 떄문에 자바스크립트 번들 최적화는 중요하다.

### 원인을 찾기위한 도구들
- Webpack Analyse, Webpack Visualizer, Webpack Bundle Analyzer 등의 다양한 도구들이 있다.
- Webpack Bundle Analyzer를 사용하면 용량별로 시각화를 해주기 떄문에 어떠한 라이브러리가 많이 사용되고 있는지 알 수 있다. 또한 사이드바를 이용해 검색과 필터가 가능하다.

### 라이브러리 중복 줄이기   
- npm에서는 tree 구조로 필요한 버전을 모두 받는 방식을 선택했다. 하지만 이방법은 과도하게 node_modules가 과도하게 커지는 문제가 있기 떄문에 이 문제를 해결하기 위해 npm은 라이브러리들이 시멘틱 semver을 지킨다고 가정한다. 하지만 이것도 모든 문제를 해결해줄수 있는 것은 아니기 때문에 dedupe 명령어를 사용해서 중복 모듈을 줄일수 있는 기능을 제공한다.
- yarn에서는 라이브러리 설치과정에서 완벽하게 처리해주기 때문에 신경쓰지 않아도 되지만, 완벽하지 않다. 그래서 만약 중복 라이브러리가 존재한다면 yarn-Deduplicate를 사용해서 중복을 제거하는 것을 추천한다.
- yarn 2에서 dedupe명령어를 제공한다고한다.
- lodash에서도 중복현상이 발생할 수 있다. 우리가 lodash를 사용하지 않더라도 우리가 사용하는 라이브러리들이 lodash를 사용할수있기 떄문이다. 이경우 Webpack의 alias기능을 사용하면 중복을 피할수있다.

우리가 사용하는 라이브러리들이 모두 tree-shaking을 지원하면 좋겠지만, 아닌 라이브러리들이 존재한다. 예를들면 lodash같은 경우가 있다. 그래서 우리는 사용되는 부분만 사용하도록 수정해야한다.
```tsx
import _ from "lodash"
import { add } from "lodash/fp"

const addOne = add(1)
_.map([1,2,3]. addOne)
```

```tsx
import _add from "lodash/fp/add"
import _map from "lodash/map"

const addOne = _add(1)
_map([1,2,3], addOne)
```

lodash 이외에 비슷한 문제가 있다면 babel-plugin-transform-imports를 사용하여 소스코드를 수정하지 않고 결과물을 최적화할 수 있다.

lodash의 일부 함수는 기능에 비해 용량이 클 수 있다. 예를들어 groupBy는 6kb에 가까운 용량을 차지한다. 왜냐하면 캐싱, shoatand 표현을 지원하기 때문이다.

Webpack은 node.js와 브라우저 환경을 통일하기 위해 polyfill을 추가할 수 있다.

라이브러리를 설치전 번들포비아를 사용해서 gzip으로 다운로드 했을때의 용량과 다운로드 속도, 각 버전별 용량을 쉽게 확인할 수 있다. 뿐만 아니라 라이브러리의 디펜던시를 미리 확인해볼 수 있다. 사용하려는 라이브러리가 용량이 조금 크더라도 이미 사용하고있는 라이브러리와 많이 겹친다면 용량 변화는 미미할테니 사용을 고려해 볼 수 있다.

추가로 비슷한 기능의 라이브러리들의 용량을 비교해주기도 한다. 특히 유용한 것은 export analysis로 tree-shaking이 되었을 때, 함수별로 얼마나 용량을 차지할지 미리 확인해볼 수 있다.

### tree-shaking
나무를 흔드는것처럼 불필요한 코드를 정적분석을 통해 필요한 코드만 가져오는 기술을 말한다.

라이브러리 제작자 관점에서 가장 중요한 것은 tree-shaking이다.

아래 코드는 사이드이펙트가 없으므로 tree-shaking이 가능하다
```tsx
function foo() {
	return `Hello World`;
}

export const a = foo();
export const b = bar();
```


아래 코드는 호출위치에 따라서 동작이 달라지는 경우 즉 사이드이펙트가 있으므로 tree-shaking이 불가능하다
```tsx
let count = 0;

function foo() {
	count++;
	return count;
}

export const a = foo();
export const b = bar();
```

번들러는 이런 사이드이펙트를 얼마나 잘 판단하는지에 따라서 tree-shaking을 잘 할 수도 못할수도 있다. 아쉽게도 webpack은 rollup에 비해 사이드이펙트를 판단을 어려워하는 편이다. 그렇기 때문에 webpack이 이해하기 쉬운 tree-shaking된 라이브러리를 만드는건 생각보다 까다로울수있다.

라이브러리를 만들때는 webpack에게 사이드이펙트 여부를 알려줘야한다. webpack은 package.json에 sideEffect란 필드를 참고해 어떤 파일에 사이드이펙트가 있는지 판단한다. 이때 사이드 이펙트가 없다면 사이드이펙트가 없는 라이브러리라고 명시해주어야한다. 하지만 사용자의 의도와 달리 사이드이펙트가 발생할수 있기 때문에 무조건 신뢰하지는 않는다. webpack은 사이드이펙트 필드가 있을때만 코드를 확인하고 tree-shaking을 시도한다. 이때 사이드이펙트를 발견하거나, 분석이 어려울때는 tree-shaking을 하지않는다.

![[Pasted image 20221113231743.png]]라이브러리를 만들때 rollup 번들러는 기본적으로 하나의 output을 만들어준다. 하지만 preserveModules란 옵션을 켜주면 오른쪽처럼 원본 소스코드와 유사한 구조의 output이 나온다. 왼쪽처럼 단일 output 파일의 경우 원본 파일의 일부에 tree-shaking이 불가능한 코드가 섞여 있다면, 최종 output은 한 파일이므로 파일 전체가  tree-shaking이 실패할 수도 있다. reserveModules옵션으로 빌드하면 이런 경우에도 일부파일만  tree-shaking이 실패하므로 이런 문제를 완화할 수 있다.

컴파일 툴을 잘 고르는 것도 중요하다. webpack에선 직접 dead 코드를 제거하기도 하지만 프로덕트 빌드에선 다른 라이브러리의(terser) 도움을 받는다. terser도 마찬가지로 사이트이펙트 유무를 판단하여 코드를 지운다. 이때 terser의 판단을 돕는 것이 pure annotation이다.
![[Pasted image 20221113231807.png]]
terser는 pure annotation 주석을 확인하면 그 코드는 사이드이펙트가 없다고 판단한다.

그런데 바벨은 pure annotation을 붙여주지만 아직 TypeScript Compiler는 붙여주지 않는다. 따라서 타입스크립트로 작성하더라도 가급적이면 바벨을 이용해 컴파일하는게 좋다. 특히나 react를 사용한다면 babel-plugin-transform-react-pure-annotaions 덕분에 pure annotation이 자동으로 삽입되므로 react-component를 라이브러리로 만들었다면 특히나 유용하다.


라이브러리가 의도와 다르게 사이드이펙트가 발생한다면 이를 찾는것은 까다롭다. 현재 가장 좋은 방법은 webpack의 stats 기능을 이용하는 것이고, webpack-cli를 쓴다면 stats옵션을 이용하고 nextjs를 이용한다면 아래와 같은 플러그인을 추가하면 웹팩이 번들링 도중에 분석한 다양한 정보를 json 포맷으로 저장할 수 있다.
![[Pasted image 20221113231845.png]]
그리고 json 파일을 앞서 설명한 webpack analyse 서비스에 올리면 모듈과 모듈의 관계를 분석할 수 있다.

Single Common Chunk를 사용하면 한두페이지에서만 라이브러리를 사용해도 모든  페이지에서 용량이 증가하는 문제가 생길 수 있다. 이런 문제를 해결하기 위해 Nextjs는 구글의 제안에 따라 다음과같이 청크를 나누고있다.
![[Pasted image 20221113231649.png]]
### dynamic import
webpack magic commnet를 이용하면 prefetch기능을 이용할 수 있다.