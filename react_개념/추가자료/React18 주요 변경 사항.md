## 1. Concurrent Mode(동시성 모드)

기존 React의 렌더링 방식은 **동기적**이었습니다. 즉, 컴포넌트가 렌더링을 시작하면, 중간에 중단하지 않고 작업을 끝까지 완료해야 했습니다. 이로 인해 렌더링 작업이 시간이 오래 걸리면 UI가 중단되는 현상(예: 입력 지연, 느린 화면 업데이트)이 발생할 수 있었습니다.

**React 18의 동시성 모드**는 렌더링 작업을 **더 작은 단위로 분할**하여, 중요한 작업(예: 사용자 입력, 애니메이션)을 우선적으로 처리하고, 덜 중요한 작업(예: 대량의 데이터 로드 및 렌더링)을 뒤로 미루거나 나중에 처리할 수 있게 합니다. 이를 통해 더 나은 사용자 경험을 제공합니다.

<br/>

## 2. 새로운 Hooks

### useId

`useId`는 컴포넌트별로 유니크한 값을 생성하는 새로운 훅입니다.

컴포넌트 내부에서 유니크한 값을 생성하는 것은 생각보다 까다롭습니다.

하나의 컴포넌트가 여러 군데에서 재사용되는 경우도 고려해야 하며, 리액트 컴포넌트 트리에서 컴포넌트가 가지는 모든 값이 겹치지 않고 모두 달라야 한다는 조건도 있습니다, 또한, SSR 환경에서 hydration이 발생할 때 서버와 클라이언트가 동일한 값을 가져야 에러가 발생하지 않아 이러한 점도 고려해야합니다.

`useId`를 사용하면 위 문제들을 자동으로 해결해 주며, 편리하게 유니크한 id값을 사용할 수 있습니다.

<br/>

### useId 사용 예시 코드

```jsx
import { useId } from 'react';

function FirstChild() {
	const id = useId();
	return <p>First ChildId: {id}</p>
}

function SecondChild() {
  const id = useId();
    return (
      <div>
        <Child/>
        <p>Second ChildId: {id}</p>
      </div>
    );
}

function App() {
  return (
    <div>
      <h1>HOME</h1>
      <FirstChild />
      <FirstChild />
      <SecondChild />
      <SecondChild />
    </div>
  );
}

export default App;
```

위 코드를 SSR시 제공되는 HTML를 보면 아래와 같습니다.

같은 컴포넌트라도 서로 인스턴스가 다르면 다른 랜덤한 값을 만들어내며, 서버 사이드와 클라이언트간에 동일한 값이 생성되어 hydration 불일치 문제도 발생하지 않습니다.

useId가 생성한 값은 :으로 감싸져 있으며, 이는 querySelector에서 작동하지 않도록 하기 위한 의도적인 결과입니다. 앞글자가 R이면 서버에서 생성된 값이며, 앞글자가 r이면 클라이언트에서 생성된 값입니다.

```jsx
<!DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>React SSR</title>
    <script defer src="main.js"></script>
  </head>
  <body>
    <div id="root">
      <div>
        <h1>HOME</h1>
        <p>
          First ChildId:
          <!-- -->:R2:
        </p>
        <p>
          First ChildId:
          <!-- -->:R3:
        </p>
        <div>
          <p>
            First ChildId:
            <!-- -->:Rs:
          </p>
          <p>
            Second ChildId:
            <!-- -->:R4:
          </p>
        </div>
        <div>
          <p>
            First ChildId:
            <!-- -->:Rt:
          </p>
          <p>
            Second ChildId:
            <!-- -->:R5:
          </p>
        </div>
      </div>
    </div>
  </body>
</html>
```

<br/>

### useTransition

`useTransition`은 비동기 작업을 처리할 때 사용되며, 특히, 복잡한 상태 업데이트나 UI 렌더링이 필요할 때 해당 작업을 우선순위가 낮은 작업으로 처리하여 사용자 인터페이스가 끊김 없이 반응할 수 있도록 도와줍니다.

<br/>

### useTransition 반환값

- `isPending` : 상태 업데이트가 지연 중인지 여부를 나타내는 불리언 값.
- `startTransition` : 긴급하지 않은 상태 업데이트를 실행하는 함수.

```jsx
const [isPending, startTransition] = useTransition();

const handleClick = () => {
  startTransition(() => {
    // 긴급하지 않은 상태 업데이트
    setState(newValue);
  });
};
```

<br/>

### useTransition 사용 예시 코드

`useTransition`을 사용해 검색과 같은 작업을 처리할 때, 검색어 입력은 즉시 반영되지만 결과 목록을 업데이트하는 작업은 우선순위를 낮춰서 UI가 끊기지 않게 처리하는 경우입니다.

```jsx
import { useEffect, useState, useTransition } from "react";

function SearchComponent() {
  const [keyword, setKeyword] = useState(""); // 입력값은 즉시 반영
  const [list, setList] = useState([]);
  const [isPending, startTransition] = useTransition();
	const ITEMS = Array.from({ length: 5000 }, (_, i) => `Item ${i + 1}`);

  const filterItems = (query) => {
    return ITEMS.filter((item) => item.includes(query));
  };

  const handleChange = (e) => {
    const value = e.target.value;
    setKeyword(value);
  };

  useEffect(() => {
    // 검색 결과 업데이트는 낮은 우선순위로 처리
    startTransition(() => {
      const filteredList = filterItems(keyword);
      setList(filteredList);
    });
  }, [keyword]);

  return (
    <div>
      <input
        type="text"
        value={keyword}
        onChange={handleChange}
        placeholder="Search..."
      />
      {/* 검색 결과가 지연될 때 로딩 메시지 표시 */}
      {isPending && <p>Updating list...</p>}
      <ul>
        {list.map((item, index) => (
          <li key={index}>{item}</li>
        ))}
      </ul>
    </div>
  );
}

export default SearchComponent;

```

- `startTransition` : 검색 결과 목록을 업데이트하는 작업을 낮은 우선순위로 처리합니다.
- `isPending` : 트랜지션이 진행 중일 때 UI에 "Updating list..."를 보여줌으로써 로딩 상태를 보여줍니다.

<br/>

### useDeferredValue

`useTransition`은 함수 실행의 우선순위를 지정하는 반면, `useDeferredValue`는 값의 업데이트 우선순위를 지정합니다. 우선순위가 높은 작업을 실행하는 동안 `useMemo`와 유사하게 이전 값을 계속 들고 있으면서 업데이트를 지연시킵니다.

이 훅은 `useMemo`와 함께 사용하면 더 효과가 좋다. 종속된 값들을 memoize 시키면 불필요한 재 랜더링을 막으면서 하위 컴포넌트나 상태의 업데이트를 지연시킬 수 있습니다.

<br/>

### useDeferredValue 사용 예시

`useDeferredValue`는 입력 필드의 값이 변경될 때, 해당 값을 즉시 반영하지 않고 지연시켜 불필요한 리렌더링을 방지하는 데 사용할 수 있습니다. 아래 예시에서는 사용자가 검색어를 입력할 때, 검색어 업데이트를 지연시켜 성능을 최적화하는 상황입니다.

```jsx
import { useState, useDeferredValue, useEffect } from "react";

function App() {
  const [keyword, setKeyword] = useState("");
  const [list, setList] = useState([]);
  const deferredInput = useDeferredValue(keyword); // 지연된 값
	const ITEMS = Array.from({ length: 5000 }, (_, i) => `Item ${i + 1}`);
	
  const filterItems = (query) => {
    // 가상 데이터 필터링 로직 (예: 5000개의 항목을 필터링)
    return ITEMS.filter((item) => item.includes(query));
  };

  useEffect(() => {
    setList(filterItems(deferredInput));
  }, [deferredInput]);

  return (
    <div>
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Search..."
      />

      <ul>
        {list.map((item, index) => (
          <li key={index}>{item}</li>
        ))}
      </ul>
    </div>
  );
}

export default App;
```

- `useDeferredValue`는 `keyword` 값을 지연시켜 성능을 최적화합니다. 즉, 사용자가 빠르게 입력할 때 매번 즉시 필터링하는 것이 아니라, 일정 시간 지연된 후 마지막 값으로 필터링 작업을 수행합니다. 이를 통해 UI 성능을 개선할 수 있습니다.
- 검색어가 입력될 때마다 바로 목록을 업데이트하지 않고, 일정 시간이 지나면 업데이트되도록 처리하여 성능을 최적화합니다.

useTransition과 useDeferredValue의 자세한 설명은 [18. useTransition & useDeferredValue](https://www.notion.so/18-useTransition-useDeferredValue-12212b90211180d6a8e1ed8dd66bad7f?pvs=21) 를 참고해주세요.

<br/>

### useSyncExternalStore

React 18부터 `useTransition`, `useDeferredValue`와 같이 렌더링을 일시중지하거나, 뒤로 미루는 등의 동시성 최적화를 도와줄 수 있는 훅들이 사용할수있습니다. 하지만 리액트에서 이렇게 동시성 렌더링이 가능해지면서, 외부 저장소의 데이터를 참조하는 컴포넌트를 렌더링할때 같은 시기에 렌더링을 했지만, 서로 다른 시점의 데이터를 참조할 수도 있는 **Tearing**문제가 발생할 수 있습니다.

**Tearing**이란 리액트에서는 하나의 state 값이 있음에도 서로 다른 값을 기준으로 렌더링되는 현상을 의미합니다. 예를 들어, startTransition으로 렌더링을 일시 중지하고 일시 중지 과정에서 값이 업데이트되는 경우 동일한 하나의 변수에 대해서 서로 다른 컴포넌트 형태가 나타날 수 있게됩니다.

리액트에서 관리하는 state라면 useTransition, useDefferedValue와 같이 내부적으로 이러한 문제를 해결하기 위한 처리를 해두었지만 관리할 수 없는 외부 데이터 소스에서는 문제가 발생하게됩니다.

여기서, 외부 데이터 소스란 리액트의 클로저 범위 밖에 있는 값들 글로벌 변수, document.body, window.innerWidth, DOM, 리액트 외부에 상태를 저장하는 라이브러리 등을 말합니다. 이 외부 데이터 소스의 Tearing 현상을 해결하는 Hook이 바로 `useSyncExternalStore`입니다.

```jsx
useSyncExternalStore(
  subscribe: (callback) => Unsubscribe
  getSanpshot: () => State
) => State
```

- 첫 번째 인수는 subscribe로 콜백 함수를 받아 스토어에 등록하는 용도로 사용됩니다. 스토에 있는 값이 변경되면 이 콜백이 호출되며, `useSyncExternalStore`를 사용하는 컴포넌트를 리렌더링합니다.
- 두 번째 인수는 컴포넌트에 필요한 스토어의 데이터를 반환하는 함수입니다. 이 함수는 스토어가 변경되지 않으면 매번 함수를 호출할 때 마다 동일한 값을 반환해야합니다. 스토어에서 값이 변경됬다면 이 값을 이전 값과 Object.is로 비교해 값이 변경되었는지 확인하고 컴포넌트를 렌더링합니다.
- 마지막 인수는 옵셔널 값으로, SSR시 내부 리액트를 hydrate하는 도증에만 사용됩니다. SSR에서 렌더링되는 훅이라면 반드시 이 값을 넘겨주어야합니다. 만약, 클라이언트와 값의 불일치가 발생한다면 오류가 발생하게됩니다.

<br/>

### useSyncExternalStore 사용 예시

`useSyncExternalStore` 를 통해 현재 윈도우의 innerWidth를 확인하는 훅입니다. **innerWidth**는 리액트 외부에 있는 데이터 값이므로 이 값의 변경 여부를 확인해 리렌더링까지 이어지게 하려면 `useSyncExternalStore`를 사용하는 것이 적절합니다.

```jsx
import { useSyncExternalStore } from "react";

function subscribe(callback) {
  window.addEventListener("resize", callback);
  return () => {
    window.removeEventListener("resize", callback);
  };
}

const useInnerWidth = () => {
  const innerWidth = useSyncExternalStore(
    subscribe,
    () => window.innerWidth,
    () => 0
  );

  return innerWidth;
};

const App = () => {
  const windowSize = useInnerWidth();
  return <p>{windowSize}</p>;
};

export default App;
```

subscribe 함수를 첫 번째 인수로 넘겨 innerWidth가 변경될 때 일어나는 콜백을 등록하였습니다.

useSyncExternalStore는 subscribe 함수의 첫 번째 인수인 콜백을 추가해 resize 이벤트가 발생할 때 마다 해당 콜백이 실행됩니다.

두 번째 인수로 현재 스토어의 값인 window.innerWidth를 마지막 인수로 SSR에서 해당 값을 알 수 없으므로 0를 주었습니다.

<br/>

### useInsertionEffect

`useInsertionEffect`는 `useLayoutEffect`가 동작하기 전에 스타일을 먼저 조작하게 해주는 훅으로, CSS-in-js 라이브러리를 위한 훅입니다.

CSS의 추가 및 수정은 브라우저에서 렌더링하는 작업 대부분을 다시 계산해 작업해야하기 때문에 리액트 입장에서는 모든 리액트 컴포넌트가 영향을 미칠 수 있는 매우 무거운 작업입니다. 따라서 리액트 17과 styled-components에서는 클라이언트 렌더링 시에 이러한 작업이 발생하지 않도록 서버 사이드에서 스타일 코드를 삽입하였습니다. 바로 이 작업을 도와주는 새로운 훅이 `useInsertionEffect`입니다.

`useInsertionEffect`의 기본적인 구조는 `useEffect`와 동일하지만 실행 시점이 다릅니다.

`useInsertionEffect`는 DOM이 실제로 변경되지 전에 동기적으로 실행됩니다. 이 훅 내부에 스타일을 삽입하는 코드를 넣어 브라우저가 레이아웃을 계산하기 전에 실행하도록 하여 좀 더 자연스러운 스타일 삽입이 가능합니다.

<br/>

**💡 useEffect vs useLayoutEffect vs useInsertionEffect**

실행 순서는 **useInsertionEffect > useLayoutEffect > useEffect** 순입니다.

`useLayoutEffect`와 비교했을 경우 실행되는 시점이 미묘하게 다릅니다. 두 훅 모두 브라우저에 DOM이 렌더링 되기 전에 실행된다는 공통점이 있지만 `useLayoutEffect`는 모든 DOM의 변경 작업이 다 끝난 이후에 실행되는 반면 `useInsertionEffect`는 DOM의 변경 작업이 이전에 실행됩니다. 브라우저가 다시 스타일을 입혀서 **DOM을 재계산하지 않아도 된다는 점**에서 차이가 나게됩니다.

<br/>

## 2. Automatic Batching

Automatic Batching는 React가 여러 상태(state) 업데이트를 하나의 렌더링 작업으로 묶어 처리하는 방식입니다. 상태 업데이트가 발생할 때마다 리렌더링을 하지 않고, 여러 개의 상태 업데이트를 모아서 한 번에 처리하는 것입니다. 이를 통해 리렌더링 횟수를 줄여 애플리케이션 성능이 개선됩니다.

<br/>

### React 18 이전 vs. 이후의 자동 배치

React 18 이전에는 **React 이벤트 핸들러** 내에서만 자동 배치가 동작했습니다. 예를 들어, 버튼 클릭과 같은 이벤트 핸들러에서 여러 상태 업데이트가 발생하면 React가 이를 자동으로 배치하여 처리했습니다. 그러나 비동기 작업(예: `setTimeout`, `Promise` 등의 콜백)에서는 배치가 동작하지 않았습니다.

**React 18**에서는 이러한 자동 배치가 **비동기 함수**와 **타이머** 등에서도 동작합니다. 예를 들어, `setTimeout`, `fetch` 콜백, 또는 `Promise` 안에서 발생하는 상태 업데이트도 자동으로 배치되어 렌더링을 한 번만 수행합니다.

```jsx
import { useState } from 'react';

function MyComponent() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState('');

  const handleClick = () => {
    setTimeout(() => {
      // React 18 이전에서는 이 두 개의 상태 업데이트가 각각 렌더링을 발생시켰습니다.
      setCount((c) => c + 1);
      setText('Updated');
    }, 1000);
  };

  return (
    <div>
      <p>{count}</p>
      <p>{text}</p>
      <button onClick={handleClick}>Update</button>
    </div>
  );
}
```

이 코드는 `setTimeout` 내에서 두 가지 상태 업데이트를 발생시키지만, **React 18**에서는 이 두 상태 업데이트가 자동으로 배치되어 한 번의 렌더링만 일어납니다

<br/>

### Automatic Batching 비활성화 방법

특정 경우에 자동 배치를 원하지 않는다면, `flushSync` 함수를 사용하여 즉시 상태 업데이트를 강제로 수행할 수 있습니다

```jsx
import { flushSync } from 'react-dom';

function handleClick() {
  flushSync(() => {
    setCount((c) => c + 1);
  });
  // 여기서 즉시 상태가 반영되고 렌더링이 발생합니다.
  flushSync(() => {
    setText('Updated');
  });
}
```

<br/>

## 3. creatRoot & hydreateRoot

`createRoot`와 `hydrateRoot` 메서드는 **React의 렌더링 엔진**을 더 효율적으로 개선한 중요한 기능입니다. 이 두 메서드는 기존의 `ReactDOM.render`와 `ReactDOM.hydrate`를 대체하는 방식으로 도입되었으며, **동시성 모드** 및 **자동 배칭** 등의 새로운 기능과 밀접하게 관련이 있습니다.

<br/>

### createRoot

React 18에서 `createRoot`는 클라이언트 렌더링을 시작할 때 사용하는 메서드로, 기존의 `ReactDOM.render`를 대체합니다. React 18에서 `ReactDOM.render`는 더 이상 권장되지 않으며, `createRoot`가 새롭게 렌더링을 관리하는 방식입니다.

```jsx
import { createRoot } from 'react-dom/client';
import App from './App';

const container = document.getElementById('root');
const root = createRoot(container);

root.render(<App />);
```

- **동시성 모드 지원**: `createRoot`는 React 18의 **동시성 모드**를 기본적으로 지원합니다. 즉, 렌더링 작업을 중단하거나 우선순위를 조정할 수 있는 기능이 포함되어 있습니다. 이를 통해 React는 큰 업데이트를 즉시 렌더링하지 않고, 중요한 작업과 덜 중요한 작업을 구분하여 사용자 경험을 개선할 수 있습니다.
- **자동 배칭**: `createRoot`를 사용할 경우, 모든 상태 업데이트는 자동으로 배칭됩니다. 이전에는 이벤트 핸들러 내부에서만 상태 업데이트가 배칭되었지만, React 18에서는 `setTimeout`, `Promise`와 같은 비동기 작업 내에서도 배칭이 적용됩니다. 이는 불필요한 리렌더링을 줄여 성능을 최적화합니다.

<br/>

### hydrateRoot

React 18에서 도입된 `hydrateRoot`는 **서버 사이드 렌더링(SSR)** 후 클라이언트에서 UI를 **하이드레이션**(hydration)하는 메서드입니다. 기존의 `ReactDOM.hydrate`는 `hydrateRoot`로 대체되었으며, 이 역시 동시성 모드와 자동 배칭 등의 기능을 포함합니다.

```jsx
import { hydrateRoot } from 'react-dom/client';
import App from './App';

const container = document.getElementById('root');
hydrateRoot(container, <App />);
```

- **동시성 모드 지원**: `hydrateRoot` 역시 동시성 모드를 지원합니다. 동시성 모드는 기존의 SSR 환경에서 서버에서 렌더링된 HTML을 클라이언트에서 하이드레이션할 때 **더 유연한 렌더링**을 가능하게 합니다. 하이드레이션 중에도 React는 중요한 사용자 이벤트를 우선적으로 처리하고, 덜 중요한 작업은 나중에 처리할 수 있습니다.
- **선택적 하이드레이션(Selective Hydration)**: `hydrateRoot`는 React 18에서 **선택적 하이드레이션** 기능을 제공합니다. 이는 서버에서 렌더링된 HTML이 클라이언트에 전달될 때, 사용자가 상호작용하는 부분만 우선적으로 하이드레이션하여 초기 성능을 최적화할 수 있습니다. 나머지 UI 부분은 백그라운드에서 천천히 하이드레이션됩니다.

<br/>

## 4. React Server Components(RSC)

**서버 컴포넌트**는 React 18에서 도입된 새로운 개념으로, React 컴포넌트를 서버에서만 렌더링하고 클라이언트로는 그 결과만을 전달합니다. 클라이언트 컴포넌트와 달리 서버 컴포넌트는 클라이언트에 JavaScript 코드나 상태 관리를 전달하지 않고, HTML과 같은 최종 결과만 전달하므로 **클라이언트의 자원 소비를 최소화**할 수 있습니다.

<br/>

### React Server Components의 특징

- **서버에서만 렌더링**: 서버 컴포넌트는 클라이언트에 전송되기 전에 서버에서 실행되고, 해당 컴포넌트의 결과만 클라이언트로 전달됩니다. 따라서 클라이언트는 컴포넌트의 JavaScript를 다운로드하거나 실행할 필요가 없습니다.
- **상태 관리 불필요**: 서버 컴포넌트는 **상태(state)**, **이벤트 핸들러** 등의 클라이언트 측 동작을 처리하지 않습니다. 서버 컴포넌트는 단순히 서버에서 데이터를 가져와서 이를 기반으로 렌더링 결과를 생성하고 클라이언트에 전달합니다.
- **클라이언트와의 결합**: 서버 컴포넌트는 클라이언트 컴포넌트와 결합되어 동작할 수 있습니다. 클라이언트 컴포넌트가 인터랙티브한 UI를 제공하는 반면, 서버 컴포넌트는 데이터를 기반으로 정적인 콘텐츠를 렌더링하는 데 주로 사용됩니다.
- **경량 클라이언트 렌더링**: 서버 컴포넌트의 결과로 클라이언트로 전달되는 것은 **순수한 HTML**이며, 클라이언트는 이를 빠르게 렌더링할 수 있습니다. 클라이언트에서 더 적은 자원을 사용하므로 성능을 최적화할 수 있습니다.

<br/>

### React Server Components의 사용

- 서버 컴포넌트는 일반적으로 `.server.js` 확장자를 사용하여 파일을 구분합니다. 이를 통해 React는 해당 컴포넌트가 서버에서만 실행된다는 것을 인식하고, 클라이언트로 전달되지 않도록 합니다.
    
    ```jsx
    // MyComponent.server.js (서버 컴포넌트)
    import React from 'react';
    
    export default function MyComponent() {
      const data = fetchDataFromServer(); // 서버 측에서만 실행되는 로직
      return <div>Data: {data}</div>;
    }
    ```
    
- 서버 컴포넌트는 서버에서만 실행되므로, **서버 측 API 호출이나 데이터베이스 쿼리**를 직접 수행할 수 있습니다. 이 방식은 클라이언트 컴포넌트에서 비동기 데이터를 처리하는 것보다 더 효율적입니다.
    
    ```jsx
    // DataFetchingComponent.server.js
    import React from 'react';
    import fetchDataFromAPI from './api';
    
    export default async function DataFetchingComponent() {
      const data = await fetchDataFromAPI();
      return (
        <div>
          <h1>Fetched Data:</h1>
          <pre>{JSON.stringify(data, null, 2)}</pre>
        </div>
      );
    }
    ```
    
- 서버 컴포넌트에서는 `useState`, `useEffect`와 같은 클라이언트 전용 훅이나 API를 사용할 수 없습니다. 이는 서버 컴포넌트가 클라이언트에서 렌더링되지 않기 때문에 의미가 없기 때문입니다.
    
    ```jsx
    // 잘못된 사용 (서버 컴포넌트에서 클라이언트 훅 사용 불가)
    export default function MyComponent() {
      const [state, setState] = useState(0); // 오류 발생
      return <div>State: {state}</div>;
    }
    ```
    
- 서버 컴포넌트와 클라이언트 컴포넌트를 결합할 수 있습니다. 서버에서 데이터나 정적인 UI를 처리한 다음, 클라이언트에서 추가적인 상호작용을 제공하는 방식입니다. 서버 컴포넌트는 데이터를 처리하고, 클라이언트 컴포넌트는 사용자와의 상호작용을 처리할 수 있습니다.
    
    ```jsx
    // MyComponent.server.js (서버 컴포넌트)
    import React from 'react';
    import InteractiveComponent from './InteractiveComponent.client'; // 클라이언트 컴포넌트 임포트
    
    export default function MyComponent() {
      const data = fetchDataFromServer();
      return (
        <div>
          <h1>Server Rendered Data: {data}</h1>
          <InteractiveComponent /> {/* 클라이언트 컴포넌트 렌더링 */}
        </div>
      );
    }
    
    ```
    
    ```jsx
    // InteractiveComponent.client.js (클라이언트 컴포넌트)
    import React, { useState } from 'react';
    
    export default function InteractiveComponent() {
      const [count, setCount] = useState(0);
      return (
        <div>
          <p>Count: {count}</p>
          <button onClick={() => setCount(count + 1)}>Increment</button>
        </div>
      );
    }
    ```
    
<br/>

### React Server Components와 기존 SSR의 차이

기존의 서버 사이드 렌더링(SSR)과 서버 컴포넌트는 비슷한 개념처럼 보일 수 있지만, 두 가지는 본질적으로 다른 방식입니다:

- **SSR**: 서버에서 전체 React 애플리케이션을 렌더링하고, 클라이언트에 해당 HTML을 전달합니다. 이후 클라이언트에서 하이드레이션을 통해 JavaScript가 실행되어 인터랙티브한 UI가 됩니다.
- **서버 컴포넌트**: 서버에서 특정 컴포넌트만 렌더링하고, 그 결과만 클라이언트로 전달하여 클라이언트 컴포넌트와 결합합니다. 클라이언트는 서버 컴포넌트의 결과만 렌더링하고, 해당 JavaScript는 실행하지 않습니다.

<br/>

## 5. Suspense

**React 18**에서는 `Suspense` 기능이 확장되고 강화되었습니다. 기존에는 `Suspense`가 주로 **코드 스플리팅**과 같은 특정 용도로만 사용되었지만, React 18에서는 **비동기 데이터 로딩**을 포함한 다양한 비동기 작업을 처리하는 데에 활용할 수 있도록 개선되었습니다

<br/>

### Suspense 주요 변경점

**1 ) 비동기 데이터 패칭 지원**

- React 18 이전에는 `Suspense`가 주로 `React.lazy`를 통한 **코드 스플리팅**에서만 사용되었습니다. 하지만 React 18부터는 서버에서 데이터를 로딩할 때도 `Suspense`를 사용할 수 있게 되었습니다. 즉, 컴포넌트가 필요한 데이터를 비동기적으로 불러오는 과정에서 그 상태를 관리할 수 있게 되었습니다.
- 이를 통해 데이터를 가져오는 동안에 **로딩 UI**를 표시하거나, 데이터가 준비되기 전에 컴포넌트를 미리 렌더링하지 않도록 제어할 수 있습니다.

**2 ) 서버 컴포넌트와 함께 사용 가능**

- React 18의 새로운 기능인 **서버 컴포넌트(Server Components)**와도 `Suspense`를 함께 사용할 수 있습니다. 서버에서 데이터를 미리 패칭하여 클라이언트에게 전달하고, 클라이언트는 이 데이터를 받아 컴포넌트를 렌더링합니다. 이 과정에서 필요한 곳에 `Suspense`를 배치하여 서버에서 데이터를 불러오는 동안 로딩 상태를 처리할 수 있습니다.

**3 ) 동시성 모드와의 통합**

- React 18에서 추가된 **동시성 기능(Concurrent Features)**과 `Suspense`가 긴밀하게 통합되었습니다. 동시성 모드에서는 React가 백그라운드에서 여러 렌더링 작업을 동시에 처리할 수 있는데, 이 과정에서 `Suspense`가 중요한 역할을 합니다.
- `Suspense`를 통해 React는 비동기 데이터 로딩이 완료될 때까지 렌더링을 지연시킬 수 있으며, 필요한 데이터를 다 불러오기 전에 불필요한 렌더링이 발생하지 않도록 방지합니다.

**4 ) 트랜지션과 결합**

- React 18의 또 다른 기능인 **트랜지션(Transitions)**과도 `Suspense`를 함께 사용할 수 있습니다. 트랜지션은 느린 UI 업데이트(예: 페이지 전환)에서 사용자 경험을 부드럽게 만들어 주는데, `Suspense`와 결합하면 트랜지션 동안 데이터를 불러오는 과정에서 로딩 상태를 더 자연스럽게 처리할 수 있습니다.

<br/>

### Suspense 사용 예시

서버 사이드 렌더링(SSR)과 `Suspense`의 통합으로, 서버에서 데이터를 미리 패칭하고 이 데이터를 기반으로 클라이언트에서 컴포넌트를 렌더링할 수 있습니다. 

```jsx
import { Suspense } from 'react';

function ServerSideComponent() {
  // 서버에서 데이터를 불러오는 동안 Suspense가 로딩 상태를 표시합니다.
  return (
    <Suspense fallback={<div>Loading from server...</div>}>
      <DataFetchingComponent />
    </Suspense>
  );
}
```

<br/>

## 참고 자료
[모던 리액트 Deep Dive](https://wikibook.co.kr/react-deep-dive/)