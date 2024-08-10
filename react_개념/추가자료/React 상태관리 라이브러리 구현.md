# React 상태관리 라이브러리 구현

## 상태관리?

> 리액트(React)에서 상태 관리(state management)는 컴포넌트 간의 데이터와 상태를 효과적으로 관리하는 방법을 의미합니다. 상태(state)는 리액트 컴포넌트 내에서 데이터의 현재 상태를 나타내며, 이 상태가 변경되면 컴포넌트는 다시 렌더링됩니다.

<br>

### 리액트의 상태관리 종류

- **로컬 상태 관리(Local State Management)**:
  - 개별 컴포넌트 내부에서 상태를 관리하는 방법입니다.
  - 리액트에서 `useState` , `useReducer` 훅을 사용하여 상태를 생성하고 관리할 수 있습니다.
  - 예를 들어, 사용자 입력 폼이나 토글 버튼처럼 다른 컴포넌트와 독립적인 상태는 로컬 상태로 관리됩니다.
- **전역 상태 관리(Global State Management)**:
  - 여러 컴포넌트에서 공유되는 상태를 관리할 때 사용됩니다.
  - 리액트의 기본적인 `Context API`를 사용하거나, 복잡한 애플리케이션에서는 `Redux`, `Recoil`, `MobX`와 같은 외부 라이브러리를 사용하여 전역 상태를 관리합니다.
  - 전역 상태 관리의 목적은 다양한 컴포넌트가 동일한 상태를 공유하고, 이를 통해 일관성 있는 UI를 유지하는 것입니다.

<br>

### useState 상태관리

```tsx
import { useState } from "react";

export default function Component() {
  // useState hook을 이용하여 상태관리
  // 초깃값 0
  const [count, setCount] = useState(0);

  // count state를 1증가 시킴
  function increment() {
    setCount((prev) => prev + 1);
  }

  // count state를 1증가 감소
  function decrement() {
    setCount((prev) => prev - 1);
  }

  return (
    <>
      <h1>useState</h1>
      <p>count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </>
  );
}
```

<br>

### useReducer 상태관리

```tsx
import { useReducer } from "react";

type State = {
  count: number;
};

enum ActionType {
  INCREMENT = "INCREMENT",
  DECREMENT = "DECREMENT",
}

type Action = {
  type: ActionType;
  payload?: State;
};

const initialState: State = { count: 0 };

const reducer = (state: State, action: Action): State => {
  switch (action.type) {
    case ActionType.INCREMENT:
      return { ...state, count: state.count + 1 };
    case ActionType.DECREMENT:
      return { ...state, count: state.count - 1 };
    default:
      throw new Error("Not Correct Action Type.");
  }
};

export default function Component() {
  const [state, dispatch] = useReducer(reducer, initialState);

  // count state를 1 증가시킴
  function increment() {
    dispatch({ type: ActionType.INCREMENT });
  }

  // count state를 1 감소시킴
  function decrement() {
    dispatch({ type: ActionType.DECREMENT });
  }

  return (
    <>
      <h1>useReducer Example</h1>
      <p>count: {state.count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </>
  );
}
```

<br>

### React hook을 이용해 상태 관리 라이브러리 구현하기

react hook을 이용하여 외부 라이브러리 처럼 전역 상태를 관리하는 라이브러리를 구현합니다.

<br>

**createStore.ts**

전역 상태 관리를 위한 store 생성 함수

```tsx
export type InitialState<T> = T extends any ? T | ((prev: T) => T) : never;

export type Store<State> = {
  get: () => State;
  set: (action: InitialState<State>) => State;
  subscribe: (cb: () => void) => () => void;
};

// store 생성 함수
export default function createStore<State extends unknown>(
  initialState: InitialState<State>
): Store<State> {
  // 클로저를 이용하여 state를 저장
  // initialState 타입이 'function'인 경우 게으른 초기화 함수 실행, 그렇지 않으면 그대로 적용
  let state =
    typeof initialState === "function" ? initialState() : initialState;

  // callback 함수는 중복되지 않도록 Set에 저장
  const callbacks = new Set<() => void>();

  // 현재의 최신 state 값을 가져옴
  const get = () => state;

  // state 값을 설정, 파라미터 값으로 설정할 State 값 혹은 State 설정 함수를 받음
  const set = (nextState: State | ((prev: State) => State)) => {
    // 인수가 함수라면 함수를 실행하고, 아니라면 그 값을 그대로 사용
    state =
      typeof nextState === "function"
        ? (nextState as (prev: State) => State)(state)
        : nextState;

    // 상태 변경 시 등록된 콜백을 모두 실행하여 리렌더링을 트리거함
    callbacks.forEach((callback) => callback());

    return state;
  };

  // 상태 변경 시 리렌더링을 트리거할 수 있는 구독 함수, 콜백 함수를 파라미터로 받음
  const subscribe = (cb: () => void) => {
    callbacks.add(cb);
    // 클린업 함수로 구독 해제를 통해 기존에 쌓인 콜백을 삭제
    return () => {
      callbacks.delete(cb);
    };
  };

  return { get, set, subscribe };
}
```

<br>

**useStore.ts**

store의 변화를 감지하여 리렌더링을 트리거하는 customhook

```tsx
import { useEffect, useState } from "react";
import { Store } from "./Store";

// 감지할 store를 파라미터로 받음
export default function useStore<State extends unknown>(store: Store<State>) {
  // store의 초깃값으로 state 생성, useState를 통해 컴포넌트의 렌더링을 유도
  const [state, setState] = useState(() => store.get());

  useEffect(() => {
    // store 값이 변경될 때마다 등록된 콜백 함수 실행하여 state 값 변경
    const unsubscribe = store.subscribe(() => {
      setState(store.get());
    });
    // 클린업 함수로 구독 해제하여 기존에 쌓인 콜백을 삭제
    return unsubscribe;
  }, [store]);

  return [state, store.set] as const;
}
```

**위에서 만든 store는 원시값의 경우 잘 적용되지만 객체의 경우 일부의 값만 변경되어도 리렌더링이 발생하여 불필요한 렌더링이 발생하게됩니다.**

이를 해결하기 위해 **변경을 감지가 필요한 값만 setState를 호출**해야합니다.

<br>

**useStoreSelector.ts**

store의 특정 값을 선택적으로 구독하는 custom hook으로 변경을 감지할 값만 리렌더링을 트리거합니다.

useStore과 다르게 파라미터로 `selector`라는 함수를 받으며, store에서 어떤 값을 가져올지 정의하는 함수입니다. `selector` 함수를 활용하여 `store.get()`을 수행합니다. **useState는 값이 변경되지 않는 한 리렌더링을 수행하지 않으므로 store의 값이 변하더라도 selector(store.get())이 변경되지 않는 한 리렌더링이 발생하지 않습니다.**

```tsx
import { useEffect, useState } from "react";
import { Store } from "./Store";

// 감지할 store와 선택할 값을 결정하는 selector 함수를 파라미터로 받음
export default function useStoreSelector<
  State extends unknown,
  Value extends unknown
>(store: Store<State>, selector: (state: State) => Value) {
  // selector 함수를 통해 선택한 값으로 state 생성
  const [state, setState] = useState(() => selector(store.get()));

  useEffect(() => {
    // store 값이 변경될 때마다 selector 함수를 사용해 변경된 값 감지
    const unsubscribe = store.subscribe(() => {
      const value = selector(store.get());
      console.log(value);
      setState(value);
    });

    // 클린업 함수로 구독 해제하여 기존에 쌓인 콜백을 삭제
    return unsubscribe;
  }, [store, selector]);

  return state;
}
```

<br>

**사용 예시 코드**

**Counter1.ts**

```tsx
import useStoreSelector from "./src/hooks/useStoreSelector";
import { store } from "./App";
import { useCallback } from "react";

export default function Counter1() {
  const count = useStoreSelector(
    store,
    useCallback((state) => state.count, [])
  );

  function increment() {
    store.set((prev) => ({
      ...prev,
      count: prev.count + 1,
    }));
  }

  function decrement() {
    store.set((prev) => ({
      ...prev,
      count: prev.count - 1,
    }));
  }
  return (
    <>
      <h1>Counter1</h1>
      <p>count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </>
  );
}
```

<br>

**Counter2.ts**

```tsx
import useStoreSelector from "./src/hooks/useStoreSelector";
import { store } from "./App";
import { useCallback } from "react";

export default function Counter1() {
  const count = useStoreSelector(
    store,
    useCallback((state) => state.count, [])
  );

  function increment() {
    store.set((prev) => ({
      ...prev,
      count: prev.count + 1,
    }));
  }

  function decrement() {
    store.set((prev) => ({
      ...prev,
      count: prev.count - 1,
    }));
  }
  return (
    <>
      <h1>Counter2</h1>
      <p>count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </>
  );
}
```

<br>

**Profile.ts**

```tsx
import { useCallback } from "react";
import { store } from "./App";
import useStoreSelector from "./src/hooks/useStore";

export default function Profile() {
  const profile = useStoreSelector(
    store,
    useCallback((state) => state.profile, [])
  );

  function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    const formdata = new FormData(e.currentTarget);
    const name = formdata.get("name") as string;
    const age = +(formdata.get("age") as string);
    if (name && age) {
      store.set((prev) => ({ ...prev, profile: { name, age } }));
      e.currentTarget.reset();
    }
  }

  return (
    <div>
      <h1>Profile</h1>
      <form onSubmit={handleSubmit}>
        <input name="name" placeholder="name" />
        <input name="age" type="number" placeholder="age" max={100} min={0} />
        <button>set</button>
      </form>
      <p>name : {profile.name}</p>
      <p>age : {profile.age}</p>
    </div>
  );
}
```

<br>

**App.ts**

```tsx
import Counter1 from "./Counter1";
import Counter2 from "./Counter2";
import Profile from "./Profile";
import createStore from "./store/Store";

export const store = createStore({ count: 0, profile: { name: "", age: 0 } });

function App() {
  return (
    <>
      <Counter1 />
      <Counter2 />
      <Profile />
    </>
  );
}

export default App;
```

위 코드 실행시 App 컴포넌트에서 생성한 store를 통해 Counter1과 Counter2 컴포넌트의 count state가 전역으로 공유되는 것을 볼 수 있습니다. Profile state 또한 정상적으로 변경되는 것을 볼 수 있습니다.

<br>

### React Context를 통한 다중 store 관리

현재 useStoreSelector hook과 store를 통해 상태관리 라이브러리처럼 동작하도록 만들었습니다.

useStoreSelector hook과 store는 반드시 하나의 스토어만 가지게됩니다. 하나의 store만을 가지게 되면 이 store는 마치 전역 변수처럼 작동하게 되어 동일한 형태의 여러개의 스토어를 가질 수 없게됩니다. 만약, 서로 다른 스코프 상에서 스토어의 구조는 동일 하지만 여러 개의 서로 다른 데이터를 공유하려면 현재 방법으로는 createStore를 통해 여러개의 store를 생성하며, 각 store의 수 만큼 useStoreSelector를 생성해야합니다. 이러한 방법은 번거럽게 store의 수 만큼 hook를 생성해야한다는 점과 이 hook이 어느 store에 사용 가능한지를 확인하려면 오직 훅의 이름이나 스토어의 이름으로 이를 확인 할 수 밖에 없다는 불편한점이 존재합니다.

이러한 점을 해결하기 위해 React의 **Context**를 활용할 수 있습니다.

Context를 활용해 해당 스토어를 하위 컴포넌트에 주입한다면 컴포넌트에서는 자신이 주입된 스토어에 대해서만 접근할 수 있게되어, 위의 문제점들을 해결할 수 있습니다.

<br>

**CounterStoreContextProvider.tsx**

```tsx
import { createContext, useRef } from "react";
import createStore, { Store } from "./store/Store";
export type CounterStore = {
  count: number;
};

// Counter Store Context 생성
export const CounterStoreContext = createContext<Store<CounterStore>>(
  createStore<CounterStore>({ count: 0 })
);

// Context Provider 컴포넌트 생성
export default function CounterStoreProvider({
  initialState,
  children,
}: {
  initialState: CounterStore;
  children: React.ReactNode;
}) {
  // ref를 통해 store 값 참조 리렌더링 방지
  // useRef는 초기값을 한 번만 설정
  // 이후에는 계속해서 동일한 참조를 유지하기 때문에 리렌더링이 발생 X
  const storeRef = useRef<Store<CounterStore>>();

  // storeRef.current 값이 없는 경우 초깃값 설정
  if (!storeRef.current) {
    storeRef.current = createStore(initialState);
  }

  // Prvoider value에 storeRef.current 값을 전달
  return (
    <CounterStoreContext.Provider value={storeRef.current}>
      {children}
    </CounterStoreContext.Provider>
  );
}
```

<br>

**useCounterContextSelector.ts**

useContext를 사용하여 store에 접근하는 hook

```tsx
import { useContext, useEffect, useState } from "react";
import {
  CounterStore,
  CounterStoreContext,
} from "../../store/CounterStoreContextProvider";

export default function useCounterContextSelector<State extends unknown>(
  selector: (state: CounterStore) => State
) {
  // useContext를 통해 store에 접근
  const store = useContext(CounterStoreContext);

  const [state, setState] = useState(() => selector(store.get()));

  useEffect(() => {
    // store 값이 변경될 때 마다 subscribe에 등록된 함수를 실행하여 state 값을 변경
    const unsubscribe = store.subscribe(() => {
      const value = selector(store.get());
      console.log(value);
      setState(value);
    });

    // 클린업 함수로 구독해제를 통해 기존에 쌓인 callback를 삭제
    return unsubscribe;
  }, [store, selector]);

  return [state, store.set] as const;
}
```

<br>

**App.tsx**

```tsx
import Counter1 from "./Counter1";
import Counter2 from "./Counter2";
import CounterStoreProvider from "./store/CounterStoreContextProvider";

function App() {
  return (
    <CounterStoreProvider initialState={{ count: 0 }}>
      <Counter1 />
      <Counter2 />
    </CounterStoreProvider>
  );
}

export default App;
```

<br>

**Counter1.tsx**

```tsx
import { useCallback } from "react";
import useCounterContextSelector from "./src/hooks/useCounterContextSelector";
import { CounterStore } from "./store/CounterStoreContextProvider";

export default function Counter1() {
  const [count, setCount] = useCounterContextSelector(
    useCallback((state: CounterStore) => state.count, [])
  );

  function increment() {
    setCount((prev) => ({
      ...prev,
      count: prev.count + 1,
    }));
  }

  function decrement() {
    setCount((prev) => ({
      ...prev,
      count: prev.count - 1,
    }));
  }
  return (
    <>
      <h1>Counter1</h1>
      <p>count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </>
  );
}
```

<br>

**Counter2.tsx**

```tsx
import { useCallback } from "react";
import useCounterContextSelector from "./src/hooks/useCounterContextSelector";
import { CounterStore } from "./store/CounterStoreContextProvider";

export default function Counter2() {
  const [count, setCount] = useCounterContextSelector(
    useCallback((state: CounterStore) => state.count, [])
  );

  function increment() {
    setCount((prev) => ({
      ...prev,
      count: prev.count + 1,
    }));
  }

  function decrement() {
    setCount((prev) => ({
      ...prev,
      count: prev.count - 1,
    }));
  }
  return (
    <>
      <h1>Counter2</h1>
      <p>count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </>
  );
}
```

<br>

위와 동일하게 Profile를관리하는 storeContext 및 selector hook를 추가생성하겠습니다.

**ProfileStoreContextProvider.tsx**

```tsx
import { createContext, useRef } from "react";
import createStore, { Store } from "./store/Store";
export type ProfileStore = {
  name: string;
  age: number;
};

export const ProfileStoreContext = createContext<Store<ProfileStore>>(
  createStore<ProfileStore>({ name: "", age: 0 })
);

export default function ProfileStoreContextProvider({
  initialState,
  children,
}: {
  initialState: ProfileStore;
  children: React.ReactNode;
}) {
  const storeRef = useRef<Store<ProfileStore>>();

  if (!storeRef.current) {
    storeRef.current = createStore(initialState);
  }

  return (
    <ProfileStoreContext.Provider value={storeRef.current}>
      {children}
    </ProfileStoreContext.Provider>
  );
}
```

<br>

**useProfileContextSelector.tsx**

```tsx
import { useContext, useEffect, useState } from "react";
import {
  ProfileStore,
  ProfileStoreContext,
} from "../../store/ProfileStoreContextProvider";

export default function useProfileContextSelector<State extends unknown>(
  selector: (state: ProfileStore) => State
) {
  const store = useContext(ProfileStoreContext);

  const [state, setState] = useState(() => selector(store.get()));

  useEffect(() => {
    // store 값이 변경될 때 마다 subscribe에 등록된 함수를 실행하여 state 값을 변경
    const unsubscribe = store.subscribe(() => {
      const value = selector(store.get());
      console.log(value);
      setState(value);
    });

    // 클린업 함수로 구독해제를 통해 기존에 쌓인 callback를 삭제
    return unsubscribe;
  }, [store, selector]);

  return [state, store.set] as const;
}
```

<br>

**Profile1.tsx**

```tsx
import { useCallback } from "react";
import useProfileContextSelector from "./src/hooks/useProfileContextSelector";

export default function Profile1() {
  const [profile, setProfile] = useProfileContextSelector(
    useCallback((state) => state, [])
  );

  function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    const formdata = new FormData(e.currentTarget);
    const name = formdata.get("name") as string;
    const age = +(formdata.get("age") as string);
    if (name && age) {
      setProfile((prev) => ({ ...prev, name, age }));
      e.currentTarget.reset();
    }
  }

  return (
    <div>
      <h1>Profile1</h1>
      <form onSubmit={handleSubmit}>
        <input name="name" placeholder="name" />
        <input name="age" type="number" placeholder="age" max={100} min={0} />
        <button>set</button>
      </form>
      <p>name : {profile.name}</p>
      <p>age : {profile.age}</p>
    </div>
  );
}
```

<br>

**Profile2.tsx**

```tsx
import { useCallback } from "react";
import useProfileContextSelector from "./src/hooks/useProfileContextSelector";

export default function Profile2() {
  const [profile, setProfile] = useProfileContextSelector(
    useCallback((state) => state, [])
  );

  function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    const formdata = new FormData(e.currentTarget);
    const name = formdata.get("name") as string;
    const age = +(formdata.get("age") as string);
    if (name && age) {
      setProfile((prev) => ({ ...prev, name, age }));
      e.currentTarget.reset();
    }
  }

  return (
    <div>
      <h1>Profile2</h1>
      <form onSubmit={handleSubmit}>
        <input name="name" placeholder="name" />
        <input name="age" type="number" placeholder="age" max={100} min={0} />
        <button>set</button>
      </form>
      <p>name : {profile.name}</p>
      <p>age : {profile.age}</p>
    </div>
  );
}
```

<br>

**App.tsx**

```tsx
import Counter1 from "./Counter1";
import Counter2 from "./Counter2";
import CounterStoreProvider from "./store/CounterStoreContextProvider";
import Profile1 from "./Profile1";
import Profile2 from "./Profile2";
import ProfileStoreContextProvider from "./store/ProfileStoreContextProvider";

function App() {
  return (
    <>
      <h1>No Provider</h1>
      <Counter1 />
      <Counter2 />
      <hr />
      <CounterStoreProvider initialState={{ count: 0 }}>
        <Counter1 />
        <Counter2 />
      </CounterStoreProvider>
      <hr />
      <ProfileStoreContextProvider initialState={{ name: "", age: 0 }}>
        <Profile1 />
        <Profile2 />
      </ProfileStoreContextProvider>
    </>
  );
}

export default App;
```

<br>

Provider가 존재하지 않아도 각각 초깃값을 정상적으로 가져오는것을 볼 수 있습니다.

이는 Provider의 작동 방식이 StoreContext를 생성 할 때 초깃값을 인수로 넘겨주었기 때문에 Provider가 없는 경우 이 초깃값을 사용하기 때문입니다. 즉, Prvoider가 없는 경우에는 `createStore`로 생성된 전역 상태가 참조되며, `ContextSelector` 훅에서 동일한 스토어를 참조하게 됩니다.

Provide가 존재하는 경우 Prvoider에 initialState에 넣은 값으로 값이 초기화됩니다.

이렇게 Context를 사용하면 store를 격리할 수 있으므로 store를 사용하는 컴포넌트는 해당 상태가 어느 store에서 온 상태인지 신경 쓰지 않아도 됩니다. 또한 부모와 자식 컴포넌트의 책임과 역할이 명시적인 코드로 나눌 수 있어 코드의 가독성과 유지보수가 향상됩니다.

<br>

### 정리

React의 hook를 이용하여 전역 상태 관리 라이브러리를 직접 구현해보았습니다.

현재 리액트 생태계에서는 많은 상태관리 외부 라이브러리(Redux, MobX, Zustand, Recoil, Jotai 등)가 존재합니다. 이런 라이브러리의 동작방식은 우리가 구현한 라이브러리의 동작 방식과 크게 다르지 않습니다.

useState, useReducer가 가지고 있는 컴포넌트 내부에서만 사용할 수 있다는 한계를 극복하기 위해 외부 스코프 어딘가에 상태를 저장하며, 이는 컴포넌트 최상단, 상태가 필요한 부모 혹은 격리된 자바스크립트 스코프 어딘가가 될 수도 있습니다. 이 외부의 상태 변경을 감지하여 컴포넌트 리렌더링 시키는 방식으로 동작하게됩니다.

<br>

### 참고 자료

[📘 wikibook - React Depp Dive](https://wikibook.co.kr/react-deep-dive/)
