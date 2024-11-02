## Hydration?

<div align="center">
  <img src="https://velog.velcdn.com/images/njt6419/post/3f36c09c-323c-42f5-9980-dead202e3211/image.webp" width="500" />
  <br/>
  [출처] <a href="https://www.franciscomoretti.com/blog/what-is-react-hydration">https://www.franciscomoretti.com/blog/what-is-react-hydration</a>
</div>

<br/>

React는 웹 페이지를 Client Side Redering(CSR)를 통해 번들링된 JS를 가져와 DOM을 렌더링합니다.

이런 방식은 많은 장점이 존재하지만 초기 렌더링 속도의 저하, SEO 저하 등과 같은 문제점이 존재합니다. 그래서 이를 해결하고자 Server Side Redering(SSR)이 사용되었습니다. SSR을 사용하여 Server Side에서 정적 페이지를 렌더링하고 이후 JS파일들도 번들링한 후 둘 다 Client Side로 보내주는 방식으로 DOM을 렌더링합니다. 하지만 그 DOM에는 동적인 이벤트가 하나도 없는 메마른 상태일 것입니다. 이 메마른 뼈데 정정 파일에 **수분을 보충(Hydration)**를 통해 HTML과 JS 코드를 서로 매칭시켜 동적인 웹사이트를 브라우저에 렌더링하였습니다.

> 즉, Hydreation 수분 보충 과정은 **서버에서 렌더링된 정적 HTML을 클라이언트에서 동적인 React 컴포넌트로 전환하는 과정**입니다.

React에서 `ReactDOM.hydreateRoot()` 함수를 사용하여 hydreate 처리를 할 수 있습니다.

```jsx
ReactDOM.hydrateRoot(domNode, reactNode, options?);
```

첫 번째 파라미터에 지정된 DOM 요소에 하위만 hydreate 처리하며, 렌더링을 통해 새로운 웹 페이지를 구성할 DOM을 생성하는 것이 아니라 DOM Tree에서 해당되는 DOM 요소를 찾아 정해진 자바스크립트 속성들을 부착시킵니다.

<br/>

## Hydreation 동작 방식

> react-dom/server를 통해 사전에 만들어진 HTML로 그려진 브라우저 DOM 노드 내부에 React 컴포넌트를 렌더링합니다.

`ReactDom.hydrateRoot()`는 Hydration 과정의 시작점이되며, 아래와 같은 과정으로 진행됩니다:

- 서버에서 렌더링된 HTML의 루트 요소를 탐색합니다.
- React 컴포넌트 트리를 생성하고, 기존 HTML 구조와 매칭시킵니다.
- 각 컴포넌트에 대한 이벤트 리스너를 연결하고 상태를 초기합니다.
- 불일치가 발생되면 경고를 발생시키고, 필요한 경우 DOM를 업데이트합니다.

새로운 DOM 트리를 만드는 프로세스와 서버에서 렌더링된 HTML이 있기 때문에 커서를 통해 비교하여 새 노드를 생성하거나 재사용합니다. 기존 DOM 트리에 커서를 두며, 새로운 DOM 노드를 만들어야 할 때 이 커서와 비교합니다. 비교 결과가 일치하면 새 노드를 만들지 않고, 기존 노드를 그대로 사용합니다. 재사용된 노드는 React Fiber 노드의 stateNode로 설정됩니다.

React Fiber Tree의 각 노드는 `beginWork()`, `completeWork()` 두 번 순회되기 때문에 기존 DOM 트리의 커서도 이에 맞춰 동기화해줘야합니다.

<br/>

### beginWork()에서의 Hydration 과정

```jsx
function beginWork(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes,
): Fiber | null {
  ...
  switch (workInProgress.tag) {
    case HostComponent:
      return updateHostComponent(current, workInProgress, renderLanes);
  }
  ...
}

function updateHostComponent(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes
) {
  pushHostContext(workInProgress);
  if (current === null) {
    tryToClaimNextHydratableInstance(workInProgress);
  }
  ....
  return workInProgress.child;
}
```

**HostComponent는 `<div>`, `<span>` 등 브라우저 네이티브 DOM요소를 나타냅니다.**

`tryToClaimNextHydratableInstance()` 함수는 서버에서 미리 렌더링된 HTML 페이지에 이미 존재하는 DOM 요소를 재사용하려고 시도하는 함수입니다.

```jsx
function tryToClaimNextHydratableInstance(fiber: Fiber): void {
  if (!isHydrating) {
    // Hydration 중이 아니라면 return
    return;
  }

  // 현재 렌더링 환경에 대한 정보를 가져온다.
  const currentHostContext = getHostContext();
  // 현재 Fiber 노드가 Hydration에 적합한지 검사
  const shouldKeepWarning = validateHydratableInstance(
    fiber.type,
    fiber.pendingProps,
    currentHostContext
  );

  // 다음 Hydration 대상 DOM 요소를 가져온다.
  // 이 요소를 현재 Fiber 노드와 연결하려고 시도한다.
  const nextInstance = nextHydratableInstance;

  // 더 이상 Hydrate 할 노드가 없거나 Hydration이 불가능한 경우
  if (
    !nextInstance ||
    !tryHydrateInstance(fiber, nextInstance, currentHostContext)
  ) {
    if (shouldKeepWarning) {
      // Hydration이 실패했고, 경고를 유지하는 경우
      warnNonHydratedInstance(fiber, nextInstance); // Hydration 실패에 대한 경고 발생
    }
    throwOnHydrationMismatch(fiber); // Hydarion 불일치가 발생했을 때 오류 발생
  }
}

function tryHydrateInstance(
  fiber: Fiber,
  nextInstance: any,
  hostContext: HostContext
) {
  // DOM 요소가 현재 Fiber 노드와 일치하여 Hydration이 가능한지 확인한다.
  const instance = canHydrateInstance(
    nextInstance,
    fiber.type,
    fiber.pendingProps,
    rootOrSingletonContext
  );

  if (instance !== null) {
    // 가능하다면 DOM 요소를 Fiber 노드의 stateNode로 설정
    fiber.stateNode = (instance: Instance);
    // 현재 Fiber를 Hydration 부모로 설정
    // 다음 Hydration 대상을 현재 요소의 첫 번째 자식으로 설정(기존 DOM의 커서가 자식으로 이동)
    // rootOrSingletonContext 플래그를 false로 설정
    hydrationParentFiber = fiber;
    nextHydratableInstance = getFirstHydratableChild(instance);
    rootOrSingletonContext = false;
    return true; // 성공
  }

  return false; // 실패
}
```

<br/>

### completeWork()에서의 Hydration 과정

```jsx
function completeWork(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes,): Fiber | null {
   switch (workInProgress.tag) {
    case HostComponent: {
      ...
      if (current !== null && workInProgress.stateNode != null) {
       ... // 기존 노드 업데이트 로직
      } else {
        ... // 새 노드 생성 로직

        // 이 노드가 Hydration 과정을 거쳤는지 확인
        const wasHydrated = popHydrationState(workInProgress);
        if (wasHydrated) {
            prepareToHydrateHostInstance(workInProgress, currentHostContext);
        } else { // Hydration 되지 않은 경우
          const rootContainerInstance = getRootHostContainer();

          // 새로운 DOM 인스턴스 생성, 자식들을 추가, stateNode 설정
          const instance = createInstance(
            type,
            newProps,
            rootContainerInstance,
            currentHostContext,
            workInProgress,
          );
          appendAllChildren(instance, workInProgress, false, false);
          workInProgress.stateNode = instance;
        }
      }
      return null;
    }
   }
  ...
}
```

Fiber 노드가 성공적으로 **Hydration** 되면 `prepareToHydrateHostInstance()` 함수를 호출합니다.

Hydration이 되지 않은 경우 React는 새로운 DOM Instance를 생성하고 자식 노드들을 추가한 후, 이를 Fiber 노드의 stateNode에 설정합니다.

이 과정은 렌더링 단계의 일부로, 실제 DOM 업데이트는 이후의 커밋 단계에서 이루어집니다.

여기서 `prepareToHydrateHostInstance()` 함수의 역할은 아래와 같습니다.

```jsx
function prepareToHydrateHostInstance(
  fiber: Fiber,
  hostContext: HostContext
): void {
  if (!supportsHydration) {
    // 현재 환경이 Hydration을 지원하는지 확인
    throw new Error(
      "Expected prepareToHydrateHostInstance() to never be called. " +
        "This error is likely caused by a bug in React. Please file an issue."
    );
  }

  // Fiber 노드에 연결된 실제 DOM 요소를 가져온다.
  const instance: Instance = fiber.stateNode;
  // 실제 Hydration 작업 수행(DOM 요소의 속성 업데이트, 이벤트 리스너 연결 등)
  const didHydrate = hydrateInstance(
    instance,
    fiber.type,
    fiber.memoizedProps,
    hostContext,
    fiber
  );

  // Hydration이 실패하고, 안전성을 성능보다 우선시하는 설정이 켜져있다면 에러를 발생시킨다.
  if (!didHydrate && favorSafetyOverHydrationPerf) {
    throwOnHydrationMismatch(fiber);
  }
}
```

[`hydrateInstance()` 내부](https://github.com/facebook/react/blob/main/packages/react-dom-bindings/src/client/ReactFiberConfigDOM.js#L1401)에서 [`hydrateProperties()`](https://github.com/facebook/react/blob/main/packages/react-dom-bindings/src/client/ReactDOMComponent.js#L2909)를 호출하여 Hydration 과정을 수행합니다.

> **`prepareToHydrateHostInstance()`는 실제 Hydration을 수행하는 함수입니다.**

<br/>

### 서버 렌더링 트리와 클라이언트 생성 트리 동기화 과정

```jsx
function popHydrationState(fiber: Fiber): boolean {
  if (!supportsHydration) {
    // Hydration을 지원하지 않는 경우
    return false;
  }

  if (fiber !== hydrationParentFiber) {
    // 현재 Fiber가 Hydarion의 부모가 아닌 경우
    return false;
  }

  if (!isHydrating) {
    // 현재 Hydration 중이 아니면
    popToNextHostParent(fiber); // 다음 호스트 부모로 이동
    isHydrating = true; // Hydration 상태 true 설정
    return false; // false return
  }

  // 노드 정리 여부 결정
  let shouldClear = false;
  if (supportsSingletons) {
    // 싱글톤 지원 여부
    if (
      fiber.tag !== HostRoot &&
      fiber.tag !== HostSingleton &&
      !(
        fiber.tag === HostComponent &&
        (!shouldDeleteUnhydratedTailInstances(fiber.type) ||
          shouldSetTextContent(fiber.type, fiber.memoizedProps))
      )
    ) {
      shouldClear = true;
    }
  } else {
    // 싱글톤 지원 X
    if (
      fiber.tag !== HostRoot &&
      (fiber.tag !== HostComponent ||
        (shouldDeleteUnhydratedTailInstances(fiber.type) &&
          !shouldSetTextContent(fiber.type, fiber.memoizedProps)))
    ) {
      shouldClear = true;
    }
  }

  // 노드 정리 실행
  if (shouldClear) {
    const nextInstance = nextHydratableInstance;
    if (nextInstance) {
      warnIfUnhydratedTailNodes(fiber);
      throwOnHydrationMismatch(fiber);
    }
  }

  // Hydration 컨텍스트 업데이트
  popToNextHostParent(fiber); // 다음 호스트 부모로 이동
  if (fiber.tag === SuspenseComponent) {
    // Suspense 처리
    nextHydratableInstance = skipPastDehydratedSuspenseInstance(fiber);
  } else {
    // 다음 Hydartion 대상 설정
    nextHydratableInstance = hydrationParentFiber
      ? getNextHydratableSibling(fiber.stateNode)
      : null;
  }
  return true;
}
```

`popHydrationState()` 함수는 각 Fiber 노드에 대해 Hydration 상태를 관리하고, 필요한 경우 정리 작업을 수행하며, Hydration 과정에서 발생할 수 있는 불일치를 감지하고 처리합니다. 이를 통해 서버에서 렌더링된 콘텐츠와 클라이언트에서 생성되는 React 트리를 효과적으로 동기화합니다.

`popToNextHostParent()` 함수는 React의 컴포넌트 트리 현재 위치에서 위로 올라가면 가장 가까운 실제 DOM 요소와 연관된 컴포넌트를 찾습니다. Hydration 과정에서 현재 처리 중인 컴포넌트의 가장 가까운 DOM 관련 부모를 찾아주는 역할을 합니다.

<br/>

### Hydration 에러 처리

Hydration 에러(Hydration Mismatch)는 주로 서버에서 렌더링한 HTML과 클라이언트에서 렌더링한 HTML이 일치하지 않을 때 발생합니다. 이 오류는 아래와 같은 상황에서 자주 발생할 수 있습니다:

- **서버와 클라이언트의 시간 차이**: 서버와 클라이언트의 시간이 다르면, 예를 들어 "현재 시간" 같은 동적 데이터가 불일치하게 됩니다.
- **동적 콘텐츠**: 서버와 클라이언트의 렌더링 결과가 달라지게 만드는 비동기 데이터 또는 랜덤 요소가 있을 때 불일치가 생길 수 있습니다.
- **브라우저 확장 프로그램의 DOM 수정**: 사용자가 설치한 확장 프로그램이 DOM을 변경하여 클라이언트와 서버 간 불일치가 발생할 수 있습니다.

**React의 Hydration 에러 해결 전략**

- **경고 메시지**: 개발자 콘솔에 경고 메시지를 표시하여 문제를 인지할 수 있게 합니다. 이를 통해 문제를 식별하고 원인을 추적할 수 있습니다.
- **클라이언트 사이드 렌더링 전환**: 불일치가 발견되면 서버에서 받은 HTML을 무시하고 클라이언트 사이드 렌더링으로 전환하여 일관된 콘텐츠를 표시합니다.
- **`suppressHydrationWarning` 속성 사용**: `suppressHydrationWarning` prop을 특정 엘리먼트에 설정하여 의도적인 불일치를 허용할 수 있습니다.

💡 **suppressHydrationWarning이란?**

`suppressHydrationWarning`는 Hydration 과정에서 발생하는 특정 경고를 억제할 수 있는 속성으로, 아래와 같은 상황에서 유용합니다:

- **서버와 클라이언트 렌더링의 불일치가 불가피할 때**: 예를 들어, 현재 시간이나 사용자 맞춤 데이터를 표시할 때 시간 차이가 발생할 수 있습니다.
- **사용자 경험에 큰 영향을 미치지 않는 차이일 때**: 중요한 데이터가 아니며 미세한 차이로 인해 큰 영향이 없을 경우 사용합니다.

```jsx
function RandomNumber() {
  return <div suppressHydrationWarning>랜덤 숫자: {Math.random()}</div>;
}
```

위 코드는 서버에서 생성된 랜덤 숫자와 클라이언트에서 생성된 랜덤 숫자가 일치하지 않아 Hydration 경고가 발생할 수 있습니다. `suppressHydrationWarning` 속성을 추가하여 이러한 불일치를 의도적으로 무시할 수 있습니다.

**주의사항**

- `suppressHydrationWaring`는 해당 엘리먼트와 그 직계 자식에만 적용됩니다.
- 남용하면 실제 문제를 숨길 수 있으므로 꼭 필요한 경우에만 사용해야 합니다.
- 가능하면 서버와 클라이언트의 렌더링 결과를 일치시키는 것이 더 좋은 해결책입니다.

<br/>

## React 18의 Hydreation

React 18에서 **Concurrent(동시성) 렌더링**이 도입되면서 Hydration 과정에도 중요한 변화가 있었습니다. 특히, Suspense와의 통합을 통해 서버에서 렌더링된 HTML을 클라이언트에서 효율적으로 Hydrate할 수 있는 새로운 방법이 추가되었습니다. 이를 통해 중요한 콘텐츠를 먼저 로드하거나, 페이지의 일부분을 선택적으로 렌더링하여 성능을 크게 개선할 수 있습니다.

<br/>

### Suspense와 Progressive Hydration

React 18에서는 **Suspense**를 사용하여 각 부분의 Hydration 우선순위를 조정할 수 있습니다. 이를 통해 중요한 콘텐츠를 먼저 렌더링하고 덜 중요한 부분은 나중에 로드하여, 사용자에게 필요한 정보를 빠르게 제공할 수 있습니다. 이를 "Progressive Hydration(점진적 Hydration)"이라고 합니다.

또한, 특정 컴포넌트를 **Suspense**로 감싸서 별도로 처리하면, 클라이언트가 중요한 콘텐츠에 더 빠르게 접근할 수 있으며 덜 중요한 콘텐츠는 백그라운드에서 로드됩니다. 이를 "Selective Hydration(선택적 Hydration)"이라고 부릅니다.

**Suspense 예시 코드**

아래 코드에서는 `ImportantComponent`를 먼저 Hydrate하고, `LessImportantComponent`는 백그라운드에서 나중에 로드되도록 설정합니다.

```jsx
import React, { Suspense } from "react";

const ImportantComponent = React.lazy(() => import("./ImportantComponent"));
const LessImportantComponent = React.lazy(() =>
  import("./LessImportantComponent")
);

function App() {
  return (
    <div>
      {/* 중요한 콘텐츠 */}
      <Suspense fallback={<div>로딩 중...</div>}>
        <ImportantComponent />
      </Suspense>

      {/* 덜 중요한 콘텐츠 */}
      <Suspense fallback={<div>추가 콘텐츠 로딩 중...</div>}>
        <LessImportantComponent />
      </Suspense>
    </div>
  );
}

export default App;
```

이 예시에서 `ImportantComponent`는 Suspense를 사용하여 Hydration 시 우선적으로 로드됩니다. `LessImportantComponent`는 두 번째 Suspense 경계를 통해 늦게 로드되며, 사용자 경험에 큰 영향을 미치지 않도록 설정됩니다.

<br/>

### Streaming SSR (서버 사이드 스트리밍 렌더링)

React 18의 **Streaming SSR**은 서버에서 HTML을 한 번에 전송하지 않고, 부분적으로 나누어 클라이언트에 전달할 수 있게 합니다. 이를 통해 전체 페이지가 로드되기 전에 중요한 콘텐츠부터 먼저 클라이언트에 표시하고, 나머지 콘텐츠는 점진적으로 로드하여 상호작용성을 높일 수 있습니다.

Streaming SSR과 Suspense를 함께 사용하면 서버에서 보내는 HTML이 클라이언트에서 부분적으로 Hydrate되므로, 사용자에게 초기 콘텐츠가 빠르게 표시되며 점진적으로 인터랙티브한 요소가 추가됩니다.

<div align="center">
  <img src="https://velog.velcdn.com/images/njt6419/post/39989c0b-a052-4723-b735-9ccf57d828d7/image.png" width="800" />
  <br/>
  [출처] <a href="https://velog.io/@huurray/React-Hydration-%EC%97%90-%EB%8C%80%ED%95%98%EC%97%AC">https://velog.io/@huurray/React-Hydration-%EC%97%90-%EB%8C%80%ED%95%98%EC%97%AC</a>
</div>

<br/>

기존 React에서는 `renderToString`을 통해 서버에서 전체 HTML을 한 번에 완성하여 클라이언트에 전달했으나, 이 방식은 클라이언트가 페이지의 모든 콘텐츠가 준비될 때까지 기다려야 하며, 전체 페이지가 한꺼번에 Hydrate되므로 **Time to Interactive(TTI)**가 길어질 수 있습니다.

React 18에서는 `pipeToNodeWritable`과 `renderToPipeableStream`을 사용하여 HTML을 작은 청크로 나누어 스트리밍 전송할 수 있습니다. 이 방식은 클라이언트가 중요한 부분을 우선 표시할 수 있게 하며, 순차적으로 Hydration을 수행하여 **First Contentful Paint(FCP)**를 개선하면서도 **Time to Interactive(TTI)**에 대한 부담을 줄입니다.

**Streaming SSR과 Progressive Hydration 예시**

아래 예시는 서버에서 일부 콘텐츠를 먼저 보내고, 나머지 부분은 추가로 전송하는 방식을 보여줍니다. React 서버 컴포넌트와 함께 Streaming SSR을 사용할 수 있습니다.

```jsx
// 서버 측 코드 (Node.js Express 예시)
import { renderToPipeableStream } from "react-dom/server";
import App from "./App";

app.get("/", (req, res) => {
  const stream = renderToPipeableStream(<App />, {
    onShellReady() {
      // HTML의 중요한 부분을 먼저 클라이언트에 전송
      res.statusCode = 200;
      stream.pipe(res);
    },
    onAllReady() {
      // 모든 콘텐츠가 준비된 후에 추가 전송
      stream.pipe(res);
    },
    onError(err) {
      console.error(err);
    },
  });
});
```

위 코드에서는 `renderToPipeableStream`을 사용하여 App 컴포넌트를 스트리밍 방식으로 전송합니다. `onShellReady` 콜백은 중요한 콘텐츠를 먼저 보내는 역할을 하며, 사용자가 페이지를 빠르게 볼 수 있도록 합니다. `onAllReady`는 전체 콘텐츠가 준비되었을 때 추가 전송을 담당하여, 나머지 Hydration을 마칩니다.

**pipeToNodeWritable을 사용한 HTML 스트리밍 예시**

```jsx
import { pipeToNodeWritable } from "react-dom/server";

const data = createServerData();
let assets = {
  "main.js": "/main.js",
  "main.css": "/main.css",
};

// pipeToNodeWritable을 사용한 HTML 스트리밍
const { startWriting, abort } = pipeToNodeWritable(
  <DataProvider data={data}>
    <App assets={assets} />
  </DataProvider>,
  res,
  {
    onReadyToStream() {
      res.statusCode = 200;
      res.setHeader("Content-Type", "text/html");
      res.write("<!DOCTYPE html>");
      startWriting(); // 초기 콘텐츠 스트리밍 시작
    },
    onError(e) {
      console.error(e);
    },
  }
);

function createServerData() {
  let done = false;
  let promise = null;
  return {
    read() {
      if (done) return;
      if (promise) throw promise;
      promise = new Promise((resolve) => {
        setTimeout(() => {
          done = true;
          promise = null;
          resolve();
        }, 1000); // 데이터 로딩 지연 시뮬레이션
      });
      throw promise;
    },
  };
}
```

이 코드에서는 `pipeToNodeWritable`를 사용하여 서버에서 클라이언트로 HTML을 스트리밍 전송합니다. **onReadyToStream** 콜백은 초기 HTML을 클라이언트에 먼저 전송하여 빠른 FCP를 제공합니다. 이후 필요한 추가 콘텐츠는 스트리밍을 통해 계속 전송되어 최종적으로 전체 Hydration을 마칩니다.

<br/>

### Selective Hydration과 Suspense

<div align="center">
  <img src="https://velog.velcdn.com/images/njt6419/post/615d1883-b0f4-4fb7-971b-9240f808f694/image.png" width="800" />
</div>

<br/>

클라이언트 측에서는 **Selective Hydration**을 통해 각 컴포넌트의 중요도에 따라 우선순위를 지정하고, 중요한 부분부터 Hydrate하여 상호작용 가능성을 빠르게 합니다. React 18에서 **Suspense**와 함께 사용할 수 있으며, 이를 통해 클라이언트가 전체 페이지를 기다리지 않고 빠르게 필요한 요소와 상호작용할 수 있습니다.

```jsx
import React, { Suspense } from "react";
import Spinner from "./src/components/Spinner";

function App() {
  return (
    <Layout>
      <Suspense fallback={<Spinner />}>
        <NavBar />
      </Suspense>
      <Suspense fallback={<Spinner />}>
        <Sidebar />
      </Suspense>
      <Content>
        <Post />
        <Suspense fallback={<Spinner />}>
          <Comments />
        </Suspense>
      </Content>
    </Layout>
  );
}

export default App;
```

이 예시에서 `NavBar`와 `Sidebar` 같은 컴포넌트는 **Suspense**로 감싸져 중요한 콘텐츠부터 우선적으로 Hydrate됩니다. 이를 통해 클라이언트는 페이지의 일부가 준비되지 않아도 주요 컴포넌트와 상호작용할 수 있으며, 사용자 경험이 개선됩니다.

<br/>

## 정리 - React Hydration 과정

### 1. Hydration 시작

Hydration은 `hydrateRoot()` 함수를 호출하면서 시작됩니다. 이 함수는 서버에서 렌더링된 HTML과 클라이언트의 React 컴포넌트 트리를 연결하여 초기 화면을 설정하는 데 사용됩니다. Hydration을 통해 서버에서 보내진 HTML에 JavaScript 이벤트와 상태 관리 기능이 추가됩니다.

### 2. Fiber Tree 순회 및 연결 시도

Hydration 과정 중 `beginWork()` 함수는 React의 Fiber 트리를 순회하며 각 Fiber 노드를 처리합니다. 이때 `tryToClaimNextHydratableInstance()` 함수는 서버에서 렌더링된 DOM 요소와 현재 Fiber 노드를 연결하기 위해 시도합니다. 이를 통해 서버와 클라이언트 간의 구조적 일치를 유지할 수 있습니다.

### 3. DOM 요소와 Fiber 노드 연결

각 Fiber 노드에 대해 `tryHydrateInstance()` 함수는 서버에서 생성된 DOM 요소가 해당 Fiber 노드와 일치하는지 확인합니다. 일치하는 경우 해당 Fiber 노드의 `stateNode`에 DOM 요소를 설정하여 실제 요소와의 연결을 완료합니다. 이를 통해 클라이언트와 서버 간의 요소 매칭이 완성됩니다.

### 4. Hydration 작업 마무리 및 정리

`completeWork()` 함수는 각 Fiber 노드의 Hydration 작업을 마무리하는 역할을 합니다. 이 단계에서 `prepareToHydrateHostInstance()` 함수는 속성 업데이트나 이벤트 리스너 연결과 같은 실제 Hydration 작업을 수행하여 서버와 클라이언트의 트리가 상호작용을 시작할 수 있도록 준비합니다.

### 5. 트리 동기화 및 불일치 처리

Hydration 중 서버에서 생성된 트리와 클라이언트에서 렌더링되는 트리가 완전히 일치하는지 확인하는 과정이 필요합니다. `popHydrationState()` 함수는 이 작업을 수행하며, 만약 서버와 클라이언트 간의 불일치를 감지하면 추가 처리를 통해 이를 정리합니다. 이는 서버와 클라이언트 트리의 구조적 일치를 유지하는 데 중요한 단계입니다.

### 6. 다음 Hydration 대상 설정

Hydration 과정이 한 Fiber 노드에서 완료되면, `popToNextHostParent()` 함수가 다음 Hydration 대상 Fiber 노드를 설정하여 다음 요소와의 매칭을 준비합니다. 이를 통해 React는 서버에서 전달된 구조와 클라이언트 트리의 매칭을 단계적으로 진행합니다.

이 과정을 통해 서버에서 미리 렌더링된 HTML과 클라이언트의 React 트리가 완전히 동기화되며, 최종적으로 React 애플리케이션이 클라이언트에서 정상적으로 구동될 수 있도록 합니다.

<br/>

## 참고 자료

[What is React Hydration?](https://velog.io/@huurray/React-Hydration-%EC%97%90-%EB%8C%80%ED%95%98%EC%97%AC)

[React Hydration의 내부 동작원리](https://velog.io/@woogur29/React-Hydration%EC%9D%98-%EB%82%B4%EB%B6%80-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC)

[React의 Hydration에 대하여](https://www.franciscomoretti.com/blog/what-is-react-hydration)
