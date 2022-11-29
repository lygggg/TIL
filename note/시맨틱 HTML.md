`시맨틱`은 "의미와 관련된"을 의미한다. 시맨틱 HTML 작성은 HTML 요소를 사용하여 모양이 아닌 각 요소의 의미를 기반으로 콘텐츠를 구조화하는 것을 의미한다.

 
보통 시맨틱 값이 없는 요소인 `<div>`와 `<span>`을 많이 사용한다.
```tsx
<div>  
<span>Three words</span>  
<div>  
<a>one word</a>  
<a>one word</a>  
<a>one word</a>  
<a>one word</a>  
</div>  
</div>  
<div>  
<div>  
<div>five words</div>  
</div>  
<div>  
<div>three words</div>  
<div>forty-six words</div>  
<div>forty-four words</div>  
</div>  
<div>  
<div>seven words</h2>  
<div>sixty-eight words</div>  
<div>forty-four words</div>  
</div>  
</div>  
<div>  
<span>five words</span>  
</div>
```

아래는 시맨틱 태그로 작성된 코드다.
```tsx
<header>  
<h1>Three words</h1>  
<nav>  
<a>one word</a>  
<a>one word</a>  
<a>one word</a>  
<a>one word</a>  
</nav>  
</header>  
<main>  
<header>  
<h1>five words</h1>  
</header>  
<section>  
<h2>three words</h2>  
<p>forty-six words</p>  
<p>forty-four words</p>  
</section>  
<section>  
<h2>seven words</h2>  
<p>sixty-eight words</p>  
<p>forty-four words</p>  
</section>  
</main>  
<footer>  
<p>five words</p>  
</footer>
```

시맨틱 태그를 사용함으로써 가져오는 장점은 코드 블록에서 요소가 의미와 구조를 제공하기 때문에 내용을 이해하지 않고도 아키텍처를 이해할 수있다. 시맨틱 마크업은 개발자가 마크업을 읽기 쉽게 만드는 것만이 아니다. 대부분 자동화 도구가 해독하기 쉬운 마크업을 만드는 것이다. 개발자 도구는 시맨틱 요소가 기계가 읽을 수 있는 구조를 제공하는 방법을 보여준다.

## 접근성 개체 모델(AOM)
브라우저는 받은 콘텐츠를 구문 분석할 때 문서 개체 모델(DOM)과 CSS 개체 모델(CSSOM)을 빌드한다. 그런 다음 접근성 트리도 구축한다. 스크린 리더와 같은 보조 장치는 AOM을 사용하여 콘텐츠를 구분 분석하고 해석한다.

시맨틱 요소를 사용하면 접근성이 높아지고 비의미적 요소를 사용하면 접근성이 떨어진다. 우리는 개발자로서 HTML의 기본 액세스 가능 특성을 보호하고 액세스 가능성을 최대화 해야한다.[AOM 검사 기능](https://developer.chrome.com/docs/devtools/accessibility/reference/#explore-tree)

## 결론
- 시맨틱 태그를 사용하면 요소의 의미와 구조를 제공하기 떄문에 내용을 이해하지 않고도 아키텍처를 이해할 수 있다.
- 코드는 자동적으로 기계가 읽는다. 우리는 기계가 더 읽기 쉽도록 더 쉬운 마크업을 만들어야한다.  시맨틱 요소는 기계가 쉽게 읽을 수 있는 구조를 제공하는 방법을 보여준다.
- 시맨틱 요소를 사용하면 접근성이 높아지고 비의미적 요소를 사용하면 접근성이 떨어진다.
- 스크린 리더 같은  보조 장치는 접근성 개체 모델을 사용해 콘텐츠를 분석하고 해석한다.

## 출처
- https://web.dev/learn/accessibility/aria-html/
- https://web.dev/learn/html/semantic-html/