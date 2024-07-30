# React & Closure

## React에서 Closure

> React 함수형 컴포넌트에서 Closure 사용하여 만들어진 hooks가 만들어졌으며, 이는 hooks가 상태와 라이프사이클을 관리하는 데 중요한 역할을 합니다. Closure를 이용하여 state를 기억하고 이를 통해 라이프사이클을 관리하게됩니다.

<br>

### Closure(클로저)란 ?

함수와 그 함수가 선언될 당시의 렉시컬 환경과의 조합입니다. 즉, 함수가 생성될 당시의 스코프에 있는 모든 변수를 기억하고, 이후에도 계속 접근할 수 있는 특성을 말합니다.

아래 예시 코드에서 countFn 함수가 종료된 후에도 getCount, increment, decrement의 함수에서는 closure를 통해 count 값에 접근할 수 있습니다.

```javascript
const countFn = (function () {
  let count = 0;

  function getCount() {
    return count;
  };

  function increment() {
    count += 1;
  };

  function decrement() {
    if(count > 0) {
    count -= 1;
    }
  }

  return { getCount, increment, decrement };
})();

const { getCount, increment, decrement } = countFn
console.log(getCount()); // 0
increment();
console.log(getCount()); // 1
decrement();
console.log(getCount()); // 0
```

<br>

### useState에서의 Closure 활용

아래 코드에서는 useState 함수의 호출은 분명 Component 내부 첫 번째 줄에서 종료되었는데, setCounter는 useState 내부의 최신 counter 값을 계속 가져올 수 있습니다.

이런 일이 가능한 이유는 바로 Closure 때문입니다.

Closure에 의해 useState가 선언된 시점에 환경(counter가 저장된 곳)이 기억되어 counter에 접근이 가능한 것입니다.

```jsx
function Component() {
  const [counter, setCounter] = useState(0);

  setCounter(1);

  console.log(counter); // 1
}
```

<br>

### useState hook 직접 구현하기

useState hook를 Closure를 활용하여 비슷하게 단계별로 직접 구현해봅니다.

**1 ) 기본 코드 구현**

아래 코드는 useState을 Closure를 활용하여 구현한 코드입니다.

하지만 현재 코드에서는 setState는 의도한 대로 동작하지 않습니다.

그 원인은 구조 분해 할당으로 state의 값를 이미 할당한 상태이기 때문에 훅 내부의 setState를 호출하더라도 state는 변경된 새로운 값을 반환하지 못하기 때문입니다.

```jsx
function useState(initalValue) {
  let state = initalValue;

  function setState(newValue) {
    state = newValue;
  }

  return [state, setState];
}

function render(Component) {
  const C = Component();
  C.render(); // 컴포넌트 렌더링

  return C;
}

function Component() {
  const [count, setCount] = useState(0);

  return {
    render: () => console.log(count),
    handle: () => {
      setCount(count + 1);
    },
  };
}

let App = render(Component); // 0
App.handle();
App = render(Component); // 0
```

<br>

**2 ) state 함수화**

이를 해결하기 위해 state를 함수를 통해 가져오도록 수정했습니다.

하지만 아래 코드는 실제 useState와는 다르게 구현되었습니다. 실제 useState와 동일하게 구현하기 위해서는 state가 함수가 아닌 변수로 선언되어야하며, React 모듈에서 useState를 가져와야합니다.

```jsx
function useState(initalValue) {
  let currentState = initalValue;

  function state() {
    return currentState;
  }

  function setState(newValue) {
    currentState = newValue;
  }

  return [state, setState];
}

function render(Component) {
  const C = Component();
  C.render(); // 컴포넌트 렌더링

  return C;
}

function Component() {
  const [count, setCount] = useState(0);

  return {
    render: () => console.log(count()),
    handle: () => {
      setCount(count + 1);
    },
  };
}

let App = render(Component); // 0
App.handle();
App = render(Component); // 1
```

<br>

**3 ) React 고차 함수 생성 및 state useState 외부 선언**

이를 해결하기 위해 React 고차 함수를 생성하고, state의 변수를 useState 외부에서 선언하도록 수정합니다.

아래 코드에서 코드가 잘 구현 되었지만 여러개의 useState 사용시 state를 덮여쓰는 문제가 발생합니다.

```jsx
const React = (function () {
  let state;

  function useState(initialValue) {
    if (state === undefined) {
      state = initialValue;
    }

    const setState = function (newValue) {
      state = newValue;
    };

    return [state, setState];
  }
  function render(Component) {
    const C = Component();
    C.render(); // 컴포넌트 렌더링

    return C;
  }

  return { useState, render };
})();

function Component() {
  const [count, setCount] = React.useState(0);
  const [text, setText] = React.useState("");

  return {
    render: () => console.log("count:", count, "text:", text),
    handle: () => {
      setCount(count + 1);
      setText("abc");
    },
  };
}

let App = React.render(Component); // count: 0, text: 0
App.handle();
App = React.render(Component); // count: "abc", text: "abc"
```

<br>

**4 ) state 구분을 위한 index 추가**

이를 해결하기 위해 고유한 index를 추가하고, 이를 통해 각 state를 관리하도록 수정합니다.

이제는 여러개의 useState가 있더라도 제대로 동작하게됩니다.

```jsx
const React = (function () {
  let global = {};
  let index = 0;

  function useState(initialState) {
    if (!global.states) {
      global.states = [];
    }
    const currentState = global.states[index] || initialState;
    const currentIndex = index;

    const setState = function (newValue) {
      global.states[currentIndex] = newValue;
    };

    index = index + 1;

    return [currentState, setState];
  }

  function render(Component) {
    index = 0; // 상태 인덱스를 초기화하여 다시 렌더링 시 올바른 상태를 가져올 수 있도록 함
    const C = Component();
    C.render(); // 컴포넌트 렌더링

    return C;
  }

  return { useState, render };
})();

function Component() {
  const [count, setCount] = React.useState(0);
  const [text, setText] = React.useState("");

  return {
    render: () => console.log("count:", count, "text:", text),
    handle: () => {
      setCount(count + 1);
      setText("abc");
    },
  };
}

let App = React.render(Component); // count: 0, text: ""
App.handle();
App = React.render(Component); // count: 1, text: "abc"
```
