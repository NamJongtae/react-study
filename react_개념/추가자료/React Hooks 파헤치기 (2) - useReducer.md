useReducer 구현채를 따라가며 useReducer가 어떻게 구현되어 있는지 살펴보겠습니다.

앞서 소개한 [React Hooks 파헤치기 (1)](https://www.notion.so/React-Hooks-1-useState-Hooks-14112b902111809cb25ec60b55be523e?pvs=21)에서 소개한 대로 useState가 동작하는 것과 유사하게 useReducer가 동작하게됩니다.  hooks의 동작 방식은 앞서 소개한 것과 동일하기 때문에 renderWithHooks 함수에서 `ReactCurrentDispatcher.current`가 마운트, 업데이트되는 부분에서 부터 살펴보겠습니다.

### renderWithHooks

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

	// current 설정 부분 마운트 시점과 업데이트 시점에 따라 변경됨
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

`ReactCurrentDispatcher.current`에 마운트 되는 시점에서 `HookDispatcherOnMount`가 업데이트 되는 시점에는 `HookDispatcherOnUpdate` 함수가 호출됩니다. 

```jsx
const HooksDispatcherOnMount: Dispatcher = {
  readContext,

  useCallback: mountCallback,
  useContext: readContext,
  useEffect: mountEffect,
  useImperativeHandle: mountImperativeHandle,
  useLayoutEffect: mountLayoutEffect,
  useMemo: mountMemo,
  useReducer: mountReducer, // mountReducer가 전달됨
  useRef: mountRef,
  useState: mountState,
  useDebugValue: mountDebugValue,
  useResponder: createResponderListener,
  useDeferredValue: mountDeferredValue,
  useTransition: mountTransition,
};

const HooksDispatcherOnUpdate: Dispatcher = {
  readContext,

  useCallback: updateCallback,
  useContext: readContext,
  useEffect: updateEffect,
  useImperativeHandle: updateImperativeHandle,
  useLayoutEffect: updateLayoutEffect,
  useMemo: updateMemo,
  useReducer: updateReducer, // updateReducer가 전달됨
  useRef: updateRef,
  useState: updateState,
  useDebugValue: updateDebugValue,
  useResponder: createResponderListener,
  useDeferredValue: updateDeferredValue,
  useTransition: updateTransition,
};
```

`HooksDispatcherOnMount` (마운트 시점) 호출 시 useReducer로  `mountReducer`가 `HooksDispatcherOnUpdate` (업데이트 시점) 호출 시 `updateReducer`가 전달됩니다.

<br/>

### mountReducer

```jsx
function mountReducer<S, I, A>(
  reducer: (S, A) => S, // 상태를 업데이트하는 리듀서 함수
  initialArg: I,       // 초기 상태 값 또는 초기화 함수에 전달할 값
  init?: I => S,       // 선택적으로 초기 상태를 가공할 수 있는 함수
): [S, Dispatch<A>] {  // 현재 상태와 디스패치 함수의 배열 반환
  // React 훅의 현재 작업 중인 Hook을 가져옵니다.
  const hook = mountWorkInProgressHook();

  // 초기 상태 변수
  let initialState;

  // init 함수가 정의되어 있다면, initialArg를 가공하여 초기 상태를 설정
  if (init !== undefined) {
    initialState = init(initialArg);
  } else {
    // init 함수가 없으면 initialArg 자체를 초기 상태로 사용
    initialState = ((initialArg: any): S);
  }

  // Hook에 메모리화된 상태와 기본 상태를 설정
  hook.memoizedState = hook.baseState = initialState;

  // 상태 업데이트를 관리하는 큐를 생성
  const queue = (hook.queue = {
    last: null,                       // 가장 최근의 업데이트를 저장
    dispatch: null,                   // 액션을 디스패치할 함수
    lastRenderedReducer: reducer,     // 마지막으로 렌더링에 사용된 리듀서
    lastRenderedState: (initialState: any), // 마지막으로 렌더링된 상태
  });

  // Dispatch 함수 생성
  const dispatch: Dispatch<A> = (queue.dispatch = (dispatchAction.bind(
    null,
    // 현재 렌더링 중인 Fiber를 전달 (React 내부 상태 관리와 연결)
    ((currentlyRenderingFiber: any): Fiber),
    queue, // 상태 업데이트 큐를 전달
  ): any));

  // 현재 상태와 디스패치 함수를 반환
  return [hook.memoizedState, dispatch];
}

```

- `mountReducer`는 React의 `useReducer` 훅과 관련된 내부 구현 함수입니다.
- 초기 상태 설정, 상태 업데이트 관리, 그리고 디스패치 함수를 반환하는 역할을 합니다.
- 인자값
    - `reducer`: 상태를 업데이트하기 위한 리듀서 함수로, `(state, action) => newState` 형태입니다.
    - `initialArg`: 초기 상태 값 또는 초기화 함수 `init`에 전달할 값입니다.
    - `init`: 선택적 초기화 함수로, 초기 상태를 가공하기 위해 사용됩니다. 예를 들어, 복잡한 초기 상태 계산이 필요한 경우 유용합니다.

`mountReducer` 함수 코드를 나누어 살펴보겠습니다.


<br/>

**mountWorkInProgressHook 호출**

```jsx
// React 훅의 현재 작업 중인 Hook을 가져옵니다.
const hook = mountWorkInProgressHook();
```

`mountWorkInProgressHook`을 호출하여 현재 컴포넌트에서 작업 중인 훅을 가져옵니다.

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

**initalState 설정**

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

**dispatch(reducer dispatch 함수)**

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

### updateReducer

updateReducer useState와 동일하게 사용되므로 앞서 설명한 [React Hooks 파헤치기 (1) - updateReducer](https://www.notion.so/React-Hooks-1-useState-Hooks-14112b902111809cb25ec60b55be523e?pvs=21)를 참고해주세요.

아래 이미지는 useReducer 전체 실행 과정을 간단하게 나타낸 흐름도입니다.

![](https://velog.velcdn.com/images/njt6419/post/4a1b687d-f3d7-487b-9fe9-4d910d3af55d/image.png)

<br/>

### 예시를 통해 전체 과정 살펴보기

```jsx
import { useReducer } from "react";

// 리듀서 함수 정의
function reducer(state, action) {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 };
    case "decrement":
      return { count: state.count - 1 };
    default:
      throw new Error("Unhandled action type");
  }
}

function App() {
  // useReducer 초기화
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  const increment = () => {
    dispatch({ type: "increment" }); // 액션 디스패치
  };

  const decrement = () => {
    dispatch({ type: "decrement" }); // 액션 디스패치
  };

  return (
    <div>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <p>Count: {state.count}</p>
    </div>
  );
}

```

<br/>

**첫 렌더링 시(`mount`)**

![](https://velog.velcdn.com/images/njt6419/post/84a8e420-7d6d-443a-a238-05d46d553223/image.png)


1. **JSX 컴파일**

- `App.jsx`가 Babel에 의해 `React.createElement` 호출로 변환됩니다.
- 컴파일 결과는 React 엘리먼트 객체입니다.


<br/>

2. **React 엘리먼트 객체 → Fiber 노드 확장**

- React 엘리먼트 객체가 **Fiber 노드**로 확장됩니다.
- Fiber 노드는 컴포넌트의 상태와 업데이트를 관리합니다.


<br/>

3. **`renderWithHooks` 호출**

- 컴포넌트가 렌더링될 때, React 내부적으로 `renderWithHooks`가 호출됩니다.
- 현재 Fiber에 연결된 훅을 초기화하는 **마운트 단계**로 진입합니다.


<br/>

4. **`mountReducer` 호출 (`useReducer` 초기화)**

- `useReducer` 훅이 호출되면 React는 `mountReducer`를 실행합니다.

```jsx
function mountReducer(reducer, initialArg, init) {
  const hook = mountWorkInProgressHook(); // 새로운 훅 객체 생성
  const initialState = init !== undefined ? init(initialArg) : initialArg;
  hook.memoizedState = initialState; // 초기 상태 저장
  hook.queue = {                     // 업데이트 큐 초기화
    last: null,
    dispatch: null,
    lastRenderedReducer: reducer,
    lastRenderedState: initialState,
  };
  const dispatch = (action) => dispatchAction(currentlyRenderingFiber, hook.queue, action);
  hook.queue.dispatch = dispatch;   // 디스패치 함수 설정
  return [hook.memoizedState, dispatch];
}
```

- `reducer`: 상태를 업데이트하는 리듀서 함수입니다.
- `initialArg`: 초기 상태입니다. 여기서는 `{ count: 0 }`가 전달됩니다.
- `hook.memoizedState`: 초기 상태가 저장됩니다.
- `hook.queue`: 상태 업데이트를 관리하는 큐가 초기화됩니다.
- `dispatch`: 디스패치 함수가 생성되어 상태 변경을 트리거할 준비를 합니다.


<br/>

5. **`mountReducer` 반환**

- `mountReducer`는 현재 상태와 `dispatch` 함수를 반환합니다.
- 즉, `[state, dispatch] = [{ count: 0 }, dispatch]`가 됩니다.


<br/>

6. **컴포넌트가 렌더링되어 `App` 함수 종료**

- Fiber 트리를 기반으로 React는 **React DOM 렌더러**를 통해 브라우저의 실제 DOM을 업데이트합니다.
- 이 단계에서 컴포넌트의 **최종 렌더링 결과가 브라우저 화면에 표시**됩니다.

<br/>

**상태 업데이트 (`dispatch`)**

![](https://velog.velcdn.com/images/njt6419/post/a489614c-26f8-43f5-9eb5-97d12124c272/image.png)


1. **`increment` 또는 `decrement` 함수 호출**

- `dispatch({ type: "increment" })` 혹은 `dispatch({ type: "decrement" })`가 호출됩니다.
- 디스패치된 액션은 `dispatchAction` 함수로 전달됩니다.

```jsx
function dispatchAction(fiber, queue, action) {
  const update = { action, next: null };
  const last = queue.last;
  if (last === null) {
    // 큐가 비어 있다면 새로운 업데이트 추가
    queue.last = update.next = update;
  } else {
    // 큐가 비어 있지 않다면 연결 리스트에 추가
    const first = last.next;
    last.next = update;
    update.next = first;
    queue.last = update;
  }
  scheduleUpdateOnFiber(fiber); // 해당 Fiber를 다시 렌더링하도록 예약
}
```

- 디스패치된 액션이 `queue`의 업데이트 리스트에 추가됩니다.


<br/>

2. **`updateReducer` 호출**

- 다시 렌더링이 시작되면, React는 `updateReducer`를 호출하여 기존 훅의 상태를 업데이트합니다.

```jsx
function updateReducer(reducer, initialArg, init) {
  const hook = updateWorkInProgressHook(); // 기존 훅 객체 가져오기
  const queue = hook.queue;
  let newState = hook.memoizedState;

  // 업데이트 큐 순회
  let update = queue.last?.next;
  do {
    const action = update.action;
    newState = reducer(newState, action); // 리듀서 실행
    update = update.next;
  } while (update !== queue.last?.next);

  hook.memoizedState = newState; // 새로운 상태 저장
  return [newState, queue.dispatch];
}
```

- **업데이트 큐**
    - 큐에 쌓인 모든 업데이트(액션)를 순차적으로 처리하며 새로운 상태를 계산합니다.
    - `reducer`를 호출하여 현재 상태와 액션을 기반으로 새로운 상태를 계산합니다.
- **상태 저장**
    - 계산된 새로운 상태가 `hook.memoizedState`에 저장됩니다.
- **새로운 상태 및 dispatch 함수 반환**
    - `[새로운 상태, dispatch 함수]`를 반환합니다.

<br/>

3. **컴포넌트 리렌더링**

- 새로운 상태(`{ count: 1 }` 등)가 적용된 상태로 컴포넌트가 다시 렌더링됩니다.
- JSX가 변환되어 DOM이 업데이트됩니다.