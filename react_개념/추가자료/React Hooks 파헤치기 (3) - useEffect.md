useEffect hook은 `ReactCurrentDispatcher.current`는 마운트시 `HooksDispatcherOnMount` 함수를 호출하며 useEffect hook에 `mountEffect` 함수가 전달되고, 업데이트시 `HooksDispatcherOnUpdate` 함수를 호출하며 useEffect hook에 `updateEffect` 함수를 전달합니다.

`ReactCurrentDispatcher.current` 에 대한 내용은 이전 글 [React Hooks 파헤치기 (1) - useState Hook의 구현채](https://velog.io/@njt6419/React-Hooks-%ED%8C%8C%ED%97%A4%EC%B9%98%EA%B8%B0-1-useState-hooks-%EA%B5%AC%ED%98%84%EC%B1%84)를 참고해주세요.

아래는 react 공식 [react-renconclier](https://github.com/facebook/react/blob/v16.12.0/packages/react-reconciler/src/ReactFiberHooks.js#L923) 소스중 일부 코드입니다.

아래 코드를 보면 마운트시 `mountEffect` 함수가 업데이트시 `updateEffect` 함수가 전달되는 것을 볼 수 있습니다.
```jsx
const HooksDispatcherOnMount: Dispatcher = {
  readContext,

  useCallback: mountCallback,
  useContext: readContext,
  useEffect: mountEffect, // mountEffect가 전달됨
  ...
};

const HooksDispatcherOnUpdate: Dispatcher = {
  readContext,

  useCallback: updateCallback,
  useContext: readContext,
  useEffect: updateEffect, // updateEffect가 전달됨
  ...
};
```

<br/>

### mountEffect

`mountEffect`는 `useEffect`  처리하는 진입점(entry point) 역할을 합니다.

```jsx
function mountEffect(
  create: () => (() => void) | void,
  deps: Array<mixed> | void | null,
): void {
	  mountEffectImpl(
	    PassiveEffect | PassiveStaticEffect, // Effect의 동작 방식을 정의
	    HookPassive, // 비동기적으로 실행되는 Effect
	    create,  // 사용자가 전달한 effect 함수입니다.
	    deps,    // 의존성 배열입니다.
	  );
}
```

- `create`
    - 이 함수는 React의 effect 실행 내용을 정의합니다.
    - 만약 `create` 함수가 cleanup 함수를 반환하면, 이 함수는 컴포넌트가 재렌더링되거나 언마운트될 때 호출됩니다.
- `deps`
    - 의존성 배열입니다. 주로 특정 값들이 변경되었을 때만 effect가 다시 실행되도록 제어합니다.
    - `null`이나 `void`가 전달되면, effect는 매 렌더링마다 실행됩니다.
- `mountEffectImpl`
    - `mountEffectImpl`는 실제로 effect를 적용하는 내부 구현을 호출합니다.
- `PassiveEffect | PassiveStaticEffect`는 React 내부에서 **Fiber 렌더링 및 훅 시스템**이 동작할 때 사용하는 **효과 태그(Effect Tag)**
입니다.
- `PassiveEffect`는 렌더링이 완료된 후 **비동기적**으로 실행되는 작업을 나타냅니다.
- `HookPassive`는 비동기적으로 실행되는 Effect를 나타냅니다.  비동기적인 Effect의 실행 시점과 클린업을 관리하는 역할을 수행합니다.

<br/>

💡 **PassiveEffect vs PassiveStaticEffect**

**PassiveEffect**
`PassiveEffect` 플래그는 React에서 `useEffect` 훅의 실행과 관련이 있습니다.

- 화면 업데이트 이후 실행해야 하는 **비동기 작업**에 주로 사용됩니다.
- 예를 들어, 데이터 요청, 타이머 설정, 이벤트 리스너 등록 등이 여기에 포함됩니다.
- `useEffect`와 연결되어 있어 브라우저의 **페인팅(rendering)** 이후에 실행됩니다.
- Fiber는 컴포넌트가 렌더링된 이후에 `PassiveEffect`가 설정된 작업을 실행합니다.
- 이 작업은 브라우저의 메인 스레드를 차단하지 않으며, 비동기적으로 수행됩니다.

**PassiveStaticEffect**

`PassiveStaticEffect` 플래그는 의존성 배열이 빈 배열인 Effect에 해당합니다.

- 이 플래그는 의존성 배열이 없거나 빈 배열인 경우, 즉 Effect가 처음 마운트될 때만 실행되며, 이후에는 재실행되지 않도록 설정됩니다. 이는 특정한 설정이나 초기화 작업을 수행하는 데 유용합니다.

<br/>

### mountEffectImpl

Effect를 설정하고 관리하는 역할을 수행합니다.

```jsx
function mountEffectImpl(fiberFlags, hookFlags, create, deps): void {
  // 현재 Hook 정보를 가져옴
  const hook = mountWorkInProgressHook();
  
  // 의존성 배열 처리
  const nextDeps = deps === undefined ? null : deps;
 
  // Fiber의 Effect 플래그를 설정
  currentlyRenderingFiber.flags |= fiberEffectTag;
  
   // Hook의 상태에 Effect를 저장
  hook.memoizedState = pushEffect(HookHasEffect | hookFlags, create, undefined, nextDeps);
}
```

- `fiberFlags`
    - `fiberFlags`는 현재 Fiber 노드에 어떤 **side effect**가 필요한지를 나타내는 플래그입니다.
    - 이 값은 React가 컴포넌트의 렌더링, 업데이트, 그리고 cleanup 과정을 제어할 수 있도록 도와줍니다. 
- `HookHasEffect | hookFlags`
    - `HookHasEffect` 플래그는 deps가 변경될 때 Effect를 실행해야 함을 표시하는 플래그입니다.
    - `hookFlags` 는 현재 훅의 상태를 나타내는 플래그입니다.  이 플래그는 여러 상태를 조합하여 나타낼 수 있으며, 예를 들어, Passive, Layout 등 다양한 Effect 타입을 포함할 수 있습니다.  
- `create`
    - `useEffect`에서 전달받는 콜백 함수로 훅의 동작을 정의합니다.
    - 반환 값은 cleanup 함수거나 `undefined`입니다.
- `deps`
    - 의존성 배열입니다.
    - 이 배열을 통해 훅이 다시 실행될 조건을 지정합니다.
    - `undefined` 또는 `null`이면 매 렌더링마다 실행됩니다. 
- `pushEffect`
    - 새로운 효과를 생성하고 `hook.memoizedState`에 저장합니다.
    - `tag`: Effect의 종류와 실행 방식을 나타내는 태그.
    - `create`: 사용자 정의 effect 함수.
    - `undefined`: cleanup 함수는 아직 정의되지 않았습니다. cleanup 함수는 훅이 재실행될 때 설정됩니다.
    - `nextDeps`: 의존성 배열입니다.

<br/>

### pushEffect

```jsx
function pushEffect(tag, create, destroy, deps) {
  const effect: Effect = {
    tag,
    create,
    destroy,
    deps,
    // Circular
    next: (null: any),
  };
  if (componentUpdateQueue === null) {
    componentUpdateQueue = createFunctionComponentUpdateQueue();
    componentUpdateQueue.lastEffect = effect.next = effect;
  } else {
    const lastEffect = componentUpdateQueue.lastEffect;
    if (lastEffect === null) {
      componentUpdateQueue.lastEffect = effect.next = effect;
    } else {
      const firstEffect = lastEffect.next;
      lastEffect.next = effect;
      effect.next = firstEffect;
      componentUpdateQueue.lastEffect = effect;
    }
  }
  return effect;
}
```

위 `pushEffect` 함수를 나누어 살펴보겠습니다.

**Effect 객체 생성**

```jsx
const effect: Effect = {
  tag,          // Effect 태그: useffect를 실행해야 하는지 여부를 표시
  create,       // 사용자가 정의한 Effect 함수
  destroy,      // cleanup 함수: 기존 Effect 제거용
  deps,         // 의존성 배열
  next: (null: any), // 순환 연결 리스트에서 다음 Effect를 가리킴 (초기값 null)
};
```

새로운 `Effect` 객체를 생성하여 훅의 동작을 정의합니다. 여기서 정의된 effect은 hook.memoizedState에 저장되며, effect의 관리를 위해 사용됩니다.

<br/>

**업데이트 큐가 없는 경우 초기화**

```jsx
if (componentUpdateQueue === null) {
  // 업데이트 큐를 새로 생성
  componentUpdateQueue = createFunctionComponentUpdateQueue();

  // 새로 생성한 Effect를 리스트의 시작과 끝으로 설정
  componentUpdateQueue.lastEffect = effect.next = effect;
}
```

`componentUpdateQueue`가 없을 경우 `createFunctionComponentUpdateQueue` 함수를 호출하여 새로 생성하고, 첫 번째 `Effect`를 큐의 시작과 끝으로 설정합니다. 

`componentUpdateQueue`는 링크드리스트 구조로 컴포넌트에 대한 모든 Effect를 추적합니다.

react는 `componentUpdateQueue`를 참조하여 **Effect 객체**를 관리하게됩니다.

새로 생성된 Effect 객체가 자신의 `next`를 자신으로 설정하여 리스트 구조를 만듭니다.

`createFunctionComponentUpdateQueue` 함수는 `useEffect`  훅이 실행될 때 필요한 정보를 저장하는 큐를 생성합니다. 이 큐는 컴포넌트 렌더링 사이클 동안 모든 Effect 객체를 추적하고, Commit 단계에서 필요한 작업을 처리할 수 있도록 합니다.

<br/>

**업데이트 큐가 이미 존재하는 경우**

```jsx
else {
  const lastEffect = componentUpdateQueue.lastEffect; // 현재 큐의 마지막 Effect

  if (lastEffect === null) {
    // 리스트가 비어 있는 경우 새 Effect를 시작과 끝으로 설정
    componentUpdateQueue.lastEffect = effect.next = effect;
  } else {
    // 리스트에 기존 Effect가 있는 경우
    const firstEffect = lastEffect.next; // 기존 리스트의 첫 번째 Effect
    lastEffect.next = effect;            // 기존 마지막 Effect의 다음을 새 Effect로 설정
    effect.next = firstEffect;           // 새 Effect의 다음을 첫 번째 Effect로 연결
    componentUpdateQueue.lastEffect = effect; // 새 Effect를 리스트의 마지막으로 갱신
  }
}
```

`componentUpdateQueue`가 이미 존재하면 새 Effect를 리스트에 추가하고 순환 구조를 유지합니다.

- **기존 리스트 연결**
    - 현재 마지막 Effect(`lastEffect`)의 `next`를 새 Effect로 연결합니다.
    - 새 Effect의 `next`를 기존 첫 번째 Effect로 연결합니다.
- **마지막 Effect 갱신**
    - 새 Effect를 큐의 마지막 Effect(`lastEffect`)로 설정.

**Effect 반환**

```jsx
return effect;
```

반환된 effect 값은 React 내부에서 관리되며, 렌더링 및 훅 실행 흐름에서 사용됩니다.

<br/>

### updateEffect

`useEffect` 훅을 실행하거나 초기화(마운트) 및 클린업(언마운트) 작업을 수행합니다. 

```jsx
function updateEffect(
  create: () => (() => void) | void,
  deps: Array<mixed> | void | null,
): void {
  // 실질적인 Effect 처리 구현 함수를 호출.
  // updateEffectImpl은 Effect를 실행, 업데이트, 또는 클린업 작업을 처리합니다.
  updateEffectImpl(
	  PassiveEffect, // 비동기적으로 실행되는 작업
	  HookPassive, // 비동기적으로 실행되는 Effect
    create, // 실행할 사용자 정의 콜백 함수.
    deps, // 의존성 배열.
  );
}
```

- `PassiveEffect`는 렌더링이 완료된 후 **비동기적**으로 실행되는 작업을 나타냅니다.
- `HookPassive`는 비동기적으로 실행되는 Effect를 나타냅니다.  비동기적인 Effect의 실행 시점과 클린업을 관리하는 역할을 수행합니다.
- **create :** `useEffect`에서 전달받는 콜백 함수입니다.
- **deps :** 의존성 배열로, 이 배열 내 값들이 변경될 때만 Effect가 다시 실행됩니다.
- `updateEffectImpl` 함수는 실제로 Effect를 설정하고 갱신하는 핵심 구현 함수입니다.
    - 전달된 인자들은 React가 Effect를 실행, 갱신, 또는 클린업할 시점을 정의하는 데 사용됩니다.

<br/>

### updateEffectImpl

실제로 Effect를 설정하고 갱신하는 핵심 구현 함수입니다.

```jsx
function updateEffectImpl(fiberFlags, hookFlags, create, deps) {
  // 현재 작업 중인 Hook을 가져옴. 이 Hook은 `workInProgress`에 연결됨.
  const hook = updateWorkInProgressHook();

  // 의존성 배열이 없으면 `nextDeps`는 null로 설정.
  const nextDeps = deps === undefined ? null : deps;
  let destroy = undefined; // 이전 cleanup 함수 초기화.

  // `currentHook`이 null이 아니면 (즉, 이 Hook이 이미 한 번 실행된 적이 있다면),
  if (currentHook !== null) {
    // 이전의 `effect` 상태를 가져옴.
    const prevEffect = currentHook.memoizedState;
    destroy = prevEffect.destroy; // 이전 cleanup 함수 참조.

    // 새로운 의존성 배열이 존재한다면,
    if (nextDeps !== null) {
      const prevDeps = prevEffect.deps; // 이전 의존성 배열을 가져옴.

      // 이전 의존성과 새로운 의존성을 비교.
      if (areHookInputsEqual(nextDeps, prevDeps)) {
        // 의존성이 동일하면 새로운 효과를 생성하지 않고 기존 효과를 유지.
        pushEffect(hookFlags, create, destroy, nextDeps);
        return;
      }
    }
  }

  // 의존성이 다르거나 처음 실행되는 경우, 새로운 효과를 스케줄링.
  sideEffectTag |= fiberEffectTag; // Fiber의 side effect 태그 갱신.
  
  // Hook의 상태를 새로운 효과로 업데이트.
  // HooksHasEffect는 deps가 변경될 때 Effect를 실행해야 함을 표시하는 플래그.
  hook.memoizedState = pushEffect(HookHasEffect | hookFlags, create, destroy, nextDeps);
}
```

위 `updateEffectImpl` 함수를 나누어 살펴보겠습니다.

**`updateWorkInProgressHook` 호출**

```jsx
const hook = updateWorkInProgressHook();
```

- 현재 진행 중인 Hook을 가져옵니다.

<br/>

**의존성 배열 확인 및 초기화**

```jsx
const nextDeps = deps === undefined ? null : deps;
```

- 의존성 배열(`deps`)이 제공되지 않으면 `null`로 설정합니다.
- 이는 의존성이 없는 경우에도 코드가 예상대로 동작하도록 하기 위함입니다.

<br/>

**기존 Hook이 있는 경우 처리**

```jsx
if (currentHook !== null) {
  const prevEffect = currentHook.memoizedState;
  destroy = prevEffect.destroy;
  
  if (nextDeps !== null) {
    const prevDeps = prevEffect.deps;
    if (areHookInputsEqual(nextDeps, prevDeps)) {
      pushEffect(hookFlags, create, destroy, nextDeps);
      return;
    }
  }
}
```

- **기존 상태 확인**
    - `currentHook`이 존재하면 이전 렌더링 시 설정된 `effect`의 상태를 확인합니다.
    - 기존 `effect`의 `destroy` (cleanup 함수)를 저장해, 필요 시 새 효과를 생성하지 않고 기존 것을 유지합니다.
- **의존성 배열 비교**
    - `areHookInputsEqual` 함수로 이전 의존성과 새 의존성을 비교합니다.
    - 의존성이 동일하면 기존 효과를 재활용하고 새로 설정하지 않습니다.

<br/>

**새로운 Effect 등록**

```jsx
sideEffectTag |= fiberEffectTag;
hook.memoizedState = pushEffect(HookHasEffect | hookFlags, create, destroy, nextDeps);
```

- **Fiber의 side effect 태그 업데이트**
    - `sideEffectTag`는 React Fiber가 어떤 작업을 수행해야 하는지 나타냅니다.
    - `|=` 연산자는 기존 태그에 새로운 effect tag(`fiberEffectTag`)를 추가합니다.
- **Hook 상태 업데이트**
    - 새로운 effect를 `pushEffect`로 생성하여 상태를 업데이트합니다.
    - `pushEffect`는 새 효과를 효과 리스트에 추가하며, 클린업 함수(`destroy`)와 의존성 배열(`nextDeps`)도 함께 저장합니다.

<br/>

### commitRoot

useEffect는 fiber 노드에 단순히 데이터 구조를 생성하게됩니다.

그럼 실제 Effect 객체가 처리되는 부분은 어디일까요?

바로 `commitRoot` 함수에서 처리되게됩니다.

초기 렌더링시에는 커밋 단계에서 `commitPassiveMountEffects` 함수를 호출하여 Effect 객체가 처리됩니다. 

이후 리렌더링이 발생하면 React는 두 개의 Fiber 트리(렌더링 이전과 렌더링 이후)를 비교하여 변경된 부분을 확인합니다. 이 과정을 통해 필요한 재조정 작업을 수행한 후, 커밋 단계에서 변경점을 실제 DOM에 반영합니다. 이때, 이전에 등록된 passive effects의 클린업을 처리하기 위해 `commitPassiveUnmountEffects`를 먼저 호출한 후, 새로운 passive effects를 실행하기 위해 `flushPassiveEffects` 함수를 호출합니다.

```jsx
function commitRoot(root) {
  const renderPriorityLevel = getCurrentPriorityLevel();
  runWithPriority(
    ImmediateSchedulerPriority,
    commitRootImpl.bind(null, root, renderPriorityLevel),
  );
  return null;
}

function commitRootImpl(
  root: FiberRoot,
  recoverableErrors: null | Array<CapturedValue<mixed>>,
  transitions: Array<Transition> | null,
  renderPriorityLevel: EventPriority,
) {
  if (
    (finishedWork.subtreeFlags & PassiveMask) !== NoFlags ||
    (finishedWork.flags & PassiveMask) !== NoFlags
  ) {
    if (!rootDoesHavePassiveEffects) {
      rootDoesHavePassiveEffects = true;
      pendingPassiveEffectsRemainingLanes = remainingLanes;
      pendingPassiveTransitions = transitions;
      scheduleCallback(NormalSchedulerPriority, () => {
        flushPassiveEffects();
        return null;
      });
    }
  }
  ...
}
```

<br/>

### **flushPassiveEffects**

```jsx
function flushPassiveEffectsImpl() {
  // 대기 중인 passive effects가 없으면 false를 반환
  if (rootWithPendingPassiveEffects === null) {
    return false;
  }

  const transitions = pendingPassiveTransitions; // 대기 중인 전환을 저장
  pendingPassiveTransitions = null; // 대기 중인 전환을 초기화

  const root = rootWithPendingPassiveEffects; // 현재 root를 가져옴
  const lanes = pendingPassiveEffectsLanes; // 대기 중인 passive effects의 lanes를 저장
  rootWithPendingPassiveEffects = null; // 대기 중인 root를 초기화
  pendingPassiveEffectsLanes = NoLanes; // 대기 중인 lanes를 초기화

  const prevExecutionContext = executionContext; // 이전 실행 컨텍스트를 저장
  executionContext |= CommitContext; // 현재 실행 컨텍스트를 CommitContext로 설정

  // 이전 passive effects의 언마운트 함수를 실행
  commitPassiveUnmountEffects(root.current);
  
  // 새로운 passive effects의 마운트 함수를 실행
  commitPassiveMountEffects(root, root.current, lanes, transitions);
  
  // ...
}

```

**useEffect는 먼저 이전 언마운트 함수를 실행한 뒤 새로운 effect 함수를 실행하게됩니다.**

여기서 역시 `commitPassiveUnmountEffects`를 먼저 호출하고 `commitPassiveMountEffects`가 호출됩니다.

`commitPassiveUnmountEffects`는 이전에 등록된 **passive effects**의 클린업을 처리하는 것입니다. 이는 자원 정리를 통해 메모리 누수를 방지하고, 새로운 효과가 실행될 준비를 하는 단계입니다.

`commitPassiveMountEffects`는 새로운 **passive effects**를 실행하는 함수로, 이 단계에서는 최신 상태에 기반하여 효과를 적용합니다. 따라서, 이전 효과의 정리가 끝난 후에 새로운 효과가 적용되는 것이 중요합니다.

<br/>

### commitPassiveUnmountEffects

```jsx
export function commitPassiveUnmountEffects(finishedWork: Fiber): void {
  // 현재 디버그 모드에서 finishedWork를 설정
  setCurrentDebugFiberInDEV(finishedWork);
  
  // 주어진 Fiber에 대해 passive 언마운트 효과를 처리
  commitPassiveUnmountOnFiber(finishedWork);
  
  // 디버그 모드에서 현재 Fiber를 초기화
  resetCurrentDebugFiberInDEV();
}

function commitPassiveUnmountOnFiber(finishedWork: Fiber): void {
  switch (finishedWork.tag) {
    case FunctionComponent:
    case ForwardRef:
    case SimpleMemoComponent: {
      // children의 effect가 먼저 clean up
      // 자식 컴포넌트의 passive 언마운트 효과를 재귀적으로 처리
      recursivelyTraversePassiveUnmountEffects(finishedWork);

      // passive 플래그가 설정된 경우, 해당 Hook의 언마운트 효과를 실행
      if (finishedWork.flags & Passive) {
        commitHookPassiveUnmountEffects(
          finishedWork,
          finishedWork.return, // nearestMountedAncestor로 부모 Fiber를 전달
          HookPassive | HookHasEffect, //HookHasEffect 플래그는 deps가 변경되지 않으면 callback이 실행x
        );
      }
      break;
    }
    ...
  }
}

function commitHookPassiveUnmountEffects(
  finishedWork: Fiber,
  nearestMountedAncestor: null | Fiber,
  hookFlags: HookFlags,
) {
  // 프로파일링이 활성화된 경우
  if (shouldProfile(finishedWork)) {
    startPassiveEffectTimer(); // passive 효과 타이머 시작
    // Hook의 언마운트 효과를 실행
    commitHookEffectListUnmount(
      hookFlags,
      finishedWork,
      nearestMountedAncestor,
    );
    recordPassiveEffectDuration(finishedWork); // passive 효과의 지속 시간 기록
  } else {
    // 프로파일링이 비활성화된 경우
    commitHookEffectListUnmount(
      hookFlags,
      finishedWork,
      nearestMountedAncestor,
    );
  }
}

```

- `commitPassiveUnmountEffects`
    
    주어진 finishedWork에 대해 passive 언마운트 효과를 처리합니다. 디버그 모드에서 현재 Fiber를 설정하고, 해당 Fiber에 대한 언마운트 효과를 호출한 후, 다시 디버그 모드를 초기화합니다.
    
- `commitPassiveUnmountOnFiber`
    
    특정 Fiber의 태그에 따라 적절한 언마운트 처리를 수행합니다.
    자식 컴포넌트의 passive 언마운트 효과를 재귀적으로 처리하는 recursivelyTraversePassiveUnmountEffects를 호출하여, 먼저 자식의 효과를 정리합니다.
    passive 플래그가 설정된 경우, commitHookPassiveUnmountEffects를 호출하여 해당 Hook의 언마운트 효과를 실행합니다. 이때, HookHasEffect 플래그를 설정하여 dependencies가 변경되지 않으면 callback이 실행되지 않도록 합니다.
    
- `commitHookPassiveUnmountEffects`
    
    주어진 Fiber의 Hook 언마운트 효과를 처리합니다.
    프로파일링이 활성화된 경우, passive 효과 타이머를 시작하고, Hook의 언마운트 효과를 실행한 후 지속 시간을 기록합니다. 프로파일링이 비활성화된 경우에는 단순히 언마운트 효과를 실행합니다.
    
<br/>

### commitHookEffectListUnmount

React는 렌더링이 완료된 후 **커밋 단계**에서 **`commitHookEffectListUnmount`** 함수가 호출되며, 언마운트되거나 의존성이 변경될 때, 클린업 함수를 실행합니다.

```jsx
function commitHookEffectListUnmount(
  flags: HookFlags,
  finishedWork: Fiber,
  nearestMountedAncestor: Fiber | null,
) {
  // Fiber 노드에서 업데이트 큐를 가져옵니다.
  const updateQueue: FunctionComponentUpdateQueue | null = (finishedWork.updateQueue: any);
  
  // 업데이트 큐의 마지막 Effect를 가져옵니다.
  // Effect는 순환 리스트 형태로 저장되어 있습니다.
  const lastEffect = updateQueue !== null ? updateQueue.lastEffect : null;
  
  // 업데이트 큐가 비어있지 않은 경우, Effect를 순회하며 처리합니다.
  if (lastEffect !== null) {
    const firstEffect = lastEffect.next; // 순환 리스트의 첫 번째 Effect를 가져옵니다.
    let effect = firstEffect;.
    // 플래그별로 필요한 것을 필터링합니다.
    do {
	    // Effect의 태그가 주어진 flags와 일치하는 경우 작업을 실행합니다.
      if ((effect.tag & flags) === flags) {
        // Unmount
        const destroy = effect.destroy; // 클린업 함수 가져옵니다.
        effect.destroy = undefined;
        if (destroy !== undefined) {
          safelyCallDestroy(finishedWork, nearestMountedAncestor, destroy); // 클린업 함수를 실행합니다.
        }
      }
      effect = effect.next; // 다음 Effect로 이동합니다.
    } while (effect !== firstEffect); // 순환 리스트를 한 바퀴 돌 때까지 반복합니다.
  }
}
```
위 `commitHookEffectListUnmount` 함수를 나누어 살펴보겠습니다.

**`updateQueue`와 `lastEffect` 가져오기**

```jsx
const updateQueue = finishedWork.updateQueue;
const lastEffect = updateQueue !== null ? updateQueue.lastEffect : null;
```

- `finishedWork.updateQueue`는 해당 Fiber 노드의 업데이트 큐를 참조합니다.
- 이 큐에는 컴포넌트에 등록된 모든 Effect가 저장되어 있습니다.
- `lastEffect`는 순환 리스트의 마지막 Effect를 가리키며, 리스트 탐색의 시작점 역할을 합니다.

<br/>

**Effect 리스트 순회**

```jsx
if (lastEffect !== null) {
  let effect = lastEffect.next; // 리스트의 첫 번째 Effect 가져오기.
  // 플래그별로 필요한 것을 필터링합니다.
  do {
     // Effect의 태그가 주어진 flags와 일치하는 경우 작업을 실행합니다.
    if ((effect.tag & flags) === flags) {
      const destroy = effect.destroy;
      if (destroy !== undefined) {
        safelyCallDestroy(finishedWork, nearestMountedAncestor, destroy); // 클린업 함수 실행.
      }
    }
    effect = effect.next; // 다음 Effect로 이동.
  } while (effect !== lastEffect.next); // 한 바퀴 돌 때까지 반복.
}

```

- `lastEffect`가 존재하면 순환 리스트를 순회합니다.
- `(effect.tag & flags) === flags`를 통해 플래그가 일치하는 경우 클린업 함수를 실행합니다.
- `effect.tag & flags는 Effect`의 tag와 flags를 비트 AND 연산합니다. 이 연산은 두 값의 각 비트가 모두 1일 때만 결과가 1이 됩니다.
- `effect.destroy`는 `useEffect`의 반환값(클린업 함수)을 참조하며, 정의되어 있으면 실행됩니다.
- 리스트의 모든 `effect`를 순회한 후 종료됩니다.

<br/>

### commitPassiveMountEffects

`commitPassiveUnmountEffects` 함수와 비슷하게 처리됩니다.

```jsx
export function commitPassiveMountEffects(
  root: FiberRoot,
  finishedWork: Fiber,
  committedLanes: Lanes,
  committedTransitions: Array<Transition> | null,
): void {
  // 현재 디버그 모드에서 finishedWork를 설정
  setCurrentDebugFiberInDEV(finishedWork);
  
  // 주어진 Fiber에 대한 passive 마운트 효과를 처리
  commitPassiveMountOnFiber(
    root,
    finishedWork,
    committedLanes,
    committedTransitions,
  );

  // 디버그 모드에서 현재 Fiber를 초기화
  resetCurrentDebugFiberInDEV();
}

function commitPassiveMountOnFiber(
  finishedRoot: FiberRoot,
  finishedWork: Fiber,
  committedLanes: Lanes,
  committedTransitions: Array<Transition> | null,
): void {
  // 이 함수를 업데이트할 때는 offscreen 트리가 hidden -> visible로 전환되거나
  // 숨겨진 트리 내에서 효과를 토글할 때 reconnectPassiveEffects도 업데이트해야 합니다.
  const flags = finishedWork.flags;
  
  switch (finishedWork.tag) {
    case FunctionComponent:
    case ForwardRef:
    case SimpleMemoComponent: {
      // Children의 Effect가 먼저 실행
      // 자식 컴포넌트의 passive 마운트 효과를 재귀적으로 처리
      recursivelyTraversePassiveMountEffects(
        finishedRoot,
        finishedWork,
        committedLanes,
        committedTransitions,
      );

      // passive 플래그가 설정된 경우, 해당 Hook의 마운트 효과를 실행
      if (flags & Passive) {
        commitHookPassiveMountEffects(
          finishedWork,
          // HookHasEffect는 deps가 변경되지 않으면 콜백이 실행되지 않도록 합니다.
          HookPassive | HookHasEffect,
        );
      }
      break;
    }
    // ... 다른 케이스 처리
  }
}

function commitHookPassiveMountEffects(
  finishedWork: Fiber,
  hookFlags: HookFlags,
) {
  // 프로파일링이 활성화된 경우
  if (shouldProfile(finishedWork)) {
    startPassiveEffectTimer(); // passive 효과 타이머 시작
    try {
      // Hook의 마운트 효과를 실행
      commitHookEffectListMount(hookFlags, finishedWork);
    } catch (error) {
      // 에러 발생 시 커밋 단계의 에러를 캡처
      captureCommitPhaseError(finishedWork, finishedWork.return, error);
    }
    recordPassiveEffectDuration(finishedWork); // passive 효과의 지속 시간 기록
  } else {
    try {
      // Hook의 마운트 효과를 실행
      commitHookEffectListMount(hookFlags, finishedWork);
    } catch (error) {
      // 에러 발생 시 커밋 단계의 에러를 캡처
      captureCommitPhaseError(finishedWork, finishedWork.return, error);
    }
  }
}
```

- `commitPassiveMountEffects`
    
    주어진 finishedWork에 대해 passive 마운트 효과를 처리합니다.
    디버그 모드에서 현재 Fiber를 설정하고, 해당 Fiber에 대한 마운트 효과를 호출한 후, 다시 디버그 모드를 초기화합니다.
    
- `commitPassiveMountOnFiber`
    
    특정 Fiber의 태그에 따라 적절한 마운트 처리를 수행합니다.
    자식 컴포넌트의 passive 마운트 효과를 재귀적으로 처리하는 `recursivelyTraversePassiveMountEffects`를 호출하여, 먼저 자식의 효과를 적용합니다.
    passive 플래그가 설정된 경우, `commitHookPassiveMountEffects`를 호출하여 해당 Hook의 마운트 효과를 실행합니다. 이때, **HookHasEffect 플래그**를 설정하여 **dependencies**가 변경되지 않으면 callback이 실행되지 않도록 합니다.
    
- `commitHookPassiveMountEffects`
    
    Fiber의 Hook 마운트 효과를 처리합니다.
    프로파일링이 활성화된 경우, passive 효과 타이머를 시작하고, Hook의 마운트 효과를 실행한 후 지속 시간을 기록합니다. 프로파일링이 비활성화된 경우에는 단순히 마운트 효과를 실행합니다. 에러 발생 시, 해당 에러를 캡처하여 처리합니다.
    
<br/>

### commitHookEffectListMount

React는 렌더링이 완료된 후 **커밋 단계**에서 `commitHookEffectListMount` 함수를 호출하여 마운트된 Effect를 실행합니다.

```jsx
function commitHookEffectListUnmount(
  flags: HookFlags,
  finishedWork: Fiber,
  nearestMountedAncestor: Fiber | null,
) {
  const updateQueue: FunctionComponentUpdateQueue | null =
    (finishedWork.updateQueue: any);
  const lastEffect = updateQueue !== null ? updateQueue.lastEffect : null;
  if (lastEffect !== null) {
    const firstEffect = lastEffect.next;
    let effect = firstEffect;
    // updateQueue의 모든 Effect를 반복하고,
    // 플래그별로 필요한 것을 필터링하면 됩니다.
    do {
      // 현재 Effect의 tag가 주어진 flags와 일치하는지 확인하여 필터링
      if ((effect.tag & flags) === flags) {
        // Unmount
        const inst = effect.inst;
        const destroy = inst.destroy;
        if (destroy !== undefined) {
          inst.destroy = undefined; // 재호출되지 않도록 처리
          safelyCallDestroy(finishedWork, nearestMountedAncestor, destroy);
        }
      }
      effect = effect.next;
    } while (effect !== firstEffect);
  }
}
```

`commitHookEffectListMount` 함수를 나누어 살펴보겠습니다.

**`updateQueue`와 `lastEffect` 가져오기**

```jsx
const updateQueue = finishedWork.updateQueue;
const lastEffect = updateQueue !== null ? updateQueue.lastEffect : null;
```

- `finishedWork.updateQueue`는 해당 Fiber 노드의 업데이트 큐를 참조합니다.
- 이 큐에는 해당 컴포넌트에 등록된 모든 Effect가 저장되어 있습니다.
- `lastEffect`는 순환 리스트의 마지막 Effect를 가리키며, 리스트 탐색의 시작점 역할을 합니다.

<br/>

**Effect 리스트 순회**

```jsx
if (lastEffect !== null) {
    const firstEffect = lastEffect.next;
    let effect = firstEffect;
    // updateQueue의 모든 Effect를 반복하고,
    // 플래그별로 필요한 것을 필터링하면 됩니다.
    do {
      // 현재 Effect의 tag가 주어진 flags와 일치하는지 확인하여 필터링
      if ((effect.tag & flags) === flags) {
        // Unmount
        const inst = effect.inst;
        const destroy = inst.destroy;
        if (destroy !== undefined) {
          inst.destroy = undefined; // 재호출되지 않도록 처리
          safelyCallDestroy(finishedWork, nearestMountedAncestor, destroy);
        }
      }
      effect = effect.next;
    } while (effect !== firstEffect);
  }
```

- `lastEffect`가 존재하면 순환 리스트를 순회합니다.
- `(effect.tag & flags) === flags`를 통해 플래그가 일치하는 경우 클린업 함수를 실행합니다.
- `effect.tag & flags는 Effect`의 tag와 flags를 비트 AND 연산합니다. 이 연산은 두 값의 각 비트가 모두 1일 때만 결과가 1이 됩니다. flags가 일치하는 경우 `create` 함수를 실행합니다.
- `effect.destroy`는 `useEffect`의 반환값(클린업 함수)을 참조하며, 정의되어 있으면 실행됩니다.
- 리스트의 모든 `effect`를 순회한 후 종료됩니다.

<br/>

### useEffect 의존성 배열과 effect.tag, flags

위에서 살펴보았듯이 React의 `useEffect`는 내부적으로 각 Hook에 대해 `effect.tag`와 `flags`를 사용하여 사이드 이펙트의 관리와 실행 조건을 결정합니다. 컴포넌트가 업데이트될 때 React는 이전 렌더링의 의존성 배열과 현재 렌더링의 의존성 배열을 비교합니다.

만약 **의존성 배열(dependency array)** 이 변경되면, React는 새로운 Effect 객체를 생성하고 기존 `effect.tag` 및 `flags`를 참조하여 이전 Effect을 언마운트 시키고 새로운 Effect를 등록하고 실행합니다. 

이러한 내부 동작과정으로 `useEffect`는 컴포넌트의 렌더링과 업데이트 과정에서 효율적으로 사이드 이펙트를 관리하게됩니다.

<br/>

### 예시를 통해 전체 과정 살펴보기

```jsx
import { useState, useEffect } from "react";

export default function App() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log("카운터 변경");
    return () => {
      console.log("클린업");
    };
  }, [count]);

  const increment = () => {
    setCount((prev) => prev + 1);
  };

  const decrement = () => {
    setCount((prev) => prev - 1);
  };

  return (
    <div>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <p>Count : {count}</p>
    </div>
  );
}
```

<br/>

**첫 렌더링이 완료된 후**

![](https://velog.velcdn.com/images/njt6419/post/239c59ca-2c0c-41c3-ae8a-ece8b1697c96/image.png)


1. **useEffect 호출** 
    - 처음으로 useEffect가 호출되어 `mountEffect` 가 실행됩니다.
        
<br/>

2. **`mountEffect` 호출**
    - 마운트 시, React는 `mountEffect`를 호출하여 새 Effect를 등록합니다.
    
    ```jsx
    function mountEffect(create, deps) {
      return mountEffectImpl(
    	  PassiveEffect | PassiveStaticEffect, 
    	  HookPassive, create, 
    	  deps
      );
    }
    ```
        
<br/>

3. **`mountEffectImpl` 호출**
    
    ```jsx
    function mountEffectImpl(fiberFlags, hookFlags, create, deps): void {
      // 현재 Hook 정보를 가져옴
      const hook = mountWorkInProgressHook();
      
      // 의존성 배열 처리
      const nextDeps = deps === undefined ? null : deps;
     
      // Fiber의 Effect 플래그를 설정
      currentlyRenderingFiber.flags |= fiberEffectTag;
      
       // Hook의 상태에 Effect를 저장
      hook.memoizedState = pushEffect(HookHasEffect | hookFlags, create, undefined, nextDeps);
    }
    
    ```
    
    - `mountWorkInProgressHook`를 통해 현재 Hook 정보를 가져옵니다.
    - 의존성 배열이 주어지면 이를 nextDeps에 저장합니다.
    - `pushEffect`를 통해 Effect를 저장합니다. 이때, create 함수와 함께 의존성 배열도 저장됩니다.
    - **`pushEffect` 호출**
        
        Effect 객체를 생성하여 업데이트 큐에 추가합니다.
        
        ```jsx
        function pushEffect(tag, create, destroy, deps) {
          const effect: Effect = {
            tag,
            create,
            destroy,
            deps,
            // Circular
            next: (null: any),
          };
          if (componentUpdateQueue === null) {
            componentUpdateQueue = createFunctionComponentUpdateQueue();
            componentUpdateQueue.lastEffect = effect.next = effect;
          } else {
            const lastEffect = componentUpdateQueue.lastEffect;
            if (lastEffect === null) {
              componentUpdateQueue.lastEffect = effect.next = effect;
            } else {
              const firstEffect = lastEffect.next;
              lastEffect.next = effect;
              effect.next = firstEffect;
              componentUpdateQueue.lastEffect = effect;
            }
          }
          return effect;
        }
        ```
        
        - effect 객체를 생성합니다.
        - 현재 updateQueue가 비었으므로, `createFunctionComponentUpdateQueue` 함수를 호출하여 componentUpdateQueue를 생성합니다.
        - 생성한 queue.lastEffect에 effect를 넣어줍니다.
        - 이렇게 생성된 effect 객체를 반환합니다.
    - 반환된 Effect를 Fiber 노드에 등록하고 의존성 배열을 설정합니다.
    
<br/>

4. **커밋단계에서 `commitPassiveMountEffects` 함수 직접 호출**
  - `commitPassiveMountEffects`함수에서  `commitHookEffectListMount`를 호출하여  새로운 create 함수(console.log(”카운터 변경”))를 실행합니다.
    
    ```jsx
    function commitHookEffectListUnmount(
      flags: HookFlags,
      finishedWork: Fiber,
      nearestMountedAncestor: Fiber | null,
    ) {
      const updateQueue: FunctionComponentUpdateQueue | null =
        (finishedWork.updateQueue: any);
      const lastEffect = updateQueue !== null ? updateQueue.lastEffect : null;
      if (lastEffect !== null) {
        const firstEffect = lastEffect.next;
        let effect = firstEffect;
        // updateQueue의 모든 Effect를 반복하고,
        // 플래그별로 필요한 것을 필터링하면 됩니다.
        do {
          // 현재 Effect의 tag가 주어진 flags와 일치하는지 확인하여 필터링
          if ((effect.tag & flags) === flags) {
            // Unmount
            const inst = effect.inst;
            const destroy = inst.destroy;
            if (destroy !== undefined) {
              inst.destroy = undefined; // 재호출되지 않도록 처리
              safelyCallDestroy(finishedWork, nearestMountedAncestor, destroy);
            }
          }
          effect = effect.next;
        } while (effect !== firstEffect);
      }
    }
    ```
    
<br/>

**업데이트 시 동작(count가 변경)**

![](https://velog.velcdn.com/images/njt6419/post/78471c85-d0ea-4134-973c-8f41b90a50c3/image.png)

1. **`updateEffect` 가 호출**
    
    ```jsx
    function updateEffect(
      create: () => (() => void) | void,
      deps: Array<mixed> | void | null,
    ): void {
      // 실질적인 Effect 처리 구현 함수를 호출.
      // updateEffectImpl은 Effect를 실행, 업데이트, 또는 클린업 작업을 처리합니다.
      updateEffectImpl(
    	  PassiveEffect, // 비동기적으로 실행되는 작업
    	  HookPassive, // 비동기적으로 실행되는 Effect
        create, // 실행할 사용자 정의 콜백 함수.
        deps, // 의존성 배열.
      );
    }
    ```
    
    - 의존성 배열이 바뀔 때(count 변경) updateEffect가 호출되어 effect를 업데이트합니다.
        
<br/>

2. **`updateEffectImpl` 호출**
    
    ```jsx
    function updateEffectImpl(fiberFlags, hookFlags, create, deps) {
      // 현재 작업 중인 Hook을 가져옴. 이 Hook은 `workInProgress`에 연결됨.
      const hook = updateWorkInProgressHook();
    
      // 의존성 배열이 없으면 `nextDeps`는 null로 설정.
      const nextDeps = deps === undefined ? null : deps;
      let destroy = undefined; // 이전 cleanup 함수 초기화.
    
      // `currentHook`이 null이 아니면 (즉, 이 Hook이 이미 한 번 실행된 적이 있다면),
      if (currentHook !== null) {
        // 이전의 `effect` 상태를 가져옴.
        const prevEffect = currentHook.memoizedState;
        destroy = prevEffect.destroy; // 이전 cleanup 함수 참조.
    
        // 새로운 의존성 배열이 존재한다면,
        if (nextDeps !== null) {
          const prevDeps = prevEffect.deps; // 이전 의존성 배열을 가져옴.
    
          // 이전 의존성과 새로운 의존성을 비교.
          if (areHookInputsEqual(nextDeps, prevDeps)) {
            // 의존성이 동일하면 새로운 효과를 생성하지 않고 기존 효과를 유지.
            pushEffect(hookFlags, create, destroy, nextDeps);
            return;
          }
        }
      }
    
      // 의존성이 다르거나 처음 실행되는 경우, 새로운 효과를 스케줄링.
      sideEffectTag |= fiberEffectTag; // Fiber의 side effect 태그 갱신.
      
      // Hook의 상태를 새로운 효과로 업데이트.
      // HooksHasEffect는 deps가 변경될 때 Effect를 실행해야 함을 표시하는 플래그.
      hook.memoizedState = pushEffect(HookHasEffect | hookFlags, create, destroy, nextDeps);
    }
    ```
    
    - updateWorkInProgressHook()를 통해 현재 작업 중인 Hook을 가져옵니다.
    - 이전 렌더링의 의존성과 새로운 의존성을 비교하여 Effect를 실행할지 결정합니다.
    - 만약, 의존성이 변경된 경우(count 변경) 새로운 effect를 등록합니다.
    - **`pusEffect` 호출**
        
        ```jsx
        function pushEffect(tag, create, destroy, deps) {
          const effect: Effect = {
            tag,
            create,
            destroy,
            deps,
            // Circular
            next: (null: any),
          };
          if (componentUpdateQueue === null) {
            componentUpdateQueue = createFunctionComponentUpdateQueue();
            componentUpdateQueue.lastEffect = effect.next = effect;
          } else {
            const lastEffect = componentUpdateQueue.lastEffect;
            if (lastEffect === null) {
              componentUpdateQueue.lastEffect = effect.next = effect;
            } else {
              const firstEffect = lastEffect.next;
              lastEffect.next = effect;
              effect.next = firstEffect;
              componentUpdateQueue.lastEffect = effect;
            }
          }
          return effect;
        }
        ```
        
        - effect 객체를 생성합니다.
        - 현재 updateQueue가 존재하므로, componentUpdateQueue.lastEffect를 가져옵니다. componentUpdateQueue를 생성합니다.
        - firstEffect에 가져온 lastEffect.next를 넣고 lastEffect.next에는 새로운 effect를 넣어줍니다.
        - 이렇게 생성된 effect 객체를 반환합니다.
    - 반환된 Effect를 Fiber 노드에 등록하고 의존성 배열을 설정합니다.
    
<br/>

3. **커밋 단계에서 `flushPassiveEffects` 함수를 호출하여 Effect 객체 처리**
    
    ```jsx
    function flushPassiveEffectsImpl() {
      // 대기 중인 passive effects가 없으면 false를 반환
      if (rootWithPendingPassiveEffects === null) {
        return false;
      }
    
      const transitions = pendingPassiveTransitions; // 대기 중인 전환을 저장
      pendingPassiveTransitions = null; // 대기 중인 전환을 초기화
    
      const root = rootWithPendingPassiveEffects; // 현재 root를 가져옴
      const lanes = pendingPassiveEffectsLanes; // 대기 중인 passive effects의 lanes를 저장
      rootWithPendingPassiveEffects = null; // 대기 중인 root를 초기화
      pendingPassiveEffectsLanes = NoLanes; // 대기 중인 lanes를 초기화
    
      const prevExecutionContext = executionContext; // 이전 실행 컨텍스트를 저장
      executionContext |= CommitContext; // 현재 실행 컨텍스트를 CommitContext로 설정
    
      // 이전 passive effects의 언마운트 함수를 실행
      commitPassiveUnmountEffects(root.current);
      
      // 새로운 passive effects의 마운트 함수를 실행
      commitPassiveMountEffects(root, root.current, lanes, transitions);
      
      // ...
    ```
    
    `commitPassiveUnmountEffects`를 호출하여 이전에 등록된 **passive effects(useEffect)** 의 클린업을 처리합니다.
    
    `commitPassiveMountEffects`를 호출하여 새로운 **passive effects(useEffect)**
    를 적용합니다.
    
<br/>

4. **`commitPassiveUnmountEffects` 함수 호출**
  - `commitPassiveUnmountEffects` 함수에서  `commitHookEffectListUnmount` 함수가 호출되며, 언마운트되거나 의존성이 변경될 때, `safelyCallDestroy` 함수(console.log(”클린업”))를 실행합니다.
    
    ```jsx
    function commitHookEffectListUnmount(
      flags: HookFlags,
      finishedWork: Fiber,
      nearestMountedAncestor: Fiber | null,
    ) {
      // Fiber 노드에서 업데이트 큐를 가져옵니다.
      const updateQueue: FunctionComponentUpdateQueue | null = (finishedWork.updateQueue: any);
      
      // 업데이트 큐의 마지막 Effect를 가져옵니다.
      // Effect는 순환 리스트 형태로 저장되어 있습니다.
      const lastEffect = updateQueue !== null ? updateQueue.lastEffect : null;
      
      // 업데이트 큐가 비어있지 않은 경우, Effect를 순회하며 처리합니다.
      if (lastEffect !== null) {
        const firstEffect = lastEffect.next; // 순환 리스트의 첫 번째 Effect를 가져옵니다.
        let effect = firstEffect;.
        // 플래그별로 필요한 것을 필터링합니다.
        do {
    	    // Effect의 태그가 주어진 flags와 일치하는 경우 작업을 실행합니다.
          if ((effect.tag & flags) === flags) {
            // Unmount
            const destroy = effect.destroy; // 클린업 함수 가져옵니다.
            effect.destroy = undefined;
            if (destroy !== undefined) {
              safelyCallDestroy(finishedWork, nearestMountedAncestor, destroy); // 클린업 함수를 실행합니다.
            }
          }
          effect = effect.next; // 다음 Effect로 이동합니다.
        } while (effect !== firstEffect); // 순환 리스트를 한 바퀴 돌 때까지 반복합니다.
      }
    }
    ```
        
<br/>

5. **`commitPassiveMountEffects` 함수 호출**
  - `commitPassiveMountEffects` 함수에서  `commitHookEffectListMount`를 호출하여 새로운 `create` 함수(console.log(”카운터 변경”))를 실행합니다.
    
    ```jsx
    function commitHookEffectListUnmount(
      flags: HookFlags,
      finishedWork: Fiber,
      nearestMountedAncestor: Fiber | null,
    ) {
    	// Fiber 노드에서 업데이트 큐를 가져옵니다.
      const updateQueue: FunctionComponentUpdateQueue | null =
        (finishedWork.updateQueue: any);
        
      // 업데이트 큐의 마지막 Effect를 가져옵니다.
      // Effect는 순환 리스트 형태로 저장되어 있습니다.
      const lastEffect = updateQueue !== null ? updateQueue.lastEffect : null;
      if (lastEffect !== null) {
        const firstEffect = lastEffect.next;
        let effect = firstEffect;
        // updateQueue의 모든 Effect를 반복하고,
        // 플래그별로 필요한 것을 필터링하면 됩니다.
        do {
          // 현재 Effect의 tag가 주어진 flags와 일치하는지 확인하여 필터링
          if ((effect.tag & flags) === flags) {
            // Unmount
            const inst = effect.inst;
            const destroy = inst.destroy;
            if (destroy !== undefined) {
              inst.destroy = undefined; // 재호출되지 않도록 처리
              safelyCallDestroy(finishedWork, nearestMountedAncestor, destroy);
            }
          }
          effect = effect.next;
        } while (effect !== firstEffect);
      }
    }
    ```
    
위 과정으로 실행되는 결과로 먼저 첫 렌더링 이후 콘솔에 “카운터 변경”이 출력되고, 다음으로 count가 변경될 때 마다 이전 클린업 함수가 실행되어 “클린업”이 출력되고 다음 create 함수(useEffect callback 함수)가 실행되어 “카운터 변경”이 출력되게됩니다.

<br/>

### 정리

**useEffect의 동작 정리**

- useEffect는 렌더링이 완료된 이후, 비동기적으로 실행됩니다. 이는 DOM 업데이트가 완료되고 브라우저가 화면에 내용을 표시한 후에 실행되므로, 화면 업데이트를 방해하지 않습니다.
- `mountEffect`는 처음 렌더링 시 사용되며, 새로운 Effect를 설정합니다.
- 첫 렌더링 이후 `commitRoot`에서 `commitPassiveMountEffects`가 실행되어 새로운 create 함수를 실행합니다.
- `updateEffect`는 기존 Effect를 갱신하거나 의존성 배열을 비교해 변화가 있을 경우 새로운 Effect를 등록합니다.
- deps(의존성 배열)이 변경될 때 마다  두 개의 fiber 트리(렌더링 이전, 렌더링 이후)를 비교해 달라진 부분에 대한 결과를 얻은 뒤(재조정) commitRoot에서 `flushPassiveEffects` 호출하고`commitPassiveUnmountEffects`를 호출하여  이전 `safelyCallDestroy`함수를 실행하고 이후 `commitPassiveMountEffects`를 호출하여 새로운 `create` 함수를 실행합니다.

여기서 `useLayoutEffect`나  `useIntersectionEffect`의 내용은 자세히 다루지 않았습니다. `tag`(Passive, Layout, Insertion)에 따라 동작 방식이 달라집니다.

- `useEffect`는 `Passive`로 등록되며, 비동기 실행됩니다.
- `useLayoutEffect`는 `Layout`로 등록되며, 동기 실행됩니다.
- `useInsertionEffect`는 `Insertion`로 등록되며, 렌더링 단계에서 실행됩니다.