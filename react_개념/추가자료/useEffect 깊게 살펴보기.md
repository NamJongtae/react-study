# useEffect 깊게 살펴보기

## useEffect란?

> 보통 useEffect는 생명주기 메서드를 대체하기 위해 만들어진 hook이라 생각하게됩니다. 하지만 정확하게는 컴포넌트의 여러 값들을 활용하여 동기적으로 side Effect를 만드는 매커니즘입니다.

<br>

### useEffect

useEffect는 첫 번째 인자로 side Effect를 발생시킬 함수, 두 번째 인자로는 의존성 배열을 전달합니다.

의존성 배열의 경우, 빈 배열, 배열, 배열을 생략할 수 있습니다.

useEffect는 의존성 배열의 값이 변경되면, 실행되게됩니다.

그럼 useEffect는 어떻게 의존성 배열의 값이 변경되는 것을 알 수 있을까요?

함수형 컴포넌트는 매번 함수를 실행하여 렌더링을 수행하게됩니다.

함수형 컴포넌트는 렌더링 시 마다 고유한 state와 props 값을 가지고 있습니다.

useEffect는 렌더링 마다 의존성 배열에 있는 값을 확인하면서 의존성 값이 이전 값과 변화 한다면 부수 효과 함수를 실행시키는 함수입니다.

state 변경시 렌더링이 발생, useState는 state 값이 변경된 것을 확인하면, console.log를 실행합니다.

```jsx
function Component() {
  const [state, setState] = useState();

  useEffect(() => {
    console.log("state change");
  }, [state]);

  //...
}
```

<br>

### cleanup 함수

`cleanup 함수`는 생명 주기 메서드 언마운트 개념과 차이가 존재합니다. 함수 컴포넌트가 리렌더링 되었을 때 의존성 변화가 있었던 당시 이전 값을 기준으로 실행되며, 이전 값을 청소해주는 함수입니다.

즉, 실행 될 때마다 이전의 `cleanup` 함수가 존재한다면 `cleanup` 함수를 실행한 다음에 콜백을 실행하게됩니다.

cleanup 함수는 주로 이벤트를 등록하고 지울때 사용됩니다.

아래 코드 실행시 useEffect에서 클릭시 cleanup 함수가 실행되고, 이후 부수효과 함수가 실행되는 것을 볼 수 있습니다.

```jsx
function Component() {
  const [count, setCount] = useState(0);

  const onClick = () => {
    setCount((prev) => prev + 1);
  };

  useEffect(() => {
    const addEvent = () => {
      console.log("click", count);
    };

    // 렌더링 시 이벤트 등록
    window.addEventListener("click", addEvent);

    // 다음 렌더링이 끝난 뒤 cleanup 함수가 실행
    return () => {
      console.log("cleanup");
      window.removeEventListener("click", addEvent);
    };
  }, [count]);

  return (
    <>
      <p>{count}</p>
      <button onClick={onClick}>increment</button>
    </>
  );
}
```

<br>

### useEffect 의존성 배열

useEffect의 의존성 배열은 아래 3가지 경우로 값을 넣을 수 있습니다.

- 의존성 배열이 존재하는 경우 : 배열의 포함된 값이 변경될 때 마다 실행됩니다.
- 의존성 배열이 빈 배열인 경우 : 비교할 의존성이 없으므로 최초 렌더링시 한 번만 실행됩니다.
- 의존성 배열이 존재하지 않는 경우 : 렌더링할 때 마다 실행됩니다.

의존성 배열이 없는 useEffect와 컴포넌트 안에서 사용하는 함수의 차이점

아래 코드에서는 매 렌더링시 마다 실행되는 console.log와 useEffect에 의해 실행되는 console.log 가 존재합니다. 이것의 차이점은 무엇일까요?

```jsx
function Component() {
  // SSR에서 실행
  console.log("렌더링1");

  // SSR(X) CSR에서 실행
  useEffect(() => {
    console.log("렌더링2");
  });
}
```

컴포넌트 내부에 실행되는 함수는 컴포넌트 렌더링 도중 실행되며, useEffect 함수 내부에서 실행되는 함수의 경우 렌더링이 완료된 이후 실행됩니다.

SSR의 관점에서 보면 useEffect는 SSR에서 실행되지 않고, CSR에서 실행되며, 컴포넌트 내부 실행 함수의 경우에는 SSR에 포함되며 SSR 도중 실행되게 됩니다. 이는 컴포넌트 내부 실행 함수가 SSR를 지연시킨다는 것을 의미하게됩니다.

만약 무거운 작업이 컴포넌트 내부에서 실행된다면, 이는 렌더링을 지연시키고, 성능 저하에 원인될 수 있습니다.

컴포넌트 내부에서 함수를 실행하는 것은 지양해야 하며, useEffect hook를 통해 렌더링 후 부수효과함수를 통해 실행하는 것이 바람직합니다.

<br>

### useEffect 직접 구현하기

useEffect hook를 비슷하게 구현해봅니다.

useEffect는 의존성 배열의 값을 이전 값과 얕은 비교를 통해 변경이 일어났는지 확인하며, 변경이 일어난 경우 부수 효과 콜백 함수를 실행하게됩니다.

```jsx
const React = (function () {
  const global = {};
  let index = 0;

  function useEffect(callback, dependencies) {
    const hooks = global.hooks;

    // 이전 훅이 있는지 확인합니다.
    let previousDependencies = global.hooks[index];

    // 의존성 배열에 값이 변경되었는지 확인합니다.
    // 이전 값이 있을 경우 이전 값을 얕은 비교를 통해 변경이 일어났는지 확인합니다.
    // 이전 값이 없는 경우 첫 렌더링인 경우이므로 변경이 일어난 것으로 취급합니다.
    let isDependeniesChanged = previousDependencies
      ? dependencies.some(
          (value, index) => !Object.is(value, previousDependencies)
        )
      : true;

    // 변경이 일어났다면 부수효과 callback 함수를 실행합니다.
    if (isDependeniesChanged) {
      callback();

      // 다음 훅이 일어날 때 비교를 위해 index를 추가해줍니다.
      index++;

      // 현재 의존성을 훅에 다시 저장합니다.
      hooks[index] = dependencies;
    }

    return { useEffect };
  }
})();
```

<br>

### useEffect 사용시 주의점

**1 ) 무한 루프**

useEffect 사용시 의도치 않게 무한 루프를 발생 시킬 수 있습니다. 이를 주의하며 사용해야합니다.

```jsx
function Component() {
  const [count, setCount] = useState(0);

  // !무한 루프 setCount에 의해 count가 변경되면, useEffect 실행
  useEffect(() => {
    setCount((prev) => prev + 1);
  }, [count]);
}
```

<br>

**2 ) eslient-disable-line, react-hooks/exhaustive-deps 경고를 무시하지 않기**

**💡 eslient-disable-line, react-hooks/exhaustive-deps 경고**

> **useEffect 인수 내부에서 사용하는 값 중 의존성 배열에 포함돼 있지 않은 값이 잇을 때 경고를 발생시킵니다.**

useEffect 사용시 필요한 의존성 배열을 생각해서 넣는 것이 중요합니다. 의존성 배열을 넣지 않는경우 의도치 못한 버그를 발생 시킬 수 있습니다.

**따라서 eslient-disable-line, react-hooks/exhaustive-deps 경고는 무시하면안됩니다.**

의존성 배열을 빈 배열로 설정하는 경우 컴포넌트를 마운트 할 때에만 어떤 동작을 실행시키는 의도로 작성됩니다. 이는 componentDidMount에 기반한 접근법으로 가급적 사용해선 안됩니다.

useEffect는 반드시 의존성 배열로 전달한 값의 변경에 의해 실행되어야 하는 훅입니다.

만약, 의존성 배열을 넘기지 않은 채 콜백 함수 내부에서 특정 값을 사용하는 것은 부수 효과가 실제로 관찰하는 값과는 별개로 작동하는 것을 의미하게됩니다.

아래 코드는 message를 props로 받아 useEffect를 통해 message 출력합니다.

만약, 정말로 useEffect를 첫 렌더링시 한 번만 실행하려고 의도한 대로라면 이런식으로 작성하기 보다는 상위 컴포넌트에서 실행하는 것이 바람직합니다.

```jsx
function Component({ message }) {
  // eslient-disable-line, react-hooks/exhaustive-deps 경고 발생
  useEffect(() => {
    console.log(message);
  }, []);

  //...
}
```

빈 배열이 아닐 때도 특정 값을 사용하지만 해당 값의 변경 시점을 피할 목적이라면 메모이제이션을 적절히 활용해 해당 값의 변화를 막거나 적당한 실행 위치를 다시 한번 생각해보는 것이 바람직합니다.

<br>

**3 ) useEffect 첫 번째 인수에 함수명 부여하기**

대부분의 경우 useEffect 첫 번째 인수 함수는 익명 함수로 나타냅니다.

useEffect의 복잡성이 높아지거나 어떤 코드인지 파악하기 어려운 경우 useEffect 첫 번째 인자 함수명을 부여하는 것이 바람직합니다. 이를 통해 해당 useEffect의 사용 목적에 대해 파악하기 쉬워집니다.

```jsx
const Component({ id }) {
  const [user, setUser] = useState(null);

  useEffect(function getUser() {
    const fetchUser = async () => {
      const res = await fetch(`https://jsonplaceholder.typicode.com/users/${id}`);
      const data = await res.json();
      setUser(user);
    };

    fetchUser();
  }, [id]);

  //...
}
```

<br>

**4 ) 거대한 useEffect 만들지 않기**

useEffect는 의존성 배열을 바탕으로 렌더링 시 의존성이 변경될 때 마다 부수 효과가 실행됩니다.

부수 효과의 크기가 커질수록 애플리케이션의 성능 저하에 원인이 되며, 의존성 배열이 거대해져 useEffect가 언제 발생하는지 정확히 파악하기 어려워집니다.

따라서, useEffect는 최대한 가볍게 만드는 것이 바랍직하며, useEffect가 거대해진 경우에는 적은 의존성 배열을 사용하는 여러 개의 useEffect로 분리하는 것이 좋습니다.

만약, 의존성 배열에 불가피하게 여러개의 변수가 들어가야 한다면 useCallback, useMemo 등으로 사전에 메모이제이션한 내용들만 useEffect에 넣는것이 좋습니다.

<br>

**5 ) 불필요한 외부 함수 만들지 않기**

useEffect가 실행하는 콜백이 외부에 불필요하게 존재하는 것은 가독성을 떨어뜨리고, 불필요한 코드가 많아집니다. 따라서, useEffect에는 불필요한 외부 함수를 만들지 않고, useEffect 내부로 가져오는것이 좋습니다.

```jsx
const Component({ id }) {
  const [user, setUser] = useState(null);

  const fetchUser = async () => {
      const res = await fetch(`https://jsonplaceholder.typicode.com/users/${id}`);
      const data = await res.json();
      setUser(data);
  };

  useEffect(function getUser() {
    fetchUser();
  }, [id, fetchUser]);

  //...
}
```

```jsx
const Component({ id }) {
  const [user, setUser] = useState(null);

  useEffect(function getUser() {
    (async () => {
      const res = await fetch(`https://jsonplaceholder.typicode.com/users/${id}`);
      const data = await res.json();
      setUser(data);
    })();
  }, [id]);

  //...
}
```

<br>

### useEffect 의존성 배열에서의 객체

JavaScript에서 객체는 참조값(메모리 주소)을 비교합니다. 따라서 객체의 속성이 모두 같더라도, 다른 메모리 주소에 저장되어 있으면 다른 객체로 판단합니다.

따라서 useEffect 의존성 배열에 객체를 넣는다면 객체의 속성이 변하지 않더라도 useEffect가 호출되게됩니다.

```jsx
import { useEffect, useState } from "react";

function App() {
  const profile = { name: "jon", age: 20 };
  const [count, setCount] = useState(0);
  const buttonHandler = () => {
    setCount((prev) => prev + 1);
  };

  useEffect(() => {
    console.log("렌더링 발생");
  }, [profile]);

  return (
    <div>
      <span>Count : {count}</span>
      <button onClick={buttonHandler}>Up Count</button>
    </div>
  );
}

export default App;
```

위 코드를 실행하면 profile이 변경되지 않았음에도 불구하고, useEffect가 호출되고 있습니다. 이를 해결하는 방법은 여러 가지가 있습니다.

**해결방법**

**1 ) Object.property 자체를 의존성 배열에 넣기**

객체가 아닌 객체 속성 자체를 의존성 배열에 넣습니다.

```jsx
useEffect(() => {
  console.log("렌더링 발생");
}, [profile.name]);
```

<br>

**2 ) JSON.stringify(Object)를 사용하기**

객체를 JSON.stringify()를 이용하여 문자열로 변환하여 사용합니다.

```jsx
const profile = { name: "jon", age: 20 };

const profileString = JSON.stringify(profile);

useEffect(() => {
  console.log("렌더링 발생");
}, [profileString]);
```

<br>

**3 ) lodash 라이브러리 isEqual 사용하기**

lodash 라이브러리의 isEqual는 객체의 모든 키-값 쌍을 재귀적으로 비교하는 깊은 비교를 하여 이를 해결할 수 있습니다.

```jsx
import { useEffect, useRef, useState } from "react";
import { isEqual } from "lodash";

function App() {
  const profileRef = useRef();
  const profile = { name: "jon", age: 20 };
  const [count, setCount] = useState(0);
  const buttonHandler = () => {
    setCount((prev) => prev + 1);
  };

  useEffect(() => {
    if (!isEqual(profileRef.current, profile)) {
      console.log("렌더링 발생");
      profileRef.current = profile;
    }
  }, [profile]);

  return (
    <div>
      <span>Count : {count}</span>
      <button onClick={buttonHandler}>Up Count</button>
    </div>
  );
}

export default App;
```
