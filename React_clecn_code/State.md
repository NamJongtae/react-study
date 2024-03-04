# 2. State

## 1. State 소개

### 상태 관리에 필요한 것

- **컴포넌트 상태**
- **전역 상태**
- **서버 상태**
- **상태 변경**
- **상태 최적화**
- **렌더링 최적화**
- **불변성**
- **상태 관리자**

<br/>

### 상태를 관리하는 이유가 무엇인가?

**상태 관리에 필요한 것들이 대해 왜 필요한지를 생각해봅시다.**

- **컴포넌트 상태** : 컴포넌트 상태는 컴포넌트의 동작이나 렌더링을 결정하는 중요한 요소입니다. 이를 통해 사용자 인터랙션에 대응하거나, 시간에 따라 변화하는 정보를 표현하게 됩니다. 따라서, 컴포넌트의 상태를 적절히 관리하는 것은 사용자 경험을 좌우하게 됩니다.
- **전역 상태** : 애플리케이션의 여러 컴포넌트가 공유하는 정보를 관리하기 위해 전역 상태가 필요합니다. 이를 통해 컴포넌트 간의 독립성을 유지하면서도 필요한 정보를 공유할 수 있게 됩니다. 전역 상태를 관리하는 것은 애플리케이션의 복잡성을 줄이고, 코드의 가독성을 높이는 데 도움이 됩니다.
- **서버 상태** : 사용자의 요청에 따라 서버에서 관리되는 데이터가 변경될 수 있습니다. 이러한 변경사항을 클라이언트에 반영하기 위해서는 서버 상태를 적절하게 관리해야 합니다. 서버 상태 관리는 데이터 일관성을 유지하고, 사용자에게 최신 정보를 제공하는 데 중요합니다.
- **상태 변경** : 상태 변경은 사용자의 인터랙션, 서버의 응답 등에 따라 발생하게 됩니다. 상태 변경을 적절히 관리하는 것은 애플리케이션의 동작을 제어하고, 예상치 못한 오류를 방지하는 데 필요합니다.
- **상태 최적화** : 상태의 변경이 많아질수록 애플리케이션의 성능에 영향을 줄 수 있습니다. 따라서, 상태 변경을 최적화하는 것은 애플리케이션의 반응성을 유지하고, 사용자 경험을 향상시키는 데 중요합니다.
- **렌더링 최적화** : 컴포넌트의 렌더링은 리소스를 많이 소모하는 작업입니다.

<br/>

## 2. 올바른 초기값 설정

**올바른 초기값 설정은 매우 중요합니다.**

**초기값은 초기 렌더링될 때 순간적으로 보여질 수 있는 값입니다.**

**초기값을 제대로 설정하지 않은 경우 렌더링 문제, 무한 루프, 타입 불일치로 의도치 않은 동작이 발생할 수 있습니다.**

**상태를 초기화하는 경우 초기값을 잘 지정해두어야 원상태로 돌아갈 수 있습니다.**

**null 처리를 할때 불필요한 예외 처리 코드를 줄일 수 있습니다.**

리액트에서 빈 값으로 state를 설정하면 초기값으로 `undefined`가 설정됩니다.

초기값이 `undefiend`가 된다면 초기 렌더링시 문제가 발생할 수 있습니다.

아래 예시 코드에서 list는 map를 이용하여 출력하고 있습니다.

하지만 list의 초기값은 undefined이기 때문에 초기 렌더링 오류가 발생하게됩니다.

물론 이 오류를 해결하기 위해 `Array.isArray()`와 같은 코드를 넣을 수 있습니다.

하지만 이런 코드는 코드를 증가시키는 불필요한 코드로 작용하게됩니다.

```jsx
import { useState, useEffect } from "react";

function InitalState() {
  const [list, setList] = useState();

  useEffect(() => {
    setList([1, 2, 3, 4, 5]);
  }, []);

  return (
    <ul>
      {list.map((el) => {
        <li key={el}>{el}</li>;
      })}
    </ul>
  );
}

export default InitalState;
```

<br/>

아래와 같이 초기값을 설정해주는 것 만으로도 충분히 해결가능하기 때문입니다.

```jsx
import { useState, useEffect } from "react";

function InitalState() {
  // 초기값 []을 주어 불필요한 예외처리 코드 제거
  const [list, setList] = useState([]);

  useEffect(() => {
    setList([1, 2, 3, 4, 5]);
  }, []);

  return (
    <ul>
      {list.map((el) => {
        <li key={el}>{el}</li>;
      })}
    </ul>
  );
}

export default InitalState;
```

<br/>

초기값 설정에서 올바른 타입 설정 또한 중요합니다.

아래 예시 코드에서 count state를 문자열로 처리 해두고 setCount()에서 숫자 1를 더해주었습니다.

이런게 된다면 문자열과 숫자가 더해져 문자열 ‘11’ 되어버려 원하는 동작을 하지 않게 됩니다.

```jsx
import { useState } from "react";

function InitalState() {
  const [count, setCount] = useState("0");
  return (
    <>
      <p> Count : {count} </p>
      <button onClick={() => setCount(count + 1)}>Increment
      <button>
    </>
  );
}

export default InitalState;
```

<br/>

## 3. 업데이트 되지 않는 값

업데이트 되지 않는 값의 경우 컴포넌트 외부로 내보내는 것이 좋습니다.

**컴포넌트가 렌더링 될 때 마다 같은 값이 재생성되기 때문입니다.**

```jsx
function NoUpdateValue() {
  const VALUE = {
    name: "REACT",
    value: "1",
  };
  //...
}
```

```jsx
const VALUE = {
  name: "REACT",
  value: "1",
};

function NoUpdateValue() {
  //...
}
```

<br/>

## 4. 플래그 값 설정하기

**플래그 값** : 프로그래밍에서 주로 특정 조건 혹은 제어를 위한 조건을 `boolean`으로 나타내는 값

아래는 input의 유효성을 검사하고 통과해야만 버튼을 활성화 시키기위해 isDisabled 라는 플래그 상태값을 사용하였습니다.

```jsx
import { useState } from "react";

function FlagValue() {
  const [emailIsValid, setEmailIsValid] = useState(false);
  const [pwIsValid, setPwIsValid] = useState(false);
  const [isDisabled, setIsDisabled] = useState(true);

  useEffect(() => {
    if (emialIsValid && pwIsValid) {
      setIsDisabled(false);
    } else {
      setIsDisabled(true);
    }
  }, []);

  return (
    <>
      <input id="email" placeholder="Email" />
      <input id="pw" placeholder="PW" />
      <button disabled={isDisabled}>로그인</button>
    </>
  );
}

export default FlagValue;
```

<br/>

하지만 이런 플래그 상태 값은 불필요한 상태값으로 작용합니다.

아래 코드처럼 플래그 상태 값을 제거하고 isDisaled라는 상수로 만들어 플래그 값을 표현식 처리하면 불필요한 상태값을 사용하지 않고도 버튼을 활성/비활성화 할 수 있습니다.

```jsx
import { useState } from "react";

function FlagValue() {
	const [emailIsValid, setEmailIsValid] = useState(false);
	const [pwIsValid, setPwIsValid] = useState(false);
	const isDisabled = !(emailIsValid && pwIsValid);

	return (
		<input id="email" placeholder="Email"/>
		<input id="pw" placeholder="PW"/>
		<button disabled={ isDisabled }>로그인</button>
	);
}

export default FlageValue;
```

<br/>

## 5. 불필요한 상태 제거하기

불필요한 상태를 만드는 것은 상태관리 값이 늘어나는 것입니다.

이로인해 렌더링에 영향을 주는 값이 늘어나기 때문에 성능 저하의 원인이 될 수 있습니다.

따라서 상태로 관리해야 하는 값인지를 자세히 생각해보면서 상태 관리를 적용해야합니다.

```jsx
const TODO_DATA = [
  {
    id: 1,
    title: "study",
    completed: false,
  },
  {
    id: 2,
    title: "walking",
    completed: true,
  },
  {
    id: 3,
    title: "eating",
    completed: true,
  },
];

function UnnecessaryState() {
  const [todoList, setTodoList] = useState(TODO_DATA);
  const [completedTodoList, setCompletedTodoList] = useState(TODO_DATA);

  useEffect(() => {
    const newList = completedTodoList.filter((todo) => todo.completed === true);
    setCompletedTodoList(newList);
  }, []);

  //...
}
```

<br/>

불필요한 completedTodoList 상태를 제거할 수 있습니다.

completedTodoList를 컴포넌트 내부의 변수로 선언합니다.

이렇게 되면 렌더링 마다 고유한 값을 가질 수 있어 상태를 사용하지 않더라도 값을 사용할 수 있습니다.

```jsx
const TODO_DATA = [
  {
    id: 1,
    title: "study",
    completed: false,
  },
  {
    id: 2,
    title: "walking",
    completed: true,
  },
  {
    id: 3,
    title: "eating",
    completed: true,
  },
];

function UnnecessaryState() {
  const [todoList, setTodoList] = useState(TODO_DATA);
  const completedTodoList = TODO_DATA.filter((todo) => todo.completed === true);

  //...
}
```

<br/>

## 6. useState 대신 useRef 활용하기

**리렌더링 방지가 필요하다면 useState 대신 useRef를 활용합니다.**

**컴포넌트의 전체적인 수명과 동일하게 지속된 정보를 일관적으로 제공해야하는 경우 useRef를 사용합니다.**

아래 코드는 isMount 상태는 상태로 사용하지 않아도 되는 값입니다.

isMount를 상태로 사용하여 불필요한 리렌더링이 많이 발생됩니다.

```jsx
import { useEffect } from "react";

function RefInsteadOfState() {
  const [isMount, setIsMount] = useState(false);

  useEffect(() => {
    if (!isMount) {
      setIsMount(true);
    }
  }, [isMount]);

  return <div>{isMount && "컴포넌트 마운트"}</div>;
}

export default RefInsteadOfState;
```

<br/>

**아래코드에서는 useRef를 사용하여 리렌더링을 방지하게됩니다.**

**이렇게 useRef를 사용하면 컴포넌트와 동일한 생명주기를 가지기 때문에 일관적인 값을 안전하게 제공할 수 있습니다.**

**반드시 DOM에서만 ref를 사용하는 것은 아닙니다.**

```jsx
import { useEffect, useRef } from "react";

function RefInsteadOfState() {
  const isMount = useRef(false);

  useEffect(() => {
    isMount.current = true;
    return () => (isMount.current = false);
  }, []);

  return <div>{isMount && "컴포넌트 마운트"}</div>;
}

export default RefInsteadOfState;
```

<br/>

## 7. 연관된 상태 단순화하기

아래 코드는 모두 연관된 상태를 모두 다른 상태로 관리하고 있습니다.

이런 상태 관리 코드는 좋지 못한 코드입니다.

연관된 코드는 여러가지 방법으로 상태를 단순화 할 수 있습니다.

```jsx
import { useEffect, useState } from "react";

function FlatState() {
  const [isLoading, setIsLoading] = useState(false);
  const [isSuccess, setIsSuccess] = useState(false);
  const [isError, setIsError] = useState(false);

  const fetchData = async () => {
    setIsLoading(true);

    await fetch("url")
      .then(() => {
        setIsLoading(false);
        setIsSuccess(true);
      })
      .catch((err) => {
        setIsLoading(false);
        setIsError(true);
      });
  };

  useEffect(() => {
    fetchData();
  }, []);

  if (isLoading) return <div>Loading...</div>;
  if (isError) return <div>Error Failed Load Data!</div>;
  if (isSuccess) return <div>Success Load Data!</div>;
}

export default FlatState;
```

<br/>

### **1 ) 연관된 상태를 열거형 데이터로 바꾸기**

```jsx
import { useEffect, useState } from "react";

const FETCH_STATE = {
  UNINITIALIZED: "uninitialized",
  LOADING: "loading",
  SUCCESS: "sucess",
  ERROR: "error",
};

function FlatState() {
  const [fetchState, setFetchState] = useState(FETCH_STATE.UNINITIALIZED);

  const fetchData = async () => {
    setPromiseState(FETCH_STATE.LOADING);

    await fetch("url")
      .then(() => {
        setPromiseState(FETCH_STATE.SUCCESS);
      })
      .catch((err) => {
        setPromiseState(FETCH_STATE.ERROR);
      });
  };

  useEffect(() => {
    fetchData();
  }, []);

  if (fetchState === FETCH_STATE.LOADING) return <div>Loading...</div>;
  if (fetchState === FETCH_STATE.ERROR)
    return <div>Error Failed Load Data!</div>;
  if (fetchState === FETCH_STATE.SUCCESS) return <div>Success Load Data!</div>;
}

export default FlatState;
```

<br/>

### 2 ) **연관된 상태를 객체로 묶어내기**

```jsx
import { useEffect, useState } from "react";

function ObjectState() {
  const [fetchState, setFetcheState] = useState({
    isLoading: false,
    isSucess: false,
    isError: false,
  });

  const fetchData = async () => {
    setFetcheState((prev) => ({ ...prev, isLoading: true }));

    await fetch("url")
      .then(() => {
        setFetcheState((prev) => ({
          ...prev,
          isLoading: false,
          isSucess: true,
        }));
      })
      .catch((err) => {
        setFetcheState((prev) => ({
          ...prev,
          isLoading: false,
          isError: true,
        }));
      });
  };

  useEffect(() => {
    fetchData();
  }, []);

  if (fetchState.isLoading) return <div>Loading...</div>;
  if (fetchState.isSucess) return <div>Error Failed Load Data!</div>;
  if (fetchState.isError) return <div>Success Load Data!</div>;
}

export default ObjectState;
```

<br/>

### 3 ) useReducer로 상태 단순화하기

여러가지 상태가 연관되어 있을때 useState 대신 useReducer를 사용하면 상태를 구조화할 수 있습니다.

```jsx
import { useEffect, useReducer } from "react";

const ACTION_TYPE = {
  LOADING: "LOADING",
  SUCCESS: "SUCCESS",
  ERROR: "ERROR",
};

const INIT_STATE = {
  isLoading: false,
  isSuccess: false,
  isError: false,
};

const reducer = (state, action) => {
  switch (action.type) {
    case ACTION_TYPE.LOADING:
      return { ...state, isLoading: true };
    case ACTION_TYPE.SUCCESS:
      return { ...state, isLoading: false, isSucess: true };
    case ACTION_TYPE.ERROR:
      return { ...state, isLoading: false, isError: true };
    default:
      return INIT_STATE;
  }
};

function StateToRedcuer() {
  const [state, dispatch] = useReducer(reducer, INIT_STATE);

  const fetchData = async () => {
    dispatch({ type: ACTION_TYPE.LOADING });

    await fetch("url")
      .then(() => {
        dispatch({ type: ACTION_TYPE.SUCCESS });
      })
      .catch((err) => {
        dispatch({ type: ACTION_TYPE.ERROR });
      });
  };

  useEffect(() => {
    fetchData();
  }, []);

  if (state === ACTION_TYPE.LOADING) return <div>Loading...</div>;
  if (state === ACTION_TYPE.ERROR) return <div>Error Failed Load Data!</div>;
  if (state === ACTION_TYPE.SUCCESS) return <div>Success Load Data!</div>;
}

export default FlatState;
```

<br/>

### 4 ) Custom Hook으로 상태 뽑아내기

확장성 있게 재사용할 수 있도록 Custom Hook으로 상태를 뽑아낼 수 있습니다.

```jsx
import { useEffect, useReducer } from "react";

const ACTION_TYPE = {
  LOADING: "LOADING",
  SUCCESS: "SUCCESS",
  ERROR: "ERROR",
};

const INIT_STATE = {
  isLoading: false,
  isSuccess: false,
  isError: false,
};

function useFetch(url) {
  const [state, dispatch] = useReducer(reducer, INIT_STATE);

  const fetchData = async () => {
    dispatch({ type: ACTION_TYPE.LOADING });

    await fetch(url)
      .then(() => {
        dispatch({ type: ACTION_TYPE.SUCCESS });
      })
      .catch((err) => {
        dispatch({ type: ACTION_TYPE.ERROR });
      });
  };

  useEffect(() => {
    fetchData();
  }, [url]);

  return state;
}

export default useFetch;
```

```jsx

import useFetch "./useFetch";

function CustomHook () {
	const { isLoading, isSuccess, isError } = useFetch("url");

  if (isLoading) return <div>Loading...</div>;
  if (isSuccess) return <div>Error Failed Load Data!</div>;
  if (isError) return <div>Success Load Data!</div>;
}
```

<br/>

## 8. 이전 상태 활용하기

`prevState`를 사용하지 않고 상태를 업데이트 한다면 예측하지 못한 에러를 발생 시킬 수 있습니다.

아래 코드에서는 setCount()를 여러번 사용하여 count를 5씩 증가 시키려고 합니다.

하지만 아래 코드는 버튼을 눌러도 1씩 count가 증가하게됩니다.

`setState`는 일반적으로 배치(batch) 업데이트를 수행하기 때문에, 여러번 `setState`를 호출하더라도 실제로 업데이트는 한 번만 일어나기때문입니다.

```jsx
import React, { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  const increaseCount = () => {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
  };

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={increaseCount}>Increment</button>
    </div>
  );
}

export default Counter;
```

<br/>

`prevState`를 사용하여 이전 상태를 받아와 새로운 상태를 계산하는 방식으로 state의 연속적인 업데이트가 가능합니다.

```jsx
import React, { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  const increaseCount = () => {
    setCount((prevState) => prevState + 1);
    setCount((prevState) => prevState + 1);
    setCount((prevState) => prevState + 1);
    setCount((prevState) => prevState + 1);
    setCount((prevState) => prevState + 1);
  };

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={increaseCount}>Increment</button>
    </div>
  );
}

export default Counter;
```
