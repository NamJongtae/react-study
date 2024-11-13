# ❓ Why - 이유 직접 찾아보기

React 공식 문서를 살펴보면서 기존에 이해하였던 내용을 더 깊게 살펴보기 위해 본질적인 내용이나 궁금한 내용을 직접 찾아보면 기록합니다. 

## 📃 목차
- [1. JSX가 어떻게 컴파일 될까?](#1-jsx가-어떻게-컴파일-될까)

- [2. Props로 ref를 사용하지 못하는 이유?](#2-props로-ref를-사용하지-못하는-이유)

- [3. React에서 State란 무엇일까?](#3-react에서-state란-무엇일까)

- [4. React의 렌더링은 어떻게 동작할까?](#4-react의-렌더링은-어떻게-동작할까)

- [5. Virtual DOM은 어떻게 동작할까?](#5-virtual-dom은-어떻게-동작할까)

- [6. useEffect는 어떻게 동작할까?](#6-useeffect는-어떻게-동작할까)

- [7. useMemo, useCallback, React.memo는 어떻게 동작할까?](#7-usememo-usecallback-reactmemo는-어떻게-동작할까)

- [8. React에서 상태관리 라이브러리가 왜 필요할까?](#8-react에서-상태관리-라이브러리가-왜-필요할까)

- [9. React Hooks의 기본 동작원리는 무엇일까?](#9-react-hooks의-기본-동작원리는-무엇일까)

- [10. React가 어떻게 SPA를 구현할까?](#10-react가-어떻게-spa를-구현할까)

<br/>

## 1. JSX가 어떻게 컴파일 될까?

JSX는 JavaScript 확장 문법으로, 컴파일 시 JavaScript 함수로 변환됩니다. 즉, JSX 코드를 작성하면 React는 이를 `React.createElement()` 함수 호출로 변환하여 실행합니다. 

```jsx
function MyComponent() {
	return <div>hello world</div>
}
```

React가 이 코드를 컴파일하면 내부적으로 `React.createElement()` 함수를 호출하는 형태로 변환됩니다.

```jsx
function MyComponent() {
	return React.createElement(div, null, "hello world");
}
```

`React.createElement()` 함수는 아래와 같은 구조를 가집니다:

```jsx
React.createElement(
  type,      // 'div'와 같은 HTML 태그 혹은 커스텀 React 컴포넌트
  props,     // 속성 (attributes), 없으면 null
  ...children // 자식 요소 (children), 여러 개일 수 있음. 여러개인 경우 배열로 전달
);

```

최종적으로 `React.createElement()`는 아래와 같은 **React Element 객체**를 반환합니다.

이 객체는 React가 실제 DOM을 생성하고 렌더링하는 데 사용됩니다.

```jsx
function MyComponent() {
	return {
  type: 'div',
  props: {
    children: 'Hello World'
  },
  key: null,
  ref: null
}
```

<br/>

## 2. Props로 ref를 사용하지 못하는 이유?

`ref`는 이미 React에서 **특별히 예약된 속성**으로, 특정 DOM 요소나 컴포넌트 인스턴스에 대한 **직접 참조를 제공**하기 위해 존재합니다. React에서 `ref`는 일반적인 데이터 전달이 아닌 **DOM 접근 및 제어를 위한 목적**으로 사용되기 때문에 일반 `props`와는 다르게 취급되기 때문입니다.

`ref`는 기본적으로 DOM 요소나 클래스 컴포넌트에만 연결됩니다. 하지만 함수형 컴포넌트에서 `ref`를 받으려면 React는 특별히 `forwardRef`라는 API를 통해 **안전하게 `ref`를 전달**하도록 합니다.
`forwardRef`는 React에게 **이 함수형 컴포넌트가 `ref`를 받을 준비가 되어 있다**는 신호를 보내며, 이 경우에만 `ref`가 해당 컴포넌트로 전달될 수 있습니다.

### 💡 ref 대신 다른 속성명으로 ref를 전달한다면?

예를 들어, ref 대신 myRef 라는 속성명으로 props를 전달한다면…

```jsx
import React, { useRef } from 'react';

// 자식 컴포넌트
function CustomInput({ myRef }) {
  return <input ref={myRef} placeholder="Enter text here" />;
}

// 부모 컴포넌트
export default function ParentComponent() {
  const inputRef = useRef(null);

  const focusInput = () => {
    if (inputRef.current) {
      inputRef.current.focus();
    }
  };

  return (
    <div>
      {/* myRef라는 이름으로 ref 전달 */}
      <CustomInput myRef={inputRef} />
      <button onClick={focusInput}>Focus the input</button>
    </div>
  );
}
```

이 경우 예약어인 `ref`를 사용하지 않으므로, `forwardRef`를 사용할 필요 없이 바로 전달할 수 있습니다. 이 방식은 간단하고 유용할 수 있지만, 실제로는 `forwardRef`를 사용하는 것과 몇 가지 차이점과 제한이 있습니다. 일반적으로는 **DOM 요소에 직접 접근하는 상황에서는 `forwardRef`를 사용하는 것이 더 안전하고 권장**됩니다. 

`forwardRef`는 React의 기본 최적화 흐름과 일치하게 동작합니다. `ref`는 React가 특별하게 관리하는 예약 속성이기 때문에, 일반 props로 `myRef`라는 이름을 사용하여 전달하면 일부 상황에서 **React의 최적화 흐름과 충돌할 수 있습니다**. 특히 라이브러리나 고급 기능을 사용할 때, React는 컴포넌트가 `ref`로 직접 접근하려는 상황을 예상하고 동작하므로 `forwardRef`를 통해 안전하게 `ref`를 전달하는 것이 좋습니다.

따라서, 웬만해서 `forwardRef()`를 사용하는것이 바람직합니다.

```jsx
import React, { useRef } from 'react';

// forwardRef를 사용하여 ref를 전달받는 CustomInput 컴포넌트
const CustomInput = forwardRef((props, ref) => (
  <input {...props} ref={ref} />
));

// 부모 컴포넌트
export default function ParentComponent() {
  const inputRef = useRef(null);

  const focusInput = () => {
    if (inputRef.current) {
      inputRef.current.focus();
    }
  };

  return (
    <div>
      {/* myRef라는 이름으로 ref 전달 */}
      <CustomInput myRef={inputRef} />
      <button onClick={focusInput}>Focus the input</button>
    </div>
  );
}
```

<br/>

## 3. React에서 State란 무엇일까?

**React에서 state란** 컴포넌트 각각의 **독립적인 데이터 저장공간**입니다. 즉, 컴포넌트가 **동적으로 변경되는 데이터**를 관리할 수 있도록 해주는 기능입니다. `state`는 주로 **사용자 입력, 네트워크 요청, 사용자 인터랙션** 등으로 인해 변할 수 있는 데이터를 다룰 때 사용됩니다.

React에서는 **`useState()` Hook**을 사용하여 `state`를 정의하고 관리합니다. 이 Hook은 현재 상태값과 그 상태를 업데이트할 수 있는 **setter 함수**를 제공합니다. `state`가 변경되면 React는 해당 컴포넌트를 **자동으로 다시 렌더링**하여 UI를 최신 상태로 유지합니다.

### 그럼 React에서 왜 State를 사용해야 할까?

React에서 `state`를 사용하는 이유는 데이터가 변경될 때마다 자동으로 UI를 업데이트하여 최신 상태를 반영하고, 각 컴포넌트가 자신의 `state`를 독립적으로 관리함으로써 다른 컴포넌트에 영향을 주지 않고 안정적으로 동작할 수 있도록 하기 때문입니다. 또한, `state`는 React의 렌더링 최적화 기법과 함께 작동하여 불필요한 재렌더링을 방지하고, 전반적인 성능을 향상시킵니다.

만약, state를 사용하지 않고, 지역 변수만을 사용하여 데이터를 처리하면 어떻게 될까?

당연히 렌더링 처리가 되지 않기 때문에 React 컴포넌트가 업데이트 되지 않을 것이고, 데이터가 변화하더라도 화면에 반영되지 않을 것입니다.

```jsx
import React from 'react';

function CounterWithoutState() {
  let count = 0;

  const handleClick = () => {
    count += 1;
    console.log(count); // 콘솔에는 증가된 값이 보이지만, 화면은 갱신되지 않음
  };

  return (
    <div>
      <p>Current Count: {count}</p>
      <button onClick={handleClick}>Increment</button>
    </div>
  );
}

export default CounterWithoutState;
```

- 버튼을 클릭해도 **화면에 표시된 `count` 값은 변하지 않습니다**. 이는 React가 변수 값의 변화를 **감지하지 못하기 때문**입니다.
- `count`는 단순히 지역 변수이기 때문에 **React의 렌더링 사이클과 연결되지 않습니다**.

따라서, `useState()` hook를 사용하여 데이터가 변경될 때마다 자동으로 UI를 업데이트하여 최신 상태를 반영하도록 해야합니다.

```jsx
import React, { useState } from 'react';

function Counter() {
  // useState를 사용하여 count라는 state를 정의
  const [count, setCount] = useState(0);

  const handleClick = () => {
    setCount(count + 1); // state를 업데이트
  };

  return (
    <div>
      <p>Current Count: {count}</p>
      <button onClick={handleClick}>Increment</button>
    </div>
  );
}

export default Counter;
```

<br/>

## 4. React의 렌더링은 어떻게 동작할까?

리액트에서는 최초 렌더링이 발생한 후 리렌더링이 발생하게됩니다.

리렌더링은 발생하는 요인은 아래와 같습니다:

- 클래스형 컴포넌트의 `setState`가 실행되는 경우
- 클래스형 컴포넌트의 `forceUpdate`가 실행되는 경우
- 함수형 컴포넌트의 `useState()`의 두번째 배열요소인 setter가 실행되는 경우
- 함수형 컴포넌트의 `useReducer()`의 두번째 배열요소인 dispatch가 실행되는 경우
- 컴포넌트의 `key props`가 변경되는 경우
- `props`가 변경되는 경우
- 부모 컴포넌트가 렌더링되는 경우

### 리액트의 렌더링 과정

1 ) 렌더링이 시작되면 리액트 컴포넌트는 루트부터 아래로 가면서 업데이트가 필요한 컴포넌트를 탐색합니다.

2 ) 업데이트가 필요한 컴포넌트를 만나면 클래스 컴포넌트는 클래스 내부의 `render()` 함수를 실행하고, 함수형 컴포넌트는 `FunctionComponent()` 그 자체를 호출하고 그 결과물을 저장합니다.

3 ) 렌더링 결과물은 JSX문법으로 구성되며, 자바스크립트로 컴파일되면서 `React.createElement()` 함수를 호출하는 구문으로 변환됩니다. 이때, `createElement()`는 자바스크립트 객체를 반환합니다.

4 ) React는 전체 컴포넌트 트리에서 렌더 결과물을 수집한 후 새로운 **Virtual DOM** 트리와 이전 Virtual DOM 트리를 비교하고, 실제 DOM에 적용해야 할 변경 사항을 계산하는 **재조정(Reconciliation)** 과정을 수행합니다. React는 이렇게 계산된 모든 변경 사항을 하나의 **동기적 시퀀스(Batch Update)**로 실제 DOM에 적용합니다.

### 렌더와 커밋 단계

렌더링의 과정은 개념적으로 크게 2가지 단계로 나뉘게됩니다.

- **렌더 단계 : 컴포넌트를 렌더링하고 변경 사항을 계산하는 모든 과정이 이루어지는 단계**
- **커밋단계 : 변경 사항을 실제 DOM에 적용하는 단계**

렌더 단계에서는 `type`, `props`, `key`의 변경을 감지하여 가상 DOM 트리에서 변경 사항을 계산하고, 새로운 가상 DOM과 이전 가상 DOM을 비교하여 업데이트가 필요한 부분을 식별합니다. 이 단계는 비동기적으로 진행되어 실제 DOM에 직접 영향을 미치지 않고, 변경 사항을 효율적으로 계산하는 데 집중합니다. 이후 커밋 단계에서는 렌더 단계에서 준비된 변경 사항을 실제 DOM에 적용하고, `componentDidMount` 및 `componentDidUpdate`와 같은 라이프사이클 메서드와 `useEffect`와 같은 부수 효과가 실행됩니다. 이 단계는 동기적으로 수행되며, 최종적으로 변경된 내용이 실제 화면에 반영됩니다. 이러한 구조 덕분에 React는 효율적인 UI 업데이트와 최적화된 렌더링을 제공합니다.

<br/>

## 5. Virtual DOM은 어떻게 동작할까?

**가상 DOM(Virtual DOM)** 은 효율적인 렌더링을 위해 실제 **DOM(Document Object Model)** 을 추상화한 JavaScript 객체입니다. 가상 DOM은 UI를 업데이트할 때 전체 DOM을 다시 렌더링하지 않고, 변화된 부분만 빠르게 찾아내어 실제 DOM에 최소한의 수정만 가하도록 돕는 역할을 합니다. 

### Virtual DOM 등장배경

브라우저에서 DOM을 조작하는 작업은 상대적으로 무겁고 성능에 부담을 주는 작업입니다.  DOM을 빈번히 변경하는 작업이 많아질수록 웹 애플리케이션의 성능이 저하됩니다.  React는 특히 SPA으로 이런 DOM 조작 작업을 관리하는것은 무겁고 성능에 부담을 주는 작업입니다.

현대 웹 애플리케이션에서는 대화형 UI가 많이 사용되면서, 실시간으로 화면을 갱신해야 하는 경우가 많습니다. 예를 들어, 소셜 미디어의 실시간 피드, 동적인 데이터 대시보드 등이 이에 해당합니다.

하지만, 각 상태 변화에 따라 실제 DOM을 매번 업데이트하면 사용자 인터페이스가 매끄럽지 않거나 반응이 느려질 수 있습니다.

**React 팀은 이러한 문제를 해결하기 위해 DOM을 직접 업데이트하지 않고, 가상의 DOM을 통해 중간 단계에서 변경 사항을 빠르게 계산한 후, 최소한의 변경만 실제 DOM에 반영하는 방식을 고안했습니다.**

### Virtual DOM의 동작과정

![1_sWfjl94Bnshi1kewFCL0gg.png](https://velog.velcdn.com/images/njt6419/post/4be14969-2eb1-4959-b402-6cdad9c9fc47/image.png)

리액트는 항상 2개의 Virtual DOM를 가지고 있습니다.

**1 ) 렌더링 이전 화면 구조를 나타내는 Virtual DOM**

**2 ) 렌더링 이후에 보이게될 화면 구조를 나타내는 Virtual DOM**

리액트는 실제 브라우저 화면에 그려지기 이전 렌더링이 발생될 상황이 되면 새로운 화면에 들어갈 내용이 담긴 가상돔을 생성합니다.

렌더링 이전 화면 구조를 나타내는 Virtual DOM과 업데이트 이후 화면 구조를 나타내는 Virtual DOM를 비교하여 정확히 어떤 부분이 변경되었는지 효율적으로 비교하는 **Diffing**을 통해 변경된 부분들을 파악하고 **Batch Update**를 통해 변경된 부분만을 실제 DOM에 한번에 반영합니다.

이런 리액트의 Virtual DOM 조작과정을 **Reconciliation(재조정)** 이라고 합니다.

리액트의 재조정 과정은  **BatchUpdate** 통해 효율적으로 진행됩니다.

리액트는 **Batch Update** 통해 변경된 모든 state를 실제 DOM에 한번에 적용시켜주기 때문입니다.

### 💡 React Fiber란?

**리액트 파이버(React Fiber)** 는 React 16에서 도입된 **재조정(reconciliation)** 엔진으로, 기존의 재조정 과정을 개선하여 UI 업데이트를 더 세밀하게 제어할 수 있게 해줍니다.

파이버는 파이버 재조정자가 관리하며, 가상 DOM과 실제 DOM을 비교해 변경 사항을 수집하며, 이 둘의 차이점을 변경에 관련된 정보를 가지고 있는 파이버를 기준으로 화면에 렌더링을 요청하는 역할을합니다.

**React Fiber의 구성요소**

- **Fiber Node**: 가상 DOM 트리의 각 노드는 Fiber로 표현되며, 기존의 가상 DOM 노드와 유사한 구조를 가지고 있습니다. 각 Fiber는 컴포넌트의 타입(type), 속성(props), 상태(state), 노드의 부모, 자식 및 형제에 대한 참조를 포함합니다.
- **우선순위(Priority)**: Fiber는 각 작업에 대해 우선순위를 설정할 수 있어 더 중요한 작업을 먼저 수행할 수 있습니다. 이를 통해 애니메이션, 스크롤 등 사용자 인터페이스와 관련된 작업을 보다 빠르게 반영할 수 있습니다.
- **알고리즘 개선**: React는 Fiber를 사용해 작업을 끊어서 수행하는 `cooperative scheduling` 방식을 채택하여, 긴 작업을 여러 프레임에 나누어 수행하게 됩니다. 이를 통해 프레임 손실 없이 매끄럽게 렌더링할 수 있습니다.

**Fiber가 수행하는 작업**

- **분할 작업(Interruptible Work)**: Fiber는 작업을 여러 작은 단위로 분할하고, 브라우저가 사용자 이벤트를 처리할 수 있도록 중간에 작업을 일시 중단하고 다시 이어서 실행할 수 있습니다.
- **병합(Merge) 및 재사용**: Fiber는 변화가 발생한 트리의 하위 노드만 다시 렌더링하고, 재사용할 수 있는 노드는 그대로 유지합니다.
- **우선순위 기반 업데이트**: 애니메이션과 같은 중요한 업데이트는 우선순위를 높게 설정해 즉각 반영하고, 덜 중요한 업데이트는 낮은 우선순위로 미뤄 성능을 최적화합니다.

React는 부모 컴포넌트가 자식 컴포넌트를 렌더링할 때, 해당 컴포넌트를 추적하기 위한 Fiber 객체를 생성합니다. **클래스형 컴포넌트**의 경우, React는 `new YourComponentType(props)`를 호출하여 컴포넌트 인스턴스를 생성하고 이를 Fiber에 저장합니다. 여기서 `this.props`는 사실 Fiber에 저장된 props의 참조를 복사한 것입니다. 반면, **함수형 컴포넌트와 Hooks**의 경우에는 컴포넌트 인스턴스 대신 `YourComponentType(props)`를 호출하여 렌더링합니다. 함수형 컴포넌트에서 사용하는 모든 훅은 Fiber 객체의 연결 리스트 형태로 저장됩니다. React가 함수형 컴포넌트를 렌더링할 때마다 해당 Fiber에서 훅 연결 리스트를 참조하며, 호출된 순서에 따라 저장된 상태 값이나 `useReducer`의 dispatch 함수를 반환합니다.

위에서 설명한 것 처럼 렌더링은 렌더와 커밋 2가지 단계로 나누어집니다. Fiber를 통해 자세히 설명하자면, 렌더 단계에서는 Fiber를 순회하며 가상 DOM의 변경 사항을 계산합니다. 이 단계에서는 실제 DOM을 변경하지 않으며, 필요에 따라 작업을 중단하거나 이어서 수행할 수 있습니다. 이후 커밋 단계에서는 Fiber에서 준비된 변경 사항을 바탕으로 실제 DOM을 업데이트합니다. 이 단계는 동기적으로 수행되어 최종 변경 사항이 화면에 반영됩니다. Fiber 시스템 덕분에 React는 복잡한 UI를 효율적으로 렌더링하고, 동적이고 상호작용이 많은 애플리케이션에서도 부드러운 사용자 경험을 제공할 수 있습니다.

**리액트 내부 코드의 파이버 객체 예시**

```jsx
function FiberNode(tag, pendingProps, key, mode) {
  // 기본 속성 초기화
  this.tag = tag;                    // 노드의 유형을 나타내는 태그
  this.key = key;                    // 리액트의 고유 키
  this.elementType = null;           // JSX 요소 타입 (예: <div>면 "div")
  this.type = null;                  // 컴포넌트 함수 또는 클래스 인스턴스
  this.stateNode = null;             // DOM 노드 또는 컴포넌트 인스턴스
  this.mode = mode;                  // 리액트 모드 (동기/비동기 여부)

  // Props & State
  this.pendingProps = pendingProps;  // 새로 전달된 props
  this.memoizedProps = null;         // 마지막 렌더링에 사용된 props
  this.memoizedState = null;         // 마지막 렌더링에 사용된 상태
  this.updateQueue = null;           // 상태 업데이트를 관리하는 큐

  // 트리 구조
  this.return = null;                // 부모 파이버 노드
  this.child = null;                 // 첫 번째 자식 파이버 노드
  this.sibling = null;               // 다음 형제 파이버 노드
  this.index = 0;                    // 자식 노드에서의 인덱스

  // 효과 및 우선순위 관리
  this.ref = null;                   // ref 속성 참조
  this.effectTag = 0;                // 이 노드에서 필요한 작업(플래그)
  this.nextEffect = null;            // 다음으로 처리할 효과 노드
  this.firstEffect = null;           // 첫 번째 효과 노드
  this.lastEffect = null;            // 마지막 효과 노드

  // 상위 컨텍스트
  this.alternate = null;             // 현재 파이버의 대체 파이버(더블 버퍼링)
  this.flags = 0;                    // 여러 상태를 나타내는 플래그
}
```
파이버는 리액트 요소와 유사하다고 느낄 수 있지만 중요한 차이점은 리액트 요소는 렌더링이 발생할 때마다 새롭게 생성되는 반면 파이버는 가급적 재사용됩니다. 파이버는 컴포넌트가 최초 마운트되는 시점에 생성되어 이후에는 가급적이면 재사용됩니다.

### 리액트 파이버 트리

![image (5).png](https://velog.velcdn.com/images/njt6419/post/507869fb-d2c8-4f13-be63-0cdc6718fd38/image.png)

파이버 트리는 React에서 2가지가 존재합니다. 하나는 현재 모습을 담은 파이버 트리, 다른 하나는 작업 중인 상태를 나타내는 **workInProgress** 트리입니다. 리액트 파이버의 작업이 끝나면 리액트는 단순히 포인터만 변경하여 **workInprogress** 트리를 현재 트리로 변경합니다. 이러한 기술을 **더블 버퍼링**이라고 합니다.

리액트에서 **더블 버퍼링**은 커밋 단계에서 수행되게됩니다. 먼저 현재 UI 렌더링을 위해 존재하는 트리인 current기준으로 모든 작업이 시작됩니다. 여기에서 업데이트가 발생하면 파이버는 리액트에서 새로 받은 데이터로 새로운 **workInProgress** 트리를 빌드하기 시작합니다. **workInprogress** 트리를 빌드하는 작업이 끝나면 다음 렌더링에 이 트리를 사용하게됩니다. 이 **workInprogress** 트리가 UI에 최종적으로 렌더링되어 반영이 완료되면 **current**가 이 **workInprogress**로 변경됩니다.

💡 **더블 버퍼링**

**더블 버퍼링**은 리액트에서 새롭게 나온 개념이 아니며, 컴퓨터 그래픽 분야에서 사용하는 용어입니다. 그래픽을 통해 화면을 표시되는 것을 그리기 위해서는 내부적으로 처리를 거쳐야 하는데, 이러한 처리를 거치게 되면 사용자에게 미처 다 그리지 못한 모습을 보이는 경우가 발생하게됩니다. 그래서 이러한 상황을 방지하기 위해 보이지 않는 곳에서 그다음으로 그려야 할그림을 미리 그린 다음, 이것이 완성되면 현재 상태를 새로운그림으로 바꾸는 기법을 의미합니다.

### 파이버의 작업 순서 예시

```html
<A1>
  <B1>안녕하세요</B1>
  <B2>
    <C1>
      <D1/>
      <D2/>
    </C1>
  </B2>
  <B3/>
</A1>
```

위 파이버의 작업은 JSX코드에서 아래와 같이 수행됩니다:

**1. A1의 beginWork()가 수행됩니다.**

**2. A1은 자식이 있으므로 B1로 이동해 beginWork()를 수행합니다.**

**3. B1은 자식이 없으므로 completeWork()가 수행됐다. 자식은 없으므로 형제(sibling)인 B2로 넘어갑니다.**

**4. B2의 beginWork()가 수행된다. 자식이 있으므로 C1로 이동합니다.**

**5. C1의 beginWork()가 수행된다. 자식이 있으므로 D1로 이동합니다.**

**6. D1의 beginWork()가 수행되고, 자식이 없으므로 형제(sibling)인 D2로 넘어갑니다.**

**7. D2의 beginWork()가 수행되고, 자식이 없으므로 completeWork()가 수행됩니다.**

**8. D2는 자식도 형제도 없으므로, 위로 이동해 C1, B2 순으로 completeWork()를 호출합니다.**

**9. B2는 형제(sibling)인 B3으로 이동해 beginWork()를 수행합니다.**

**10. B3의 completeWork()가 수행되면 반환해 상위로 타고 올라갑니다.**

**11. A1의 completeWork()가 수행됩니다.**

**12. 루트 노드가 완성되면 commitWork()가 수행되고 이 중 변경 사항을 비교해 업데이트가 필요한 변경사항이 DOM에 반영됩니다.**

위 파이버 트리를 도식화 하면 아래와 같습니다.

![2024-03-09T08_16_51.webp](https://velog.velcdn.com/images/njt6419/post/7ae13a52-21d2-45a1-9a98-5215814199e8/image.webp)

<br/>

## 6. useEffect는 어떻게 동작할까?

`useEffect` hook은 컴포넌트에서 **부수 효과(side effect)**를 관리하기 위한 훅입니다.

`useEffect`는 클래스형 컴포넌트의 `componentDidMount`, `componentDidUpdate`, `componentWillUnmount`의 기능을 합친 훅으로 이해할 수 있습니다.

### 내부 동작 원리

`useEffect`는 기본적으로 **렌더링 이후 비동기적으로 실행**됩니다. 즉, React가 렌더링을 완료하고 화면에 변경사항을 반영한 후 `useEffect`가 실행됩니다. 이를 통해 렌더링 성능에 영향을 주지 않고 부수 효과를 처리할 수 있습니다.

React는 `useEffect`를 두 가지 경우에 실행합니다:

- **첫 번째 렌더링 시**: 의존성 배열이 `[]`일 경우에만 `useEffect`가 첫 번째 렌더링 후에 한 번 실행됩니다.
- **의존성 값이 변경될 때**: 의존성 배열이 주어진 경우, 배열 내의 값 중 하나라도 변경되면 `useEffect`가 다시 실행됩니다.

`useEffect`는 내부적으로 **의존성 배열을 저장하고 비교**하여 효과를 실행할지를 결정합니다. 이때 React는 다음과 같은 방식으로 의존성 배열을 처리합니다:

- `useEffect`가 호출될 때마다 React는 내부적으로 의존성 배열을 기록합니다.
- 이후 렌더링 시 `useEffect`가 다시 호출되면 이전에 저장된 의존성 배열과 현재의 의존성 배열을 비교합니다. 이때, `Object.is` 를 기반으로 얕은 비교를 수행하여 의존성 배열을 비교합니다.
- **배열 내의 값이 변경되었는지**를 확인하여, 변경된 경우에만 `useEffect`의 콜백 함수를 재실행합니다.

`useEffect`는 **클린업 함수를 반환**할 수 있으며, 이 함수는 다음 상황에서 호출됩니다:

- 컴포넌트가 언마운트될 때: 주로 이벤트 리스너 제거, 타이머 해제 등 리소스를 해제할 때 사용합니다.
- `useEffect`가 다시 실행되기 직전에: 이전 `useEffect`의 효과를 제거하고 새롭게 실행하기 위해 호출됩니다.

### 동작 방식과 용도

- **컴포넌트가 마운트될 때** (`componentDidMount`): 의존성 배열이 빈 배열 `[]`이면 `useEffect`는 컴포넌트가 마운트될 때 한 번만 실행됩니다.
    
    ```jsx
    useEffect(() => {
      console.log("컴포넌트가 처음 마운트되었습니다.");
    }, []);
    ```
    
- **특정 값이 변경될 때** (`componentDidUpdate`): 의존성 배열에 값이 포함된 경우, 해당 값이 변경될 때마다 `useEffect`가 실행됩니다.
    
    ```jsx
    useEffect(() => {
      console.log("count가 변경되었습니다.", count);
    }, [count]);
    ```
    
    위 코드에서는 `count`가 변경될 때마다 `useEffect`가 실행됩니다.
    
- **클린업 함수** (`componentWillUnmount`): `useEffect` 안에서 함수가 반환되면, 컴포넌트가 언마운트될 때 또는 다음 렌더링 전에 실행됩니다. 이를 **클린업 함수**라고 부르며, 타이머 정리, 구독 해제 등 리소스를 정리할 때 주로 사용합니다.
    
    ```jsx
    useEffect(() => {
      const timer = setInterval(() => {
        console.log("타이머 실행 중");
      }, 1000);
    
      // 클린업 함수
      return () => {
        clearInterval(timer);
        console.log("타이머가 정리되었습니다.");
      };
    }, []);
    ```
    
### 💡 useEffect (부수 효과)사용을 최소화 해야하는 이유

`useEffect`는 렌더링이 완료된 후에 비동기적으로 실행됩니다. 컴포넌트 상태나 의존성 값이 변경될 때마다 `useEffect`는 렌더링 이후에 한 번씩 실행되므로, 의존성 값이 자주 변경될 경우 `useEffect`가 빈번하게 실행될 수 있습니다. 이로 인해 다음과 같은 성능 문제가 발생할 수 있습니다:

- **불필요한 연산 발생**: 매 렌더링마다 부수 효과가 다시 실행되면 불필요한 데이터 요청이나 상태 계산이 발생할 수 있습니다.
- **리렌더링 증가**: `useEffect` 내에서 상태를 설정하거나 다른 상태를 업데이트하는 경우, 추가 리렌더링이 발생하여 성능 저하로 이어질 수 있습니다.

### 💡 useEffect의 의존성 배열을 ESLint 규칙과 일치시켜야 하는 이유

의존성 배열에 **필요한 모든 변수를 포함**해야 `useEffect`가 의도한 대로만 실행됩니다. 만약 필요한 변수를 의존성 배열에 포함하지 않으면, `useEffect` 내부에서 사용하는 값이 예상치 못하게 변경되더라도 `useEffect`가 다시 실행되지 않아 부작용이 발생할 수 있습니다.

useEffect의 의존성은 선택적으로 사용하는 것이 아닌 side Effect의 코드에서 사용되는 모든 반응형 값은 의존성 목록에 선언되어야합니다.

<br/>

## 7. useMemo, useCallback, React.memo는 어떻게 동작할까?

### useMemo : 값 메모이제이션

`useMemo`는 계산이 **비용이 큰 값**을 메모이제이션하여, 의존성이 변경되지 않는 한 동일한 값을 반환하게 합니다. `useMemo`를 사용하면 매번 렌더링 때마다 값을 다시 계산하지 않고 **이전에 계산한 값을 재사용**하므로 성능이 향상됩니다.

**useMemo 내부 구현 코드**

```jsx
export function useMemo<T>(
  create: () => T,
  deps: Array<mixed> | void | null,
): T {
  const dispatcher = resolveDispatcher();
  return dispatcher.useMemo(create, deps);
}
```

- `useMemo`는 `create` 함수와 `deps` 배열을 인자로 받습니다.
- 실제 구현은 `dispatcher.useMemo`로 위임됩니다.
- `resolveDispatcher` 함수는 현재 React의 렌더링 단계에 따라 적절한 dispatcher를 반환합니다.

```jsx
function resolveDispatcher() {// Hooks에서 호출하는 함수 코드.
  const dispatcher = ReactCurrentDispatcher.current; // ReactCurrentDispatcher의 current를 본다.
  if (__DEV__) { // 리액트의 경우 개발모드에서만 에러를 출력하고 프로덕트에서 최적화 하는 부분이 존재한다.
    if (dispatcher === null) { // 잘못된 호출에 대한 에러 출력.
      console.error(
        'Invalid hook call. Hooks can only be called inside of the body of a function component. This could happen for' +
          ' one of the following reasons:\n' +
          '1. You might have mismatching versions of React and the renderer (such as React DOM)\n' +
          '2. You might be breaking the Rules of Hooks\n' +
          '3. You might have more than one copy of React in the same app\n' +
          'See https://reactjs.org/link/invalid-hook-call for tips about how to debug and fix this problem.',
      );
    }
  }
  // Will result in a null access error if accessed outside render phase. We
  // intentionally don't throw our own error because this is in a hot path.
  // Also helps ensure this is inlined.
  return ((dispatcher: any): Dispatcher);
}
```

- **`ReactCurrentDispatcher` 객체**
    - `ReactCurrentDispatcher`는 React의 내부 객체로, `current` 속성을 통해 **현재 활성화된 디스패처**를 참조합니다.
    - 이 `current`는 React의 렌더링 단계에 따라 **마운트 단계**에서는 `mountDispatcher`, **업데이트 단계**에서는 `updateDispatcher`로 설정됩니다.
- **`resolveDispatcher` 함수**
    - `resolveDispatcher`는 `ReactCurrentDispatcher.current`를 읽어, 현재 활성화된 디스패처를 가져옵니다.
    - 만약 `current`가 `null`이라면, React는 컴포넌트 외부에서 훅을 호출했거나 아직 디스패처가 초기화되지 않은 경우입니다. 이때 에러를 발생시켜 **훅을 잘못된 위치에서 호출하지 않도록 보호**합니다.
    - 정상적으로 `current`에 값이 있다면, 해당 디스패처를 반환합니다.
    

**dispatcher**

```jsx
function mountMemo<T>(
  nextCreate: () => T,
  deps: Array<mixed> | void | null,
): T {
  const hook = mountWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  const nextValue = nextCreate();
  hook.memoizedState = [nextValue, nextDeps];
  return nextValue;
}

function updateMemo<T>(
  nextCreate: () => T,
  deps: Array<mixed> | void | null,
): T {
  const hook = updateWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  const prevState = hook.memoizedState;
  if (prevState !== null) {
    if (nextDeps !== null) {
      const prevDeps: Array<mixed> | null = prevState[1];
      if (areHookInputsEqual(nextDeps, prevDeps)) {
        return prevState[0];
      }
    }
  }
  const nextValue = nextCreate();
  hook.memoizedState = [nextValue, nextDeps];
  return nextValue;
}
```

- **`mountMemo`**: 처음 `useMemo`가 호출될 때 실행되는 함수입니다.
    - `mountWorkInProgressHook()`을 호출하여 현재 훅의 상태를 설정할 공간을 가져옵니다.
    - `nextCreate()`를 호출하여 메모이제이션할 **값을 계산**하고, 이를 `memoizedState`에 저장합니다.
    - 의존성 배열(`deps`)이 전달되면 이를 함께 저장하여, 이후 렌더링 시 변경 여부를 확인할 수 있습니다.
- **`updateMemo`**: 이미 마운트된 `useMemo` 훅이 업데이트될 때 호출됩니다.
    - `updateWorkInProgressHook()`을 통해 현재 훅의 상태에 접근합니다.
    - `memoizedState`에서 **이전 값과 이전 의존성 배열**을 가져옵니다.
    - `areHookInputsEqual`을 사용하여 **이전 의존성과 현재 의존성을 비교**합니다. 의존성이 같으면 이전에 계산된 값을 반환하여 불필요한 재계산을 방지합니다.
    - 만약 의존성이 달라졌다면, `nextCreate()`를 호출하여 새 값을 계산하고, `memoizedState`에 새로운 값과 의존성 배열을 저장합니다.
    

의존성 배열은 `useMemo`의 두 번째 인자로 전달되며, 이 배열의 값들이 변경될 때만 메모이제이션된 값을 재계산합니다. React는 이전 렌더링의 의존성 값들과 현재 렌더링의 값들을 비교합니다:
내부적으로는 `Object.is`를 사용하여 비교합니다.

```jsx
function areHookInputsEqual(
  nextDeps: Array<mixed>,
  prevDeps: Array<mixed> | null,
) {
  if (prevDeps === null) {
    return false;
  }
  for (let i = 0; i < prevDeps.length && i < nextDeps.length; i++) {
    if (Object.is(nextDeps[i], prevDeps[i])) {
      continue;
    }
    return false;
  }
  return true;
}

```

`Object.is`를 사용하여 각 의존성을 비교합니다. 모든 의존성이 동일하면 `true`를 반환하여 메모이제이션된 값을 재사용합니다.

**동작 원리 정리**

```jsx
const memoizedValue = useMemo(() => {
  return computeExpensiveValue(a, b);
}, [a, b]);
```

- React는 `useMemo` 내부의 계산이 최초 렌더링에서 한 번 수행된 후 결과 값을 **캐시**에 저장합니다.
- 이후 리렌더링 시 `useMemo`는 의존성 배열을 확인하여, `a`나 `b` 값이 변경되지 않으면 **캐시된 값**을 반환합니다.
- 의존성 배열의 값이 변경되면 `useMemo`는 내부의 계산을 다시 실행하고, 결과 값을 업데이트하여 캐시에 저장합니다.

### useCallback : 함수 메모이제이션

`useCallback`은 **특정 함수의 메모이제이션**을 수행하여, 함수가 불필요하게 재생성되지 않도록 합니다. 이 훅은 **콜백 함수가 동일한 참조값**을 가지도록 유지하고, 의존성이 변경되지 않는 한 **이전의 함수 참조를 재사용**합니다. 주로 `props`로 자식 컴포넌트에 함수를 전달할 때 사용합니다.

```jsx
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

**useCallback 내부 구현 코드**

```jsx
function useCallback<T extends (...args: any[]) => any>(
  callback: T,
  deps: Array<any> | void | null,
): T {
  return useMemo(() => callback, deps);
}
```

- **`useMemo` 호출**
    - `useCallback`은 `useMemo`를 통해 내부적으로 동일한 함수 참조를 관리합니다.
    - `useMemo`의 첫 번째 인자로 `() => callback`을 전달하여 **callback 함수 자체**가 아닌, **함수를 반환하는 래퍼 함수**를 사용합니다.
    - 이렇게 함으로써, `callback` 함수의 참조가 유지되며 `useCallback`이 함수 메모이제이션 훅으로 작동하게 됩니다.
- **의존성 배열에 따라 메모이제이션**
    - `deps` 배열이 전달되면, `useMemo`는 이 배열의 값이 변경될 때에만 새 함수를 생성합니다.
    - `deps` 배열 내 값이 변하지 않으면 `useMemo`는 이전에 메모이제이션된 함수 참조를 반환하여, `callback` 함수가 불필요하게 재생성되지 않도록 합니다.
    - 

**`useCallback`이 `useMemo`를 래핑하는 이유?**

`useCallback`은 함수 자체를 메모이제이션하는 것이기 때문에 `useMemo`를 통해 동일한 의존성 배열 기반 메모이제이션 원리를 활용합니다. `useMemo`는 원래 **계산된 값**을 반환하는 데 목적이 있지만, `useCallback`은 이 계산된 값이 **함수 참조**가 되도록 응용하여 메모이제이션을 실현합니다.

**동작 원리 정리**

- React는 `useCallback` 내부의 함수를 캐시에 저장하고, 의존성 배열을 확인하여 해당 함수가 필요할 때 **이전 함수를 재사용**합니다.
- 의존성이 변경되지 않으면 기존 함수를 재사용하고, 변경되면 새로운 함수를 생성합니다.

### React.memo : 컴포넌트 메모이제이션

`React.memo`는 컴포넌트의 불필요한 렌더링을 방지하기 위해 사용되는 고차 컴포넌트(Higher Order Component, HOC)입니다. `React.memo`는 **컴포넌트를 메모이제이션**하여, 해당 컴포넌트의 `props`가 변경되지 않는 한 **재렌더링을 하지 않도록** 합니다. `React.memo`로 감싸진 컴포넌트는 이전 `props`와 새로운 `props`를 비교하여 동일한 경우 컴포넌트를 재사용합니다.

```jsx
const MyComponent = React.memo(Component, [areEqual]);
```

**React.memo 내부 구현 코드**

React 내부에서 `React.memo`의 실제 구현 코드는 더 복잡합니다.

```jsx
function memo(type, compare) {
  const elementType = {
    $$typeof: REACT_MEMO_TYPE,
    type,
    compare: compare === undefined ? null : compare,
  };
  return elementType;
}
```

- `$$typeof`: React 내부에서 사용하는 **고유한 식별자**로, 이 경우 `REACT_MEMO_TYPE`을 사용하여 메모이제이션된 컴포넌트를 구분합니다.
- `type`: 실제 렌더링할 원래 컴포넌트입니다.
- `compare`: props 비교 함수로, 주어지지 않으면 기본적으로 `null`입니다.

이렇게 반환된 `elementType`은 React의 **Reconciler(재조정자)** 에서 처리됩니다. `memo`를 사용한 컴포넌트는 React가 내부적으로 props 비교를 수행하여 렌더링 최적화를 합니다.

**동작 원리 정리**

- `React.memo`는 기본적으로 얕은 비교(Shallow Comparison)를 통해 **이전 `props`와 새로운 `props`가 같은지**를 판단합니다.
- 두 객체가 같다면 재렌더링을 하지 않고 이전 컴포넌트를 그대로 사용합니다.
- 다르면 컴포넌트를 다시 렌더링합니다.

<br/>

## 8. React에서 상태관리 라이브러리가 왜 필요할까?

  React는 기본적으로 **컴포넌트 단위**로 상태를 관리합니다. 하지만 프로젝트 규모가 커지거나 복잡도가 증가하면서 React의 기본 상태 관리만으로는 한계에 부딪히게 됩니다.

### 1 ) **다수의 컴포넌트 간 상태 공유**

- 서로 다른 컴포넌트들이 **동일한 상태**를 필요로 하는 경우, 해당 상태를 최상위 부모 컴포넌트로 끌어올려야 합니다(Prop Drilling).
- 예를 들어, `Navbar`, `Sidebar`, `Content`가 모두 **로그인 상태**를 알아야 한다면, 이를 최상위 컴포넌트에서 관리하고 **props로 전달**해야 합니다.
- 이 경우, 상태를 **중간 컴포넌트**를 거쳐 전달해야 하는 문제가 발생할 수 있습니다.

### 2 ) **Prop Drilling 문제**

- 특정 상태를 자식 컴포넌트로 전달하기 위해 불필요하게 여러 단계의 컴포넌트에 **props**를 넘겨줘야 하는 상황입니다.
- 이로 인해 코드가 지저분해지고, 컴포넌트 간의 **결합도**가 높아집니다.

### 3 ) **전역 상태(Global State)의 필요성**

- 다수의 페이지 또는 모듈에서 **공통으로 접근해야 하는 상태**가 있는 경우(예: 사용자 인증 정보, 테마 설정 등).
- 전역 상태를 쉽게 관리하지 않으면 코드가 복잡해지고 오류가 발생하기 쉽습니다.

### 4 ) **복잡한 비동기 로직**

- `useEffect`와 같은 React 훅을 사용하여 **API 호출**이나 **비동기 작업**을 관리할 수 있지만, 상태와 관련된 여러 비동기 로직이 복잡하게 얽히면 관리하기 어려워집니다.
- 예를 들어, 여러 API 요청의 상태(로딩, 성공, 실패 등)를 개별적으로 관리하는 것은 까다롭습니다.

### 💡 언제 상태 관리 라이브러리를 도입해야 하는가?

무조건 상태 관리 라이브러리를 도입하는 것은 좋지 않습니다. 오히려 상태 관리의 복잡성 증대, 성능 저하, 유지 보수 어려움 등의 문제를 초래할 수 있습니다. 복잡하지 않은 프로젝트인 경우 React에서 기본적으로 제공하는 상태관리 도구 useState, useReducer, contextAPI 등을 사용하는 것이 좋습니다.

상태 관리 라이브러리는 아래와 같은 상황에서 도입하는것이 좋습니다:

- **컴포넌트 간 상태 공유**가 빈번한 경우
- **전역 상태**(예: 사용자 정보, 테마 설정)가 필요할 때
- **비동기 데이터 페칭**과 상태 관리가 복잡할 때
- Prop Drilling 문제가 코드의 가독성을 떨어뜨릴 때

상태 관리 라이브러리를 도입할 때는 정말 필요한 라이브러리인지 아님 React에서 제공하는 상태 관리 도구를 통해 충분히 해결 가능한지 생각해보는 것이 중요합니다.

<br/>

## 9. React Hooks의 기본 동작원리는 무엇일까?

React Hooks는 React 내부에서 **클로저(closure)** 와 **배열**을 사용하여 상태와 함수 호출을 관리합니다. Hooks는 함수형 컴포넌트가 렌더링될 때마다 **순차적으로 호출**되며, React는 이를 통해 각 컴포넌트의 상태를 추적합니다.

### 기본 개념

- React는 **각 컴포넌트마다 고유한 메모리 공간**을 사용하여 상태를 저장합니다.
- `useState`나 `useEffect` 같은 Hook은 **렌더링 순서**에 따라 호출되므로, React는 호출된 순서대로 상태를 추적하고 관리합니다.
- Hook의 호출 순서가 바뀌지 않는다는 전제하에, React는 **Hook이 호출된 순서대로 상태를 매핑**합니다.

**React Hooks 일부 소스 코드** 

```jsx
export function useState<S>(
  initialState: (() => S) | S, // 초기 상태값 타입 콜백 함수 방식과 일반 방식 두가지.
): [S, Dispatch<BasicStateAction<S>>] { 
// 반환하는 타입 S는 상태 Dispatch<action<상태>>는 setState
  const dispatcher = resolveDispatcher(); // dispatcher라는 resolveDispatcher 함수 반환값을 받고
  return dispatcher.useState(initialState); // 거기서 useState라는 메서드(함수를 실행해서 상태를 넣어준다.)
}
export function useReducer<S, I, A>(
  reducer: (S, A) => S,
  initialArg: I,
  init?: I => S,
): [S, Dispatch<A>] {
  const dispatcher = resolveDispatcher();
  return dispatcher.useReducer(reducer, initialArg, init);
}

export function useRef<T>(initialValue: T): {|current: T|} {
  const dispatcher = resolveDispatcher();
  return dispatcher.useRef(initialValue);
}

export function useEffect(
  create: () => (() => void) | void,
  deps: Array<mixed> | void | null,
): void {
  const dispatcher = resolveDispatcher();
  return dispatcher.useEffect(create, deps);
}

export function useInsertionEffect(
  create: () => (() => void) | void,
  deps: Array<mixed> | void | null,
): void {
  const dispatcher = resolveDispatcher();
  return dispatcher.useInsertionEffect(create, deps);
}

export function useLayoutEffect(
  create: () => (() => void) | void,
  deps: Array<mixed> | void | null,
): void {
  const dispatcher = resolveDispatcher();
  return dispatcher.useLayoutEffect(create, deps);
}
...Hooks 생략...

// resolveDispatcher와 Hooks의 상태에 대한 코드 역시 같은 파일내에 존재한다.
import ReactCurrentDispatcher from './ReactCurrentDispatcher';
// 액션 함수 type과 Dispatch 타입
type BasicStateAction<S> = (S => S) | S;
type Dispatch<A> = A => void;

function resolveDispatcher() {// Hooks에서 호출하는 함수 코드.
  const dispatcher = ReactCurrentDispatcher.current; // ReactCurrentDispatcher의 current를 본다.
  if (__DEV__) { // 리액트의 경우 개발모드에서만 에러를 출력하고 프로덕트에서 최적화 하는 부분이 존재한다.
    if (dispatcher === null) { // 잘못된 호출에 대한 에러 출력.
      console.error(
        'Invalid hook call. Hooks can only be called inside of the body of a function component. This could happen for' +
          ' one of the following reasons:\n' +
          '1. You might have mismatching versions of React and the renderer (such as React DOM)\n' +
          '2. You might be breaking the Rules of Hooks\n' +
          '3. You might have more than one copy of React in the same app\n' +
          'See https://reactjs.org/link/invalid-hook-call for tips about how to debug and fix this problem.',
      );
    }
  }
  // Will result in a null access error if accessed outside render phase. We
  // intentionally don't throw our own error because this is in a hot path.
  // Also helps ensure this is inlined.
  return ((dispatcher: any): Dispatcher);
}
```

### React Hooks의 관리 구조

- **Hook State**: 각 컴포넌트마다 **Hook 상태 배열**이 생성됩니다.
- **렌더링 사이클**: 컴포넌트가 렌더링될 때마다 `useState`, `useEffect` 등 모든 Hook이 **순서대로 호출**됩니다.
- **Hook Index**: React는 현재 컴포넌트 내에서 **몇 번째 Hook이 호출되었는지 추적**합니다.

<br/>

## 10. React가 어떻게 SPA를 구현할까?

리액트에서 SPA를 구현할 때는 **클라이언트 측 라우팅(Client-side Routing)**을 사용합니다. 보통 웹 애플리케이션에서는 URL이 변경될 때마다 서버로 요청을 보내 페이지를 새로 로드하지만, 리액트에서는 `react-router` 같은 라우팅 라이브러리를 활용하여 클라이언트 측에서 라우팅을 처리합니다.

- `react-router`는 페이지 전환이 필요할 때 브라우저의 `history API`를 이용하여 URL을 변경하지만 실제로 서버에 요청을 보내지 않습니다.
- 이를 통해 URL은 변경되지만 페이지 전체가 새로 고쳐지지 않으며, 오직 필요한 컴포넌트만 교체되거나 업데이트됩니다.

### React Router의 동작 원리

React Router는 브라우저의 **History API**(`pushState`, `replaceState`, `popstate` 이벤트 등)를 사용하여 **URL을 변경**합니다. 이렇게 변경된 URL을 감지하고, 해당 URL에 맞는 컴포넌트를 **클라이언트 사이드에서 렌더링**합니다.

**동작 과정**

- **초기 설정**: 애플리케이션을 `<BrowserRouter>`로 감싸면 React Router는 **히스토리 객체**를 생성하고 URL을 추적합니다.
- **라우팅 구성**: `<Routes>` 컴포넌트 내에 여러 개의 `<Route>`를 정의하여 **경로별 컴포넌트**를 지정합니다.
- **URL 변경 감지**: `<Link>`나 `useNavigate`를 통해 URL이 변경되면, React Router는 **전체 페이지를 다시 로드하지 않고** 컴포넌트를 교체합니다.
- **렌더링**: 변경된 URL에 따라 **일치하는 경로의 컴포넌트**를 렌더링합니다.

### React Router의 내부 동작 코드

실제 React Router는 더 복잡한 최적화 및 다양한 기능이 포함되어 있지만, 여기서는 기본적인 작동 방식을 설명합니다.

**`BrowserRouter` 구현 예시**
`BrowserRouter`는 HTML5 **History API**를 사용하여 브라우저의 URL을 제어합니다. 사용자가 브라우저의 뒤로 가기 버튼을 클릭하거나 `navigate` 함수를 통해 경로를 변경할 때, `location` 상태를 업데이트하여 해당 경로에 맞는 컴포넌트를 렌더링합니다.

```jsx
import { useState, useEffect, createContext, useContext } from 'react';

// Router Context 생성 (다른 컴포넌트에서 라우터 상태를 접근할 수 있게 함)
const RouterContext = createContext();

// BrowserRouter 컴포넌트
export function BrowserRouter({ children }) {
  // 현재 경로를 저장하는 상태값
  const [location, setLocation] = useState(window.location.pathname);

  // 브라우저의 뒤로 가기/앞으로 가기 등의 이벤트를 감지하여 경로 변경
  useEffect(() => {
    const handlePopState = () => setLocation(window.location.pathname);
    window.addEventListener('popstate', handlePopState);
    
    // 컴포넌트가 언마운트될 때 이벤트 제거
    return () => window.removeEventListener('popstate', handlePopState);
  }, []);

  // navigate 함수: 브라우저의 URL을 변경하고 상태 업데이트
  const navigate = (path) => {
    window.history.pushState({}, '', path); // 브라우저의 주소창 URL 변경
    setLocation(path); // 상태 업데이트로 컴포넌트 리렌더링 유도
  };

  // 자식 컴포넌트에서 라우팅 정보에 접근할 수 있도록 Context로 제공
  return (
    <RouterContext.Provider value={{ location, navigate }}>
      {children}
    </RouterContext.Provider>
  );
}

// 현재 경로를 반환하는 Hook
export const useLocation = () => useContext(RouterContext).location;

// 경로를 변경하는 Hook
export const useNavigate = () => useContext(RouterContext).navigate;
```

- **`useState`로 `location` 관리**
    - 현재 URL 경로(`window.location.pathname`)를 상태로 관리합니다.
    - URL이 변경되면 `location` 상태가 업데이트됩니다.
- **`useEffect`로 URL 변경 감지**
    - `popstate` 이벤트를 사용하여 브라우저의 뒤로 가기/앞으로 가기 시 URL 변경을 감지합니다.
    - URL이 변경되면 `setLocation`을 호출하여 경로를 갱신합니다.
- **`navigate` 함수**
    - `window.history.pushState()`를 사용하여 브라우저의 URL을 변경합니다.
    - 페이지 전체를 새로고침하지 않고 경로만 변경합니다.
- **Context API 활용**
    - `RouterContext`를 사용하여 **라우팅 상태**(`location`, `navigate`)를 하위 컴포넌트에서 접근할 수 있도록 합니다.
    

**`Routes`와 `Route` 구현 예시**

**`Routes`** 컴포넌트는 현재 경로에 맞는 **자식 컴포넌트**를 렌더링하는 역할을 합니다. **`Route`** 컴포넌트는 특정 경로(`path`)에 대해 렌더링할 컴포넌트를 정의합니다.

```jsx
export function Routes({ children }) {
  const location = useLocation(); // 현재 경로 가져오기
  let element = null;

  // 자식 컴포넌트들(`Route`)을 순회하면서 현재 경로와 일치하는 컴포넌트를 찾음
  children.forEach((child) => {
    if (child.props.path === location) {
      element = child;
    }
  });

  return element; // 일치하는 경로가 없으면 null 반환
}

export function Route({ path, element }) {
  return element; // 일치하는 경로의 컴포넌트 렌더링
}
```

- **`useLocation`으로 현재 경로 가져오기**
    - `useLocation` 훅을 사용하여 현재 브라우저의 경로를 가져옵니다.
- **`children.forEach`를 통해 경로 매칭**
    - `<Routes>`의 자식 요소로 전달된 모든 `<Route>` 컴포넌트를 순회합니다.
    - 현재 경로(`location`)와 `<Route>`의 `path` 속성이 일치하는지 확인합니다.
- **매칭된 컴포넌트 렌더링**
    - 현재 경로와 일치하는 `<Route>` 컴포넌트를 찾으면 해당 컴포넌트를 렌더링합니다.
    - 일치하는 컴포넌트가 없으면 `null`을 반환합니다.

**`Link` 구현 예시**

**`Link`** 컴포넌트는 클릭 시 브라우저의 URL을 변경하고, **페이지 새로고침 없이** 해당 경로에 맞는 컴포넌트를 렌더링합니다.

```jsx
export function Link({ to, children }) {
  const navigate = useNavigate();

  const handleClick = (event) => {
    event.preventDefault();
    navigate(to);
  };

  return <a href={to} onClick={handleClick}>{children}</a>;
}
```

- **`useNavigate` 훅 사용**
    - `useNavigate` 훅을 사용하여 URL을 변경하는 `navigate` 함수를 가져옵니다.
- **`handleClick` 이벤트**
    - `event.preventDefault()`를 사용하여 `<a>` 태그의 기본 동작(페이지 새로고침)을 방지합니다.
    - `navigate(to)`를 호출하여 지정된 경로로 URL을 변경합니다.
- **클라이언트 사이드 네비게이션**
    - 이 방식으로 페이지 전체를 다시 로드하지 않고 URL만 변경하여 **SPA처럼 동작**하도록 만듭니다.