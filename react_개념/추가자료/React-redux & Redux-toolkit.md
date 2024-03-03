# React-redux & Redux-toolkit

## 1. Redux란 ?

> 자바스크립트 상태관리 라이브러리로 중앙집중식 저장소를 이용하여 어플리케이션의 상태를 예측하고 일관성있게 관리하기 위해 사용됩니다.
> 어플리케이션 규모가 커지거나 상태가 복잡해질 때 유용하게 사용할 수 있습니다.

<br/>

### Redux를 사용하는 이유

- 중앙 집중화된 상태관리 : Redux는 어플리케이션의 상태를 하나의 중앙 저장소에 저장하고 관리하기 때문에, 여러 컴포넌트에서 동일한 상태에 접근하고 관리할 수 있도록 해주기 때문에 상태변화를 추적하고 디버깅하기 유리합니다.
- 예측 가능한 상태변화 : Redux는 상태변화를 순수한 함수인 reducer를 통해 관리 하기 때문에, 이전 상태와 액션을 기반으로 다음 상태를 예측 가능함으로 예측 불가한 동작을 예방할 수 있습니다.
- 시간 여행 및 디버깅 : Redux는 개발자가 상태 변화를 쉽게 디버깅하고 분석할 수 있습니다. 특히 Redux의 시간 여행 기능을 사용하여 특정 시점의 상태와 이전 상태를 비교하여 버그나 문제를 해결할 수 있습니다.
- 상태 공유 및 전파의 용이 : 여러 컴포넌트 간에 상태를 공유하거나 특정 상태 변화를 다른 부분으로 전파하기가 용이합니다. 컴포넌트 간의 데이터 전달을 더 효율적으로 처리할 수 있습니다.

<br/>

### Redux의 구성 및 작동원리

![redux](https://github.com/NamJongtae/react-study/assets/113427991/e15b8fae-ff71-41ea-819e-34e1ce07ec18)

- `store` : 어플리케이션의 전체 상태를 포함하는 단일 객체 저장소입니다. 상태의 모든 변경사항은 스토어에서 관리됩니다.
- `state` : 어플리케이션의 데이터를 담고 있는 객체입니다.
- `getState` : 스토어에서 해당되는 현재 상태를 가져오는 메서드입니다.
- `reducer` : 상태 변화를 일으키는 함수로 현재 상태와 전달받은 액션 객체를 파라미터로 받아 옵니다. 두 값을 참고하여 새로운 값을 만들어서 반환해 줍니다. reducer에서는 상태의 불변성을 유지하면서 데이터에 변화를 발생시켜야 합니다.
- `action` : 상태를 변경하려는 의도나 목적을 설명하는 객체입니다. 어플리케이션에서 어떤 일이 발생했는지 나타내며, type 속성을 반드시 포함해야 합니다. 그외 속성 값들을 추가가능하며, 나중에 업데이트할 상태를 나타냅니다.
- `dispatch` : 액션을 스토어에 보내 상태 변화를 요청하는 메서드입니다. 액션을 dispatch하면 해당 액션에 맞는 리듀서가 호출되어 상태가 업데이트됩니다.
- `subscribe` : 스토어의 변경 사항을 감지하고 리스너 함수를 호출하는 메서드입니다. Redux 스토어 내부의 상태가 변경되면, subscribe를 사용하여 등록된 리스너 함수가 호출되어 변경 사항을 처리할 수 있습니다.

<br/>

### Redux 세 가지 원칙

- 단일 스토어 : 하나의 어플리케이션 안에는 하나의 스토어만 존재해야합니다. 여러 개의 스토어를 사용하는 것이 가능하지만 상태 관리가 복잡해질 수 있어 권장되지 않습니다.
- 읽기 전용 상태 : redux 상태는 읽기전용 입니다. 상태를 업데이트할 때 기존의 객체를 변경시키지 않고 새로운 객체를 생성해주어야 합니다. redux에서는 불변성을 유지해야하는 이유는 내부적으로 데이터가 변경되는 것을 감지하기 위해 얕은 비교 검사를 하기 때문입니다.
- 순수 함수 : 변화를 일으키는 redcer 함수는 순수한 함수여야 합니다.

**💡 reducer 순수함수의 조건**
- reducer 함수는 이전 상태와 액션 객체를 파라미터로 받습니다.
- 파라미터 값 이외에는 의존하면 안됩니다.
- 이전 상태는 절대로 변경해선 안되며, 변경을 준 새로운 상태 객체를 만들어서 반환해야합니다.
- 같은 파라미터로 호출된 reducer 함수는 언제나 똑같은 출력을 해야합니다.

<br/>

## 2. React-redux ?

> Redux를 React와 연동하여 사용하기 편리하도록 만든 라이브러입니다.

<br/>

### React-redux 구성

- `Provider` : Provider 컴포넌트는 모든 컴포넌트에서 스토어에 접근할 수 있도록
  합니다.
- `createStore()` : createStore() 메서드는 인자 값으로 reducer 함수를 받으며, 스토어를 생성해주는 역할을 합니다.
- `reducer` : 상태 변화를 일으키는 함수로 현재 상태와 전달받은 액션 객체를 파라미터로 받아 옵니다. 두 값을 참고하여 새로운 값을 만들어서 반환해 줍니다. reducer에서는 상태의 불변성을 유지하면서 데이터에 변화를 발생시켜야 합니다.
- `useSelector()` : 스토어의 상태를 선택적으로 가져오는 역할하는 hook입니다. 인자로 콜백함수을 받으며, 이 함수는 스토어의 상태 인자로 받아 선택한 상태를 반환합니다.
- `useDispatch()` : 액션을 dispatch 하는데 사용되는 hook으로, 스토어의 dispatch 메서드를 반환합니다. 반환된 dispatch 메서드는 인자 값으로 action를 받습니다.
- `disptach()` : 액션을 스토어에 보내 상태 변화를 요청하는 메서드입니다. 액션을 dispatch하면 해당 액션에 맞는 리듀서가 호출되어 상태가 업데이트됩니다.

<br/>

### React-redux 사용 예시 코드

**index.js**

`Provider`로 리덕스를 사용할 컴포넌트를 감싸고, 사용할 `store`를 넣어줍니다.

```jsx
import React from "react";
import ReactDOM from "react-dom/client";
import ReduxTest, { store } from "./ReduxTest";
import { Provider } from "react-redux";

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(
  <React.StrictMode>
    <Provider store={store}>
      <ReduxTest />
    </Provider>
  </React.StrictMode>
);
```

<br/>

**ReduxTest.jsx**

```jsx
import React from "react";
import { createStore } from "redux";
import { useSelector, useDispatch } from "react-redux";

// action type 설정
const ACTION_TYPES = {
  increment: "INCREMENT",
  decrement: "DECREMENT",
};

// reducer 함수 정의
export function reducer(state = { counter: 0 }, action) {
  switch (action.type) {
    case ACTION_TYPES.increment:
      // 카운터 증가
      return { counter: state.counter + 1 };

    case ACTION_TYPES.decrement:
      // 카운터 감소
      return { counter: state.counter - 1 };

    default:
      return state;
  }
}
// store 생성
export const store = createStore(reducer);

export default function ReduxTest() {
  // counter state 가져오기
  const counter = useSelector((state) => state.counter);
  // disptach 함수 가져오기
  const dispatch = useDispatch();

  // disptach를 통해 increment action 전달
  const handleIncrement = () => {
    dispatch({ type: ACTION_TYPES.increment });
  };
  // disptach를 통해 decrement action 전달
  const handleDecrement = () => {
    dispatch({ type: ACTION_TYPES.decrement });
  };

  return (
    <>
      <div>redux counter</div>
      <div>{counter}</div>
      <button onClick={handleIncrement}>+</button>
      <button onClick={handleDecrement}>-</button>
    </>
  );
}
```

<br/>

## 3. Redux-toolkit ?

> Reudx를 보다 편리하게 사용하기 위해 개발된 라이브러리입니다.

<br/>

### Redux-toolkit의 장점

- 액션 및 리듀서 관리: createSlice를 통해 액션 생성자 함수와 리듀서를 한 번에 생성하고 관리할 수 있습니다. 이를 통해 코드의 중복을 줄이고 관련된 로직을 한 곳에서 관리할 수 있습니다.
- 불변성 유지: createSlice와 configureStore 등의 함수들이 내부적으로 불변성을 유지하는 코드를 자동으로 생성해줍니다. 이를 통해 불변성 관련 작업에 대한 걱정을 덜 수 있습니다.
- 타입스크립트 지원: redux-toolkit은 타입스크립트와 함께 사용할 때 높은 타입 안정성을 제공합니다. 리듀서와 상태에 대한 타입 추론이 자동으로 이루어지며, 타입 에러를 사전에 방지할 수 있습니다.
- 비동기 로직 처리: createAsyncThunk를 사용하여 비동기 로직을 더 쉽게 처리할 수 있습니다. 비동기 작업의 상태를 자동으로 관리하고 처리 결과를 상태로 업데이트할 수 있습니다.
- Immer 통합: redux-toolkit은 내부적으로 Immer 라이브러리를 사용하여 불변성을 유지하면서 더 쉽게 상태를 업데이트할 수 있게 합니다.
- 간결한 코드: redux-toolkit을 사용하면 기존 Redux 코드보다 더 간결하고 읽기 쉬운 코드를 작성할 수 있습니다.

<br/>

### 기존 React-redux와 사용 차이점

- `configureStore()` : createStroe()와 비슷한 역할을 수행하며, 더 편리하게 스토어를 생성하고 설정하는 데 도움을 주는 함수입니다. 인자 값으로는 필수 값인 reducer, middleware 등이 올 수 있습니다. reducer 인자 값은 객체로 형태로 들어가며, 리듀서 슬라이스 객체들을 받습니다.
- `createSlice()` : 리듀서와 액션 생성자 함수를 한 번에 생성하고 관리하기 위해 사용됩니다. 인자 값으로 name, initialState, reducers를 받습니다.
- `dispatch()` : 기존 disptach와 동일한 기능을 하지만, 기존 dispatch와 다르게 **slice.actions**를 통해 액션 생성자 함수와 리듀서 함수에 접근할 수 있습니다. 이를 통해 액션 생성자 함수를 직접 가져오는 것보다 편리하게 해당되는 reduce를 호출할 수 있습니다.

<br/>

### React-toolkit 사용 예시 코드

**index.js**

```jsx
import React from "react";
import ReactDOM from "react-dom/client";
import ReduxTest, { store } from "./ReduxTest";
import { Provider } from "react-redux";

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(
  <React.StrictMode>
    <Provider store={store}>
      <ReduxTest />
    </Provider>
  </React.StrictMode>
);
```

<br/>

**counterSlice.js**

```jsx
import { createSlice } from "@reduxjs/toolkit";
// slice 생성
export const counterSlice = createSlice({
  name: "counterSlice",
  // 상태 초기값 설정
  initialState: {
    counter: 0,
  },
  // reducer 함수 설정
  reducers: {
    // counter 증가
    increment: (state) => {
      state.counter += 1;
    },
    // count 감소
    decrement: (state) => {
      state.counter -= 1;
    },
  },
});
```

<br/>

**ReduxTest.jsx**

```jsx
import { configureStore } from "@reduxjs/toolkit";
import { counterSlice } from "./counterSlice";
import { useSelector, useDispatch } from "react-redux";
// store 생성
export const store = configureStore({
  // reducer 적용
  reducer: {
    counterSlice: counterSlice.reducer,
  },
});

export default function ReduxTest() {
  // counterSlice에서 counter state를 가져옴
  const counter = useSelector((state) => state.counterSlice.counter);
  // dispatch 메서드 가져옴
  const dispatch = useDispatch();
  // dispatch를 통해 increment action 전달
  const handleIncrement = () => {
    dispatch(counterSlice.actions.increment());
  };
  // dispatch를 통해 decremetn action 전달
  const handleDecrement = () => {
    dispatch(counterSlice.actions.decrement());
  };

  return (
    <>
      <div>redux counter</div>
      <div>{counter}</div>
      <button onClick={handleIncrement}>+</button>
      <button onClick={handleDecrement}>-</button>
    </>
  );
}
```

<br/>

### 참고 사이트

[[코딩병원] react-redux 정리 및 예제](https://itprogramming119.tistory.com/entry/React-react-redux-예제)
