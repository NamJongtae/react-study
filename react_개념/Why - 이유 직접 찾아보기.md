# ❓ Why - 이유 직접 찾아보기

React 공식 문서를 살펴보면서 기존에 이해하였던 내용을 더 깊게 살펴보기 위해 본질적인 내용이나 궁금한 내용을 직접 찾아보면 기록합니다. 

## 📃 목차
- [1. JSX가 어떻게 컴파일 될까?](#1-jsx가-어떻게-컴파일-될까)

- [2. Props로 ref를 사용하지 못하는 이유?](#2-props로-ref를-사용하지-못하는-이유)

- [3. React에서 State란 무엇일까?](#3-react에서-state란-무엇일까)

- [4. React의 렌더링은 어떻게 동작할까?](#4-react의-렌더링은-어떻게-동작할까)

- [5. Virtual DOM은 어떻게 동작할까?](#5-virtual-dom은-어떻게-동작할까)

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
