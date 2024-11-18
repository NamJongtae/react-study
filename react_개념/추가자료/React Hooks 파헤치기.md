> React Hooks는 React 내부에서 **클로저(closure)** 와 **배열**을 사용하여 상태와 함수 호출을 관리합니다. Hooks는 함수형 컴포넌트가 렌더링될 때마다 **순차적으로 호출**되며, React는 이를 통해 각 컴포넌트의 상태를 추적합니다.

### **React Hooks 소스 코드 살펴보기**

아래는 react 공식 github에서 제공하는 [ReactHooks](https://github.com/facebook/react/blob/v16.12.0/packages/react/src/ReactHooks.js) 소스 코드입니다.

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

- 모든 훅에서 **dispatcher**라는 객체를 `resolveDispatcher` 함수를 통해 호출합니다.
    
    **dispatcher** 객체는 현재 렌더링 중인 컴포넌트와 관련된 Hook 상태 및 동작을 관리합니다. `resolveDispatcher`는 렌더링 컨텍스트에서 현재 활성화된 `dispatcher`를 가져오며, 이는 React의 상태 관리 시스템(Fiber)와 직접 연결됩니다. 이를 통해 각 훅은 자신의 상태를 React 시스템에 안전하게 등록하고 관리할 수 있습니다.
    
- **dispatcher** 객체를 통해 Hook를 호출합니다.
    
    **dispatcher** 객체는 `useState`, `useEffect`, `useReducer` 등의 Hook 메서드를 포함하고 있으며, 각각의 메서드는 React 내부에서 필요한 상태 관리 또는 부수 효과 처리를 담당합니다. 각 Hook 호출은 `dispatcher` 객체의 메서드를 통해 실행되며, 이를 통해 React는 렌더링 단계에서 상태와 부수 효과를 추적하고 업데이트를 예약합니다.
    
<br/>

### React Hooks의 구현채 살펴보기

**useState** [ReactHooks](https://github.com/facebook/react/blob/v18.3.1/packages/react/src/ReactHooks.js)의 구현채를 따라가며 ReactHooks에 대해 살펴보겠습니다.

```jsx
export function useState<S>(
  initialState: (() => S) | S, // 초기 상태값 타입 콜백 함수 방식과 일반 방식 두가지.
): [S, Dispatch<BasicStateAction<S>>] { 
// 반환하는 타입 S는 상태 Dispatch<action<상태>>는 setState
  const dispatcher = resolveDispatcher(); // dispatcher라는 resolveDispatcher 함수 반환값을 받고
  return dispatcher.useState(initialState); // 거기서 useState라는 메서드(함수를 실행해서 상태를 넣어준다.)
}
```

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

- `resolveDispatcher`는 `ReactCurrentDispatcher.current`를 읽어, 현재 활성화된 디스패처를 가져옵니다.
- 만약 `current`가 `null`이라면, React는 컴포넌트 외부에서 훅을 호출했거나 아직 디스패처가 초기화되지 않은 경우입니다. 이때 에러를 발생시켜 **훅을 잘못된 위치에서 호출하지 않도록 보호**합니다.
- 정상적으로 `current`에 값이 있다면, 해당 디스패처를 반환합니다.

<br/>

```jsx
const ReactCurrentDispatcher = {
/**
* @internal
* @type {ReactComponent}
*/
  current: (null: null | Dispatcher),
}

export default ReactCurrentDispatcher
```

- `ReactCurrentDispatcher`는 React의 내부에서 **훅(hook)** 호출과 관련된 동작을 관리하기 위해 사용되는 전역 객체입니다.
- React는 컴포넌트가 렌더링되는 동안 각 훅(`useState`, `useEffect`, `useContext` 등)의 호출을 추적하고, 적절한 동작(예: 상태 반환, 부수 효과 등록 등)을 수행하기 위해 이 객체를 활용합니다.
- `Dispatcher`는 훅의 동작을 정의하는 메서드들의 모음입니다. 예를 들어:
    - `useState`가 호출되었을 때 상태를 반환하고 업데이트 함수(setter)를 제공하는 로직
    - `useEffect`가 호출되었을 때 부수 효과를 등록하는 로직

React는 컴포넌트의 초기 렌더링, 업데이트, 서버 사이드 렌더링 등 다양한 환경에서 다른 Dispatcher를 사용합니다. 각 환경에 맞는 훅 동작을 위해 Dispatcher를 전환합니다.

<br/>

**렌더링 중 Dispatcher**

렌더링 중 훅 호출을 처리하는 Dispatcher입니다. 함수형 컴포넌트가 렌더링되는 동안 사용됩니다.

```jsx
const HooksDispatcherOnMount = {
  useState: mountState,
  useEffect: mountEffect,
  useContext: readContext,
  // 기타 훅들...
};
```

<br/>

**업데이트 중 Dispatcher**

컴포넌트가 업데이트되는 중에 사용되는 Dispatcher입니다.

```jsx
const HooksDispatcherOnUpdate = {
  useState: updateState,
  useEffect: updateEffect,
  useContext: readContext,
  // 기타 훅들...
};
```

<br/>

**서버 사이드 렌더링 Dispatcher**

서버 사이드 렌더링 환경에서 사용되는 Dispatcher입니다.

```jsx
const HooksDispatcherOnServer = {
  useState: unsupportedHook,
  useEffect: unsupportedHook,
  useContext: readContext,
  // 기타 훅들...
};
```

<br/>

`ReactCurrentDispatcher`는 **react-reconciler** 패키지의 **ReactFiberHook** 모듈에서 결정됩니다.

이는 **Fiber**를 통해 **Reconciler**에서 Hook를 관리하는 것을 의미합니다.

react에서 단순히 react element만을 알고 있으며, 실제 Hook을 포함한 element 객체는 모릅니다.

react element를 fiber로 확장하는 역할은 reconciler가 하게됩니다.

JSX는 `createElement` 함수로 변환되어 호출된 후 tagName이 함수인 경우 함수형 컴포넌트를 재귀적으로 실행 시켜 중첩객체로 변환시켜줍니다.

```jsx
export const createElement = (tagName, props, ...children) => {
  if(typeof tagName=== 'function') {
	  return tagName(props, ...children);
  }
  return {
    tagName,
    props,
    children,
  };
}
```

여기서 React는 중첩적으로  함수를 바로 호출하는 것이 아니라 일단 reconciler의 렌더 과정에서 함수형 컴포넌트를 각각 호출합니다. 이때, ReactCurrentDispatcher.current에 적절한 hooks 함수를 주입하여 함수형 컴포넌트에서 사용할 수 있도록합니다.

위 과정을 그림으로 나타내면 아래와 같습니다:

![](https://velog.velcdn.com/images/njt6419/post/a3c33bab-85fd-48c8-8ad3-e4fdcea23fd7/image.png)


<br/>

### Reconciler(재조정자) 란?

Reconciler은 전체 컴포넌트 트리에서 렌더 결과물을 수집한 후 새로운 **Virtual DOM** 트리와 이전 Virtual DOM 트리를 비교하고, 실제 DOM에 적용해야 할 변경 사항을 계산하는 작업을 수행합니다. 이를 **재조정(Reconciliation)** 이라고 하며, 재조정을 통해 React는 상태 변경 시 실제 DOM 조작을 최소화하며 최적화된 방식으로 업데이트를 수행합니다.

![](https://velog.velcdn.com/images/njt6419/post/cc56eba7-184d-4b01-9b72-3c35e4503dc1/image.png)

리액트는 항상 2개의 Virtual DOM를 가지고 있습니다.

**1 ) 렌더링 이전 화면 구조를 나타내는 Virtual DOM**

**2 ) 렌더링 이후에 보이게될 화면 구조를 나타내는 Virtual DOM**

리액트는 실제 브라우저 화면에 그려지기 이전 렌더링이 발생될 상황이 되면 새로운 화면에 들어갈 내용이 담긴 가상돔을 생성합니다.

렌더링 이전 화면 구조를 나타내는 Virtual DOM과 업데이트 이후 화면 구조를 나타내는 Virtual DOM를 비교하여 정확히 어떤 부분이 변경되었는지 효율적으로 비교하는 Diffing을 통해 변경된 부분들을 파악하고 **Scheduling(스케줄링)**을 통해 변경사항을 즉시 반영하지 않고, 우선순위를 기반으로 작업을 스케줄링하여 변경된 부분만을 실제 DOM에 한번에 반영합니다.

<br/>

### Fiber 란?

**React Fiber**는 React 16에서 도입된 **새로운 재조정(Reconciliation) 알고리즘**입니다. Fiber는 React의 렌더링 과정을 최적화하고 더 유연하게 만들기 위해 설계되었습니다. 

React는 애플리케이션의 모든 컴포넌트 인스턴스를 추적하기 위해 **Fiber**라는 자료 구조를 사용합니다. Fiber는 컴포넌트 메타데이터와 관련된 핵심 정보를 담고 있는 객체로, React가 렌더링과 업데이트 과정을 최적화하는 데 중요한 역할을 합니다.

각 Fiber 객체는 현재 렌더링 중인 컴포넌트와 관련된 여러 정보를 포함하고 있습니다. 주요 메타데이터는 다음과 같습니다:

- **컴포넌트 유형**: Fiber는 어떤 타입의 컴포넌트가 렌더링되어야 하는지에 대한 정보를 담고 있습니다.
- **Props와 State**: 현재 컴포넌트가 렌더링될 때 사용되는 props와 state 값을 포함하고 있으며, 이 값들은 Fiber 객체에서 추적됩니다.
- **트리 구조 정보**: 부모, 형제, 자식 컴포넌트를 가리키는 포인터들이 있어, React가 컴포넌트 트리 내에서 관계를 파악할 수 있게 해 줍니다.
- **렌더링 상태를 위한 내부 메타데이터**: React가 렌더링과 업데이트를 추적하기 위해 필요한 내부 데이터를 포함하고 있습니다.

Fiber는 기본적으로 **React의 비동기 렌더링을 지원**하며, 렌더링 성능을 개선하기 위해 도입된 자료 구조입니다. 전통적인 React 렌더링에서는 업데이트가 발생할 때 전체 컴포넌트를 한 번에 렌더링했습니다. 그러나 Fiber를 사용하면서부터는 React가 작업을 작은 단위로 나누어, 프레임 단위로 작업을 중단하고 다시 시작할 수 있게 되었습니다. 이를 통해 브라우저의 사용자 인터페이스가 끊김 없이 반응할 수 있게 되며, 작업이 중간에 종료되거나 긴급 업데이트가 발생할 경우 우선순위에 따라 작업을 재조정할 수 있습니다.

React는 부모 컴포넌트가 자식 컴포넌트를 렌더링할 때, 해당 컴포넌트를 추적하기 위한 Fiber 객체를 생성합니다. **클래스형 컴포넌트**의 경우, React는 `new YourComponentType(props)`를 호출하여 컴포넌트 인스턴스를 생성하고 이를 Fiber에 저장합니다. 여기서 `this.props`는 사실 Fiber에 저장된 props의 참조를 복사한 것입니다. 반면, **함수형 컴포넌트와 Hooks**의 경우에는 컴포넌트 인스턴스 대신 `YourComponentType(props)`를 호출하여 렌더링합니다. 함수형 컴포넌트에서 사용하는 모든 훅은 Fiber 객체의 연결 리스트 형태로 저장됩니다. React가 함수형 컴포넌트를 렌더링할 때마다 해당 Fiber에서 훅 연결 리스트를 참조하며, 호출된 순서에 따라 저장된 상태 값이나 `useReducer`의 dispatch 함수를 반환합니다.

렌더링은 렌더와 커밋 2가지 단계로 나누어집니다. Fiber를 통해 자세히 설명하자면, 렌더 단계에서는 Fiber를 순회하며 가상 DOM의 변경 사항을 계산합니다. 이 단계에서는 실제 DOM을 변경하지 않으며, 필요에 따라 작업을 중단하거나 이어서 수행할 수 있습니다. 이후 커밋 단계에서는 Fiber에서 준비된 변경 사항을 바탕으로 실제 DOM을 업데이트합니다. 이 단계는 동기적으로 수행되어 최종 변경 사항이 화면에 반영됩니다. Fiber 시스템 덕분에 React는 복잡한 UI를 효율적으로 렌더링하고, 동적이고 상호작용이 많은 애플리케이션에서도 부드러운 사용자 경험을 제공할 수 있습니다.

<br/>

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

<br/>

### 리액트 파이버 트리

![](https://velog.velcdn.com/images/njt6419/post/ad650b03-9e9c-4af3-8a07-2c6cebe5bcc1/image.png)

파이버 트리는 React에서 2가지가 존재합니다. 하나는 현재 모습을 담은 **current** 파이버 트리, 다른 하나는 작업 중인 상태를 나타내는 **workInProgress** 트리입니다. 리액트 파이버의 작업이 끝나면 리액트는 단순히 포인터만 변경하여 **workInprogress** 트리를 현재 트리로 변경합니다. 이러한 기술을 **더블 버퍼링**이라고 합니다.

리액트에서 **더블 버퍼링**은 커밋 단계에서 수행되게됩니다. 먼저 현재 UI 렌더링을 위해 존재하는 트리인 current기준으로 모든 작업이 시작됩니다. 여기에서 업데이트가 발생하면 파이버는 리액트에서 새로 받은 데이터로 새로운 **workInProgress** 트리를 빌드하기 시작합니다. **workInprogress** 트리를 빌드하는 작업이 끝나면 다음 렌더링에 이 트리를 사용하게됩니다. 이 **workInprogress** 트리가 UI에 최종적으로 렌더링되어 반영이 완료되면 **current**가 이 **workInprogress**로 변경됩니다.

💡 **더블 버퍼링**

**더블 버퍼링**은 리액트에서 새롭게 나온 개념이 아니며, 컴퓨터 그래픽 분야에서 사용하는 용어입니다. 그래픽을 통해 화면을 표시되는 것을 그리기 위해서는 내부적으로 처리를 거쳐야 하는데, 이러한 처리를 거치게 되면 사용자에게 미처 다 그리지 못한 모습을 보이는 경우가 발생하게됩니다. 그래서 이러한 상황을 방지하기 위해 보이지 않는 곳에서 그다음으로 그려야 할그림을 미리 그린 다음, 이것이 완성되면 현재 상태를 새로운그림으로 바꾸는 기법을 의미합니다.

<br/>

### 파이버의 작업 순서

```html
// JSX
<A1>
  <B1>안녕하세요</B1>
  <B2>
    <C1>
      <D1 />
      <D2 />
    </C1>
  </B2>
  <B3 />
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

![](https://velog.velcdn.com/images/njt6419/post/061302bb-b73f-4c54-a102-564b4b0e7ad4/image.webp)

여기서 만약 **setState**로 인한 업데이트가 발생하면 어떻게 될까요? 이미 리액트에는 앞서 만든 **current** 트리가 존재하고 **setState**로 인한 업데이트 요청을 받아 **workInProgress** 트리를 다시 빌드하기 시작합니다 최초 렌더링 시에는 모든 파이버를 새롭게 만들어야 했지만, 이미 파이버가 존재하므로 새로 생성하지 않고 기존의 파이버에서 업데이트된 **props**를 받아 파이버 내부에서 처리합니다. 새로운 파이버를 만드는 것은 **리소스 낭비**라고 볼 수 있습니다. 따라서 기존의 파이버 객체를 재활용하여 내부 속성값만 초기화하거나 바꾸는 형태로 트리를 업데이트합니다. 이것이 앞에서 설명한 “가급적이면 새로운 파이버를 생성하지 않는다.”가 바로 이것입니다.

리액트는 이러한 작업을 파이버 단위로 나누어서 수행하며, 애니메이션이나 사용자가 입력하는 작업은 우선순위가 높은 작업으로 분리하거나 목록을 렌더링하는 등의 작업의 우선순위가 낮은 작업으로 분리해 최적의 순위로 작업을 완료하도록합니다.

<br/>

### renderWithHooks

reconciler의 [ReactFiberHooks.rednerWithHooks](https://github.com/facebook/react/blob/v16.12.0/packages/react-reconciler/src/ReactFiberHooks.js#L375) 함수를 이용하여 ReactCurrentDispatcher.current에 적절한 hooks 함수를 주입하게됩니다.

```jsx
export function renderWithHooks(
  current: Fiber, // 이전 렌더링에서 사용된 Fiber 노드
  workInProgress: Fiber, // 현재 렌더링 중인 Fiber 노드
  Component: any, // 렌더링할 함수형 컴포넌트
  props: any,
  refOrContext: any, // ref나 Context 정보
  nextRenderExpirationTime: ExpirationTime // 이번 렌더링의 만료시간
): any {
  renderExpirationTime = nextRenderExpirationTime;
  currentlyRenderingFiber = workInProgress; // 현재 작업 중인 fiber를 전역으로 잡아둠
  nextCurrentHook = current !== null ? current.memoizedState : null;

  ReactCurrentDispatcher.current =
    nextCurrentHook === null ? HooksDispatcherOnMount : HooksDispatcherOnUpdate;

  let children = Component(props, refOrContext);

  /*컴포넌트 재호출 로직..*/

  const renderedWork = currentlyRenderingFiber;
  renderedWork.memoizedState = firstWorkInProgressHook;

  ReactCurrentDispatcher.current = ContextOnlyDispatcher;

  currentlyRenderingFiber = null;
   /*...*/
}
```

**memoizedState**에 **firstWorkInProgressHook**를 저장합니다.

**firstWorkInProgressHook**는 전역변수에 저장되어 있는 hook 연결리스트의 head입니다.

**memoizedState를 통해 currnet의 존재 유무로 mount 되어야하는지 update 되어야하는지 여부를 판단합니다.**

renderWithHooks는 VirtualDOM wokInProgress 트리가 순회될 때, 특정 Fiber가 렌더 되어야하는 상황일 때 호출됩니다. 여기서 인자로 들어오는 currnet와 workInProgress는 렌더가 필요한 Fiber 노드가됩니다.

아래 부분은 모듈 전체에서 쓰는 전역 변수입니다.

```jsx
renderExpirationTime = nextRenderExpirationTime;
currentlyRenderingFiber = workInProgress; // 현재 작업 중인 fiber를 전역으로 잡아둠
nextCurrentHook = current !== null ? current.memoizedState : null;
```

파일 초반 부분을 살펴보면 위 전역 변수 선언과 초기화 부분을 살펴볼 수 있습니다.

```jsx
// These are set right before calling the component.
let renderExpirationTime: ExpirationTime = NoWork;
// The work-in-progress fiber. I've named it differently to distinguish it from
// the work-in-progress hook.
let currentlyRenderingFiber: Fiber | null = null;

// Hooks are stored as a linked list on the fiber's memoizedState field. The
// current hook list is the list that belongs to the current fiber. The
// work-in-progress hook list is a new list that will be added to the
// work-in-progress fiber.
let currentHook: Hook | null = null;
let nextCurrentHook: Hook | null = null;
let firstWorkInProgressHook: Hook | null = null;
let workInProgressHook: Hook | null = null;
let nextWorkInProgressHook: Hook | null = null;

let remainingExpirationTime: ExpirationTime = NoWork;
let componentUpdateQueue: FunctionComponentUpdateQueue | null = null;
let sideEffectTag: SideEffectTag = 0;

```

<br/>

**Dispatcher 설정 (Mount vs Update)**

```jsx
if (__DEV__) {
  if (nextCurrentHook !== null) {
    ReactCurrentDispatcher.current = HooksDispatcherOnUpdateInDEV;
  } else if (hookTypesDev !== null) {
    ReactCurrentDispatcher.current = HooksDispatcherOnMountWithHookTypesInDEV;
  } else {
    ReactCurrentDispatcher.current = HooksDispatcherOnMountInDEV;
  }
} else {
  ReactCurrentDispatcher.current =
    nextCurrentHook === null
      ? HooksDispatcherOnMount
      : HooksDispatcherOnUpdate;
}
```

- **Mount** 단계인지 **Update** 단계인지에 따라 **Dispatcher**를 설정합니다.
- Dispatcher는 `useState`, `useEffect` 등 Hook 호출 시 어떤 함수를 사용할지 결정합니다.

<br/>

**컴포넌트 렌더링**

```jsx
let children = Component(props, refOrContext);
```

실제 컴포넌트 함수를 호출하여 **자식 요소**를 렌더링합니다.

<br/>

**Render Phase Updates 처리 (렌더링 도중 업데이트 발생 시)**

```jsx
if (didScheduleRenderPhaseUpdate) {
  do {
    didScheduleRenderPhaseUpdate = false;
    numberOfReRenders += 1;
    if (__DEV__) {
      ignorePreviousDependencies = false;
    }

    nextCurrentHook = current !== null ? current.memoizedState : null;
    nextWorkInProgressHook = firstWorkInProgressHook;

    currentHook = null;
    workInProgressHook = null;
    componentUpdateQueue = null;

    if (__DEV__) {
      hookTypesUpdateIndexDev = -1;
    }

    ReactCurrentDispatcher.current = __DEV__
      ? HooksDispatcherOnUpdateInDEV
      : HooksDispatcherOnUpdate;

    // 컴포넌트를 다시 렌더링
    children = Component(props, refOrContext);
  } while (didScheduleRenderPhaseUpdate);

  renderPhaseUpdates = null;
  numberOfReRenders = 0;
}
```

- 렌더링 도중에 `useState`나 `useReducer`로 인한 업데이트가 발생한 경우, 컴포넌트를 다시 렌더링합니다.
- 이 과정은 **무한 렌더링 방지**를 위해 최대 재렌더링 횟수가 설정됩니다.

<br/>

**Dispatcher 초기화 및 Hook 목록 저장**

```jsx
ReactCurrentDispatcher.current = ContextOnlyDispatcher;

const renderedWork: Fiber = (currentlyRenderingFiber: any);
renderedWork.memoizedState = firstWorkInProgressHook;
renderedWork.expirationTime = remainingExpirationTime;
renderedWork.updateQueue = (componentUpdateQueue: any);
renderedWork.effectTag |= sideEffectTag;

if (__DEV__) {
  renderedWork._debugHookTypes = hookTypesDev;
}
```

- 렌더링이 완료된 후, **Fiber 객체에 Hook 상태를 저장**합니다.
- `effectTag`를 통해 **Side Effect**가 있는지 표시합니다.

<br/>

**Hook 개수 검증 및 초기화**

```jsx
	const didRenderTooFewHooks =
	  currentHook !== null && currentHook.next !== null;
	
	renderExpirationTime = NoWork;
	currentlyRenderingFiber = null;
	
	currentHook = null;
	nextCurrentHook = null;
	firstWorkInProgressHook = null;
	workInProgressHook = null;
	nextWorkInProgressHook = null;
	
	if (__DEV__) {
	  currentHookNameInDev = null;
	  hookTypesDev = null;
	  hookTypesUpdateIndexDev = -1;
	}
	
	remainingExpirationTime = NoWork;
	componentUpdateQueue = null;
	sideEffectTag = 0;

	invariant(
	  !didRenderTooFewHooks,
	  'Rendered fewer hooks than expected. This may be caused by an accidental early return statement.',
	);

   return children;
	 }
```

- **모든 상태를 초기화**하고, **렌더링된 자식 요소를 반환**합니다.
- Hook 개수가 예상보다 적으면 경고를 발생시킵니다.

renderWithHooks의 모든 일이 끝나면 하나의 fiber를 render 한 것 이므로 모든 전역변수를 초기화시키는 코드가 실행됩니다.

```jsx
 renderExpirationTime = NoWork;
  currentlyRenderingFiber = null;

  currentHook = null;
  nextCurrentHook = null;
  firstWorkInProgressHook = null;
  workInProgressHook = null;
  nextWorkInProgressHook = null;
```

아래 부분을 보면 nextCurrnetHook을 currnet가 없는 경우 current의 memoizedState로 설정하고 있습니다.

```jsx
nextCurrentHook = current !== null ? current.memoizedState : null;
```

fiber에서는 hook이라는 이름 대신 memoizedState 프로퍼티에 hook을 저장합니다.

```jsx
const fiber = {
	//...
	memoizedState: [
	  {
	    state: 1,
	  },
	]
	//...
}
```

이런 형태로 여러개의 hooks가 순차적으로 배열에 쌓이게됩니다.

예를 들어, useState를 여러개 사용하는 경우

```jsx
const App = () => {
  const [count, setCount] = useState(0);
  const [text, setText] = useState('');
  //...
}
```

```jsx
const fiber = {
	//...
	memoizedState: [
	  {
	    state: 1,
	  },
	  {
		  state: ''
	  },
	]
	//...
}
```

이런식으로 hooks가 쌓이게 되면서 하나의 Fiber 내부에서 hook의 실행 순서가 보장됩니다.

아래 부분은 hooks의 주입부입니다.

```jsx
ReactCurrentDispatcher.current =
    nextCurrentHook === null ? HooksDispatcherOnMount : HooksDispatcherOnUpdate;
```

여기서 nextCurrentHook이 null인 경우는 기존의 current VirtualDOM 요소의 hook이 없는 경우로 current가 없는 경우이며, 바로 **VirtualDOM이 마운트되는 시점**입니다.

nextCurrentHook가 존재하는 경우는 기존의 current VirtualDOM 요소의 hook이 존재하는 경우로 기존에 마운트된 VirtualDOM인 current의 hook을 업데이트 하도록 HooksDispatcherOnUpdate 객체를 불러옵니다.

따라서 current의 여부에 따라 hook 함수가 달라지게됩니다.

<br/>

### MoutState

마운트 시점에 불러와지는 useState인 mounState

```jsx
function mountState<S>(
  initialState: (() => S) | S,
): [S, Dispatch<BasicStateAction<S>>] {
  const hook = mountWorkInProgressHook();
  if (typeof initialState === 'function') {
    initialState = initialState();
  }
  hook.memoizedState = hook.baseState = initialState;
  const queue = (hook.queue = {
    last: null,
    dispatch: null,
    lastRenderedReducer: basicStateReducer,
    lastRenderedState: (initialState: any),
  });
  const dispatch: Dispatch<
    BasicStateAction<S>,
  > = (queue.dispatch = (dispatchAction.bind(
    null,
    // Flow doesn't know this is non-null, but we do.
    ((currentlyRenderingFiber: any): Fiber),
    queue,
  ): any));
  return [hook.memoizedState, dispatch];
}
```

`mountWorkInProgressHook` 함수를 통해 hook 객체를 받아오는 것을 볼 수 있습니다.

<br/>

**mountWorkInProgressHook**

```jsx
function mountWorkInProgressHook(): Hook {
  const hook: Hook = {
    memoizedState: null,

    baseState: null,
    queue: null,
    baseUpdate: null,

    next: null,
  };

  if (workInProgressHook === null) {
    // This is the first hook in the list
    firstWorkInProgressHook = workInProgressHook = hook;
  } else {
    // Append to the end of the list
    workInProgressHook = workInProgressHook.next = hook;
  }
  return workInProgressHook;
}
```

`mountWorkInProgressHook` 이 처음 호출되는 시점에는 무조건 **firstWorkInProgressHook**과 **workInProgressHook**이 여기서 만든 기본 hook 객체로 할당됩니다.

여기서 **firstWorkInProgress**는 첫번째 훅 링크드 리스트의 시작부를 나타냅니다.

이후 호출시에는 workInProgress가 이미 존재하므로 가장 최근에 만들어진 **workInProgressHook** 다음에 hook을 붙여주는 형태로 hook을 링크드리스트로 만들어줍니다.

이 부분을 컴포넌트르 호출한 뒤 **renderWithHooks**에서 해주고있습니다.

```jsx
//...
let children = Component(props, refOrContext); // 컴포넌트 호출
	//...
  // We can assume the previous dispatcher is always this one, since we set it
  // at the beginning of the render phase and there's no re-entrancy.
  ReactCurrentDispatcher.current = ContextOnlyDispatcher;

	// 여기서 작업이 완료된 Fiber의 memoziedState에 첫번째 훅(head)를 붙여줍니다.
  const renderedWork: Fiber = (currentlyRenderingFiber: any);
  renderedWork.memoizedState = firstWorkInProgressHook;
  
  renderedWork.expirationTime = remainingExpirationTime;
  renderedWork.updateQueue = (componentUpdateQueue: any);
  renderedWork.effectTag |= sideEffectTag;
  //...
```

<br/>

**initalState 설정 부분**

```jsx
if (typeof initialState === 'function') {
  initialState = initialState();
}
hook.memoizedState = hook.baseState = initialState;
```

인자로 받은 initalState 타입에 따라 다르게 처리됩니다. 함수인 경우, 이걸 실행시킨 반환값을 initalState로 만듭니다. 만든 hook의 memoizedState에 initalState를 저장합니다.

<br/>

**queue 설정**

```jsx
const queue = (hook.queue = {
  last: null,
  dispatch: null,
  lastRenderedReducer: basicStateReducer,
  lastRenderedState: (initialState: any),
 });
```

이 큐를 통해 여러번의 setState를 한번에 처리합니다. 업데이트 대기열로 사용됩니다.

- `last`: 가장 최근에 추가된 상태 업데이트를 저장합니다. (링크드 리스트 구조에서 사용)
- `dispatch`: 상태를 변경할 때 호출되는 **디스패치 함수**입니다.
- `lastRenderedReducer`: 기본적으로 `basicStateReducer`를 사용합니다. 이 리듀서는 단순히 새로운 상태 값을 반환합니다.
- `lastRenderedState`: 현재 렌더링된 상태 값을 저장합니다.

<br/>

**dispatch(setState)**

```jsx
const dispatch: Dispatch<
  BasicStateAction<S>,
> = (queue.dispatch = (dispatchAction.bind(
  null,
  // Flow doesn't know this is non-null, but we do.
  ((currentlyRenderingFiber: any): Fiber),
  queue,
  ): any));
return [hook.memoizedState, dispatch];
```

setState의 구현 부분으로 queue.dispatch 값에 `dispatchAction(currentRenderingFiber, queue)`이라는 함수를 저장시켰습니다. 지금 작업중인 Fiber를 넘겨주고, 업데이트 대기열을 넘겨서 업데이트시킵니다.

- `dispatchAction` 함수를 `dispatch`로 바인딩합니다.
    - `currentlyRenderingFiber`: 현재 렌더링 중인 Fiber 노드에 접근합니다.
    - `queue`: 앞서 생성한 큐 객체를 참조합니다.
    - **`dispatch` 함수를 클로저로 생성합니다.** 이 클로저는 렌더링 당시 `currentlyRenderingFiber`와 `queue`를 캡처합니다.
- `dispatch` 함수는 **상태 업데이트 요청을 큐에 추가**하고, React가 다시 렌더링하도록 트리거합니다.
    - `dispatchAction`은 React 내부에서 상태 업데이트를 처리하는 핵심 함수입니다.

<br/>

### updateState

```jsx
function updateState<S>(
  initialState: (() => S) | S,
): [S, Dispatch<BasicStateAction<S>>] {
  return updateReducer(basicStateReducer, (initialState: any));
}
```

updateState는 **updateReducer**로 구현되어있습니다.

- `updateState` 함수는 React 컴포넌트가 이미 마운트된 후에 다시 렌더링될 때(`update phase`) 호출됩니다.
- `updateState`는 내부적으로 `updateReducer` 함수를 호출합니다.
    - 여기서 `basicStateReducer`는 단순히 새로운 상태를 반환하는 기본 리듀서 함수입니다.
    
    ```jsx
    function basicStateReducer<S>(state: S, action: BasicStateAction<S>): S {
      return typeof action === 'function' ? action(state) : action;
    }
    ```
    
    - `initialState`는 무시되며, 이전에 저장된 상태를 가져옵니다

<br/>

### **updateReducer**

```jsx
function updateReducer<S, I, A>(
  reducer: (S, A) => S,
  initialArg: I,
  init?: I => S,
): [S, Dispatch<A>] {
  const hook = updateWorkInProgressHook();
	const queue = hook.queue;
  invariant(
    queue !== null,
    'Should have a queue. This is likely a bug in React. Please file an issue.',
  );

  queue.lastRenderedReducer = reducer;

  if (numberOfReRenders > 0) {
	  //...
  }

  // The last update in the entire queue
  const last = queue.last;
  // The last update that is part of the base state.
  const baseUpdate = hook.baseUpdate;
  const baseState = hook.baseState;

  // Find the first unprocessed update.
  let first;
  if (baseUpdate !== null) {
    if (last !== null) {
      // For the first update, the queue is a circular linked list where
      // `queue.last.next = queue.first`. Once the first update commits, and
      // the `baseUpdate` is no longer empty, we can unravel the list.
      last.next = null;
    }
    first = baseUpdate.next;
  } else {
    first = last !== null ? last.next : null;
  }
  if (first !== null) {
	  //...
  }

  const dispatch: Dispatch<A> = (queue.dispatch: any);
  return [hook.memoizedState, dispatch];
}
```

- `updateReducer` 함수는 `useReducer`와 `useState`를 모두 처리할 수 있는 범용 함수입니다.
- 이 함수는 `reducer`, `initialArg`, `init`을 인자로 받아 새로운 상태를 계산하고 반환합니다.
- `reducer`는 상태를 업데이트하기 위해 사용됩니다.

**updateReducer** 코드를 나누어 자세히 살펴보겠습니다.

<br/>

**Hook 객체 가져오기**

```jsx
const hook = updateWorkInProgressHook();
const queue = hook.queue;
```

- `updateWorkInProgressHook()`을 호출하여 **현재 Fiber 노드에 연결된 Hook**을 가져옵니다.
- `queue`는 상태 업데이트를 관리하는 **큐 객체**입니다.
    - 이 큐는 이전 렌더링에서 발생한 **상태 업데이트(action)** 들을 연결 리스트로 관리합니다.

**Queue 유효성 검사**

```jsx
invariant(
  queue !== null,
  'Should have a queue. This is likely a bug in React. Please file an issue.',
);
```

- `queue`가 없으면 React 내부에서 오류가 발생했음을 의미합니다.

<br/>

**Reducer 설정**

```jsx
queue.lastRenderedReducer = reducer;
```

- 현재 사용 중인 리듀서를 큐에 저장합니다.
- 이는 이후 업데이트에서 동일한 리듀서를 사용할 수 있도록 합니다.

<br/>

**렌더링 중 상태 업데이트 처리**

```jsx
if (numberOfReRenders > 0) {
  const dispatch: Dispatch<A> = (queue.dispatch: any);
  if (renderPhaseUpdates !== null) {
    const firstRenderPhaseUpdate = renderPhaseUpdates.get(queue);
    if (firstRenderPhaseUpdate !== undefined) {
      renderPhaseUpdates.delete(queue);
      let newState = hook.memoizedState;
      let update = firstRenderPhaseUpdate;
      do {
        const action = update.action;
        newState = reducer(newState, action);
        update = update.next;
      } while (update !== null);

      if (!is(newState, hook.memoizedState)) {
        markWorkInProgressReceivedUpdate();
      }

      hook.memoizedState = newState;
      if (hook.baseUpdate === queue.last) {
        hook.baseState = newState;
      }

      queue.lastRenderedState = newState;
      return [newState, dispatch];
    }
  }
  return [hook.memoizedState, dispatch];
}
```

- **렌더링 중**(`render phase`) 상태 업데이트가 발생한 경우 이를 처리합니다.
    - `renderPhaseUpdates`: 현재 렌더링 중 발생한 상태 업데이트를 저장하는 맵입니다.
    - 만약 `renderPhaseUpdates`가 존재한다면, 이를 처리하여 **새로운 상태를 계산**합니다.
- 상태가 변경되면 **Fiber에 변경 사항**이 있음을 표시합니다.

<br/>

**일반적인 상태 업데이트 처리**

```jsx
const last = queue.last;
const baseUpdate = hook.baseUpdate;
const baseState = hook.baseState;

let first;
if (baseUpdate !== null) {
  if (last !== null) {
    last.next = null;
  }
  first = baseUpdate.next;
} else {
  first = last !== null ? last.next : null;
}

```

- `last`: 큐에서 마지막으로 추가된 업데이트입니다.
- `baseUpdate`: 현재 렌더링에서 사용된 **마지막 업데이트**입니다.
- `baseState`: 마지막 업데이트 이전의 **기본 상태**입니다.
- `baseUpdate`가 존재하면, **이전 업데이트 이후의 첫 번째 업데이트**부터 순회합니다.
- 만약 `baseUpdate`가 없다면, **큐의 첫 번째 업데이트**부터 순회합니다.
- `last.next = null`은 큐가 **순환 링크드 리스트 형태**이기 때문에, 순환 참조를 끊어줍니다.

<br/>

**상태 업데이트 처리**

```jsx
if (first !== null) {
  let newState = baseState; // 새로운 상태를 기본 상태로 초기화
  let newBaseState = null; // 새로운 기본 상태
  let newBaseUpdate = null; // 새로운 기본 업데이트
  let prevUpdate = baseUpdate; // 이전 업데이트를 추적
  let update = first; // 현재 처리 중인 업데이트
  let didSkip = false; // 우선순위가 낮아 건너뛴 업데이트가 있는지 여부

  do {
    const updateExpirationTime = update.expirationTime;
    if (updateExpirationTime < renderExpirationTime) {
      // 현재 렌더링 우선순위보다 낮은 업데이트는 건너뜀
      if (!didSkip) {
        didSkip = true;
        newBaseUpdate = prevUpdate; // 건너뛴 업데이트를 새로운 기본 업데이트로 설정
        newBaseState = newState; // 건너뛴 시점의 상태를 새로운 기본 상태로 설정
      }
      // 남아있는 업데이트의 우선순위를 추적
      if (updateExpirationTime > remainingExpirationTime) {
        remainingExpirationTime = updateExpirationTime;
        markUnprocessedUpdateTime(remainingExpirationTime);
      }
    } else {
      // 충분한 우선순위가 있는 업데이트는 처리
      markRenderEventTimeAndConfig(
        updateExpirationTime,
        update.suspenseConfig,
      );

      // 미리 계산된 `eagerState`가 있고, 현재 리듀서와 동일하다면 사용
      if (update.eagerReducer === reducer) {
        newState = ((update.eagerState: any): S);
      } else {
        const action = update.action;
        newState = reducer(newState, action); // 리듀서를 사용해 새로운 상태 계산
      }
    }
    prevUpdate = update; // 현재 업데이트를 이전 업데이트로 설정
    update = update.next; // 다음 업데이트로 이동
  } while (update !== null && update !== first);
```

- **업데이트 큐**를 순회하면서 각 업데이트를 처리합니다.
- `expirationTime`을 사용하여 **우선순위가 낮은 업데이트**는 건너뜁니다.
- `didSkip` 플래그를 사용하여 **건너뛴 업데이트가 있는지** 확인합니다.
- **우선순위가 높은 업데이트**는 `reducer`를 사용하여 상태를 계산합니다.

<br/>

**새로운 기본 상태와 기본 업데이트 설정**

```jsx
  if (!didSkip) {
    newBaseUpdate = prevUpdate; // 모든 업데이트가 처리되었을 때 마지막 업데이트를 기본 업데이트로 설정
    newBaseState = newState; // 모든 업데이트가 처리되었을 때 최종 상태를 기본 상태로 설정
  }
```

만약 **모든 업데이트가 처리되었고 건너뛴 업데이트가 없다면**, 최종 상태를 새로운 기본 상태로 설정합니다.

<br/>

**상태 변경 확인 및 Fiber에 반영**

```jsx
  if (!is(newState, hook.memoizedState)) {
    markWorkInProgressReceivedUpdate(); // 새로운 상태가 이전 상태와 다를 경우, 업데이트를 표시
  }

  hook.memoizedState = newState; // Hook에 새로운 상태를 저장
  hook.baseUpdate = newBaseUpdate; // 새로운 기본 업데이트 저장
  hook.baseState = newBaseState; // 새로운 기본 상태 저장

  queue.lastRenderedState = newState; // 마지막으로 렌더링된 상태 업데이트
}
```

- **상태가 변경되었는지 확인**하고, 변경된 경우 **Fiber에 변경 사항을 표시**합니다.
- `memoizedState`, `baseUpdate`, `baseState`를 각각 최신 값으로 업데이트합니다.

<br/>

**결과 반환**

```jsx
const dispatch: Dispatch<A> = (queue.dispatch: any);
return [hook.memoizedState, dispatch];
```

최종적으로 `useState`와 동일하게 `[현재 상태, dispatch 함수]`를 반환합니다.

<br/>

### **updateWorkInProgressHook**

```jsx
function updateWorkInProgressHook(): Hook {
  // This function is used both for updates and for re-renders triggered by a
  // render phase update. It assumes there is either a current hook we can
  // clone, or a work-in-progress hook from a previous render pass that we can
  // use as a base. When we reach the end of the base list, we must switch to
  // the dispatcher used for mounts.
  if (nextWorkInProgressHook !== null) {
    // There's already a work-in-progress. Reuse it.
    workInProgressHook = nextWorkInProgressHook;
    nextWorkInProgressHook = workInProgressHook.next;

    currentHook = nextCurrentHook;
    nextCurrentHook = currentHook !== null ? currentHook.next : null;
  } else {
    // Clone from the current hook.
    invariant(
      nextCurrentHook !== null,
      'Rendered more hooks than during the previous render.',
    );
    currentHook = nextCurrentHook;

    const newHook: Hook = {
      memoizedState: currentHook.memoizedState,

      baseState: currentHook.baseState,
      queue: currentHook.queue,
      baseUpdate: currentHook.baseUpdate,

      next: null,
    };

    if (workInProgressHook === null) {
      // This is the first hook in the list.
      workInProgressHook = firstWorkInProgressHook = newHook;
    } else {
      // Append to the end of the list.
      workInProgressHook = workInProgressHook.next = newHook;
    }
    nextCurrentHook = currentHook.next;
  }
  return workInProgressHook;
}
```

이 함수는 **현재 작업 중인 Hook을 가져와 업데이트**합니다.
**updateWorkInProgressHook** 코드를 나누어 자세히 살펴보겠습니다.

<br/>

**nextWorkInProgressHook이 존재하는 경우**

```jsx
if (nextWorkInProgressHook !== null) {
  // 이미 진행 중인 Hook이 있는 경우, 이를 재사용합니다.
  workInProgressHook = nextWorkInProgressHook; // 현재 Hook을 진행 중인 Hook으로 설정
  nextWorkInProgressHook = workInProgressHook.next; // 다음 Hook으로 이동

  // 현재 Fiber 노드에서 사용 중인 Hook을 가져옵니다.
  currentHook = nextCurrentHook;
  nextCurrentHook = currentHook !== null ? currentHook.next : null;
}
```

이 부분은 render 도중 rerneder가 일어난 경우입니다.

- `nextWorkInProgressHook` 가 `null`이 아니라면, **이전에 사용했던 진행 중인 Hook이 존재**한다는 의미입니다.
    - `workInProgressHook`을 **기존 Hook으로 설정**하고, `nextWorkInProgressHook`을 다음 Hook으로 이동시킵니다.
- `currentHook`은 현재 Fiber에서 사용 중인 Hook을 가리키며, **다음 Hook**으로 이동합니다.
    - `nextCurrentHook`이 존재하면 `next`를 통해 연결된 다음 Hook을 가리킵니다.

<br/>

**nextWorkInProgressHook이 없는 경우 (새로운 Hook 생성)**

```jsx
else {
    // 현재 Hook이 없으면 새로운 Hook을 생성합니다.
    invariant(
      nextCurrentHook !== null,
      'Rendered more hooks than during the previous render.',
    );
    currentHook = nextCurrentHook;

    const newHook: Hook = {
      memoizedState: currentHook.memoizedState,

      baseState: currentHook.baseState,
      queue: currentHook.queue,
      baseUpdate: currentHook.baseUpdate,

      next: null,
    };

    if (workInProgressHook === null) {
      // This is the first hook in the list.
      workInProgressHook = firstWorkInProgressHook = newHook;
    } else {
      // Append to the end of the list.
      workInProgressHook = workInProgressHook.next = newHook;
    }
    nextCurrentHook = currentHook.next;
  }
  return workInProgressHook;
}
```

- `nextWorkInProgressHook` 가 `null`인 경우, 이전 렌더링에서 **Hook을 모두 사용**했기 때문에 새로운 Hook을 생성해야 합니다.
- `invariant`는 조건이 충족되지 않으면 **에러 메시지**를 출력합니다.
    - 여기서는 `nextCurrentHook`이 `null`이면 **Hook의 개수가 맞지 않음을 의미**합니다.

<br/>

**현재 Hook을 복제하여 새로운 Hook 생성**

```jsx
    currentHook = nextCurrentHook;

    const newHook: Hook = {
      memoizedState: currentHook.memoizedState, // 현재 Hook의 상태를 복사
      baseState: currentHook.baseState, // 기본 상태를 복사
      queue: currentHook.queue, // 상태 업데이트 큐 복사
      baseUpdate: currentHook.baseUpdate, // 기본 업데이트 복사
      next: null, // 새로운 Hook이므로 `next`는 null로 설정
    };
```

- `currentHook`을 현재 Fiber에서 사용 중인 Hook으로 설정합니다.
- **새로운 Hook 객체**를 생성하여, 기존 `currentHook`의 속성들을 복사합니다.
    - `memoizedState`: 최신 상태 값을 복사합니다.
    - `baseState`: 기본 상태 값을 복사합니다.
    - `queue`: 상태 업데이트를 관리하는 큐를 복사합니다.
    - `baseUpdate`: 마지막으로 처리된 업데이트를 복사합니다.
    - `next`: 새로 생성된 Hook이기 때문에 `null`로 설정합니다.
    
<br/>

**새로운 Hook을 작업 중인 Hook 목록에 추가**

```jsx
if (workInProgressHook === null) {
  // 첫 번째 Hook인 경우, 새로운 Hook을 목록의 시작으로 설정합니다.
  workInProgressHook = firstWorkInProgressHook = newHook;
  } else {
    // 첫 번째 Hook이 아닌 경우, 목록의 끝에 새로운 Hook을 추가합니다.
    workInProgressHook = workInProgressHook.next = newHook;
  }

  // 다음 Hook으로 이동
  nextCurrentHook = currentHook.next;
}
```

- **현재 작업 중인 Hook 목록이 비어있는지 확인**합니다.
    - 비어있다면, `firstWorkInProgressHook`과 `workInProgressHook` 모두 **새로 생성한 Hook을 가리키도록 설정**합니다.
    - 비어있지 않다면, 현재 목록의 **끝에 새로운 Hook을 추가**합니다.
- `nextCurrentHook`을 다음 Hook으로 설정하여 **목록을 순회**합니다.

<br/>

**workInProgressHook 반환**

```jsx
  return workInProgressHook;
}
```

**최종적으로 업데이트된 Hook**을 반환합니다.

<br/>

**updateReducer**에서 `updateWorkInProgressHook()` 이후 부분을 살펴보겠습니다.

```jsx
const queue = hook.queue;
  invariant(
    queue !== null,
    'Should have a queue. This is likely a bug in React. Please file an issue.',
  );

  queue.lastRenderedReducer = reducer;

  if (numberOfReRenders > 0) {
	  //...
  }

  // The last update in the entire queue
  const last = queue.last;
  // The last update that is part of the base state.
  const baseUpdate = hook.baseUpdate;
  const baseState = hook.baseState;

  // Find the first unprocessed update.
  let first;
  if (baseUpdate !== null) {
    if (last !== null) {
      // For the first update, the queue is a circular linked list where
      // `queue.last.next = queue.first`. Once the first update commits, and
      // the `baseUpdate` is no longer empty, we can unravel the list.
      last.next = null;
    }
    first = baseUpdate.next;
  } else {
    first = last !== null ? last.next : null;
  }
  if (first !== null) {
	  //...
  }
```

이 부분은 state를 업데이트 하는 부분으로 queue에서 업데이트할 요소를 꺼내와서 기존의 base State와 memoizedState를 적절히 사용하여 모든 업데이트를 처리하는 과정입니다. 이 과정에서 인자로 넘겨준 **basicStateReducer**가 사용됩니다.

```jsx
  const dispatch: Dispatch<A> = (queue.dispatch: any);
  return [hook.memoizedState, dispatch];
```

이 부분은 업데이트가 끝난 memoizedState와 queue.dispatch 함수를 배열에 감싸 반환합니다.

<br/>

### 예시 코드를 통해 전체 과정 살펴보기

```jsx
import { useState } from "react";

function App() {
  const [count, setCount] = useState(0);

  const increment = () => {
    setCount(prev => prev + 1);
  }

  const decrement = () => {
    setCount(prev => prev - 1);
  }

  return(
    <div>
      <button onClick={increment}>+</button>
      <button onClick={increment}>-</button>
      <p>Count : {count}</p>
    </div>
  )
}
```

**첫 렌더링 시**

1. **JSX 컴파일**
    - `App.jsx`가 **Babel**에 의해 `React.createElement` 함수 호출로 변환됩니다.
    - 컴파일된 결과는 React 엘리먼트 객체입니다.
2. **React 엘리먼트 객체 → Fiber로 확장**
    - React 엘리먼트 객체가 **Fiber 노드**로 확장됩니다.
    - 이 Fiber 노드는 컴포넌트의 상태와 업데이트를 관리합니다.
3. **초기 렌더링 - `renderWithHooks` 호출**
    - 컴포넌트가 렌더링될 때, `renderWithHooks` 함수가 호출됩니다.
    - 이때 `current` Fiber가 존재하지 않으므로, React는 **마운트 단계**로 진입합니다.
4. **`mountState` 호출 (`useState` 초기화)**
    - `useState` 훅이 호출되면, `mountState` 함수가 실행됩니다.
    - `mountWorkInProgressHook()` 함수가 호출되어 새로운 훅 객체를 생성합니다.
5. **`mountWorkInProgressHook()` 실행**
    - 새로운 `hook` 객체가 생성됩니다.
    - 이 `hook` 객체는 **Fiber 노드의 `memoizedState`에 연결**됩니다.
6. **`useState`의 초기 상태 설정**
    - `mountState`에서 `initialState` (여기서는 `0`)가 `hook.memoizedState`에 저장됩니다.
    - `hook.queue` 객체가 초기화됩니다.
        
        ```tsx
        const queue = (hook.queue = {
          last: null,
          dispatch: null,
          lastRenderedReducer: basicStateReducer,
          lastRenderedState: hook.memoizedState,
        });
        ```
        
    - `queue.lastRenderedState`에 **초기 상태** `0`이 저장됩니다.
7. **`dispatch` (즉, `setState`) 함수 생성**
    - `dispatch` 함수가 생성되어 `hook.queue.dispatch`에 저장됩니다.
    - 이 `dispatch`는 `setState` 역할을 하며, 상태를 업데이트할 때 사용됩니다.
8. **`mountState` 반환**
    - **초기 상태**와 `setState` 함수(`dispatch`)가 반환됩니다.
    - 즉, `[count, setCount] = [0, dispatch]`가 됩니다.
9. **컴포넌트가 렌더링되어 `App` 함수 종료**:
    - JSX가 변환된 React 엘리먼트가 생성되고, 브라우저에 렌더링됩니다.

**상태 업데이트 (`setCount`)**

1. **`increment` 또는 `decrement` 함수 호출 시**
    - `setCount(prev => prev + 1)` 혹은 `setCount(prev => prev - 1)`이 호출됩니다.
    - 이때, `dispatchAction` 함수가 실행됩니다.
2. **`updateState` 호출 (상태 업데이트)**
    - `dispatchAction` 내부에서 `updateState`가 호출됩니다.
    - `updateWorkInProgressHook()`을 호출하여 **기존의 훅 객체**를 가져옵니다.
3. **업데이트 큐 처리**
    - `updateState`에서 `hook.queue`에 쌓인 **업데이트 큐**를 순회합니다.
    - `basicStateReducer`가 실행되어 새로운 상태를 계산합니다.
        
        ```tsx
        function basicStateReducer(state, action) {
          return typeof action === 'function' ? action(state) : action;
        }
        ```
        
4. **새로운 상태 설정**
    - 새로 계산된 상태(`newState`)가 `hook.memoizedState`와 `queue.lastRenderedState`에 저장됩니다.
5. **컴포넌트 리렌더링 트리거**
    - `scheduleUpdateOnFiber` 함수가 호출되어 해당 Fiber가 업데이트 큐에 추가됩니다.
    - React는 이 Fiber를 다시 렌더링합니다.
6. **`renderWithHooks`가 다시 호출됨**
    - 이번에는 `updateState`가 호출됩니다 (초기 렌더링 시 `mountState`와 다름).
    - 이전에 저장된 상태와 `dispatch` 함수가 반환됩니다.
7. **컴포넌트 리렌더링 완료**
    - 새로운 상태(`count`)가 반영된 컴포넌트가 렌더링됩니다.

<br/>

### 정리

![](https://velog.velcdn.com/images/njt6419/post/7443394a-08ee-4bde-9df0-a916b87d43fe/image.png)

- hook은 Fiber 객체에 memoizedState 프로퍼티 키값으로 저장됩니다.
- hook은 여러개를 사용할 수 있으며, 여러가지 값을 가지도록 관리됩니다. 이를 위해 hook은 링크드리스트로 구현됩니다.
- current(이미 존재하는 Virtual DOM) 여부에 따라 hooks의 구현체(mount, update)가 달라지게됩니다.
- 이걸 연결하는 객체가 ReactCurrentDispatcher.curent입니다.

<br/>

### 참고 자료

[진짜 리액트는 어떻게 생겼나(1) - useState 따라가며 흐름잡기](https://0422.tistory.com/321)

[진짜 리액트는 어떻게 생겼나(2) - renderWithHooks와 훅의 본체](https://0422.tistory.com/322?category=1139977)

[[김용찬 | 위키북스] 모던 리액트 Deep Dive](https://wikibook.co.kr/react-deep-dive/)