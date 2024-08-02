# useReducer 깊게 살펴보기

## useReducer

> useReducer는 useState와 비슷한 역할을 하는 상태관리 hook으로 좀 더 복잡한 상태값을 미리 정해둔 틀에 따라 구조적으로 관리할 수 있도록 하는 hook입니다.

<br>

```jsx
// useReducer 선언 형태
const [state, dispatch] = useReducer(reducer, initalState, init);
```

**반환값**

- state : 현재 useReducer가 가지고 있는 값을 의미합니다.
- dispatch : state를 업데이트하는 함수입니다.
  - reducer 함수를 실행 시키는 역할을 합니다.
  - action 객체를 인자로 받으며 action 객체는 어떤 행동인지를 나타내는 type 속성과 해당 행동과 관련된 데이터(payload)를 담고 있습니다.
  - action을 이용하여 컴포넌트 내에서 state의 업데이트를 발생시킵니다.

**매개변수**

- reducer : useReducer의 기본 action을 정의하는 함수입니다.
- initalState : useReducer의 초기값을 설정합니다.
- init(optional) : 필수값이 아니며, 넘겨주는 init 함수가 존재한다면 게으른 초기화가 일어나면 initalState를 인수로 init 함수가 실행됩니다.

<br>

### useReducer의 사용

useReducer는 복잡해 보일 수 있지만, 복잡한 형태의 state를 사전에 정의된 dispatcher로만 수정할 수 있도록 제한하여 state 값을 변경하는 과정을 제한적으로 두고 이에 대한 변경을 빠르게 확인할 수 있게합니다.

단순한 number, boolean과 같이 간단한 값을 관리하는 것은 useState로 충분하지만 state하나가 가져야할 값이 복잡하고 이를 수정하는 경우의 수가 많다면 state를 useReducer로 관리하는 것이 state를 훨씬 관리하기 쉬워집니다.

아래는 예시 코드는 useReducer를 사용하여 구현한 todolist 예시 코드입니다.

```tsx
import React, { useReducer, useState } from "react";

type Todo = {
  id: number;
  text: string;
  completed: boolean;
};

type State = {
  todos: Todo[];
};

enum ActionType {
  ADD_TODO = "ADD_TODO",
  TOGGLE_TODO = "TOGGLE_TODO",
  REMOVE_TODO = "REMOVE_TODO",
}

type Action =
  | { type: ActionType.ADD_TODO; payload: string }
  | { type: ActionType.TOGGLE_TODO; payload: number }
  | { type: ActionType.REMOVE_TODO; payload: number };

export const initialState: State = {
  todos: [],
};

export const reducer = (state: State, action: Action): State => {
  switch (action.type) {
    case ActionType.ADD_TODO:
      const newTodo: Todo = {
        id: Date.now(),
        text: action.payload,
        completed: false,
      };
      return { ...state, todos: [...state.todos, newTodo] };
    case ActionType.TOGGLE_TODO:
      return {
        ...state,
        todos: state.todos.map((todo) =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        ),
      };
    case ActionType.REMOVE_TODO:
      return {
        ...state,
        todos: state.todos.filter((todo) => todo.id !== action.payload),
      };
    default:
      throw new Error("Unhandled action type");
  }
};

export default function TodoApp() {
  const [state, dispatch] = useReducer(reducer, initialState);
  const [newTodoText, setNewTodoText] = useState("");

  const handleAddTodo = () => {
    if (newTodoText.trim() !== "") {
      dispatch({ type: ActionType.ADD_TODO, payload: newTodoText });
      setNewTodoText("");
    }
  };

  return (
    <div>
      <h1>Todo List with useReducer</h1>
      <input
        type="text"
        value={newTodoText}
        onChange={(e) => setNewTodoText(e.target.value)}
        placeholder="Enter a new todo"
      />
      <button onClick={handleAddTodo}>Add Todo</button>
      <ul>
        {state.todos.map((todo) => (
          <li
            key={todo.id}
            style={{ textDecoration: todo.completed ? "line-through" : "none" }}
          >
            {todo.text}
            <button
              onClick={() =>
                dispatch({ type: ActionType.TOGGLE_TODO, payload: todo.id })
              }
            >
              {todo.completed ? "Undo" : "Complete"}
            </button>
            <button
              onClick={() =>
                dispatch({ type: ActionType.REMOVE_TODO, payload: todo.id })
              }
            >
              Remove
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

<br>

### useReducer와 useState

useReducer와 useState는 세부 동작과 쓰임에만 차이가 있을 뿐, 클로저를 활용해 값을 저장해서 state를 관리한다는 사실은 동일합니다.

따라서, useRedcuer로 useState을 useState로 useReducer를 구현해 볼 수 있습니다.

<br>

### useState로 useReducer구현하기

```tsx
import { useCallback, useMemo, useState } from "react";

// reducer 함수 타입 정의
// S: state, A: action
type Reducer<S, A> = (state: S, action: A) => S;

export default function useMyReducer<S, A>(
  reducer: Reducer<S, A>,
  initialArg: S | (() => S),
  init?: (arg: S) => S
) {
  // init 함수 존재시 init 함수에 initalArg를 인자로 주고, 함수를 실행
  // init 함수가 없을시 initalArg를 초기값으로 설정
  const [state, setState] = useState<S>(
    init ? () => init(initialArg as S) : (initialArg as S)
  );

  // useCallback: 리렌더링될 때마다 dispatch 함수가 새로 생성되는 것을 방지
  // setState에 reducer 함수를 사용하여 dispatch 함수 구현
  // reducer 함수에 prev를 통해 state 값 전달
  const dispatch = useCallback(
    (action: A) => {
      setState((prev: S) => reducer(prev, action));
    },
    [reducer]
  );

  // 컴포넌트가 리렌더링될 때마다 새로운 배열이 생성되는 것을 방지
  return useMemo(
    () => [state, dispatch] as [S, (action: A) => void],
    [state, dispatch]
  );
}
```

<br>

### useReducer로 useState 구현하기

```tsx
import { useReducer } from "react";

// state 타입 정의
type State<T> = T;

// action 타입 정의
type Action<T> = { type: "SET_STATE"; payload: T };

const reducer = <T,>(state: State<T>, action: Action<T>): State<T> => {
  switch (action.type) {
    case "SET_STATE":
      return action.payload;
    default:
      throw new Error("Unhandled action type");
  }
};

export default function useMyState<T>(initialState: T) {
  const [state, dispatch] = useReducer(reducer, initialState);

  const setState = (newState: T) => {
    dispatch({ type: "SET_STATE", payload: newState });
  };

  return [state, setState] as [T, (newState: T) => void];
}
```

<br>

### useMyReducer, useMyState 사용 예시

```tsx
import useMyReducer from "@/useMyReducer";
import useMyState from "@/useMyState";

type State = {
  count: number;
};

type Action = {
  type: ActionType;
  payload?: State;
};

enum ActionType {
  INCREMENT = "INCREMENT",
  DECREMENT = "DECREMENT",
  RESET = "RESET",
}

const reducer = (state: State, action: Action) => {
  switch (action.type) {
    case "INCREMENT": {
      return { ...state, count: state.count + 1 };
    }
    case "DECREMENT":
      return { ...state, count: state.count - 1 };
    case "RESET":
      return init(action.payload || { count: 0 });
    default:
      return state;
  }
};

const init = (state: State) => {
  return state;
};

export default function Component() {
  const [state, dispatch] = useMyReducer<State, Action>(
    reducer,
    { count: 0 },
    init
  );
  const [actionText, setActionText] = useMyState("");

  const handleIncrement = () => {
    dispatch({ type: ActionType.INCREMENT });
    setActionText("INCREMENT");
  };

  const handleDecrement = () => {
    dispatch({ type: ActionType.DECREMENT });
    setActionText("DECREMENT");
  };

  const handleReset = (initalCount: number) => {
    dispatch({ type: ActionType.RESET, payload: { count: initalCount || 0 } });
    setActionText("RESET");
  };

  return (
    <div>
      <h1 className="text-3xl text-center mt-5">useReducer & useState</h1>
      <p>count : {state.count}</p>
      <button onClick={handleIncrement}>+</button>
      <button onClick={handleDecrement}>-</button>
      <button onClick={() => handleReset(10)}>reset</button>
      <p>action : {actionText}</p>
    </div>
  );
}
```

<br>

### useReducer의 장점

- **복잡한 상태 로직 관리** : `useReducer`는 복잡한 상태 전환 로직을 명확하게 정의할 수 있습니다. 여러 상태가 관련된 경우, 상태 전환을 하나의 함수에서 처리할 수 있어 가독성이 높아집니다.
- **디버깅** : 상태 전환 로직이 모두 `reducer` 함수 안에 있기 때문에, 상태가 어떻게 변화하는지 쉽게 추적하고 예측할 수 있습니다.
- **테스트** : `reducer` 함수는 순수 함수여야 하기 때문에 테스트하기 쉽습니다. 특정 액션에 대해 상태가 어떻게 변하는지 단위 테스트가 가능합니다.
- **최적화**: 여러 하위 컴포넌트가 상태를 공유할 때, `useReducer`를 사용하면 불필요한 렌더링을 줄일 수 있습니다.

👉 [공식 문서](https://react.dev/learn/extracting-state-logic-into-a-reducer#comparing-usestate-and-usereducer)에서 useReducer의 장점에 대해 살펴볼 수 있습니다.

<br>

### useReducer는 주로 언제 사용할까?

**1 ) state의 형태가 복잡할 때**

복잡한 상태 로직 관리가 필요한 경우 상태 전환을 하나의 함수에서 처리할 수 있어 가독성이 향상되고 유지보수가 쉬워집니다.

**2 ) 분산된 setState 로직을 한 곳에 모으고 싶을 때**

reducer 함수는 컴포넌트 외부로 분리되어 있으며, 상태 전환 로직이 모두 reducer 함수 안에 있기 때문에 상태 관리 오류 발생 시 reducer 함수를 확인하면 되므로 디버깅에 용이합니다.

<br>

### 정리

useReducer나 useState 둘 다 상태관리에 사용되는 hook이며, 세부 동작과 쓰임에만 차이가 있을 뿐, 결국 클로저를 활용해 값을 저장하여 state를 관리합니다.

따라서, useReducer와 useState는 상황에 맞게 적절히 사용하면 됩니다.
