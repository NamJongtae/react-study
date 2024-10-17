## Context 변경 사항

React 19에서는 기존의 `Context.Provider` 대신 `<Context>` 자체를 Provider로 사용하는 새로운 방식을 도입했습니다.

### 이전 방식

```jsx
import { createContext, useContext, useState } from "react";

// Context 생성
const ThemeContext = createContext();

// 부모 컴포넌트
const ParentComponent = () => {
  const [theme, setTheme] = useState("dark");
  const changeTheme = (theme) => {
    setTheme(theme);
  };

  return (
    // Context.Provider를 사용하여 값 전달
    <ThemeContext.Provider value={{ theme, changeTheme }}>
      <ChildComponent />
    </ThemeContext.Provider>
  );
};

// 자식 컴포넌트
const ChildComponent = () => {
  const { theme } = useContext(ThemeContext);
  return <div className={theme}>Hello world!</div>;
};

export default ParentComponent;
```

<br/>

### 변경된 방식

```jsx
import { createContext, useContext, useState } from "react";

// Context 생성
const ThemeContext = createContext();

// 부모 컴포넌트
const ParentComponent = () => {
  const [theme, setTheme] = useState("dark");
  const changeTheme = (theme) => {
    setTheme(theme);
  };

  return (
    // Context 자체를 사용하여 값 전달
    <ThemeContext value={{ theme, changeTheme }}>
      <ChildComponent />
    </ThemeContext>
  );
};

// 자식 컴포넌트
const ChildComponent = () => {
  const { theme } = useContext(ThemeContext);
  return <div className={theme}>Hello world!</div>;
};

export default ParentComponent;
```

<br/>

## ForwardRef 변경사항

React 19에서는 ref를 prop으로 직접 전달할 수 있게 되었습니다.

이제는 **forwardRef**를 사용할 필요가 없으며, ref를 prop으로 직접 전달하고 사용할 수 있습니다.

### 이전 방식

```jsx
import { useRef, forwardRef } from "react";

// forwardRef를 사용하여 ref를 전달받는 컴포넌트 생성
const MyComponent = forwardRef((props, ref) => {
  return <div ref={ref}>Hello</div>;
});

export default function App() {
  const myRef = useRef("MyRef");

  return <MyComponent ref={myRef} />;
}
```

<br/>

### 변경된 방식

```jsx
import { useRef } from "react";

// ref를 prop으로 직접 전달받는 컴포넌트 생성
const MyComponent = ({ ref }) => {
  return <div ref={ref}>Hello</div>;
};

export default function App() {
  const myRef = useRef("MyRef");

  return <MyComponent ref={myRef} />;
}
```

<br/>

## Ref의 Cleanup 함수

`ref` 콜백에서 Cleanup 함수를 반환하는 것을 지원합니다.

이를 통해 DOM 노드가 제거되거나 변경될 때 클린업 작업을 더 간편하게 처리할 수 있습니다.

이 기능을 활용하면, DOM 노드의 생성과 제거를 보다 효율적으로 관리할 수 있습니다.

<br/>

### 기존 Ref Cleanup 수행

기존에는 ref 콜백 함수에서 DOM 노드가 제거될 때 `null`이 인자로 전달되었습니다.

이를 통해 클린업 작업을 수행할 수 있었습니다.

```jsx
import React, { useState } from "react";

const MyComponent = () => {
  const refCallback = (node) => {
    if (node) {
      console.log("Node mounted:", node);
    } else {
      console.log("Node unmounted");
    }
  };

  return <div ref={refCallback}>Hello</div>;
};

export default MyComponent;
```

이 예제에서는 `refCallback` 함수가 DOM 노드가 마운트되거나 언마운트될 때 호출됩니다.

`node`가 `null`로 전달될 때는 DOM 노드가 제거되었음을 알 수 있으며, 이를 통해 클린업 작업을 수행할 수 있습니다.

<br/>

### Ref Callback과 리렌더링

React Element의 **ref는 컴포넌트가 렌더링된 이후 실행되는 함수로** 이 함수는 렌더링된 DOM node를 인자로 받고, 해당 컴포넌트가 언마운트되면 한번 더 호출되어 null을 인자로 받습니다.

이 메커니즘은 이전 콜백 함수가 **`null`**을 전달받아 역할을 종료하고, 새로운 콜백 함수가 현재 DOM 노드를 전달받는 방식입니다.

```jsx
import React, { useState } from "react";

const MyComponent = () => {
  const [count, setCount] = useState(0);

  const refCallback = (node: HTMLDivElement | null) => {
    console.log(count, node);
  };

  return (
    <div ref={refCallback}>
      <button onClick={() => setCount((c) => c + 1)}>Increment</button>
    </div>
  );
};

export default MyComponent;
```

이 경우, 버튼 클릭으로 인해 컴포넌트가 재렌더링되며, `refCallback` 함수도 호출됩니다.

이 메커니즘 덕분에 이전 콜백 함수는 `null`을 전달받아 역할을 종료하고, 새로운 콜백 함수는 현재 DOM 노드를 전달받습니다.

<br/>

**Ref Callback 메모이제이션**

```jsx
import React, { useState, useCallback } from "react";

const MyComponent = () => {
  const [count, setCount] = useState(0);

  const refCallback = useCallback((node) => {
    console.log(node);
  }, []); // 빈 배열로 메모이제이션

  return (
    <div ref={refCallback}>
      <button onClick={() => setCount((c) => c + 1)}>Increment</button>
    </div>
  );
};

export default MyComponent;
```

`refCallback`이 빈 의존 배열 `[]`로 메모이제이션되어, DOM 노드가 변경되거나 컴포넌트가 재렌더링되더라도 동일한 콜백 함수가 유지됩니다.

<br/>

### Ref Cleanup 함수 사용 예시

이 코드에서는 `useEffect`를 사용하여 DOM 노드가 나타날 때와 사라질 때 로그를 출력하려고 하지만, `useEffect`는 의존 배열이 비어 있는 상태에서 처음 렌더링 때만 실행되므로, 이후 상태 변화에 따른 DOM 노드의 생성/제거 시 제대로 작동하지 않습니다.

```jsx
import React, { useState, useEffect, useRef } from "react";

const MyComponent = () => {
  const [isVisible, setIsVisible] = useState(true);
  const divRef = useRef(null);

  useEffect(() => {
    if (divRef.current) {
      console.log("DOM 노드가 생성되었습니다:", divRef.current);

      // 클린업 함수
      return () => {
        console.log("DOM 노드가 제거되었습니다:", divRef.current);
      };
    }
  }, []); // 의존성 배열이 비어 있어 처음에만 실행됨

  return (
    <div>
      {isVisible && <div ref={divRef}>안녕하세요!</div>}
      <button onClick={() => setIsVisible((prev) => !prev)}>
        {isVisible ? "숨기기" : "보이기"}
      </button>
    </div>
  );
};

export default MyComponent;
```

- 이 코드에서 `useEffect`는 컴포넌트가 처음 렌더링될 때만 실행되고, `div`가 제거되거나 다시 나타날 때는 `useEffect`가 다시 실행되지 않기 때문에 클린업이 정상적으로 동작하지 않습니다.
- 따라서 버튼을 클릭해 `div`를 숨기거나 다시 보여줄 때 로그가 출력되지 않습니다.

<br/>

**Ref Cleanup 함수를 사용하여 해결**

아래 코드는 `useEffect` 대신 **React 19의 `ref` 콜백 클린업 기능**을 사용하여 문제를 해결한 예시입니다. 이 방식은 `div` 요소가 DOM에 추가되거나 제거될 때마다 제대로 된 클린업 작업을 수행할 수 있습니다.

```jsx
import React, { useState, useCallback } from "react";

const MyComponent = () => {
  const [isVisible, setIsVisible] = useState(true);

  // ref 콜백을 사용해 DOM 노드가 생성/제거될 때마다 실행
  const refCallback = useCallback((node) => {
    if (node) {
      // DOM 노드가 생성되었을 때
      console.log("DOM 노드가 생성되었습니다:", node);

      // 클린업 함수 반환
      return () => {
        // DOM 노드가 제거되었을 때
        console.log("DOM 노드가 제거되었습니다:", node);
      };
    }
  }, []);

  return (
    <div>
      {isVisible && <div ref={refCallback}>안녕하세요!</div>}
      <button onClick={() => setIsVisible((prev) => !prev)}>
        {isVisible ? "숨기기" : "보이기"}
      </button>
    </div>
  );
};

export default MyComponent;
```

- `refCallback`은 DOM 노드가 **생성될 때**와 **제거될 때** 각각 호출됩니다.
- DOM 노드가 마운트되면 `"DOM 노드가 생성되었습니다"`라는 로그가 출력되고, 언마운트되면 `"DOM 노드가 제거되었습니다"` 로그가 출력됩니다.
- `useEffect` 대신 `ref` 콜백을 사용함으로써 상태가 변경될 때마다 적절한 클린업이 이루어집니다.

<br/>

## Meta tags

기존에는 HTML에서 <title>, <link> 및 <meta>와 같은 문서 메타데이터 태그는 문서의 <head> 섹션에 배치 되도록 예약되어 있었습니다. React에서는 앱에 적합한 메타 데이터가 무엇인지 결정하는 구성요소가 <head>를 렌더링하는 위치에서 매우 멀리 떨어져있거나 React가 <head>를 전혀 렌더링하지 않을 수 있습니다. 이러한 요소를 효과에 수동으로 삽입하거나 react-helmet과 같은 라이브러리를 통해 삽입해야 했으며 서버에서 React 애플리케이션을 렌더링할 때 신중한 처리가 필요했습니다.

리액트 19에서는 컴포넌트 자체에 문서 메타데이터 태그를 렌더링하는 기능이 추가됩니다.

```jsx
function BlogPost({ post }) {
  return (
    <article>
      <h1>{post.title}</h1>
      <title>{post.title}</title>
      <meta name="author" content="Josh" />
      <link rel="author" href="https://twitter.com/joshcstory/" />
      <meta name="keywords" content={post.keywords} />
      <p>Eee equals em-see-squared...</p>
    </article>
  );
}
```

리액트가 위 컴포넌트를 렌더링할 때, `<title>`, `<link>`, `<meta>` 태그는 문서 `<head>` 섹션에 위치하도록 자동으로 호이스팅 처리합니다. 메타데이터 태그를 기본적으로 지원함에 따라 클라이언트 전용 앱, 스트리밍 SSR 및 서버 컴포넌트와 함께 동작하는 것도 가능합니다.

<br/>

💡 **그럼에도 불구하고 메타데이터 라이브러리가 필요할 수 있습니다.**

간단한 경우 문서 메타데이터 태그를 렌더링하는 것이 적합할 수 있으나, 라이브러리는 일반 메타데이터를 현재 라우트에 따라 특정 메타데이터로 재정의하는 등 더 우수한 기능을 제공할 수 있습니다. 추가된 기능을 통해 [`react-helmet`](https://github.com/nfl/react-helmet)과 같은 프레임워크와 라이브러리가 메타데이터 태그를 지원하기 쉽게 만들며, 대체가 아닌 보완의 역할을 수행합니다.

<br/>

## useDeferrdValue

React 19에서는 `useDeferredValue`의 두 번째 인자로 초기값을 설정할 수 있는 기능이 추가되었습니다. 이 기능은 컴포넌트의 최초 렌더링 시에도 트랜지션을 적용하여 사용자 인터페이스를 부드럽게 제공할 수 있도록 합니다.

```jsx
import React, { useState } from "react";
import { useDeferredValue } from "react";

const SearchPage = () => {
  const [query, setQuery] = useState("");
  const deferredQuery = useDeferredValue(query, "");

  return (
    <div>
      <input
        type="text"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      <SearchResults query={deferredQuery} />
    </div>
  );
};

const SearchResults = ({ query }) => {
  // 이 컴포넌트는 `query`가 변경될 때마다 다시 렌더링됩니다.
  return <div>Search results for: {query}</div>;
};
```

- 새로운 기능으로 `useDeferredValue`의 두 번째 인자로 초기값을 설정할 수 있습니다. 이 초기값은 컴포넌트의 최초 렌더링 시에 사용됩니다. 위 예제에서는 빈 문자열 **`''`**을 초기값으로 설정했습니다.
- 이 초기값은 컴포넌트의 최초 렌더링 시 `deferredQuery`에 설정되며, 이후 `query`의 값이 업데이트되면 새로운 값으로 트랜지션이 발생합니다.
- 초기값을 사용하면 트랜지션을 통해 상태가 즉시 업데이트되기 전에 부드러운 사용자 인터페이스를 제공할 수 있습니다. 예를 들어, 입력 필드에서 텍스트를 입력하는 동안 `deferredQuery`는 지연된 값을 사용하여 검색 결과를 렌더링합니다. 이는 입력 필드의 변화에 즉시 반응하지 않고, 지연된 값을 사용하여 결과를 표시함으로써 사용자 인터페이스를 부드럽게 유지합니다.

<br/>

## 에러 핸들링 개선

React 19에서는 에러 핸들링을 세밀하게 제어할 수 있는 새로운 기능이 도입되었습니다.

이제 `createRoot`와 `hydrateRoot` 메서드에서 `onCaughtError`와 `onUncaughtError` 옵션을 사용하여 에러 처리 방식을 조정할 수 있습니다.

<br/>

### 기존의 문제점

기존에는 `Error Boundary`를 사용하여 에러를 잡았지만, 여전히 콘솔에 `console.error`로 에러가 출력되는 문제가 있었습니다.

이로 인해 개발자는 에러가 정상적으로 처리되었음에도 불구하고 불필요한 에러 메시지를 보게 되었고, 이는 개발 과정에서 혼란을 초래할 수 있었습니다.

<br/>

### React 19 개선점

React 19에서는 에러 핸들링을 더 세밀하게 제어할 수 있는 방법이 제공됩니다.

`createRoot`와 `hydrateRoot`의 `onCaughtError` 및 `onUncaughtError` 옵션을 사용하여 콘솔에 에러 메시지를 출력하는 기본 동작을 덮어쓸 수 있습니다.

`onCaughtError`는 `Error Boundary`가 잡은 에러를 처리할 때 사용됩니다.

이 옵션을 사용하여 에러가 콘솔에 기록되는 방식을 제어할 수 있습니다.

```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';

const App = () => {
  // 의도적으로 에러를 발생시키는 컴포넌트
  throw new Error('Something went wrong!');
  return <div>Hello, React 19!</div>;
};

// createRoot를 사용하여 에러 핸들링 설정
ReactDOM.createRoot(document.getElementById('root')!, {
  onCaughtError: (error) => {
    // 에러를 info로 기록
    console.info('Caught an error:', error);
  },
}).render(<App />);
```

이 코드에서 `onCaughtError` 옵션에 제공된 함수는 `Error Boundary`가 잡은 에러를 처리합니다. 기본적으로 `console.error`로 출력되는 에러를 `console.info`로 기록하여 개발자 혼란을 줄일 수 있습니다.

<br/>

`onUncaughtError`는 `Error Boundary`가 잡지 못한 에러를 처리하는 옵션입니다.

이 옵션을 사용하면 애플리케이션 전체에서 발생한 예기치 않은 에러를 처리할 수 있습니다.

```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';

const App = () => {
  // 의도적으로 에러를 발생시키는 컴포넌트
  throw new Error('Something went wrong!');
  return <div>Hello, React 19!</div>;
};

// createRoot를 사용하여 에러 핸들링 설정
ReactDOM.createRoot(document.getElementById('root')!, {
  onCaughtError: (error) => {
    console.info('Caught an error:', error);
  },
  onUncaughtError: (error) => {
    console.error('Uncaught error:', error);
  },
}).render(<App />);
```

`onUncaughtError`를 사용하여, 애플리케이션에서 발생한 에러를 `console.error`로 기록합니다. 이는 `Error Boundary`가 에러를 잡지 못한 경우에 유용합니다.

<br/>

## Stylesheet precedence 속성

React 19에서는 precedence 속성을 제공하여 스타일시트의 삽입 순서를 DOM내에서 관리할 수 있게 되었습니다. 이를 통해 외부 스타일시트가 로드된 후 해당 스타일 규칙에 맞게 콘텐츠가 제대로 표시되도록 할 수 있습니다.

아래 예시를 보면, bar 스타일시트는 high precedence를 가지며, 기본 precedence를 가진 foo 보다 우선적으로 로드되고 DOM 상에서도 앞에 위치합니다. baz 스타일시트는 ComponentOne 에서 사용된 foo 와 동일한 precedence를 가지므로, DOM에서 foo 와 bar 사이에 위치하게 됩니다.

```jsx
function ComponentOne() {
  return (
    <Suspense fallback="loading...">
      <link rel="stylesheet" href="foo" precedence="default" />
      <link rel="stylesheet" href="bar" precedence="high" />
      <article class="foo-class bar-class">
        {...}
      </article>
    </Suspense>
  )
}

function ComponentTwo() {
  return (
    <div>
      <p>{...}</p>
      <link rel="stylesheet" href="baz" precedence="default" />  <-- foo와 bar 사이에 삽입됩니다.
    </div>
  )
}
```

```jsx
function App() {
  return (
    <>
      <ComponentOne />
      ...
      <ComponentOne /> // DOM에 중복된 스타일시트 링크를 생성하지 않습니다.
    </>
  );
}
```

서버 사이드 렌더링 중 리액트는 `<head>`에 스타일시트를 포함시킵니다. 이로써 브라우저가 로드될 때까지 화면이 그려지지 않도록 합니다. 이미 스트리밍을 시작한 후에 스타일시트가 늦게 발견되더라도, 리액트는 해당 스타일시트에 의존하는 서스펜스 바운더리의 콘텐츠를 공개하기 전에 스타일시트가 클라이언트의 `<head>`에 삽입되도록 보장합니다.

클라이언트 사이드 렌더링 도중 리액트는 새로 렌더링 된 스타일시트가 로드될 때까지 대기한 후 렌더링을 커밋합니다. 이 컴포넌트를 애플리케이션 내 여러 위치에서 렌더링하는 경우 리액트는 스타일시트를 문서에 한 번만 포함시킵니다.

React 19의 precedence 속성은 특히 CSS-in-JS 라이브러리와 함께 사용할 때
매우 유용합니다. 이 속성을 통해 스타일시트의 우선순위를 명시적으로 지정할 수 있
어, 여러 스타일링 전략을 보다 효율적으로 적용할 수 있습니다.

주요 개선 사항 정리하면 아래와 같습니다:

- 서버사이드 렌더링 시: 컴포넌트가 렌더링될 때 스타일시트를 삽입할 수 있습니다.
- 중복 방지: 여러 곳에서 동일한 컴포넌트를 렌더링할 경우, CSS는 한 번만 삽입됩니다.
- 모듈 번들러와의 통합: 모듈 번들러가 이 기능을 활용하면 성능과 관리 측면에서이점을 얻을 수 있습니다.

<br/>

## 참고 사이트

https://velog.io/@typo/react-19-beta

https://ko.react.dev/blog/2024/04/25/react-19
