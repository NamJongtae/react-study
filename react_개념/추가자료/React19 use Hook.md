## use Hook?

> `use()` 훅은 비동기 데이터 처리와 컨텍스트 사용을 크게 단순화하는 강력한 기능을 제공합니다. 이 훅은 Promise와 같은 리소스를 읽어올 수 있으며, `Suspense` 및 오류 경계(error boundaries)와 원활하게 통합됩니다.

<br/>

### use Hook의 주요 기능

- **비동기 데이터 처리**: Promise와 함께 사용하면, 해당 컴포넌트는 Promise가 해결될 때까지 렌더링을 중단하고, 데이터를 기다리는 동안 `Suspense`를 통해 로딩 상태를 자동으로 처리할 수 있습니다.

- **에러 처리** : `use()` 훅과 함께 `ErrorBoundary`를 사용해 비동기 작업에서 발생하는 에러를 안전하게 처리할 수 있습니다. 네트워크 요청에서 오류가 발생했을 때 Promise를 명시적으로 reject하거나, `throw`를 통해 에러를 발생시키면 `ErrorBoundary`가 이를 감지해 오류 화면을 출력할 수 있습니다. 이때 오류는 `ErrorBoundary`의 `fallback` 속성에 정의된 컴포넌트로 대체됩니다.

- **유연한 컨텍스트 접근**: `useContext()`와 유사하지만, `use()`는 반복문이나 조건문 안에서도 호출할 수 있습니다. 이를 통해 컨텍스트 값을 더욱 유연하게 처리할 수 있어 코드 구조가 간결해집니다.

- **조건부 렌더링**: 기존의 훅들과 달리 `use()`는 컴포넌트 상단에 있을 필요 없이, 조건문이나 반복문 내에서 자유롭게 호출할 수 있어 더욱 유동적인 컴포넌트 작성이 가능합니다.

<br/>

### use Hook 사용하기

**기존 useEffect를 사용하여 비동기 데이터 처리**

```jsx
import { useEffect, useState } from "react";

const DataComponent = ({ data }) => {
  return (
    <>
      <p>ID : {data.id}</p>
      <p>UserID : {data.userId}</p>
      <p>Title : {data.title}</p>
      <p>completed : {data.completed ? "true" : "false"}</p>
    </>
  );
};

function App() {
  const [data, setData] = useState({});
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    setLoading(true);
    fetch("https://jsonplaceholder.typicode.com/todos/1")
      .then((response) => response.json()) // response.json()은 Promise이므로 여기에 처리
      .then((data) => setData(data)) // 데이터를 받아 setData로 저장
      .catch((error) => console.log(error))
      .finally(() => setLoading(false)); // 요청이 끝나면 loading 상태 false
  }, []);

  if (loading) return <div>loading...</div>;

  return <DataComponent data={data} />;
}

export default App;
```

<br/>

**use Hook를 사용하여 비동기 데이터 처리**

```jsx
import { use, Suspense } from "react";

const fetchData = fetch("https://jsonplaceholder.typicode.com/todos/1")
  .then((response) => {
    return response.json();
  })
  .catch((error) => console.log(error));

const DataComponent = () => {
  const data = use(fetchData);

  return (
    <>
      <p>ID : {data.id}</p>
      <p>UserID : {data.userId}</p>
      <p>Title : {data.title}</p>
      <p>completed : {data.completed ? "true" : "false"}</p>
    </>
  );
};

function App() {
  return (
    <Suspense fallback={<p>Loading...</p>}>
      <DataComponent />
    </Suspense>
  );
}

export default App;

```

<br/>

### Error Bounary로 에러 처리하기

**use Hook**은 **Erro rBoundary**로 에러를 처리할 수 있습니다.

**Error Boundary**는 에러가 발생한 부분 대신 오류 메시지와 같은 fallback UI를 표시할 수 있는 특수 컴포넌트입니다. 

**Error Boundary**를 직접 생성할 수 있고, 라이브러리를 통해 간단하게 사용할 수도 있습니다.

<br/>

**ErrorBoundary 직접 생성하기**

```jsx
import React from "react";

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // 다음 렌더링에서 폴백 UI가 보이도록 상태를 업데이트 합니다.
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // 에러 리포팅 서비스에 에러를 기록할 수도 있습니다.
    // logErrorToMyService(error, errorInfo);
    console.log(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // 폴백 UI를 커스텀하여 렌더링할 수 있습니다.
      return <h1>Something went wrong.</h1>;
    }

    // eslint-disable-next-line react/prop-types
    return this.props.children;
  }
}

export default ErrorBoundary;
```

<br/>

**react-error-boundary 설치하기**

```bash
npm install react-error-boundary;
```

<br/>

**Error Boundary 설정하기**

```jsx
import { use, Suspense } from "react";
import { ErrorBoundary } from "react-error-boundary";

// 🔴 fetch url를 잘못입력하여 에러 발생
const fetchData = fetch("https://jsonplaceholder.typicode.com/todoss/1") 
  .then((response) => {
    if (!response.ok) {
      throw new Error("Network response was not ok");
    }
    return response.json();
  })
  .catch((error) => console.log(error));

const DataComponent = () => {
  const data = use(fetchData);

  return (
    <>
      <p>ID : {data.id}</p>
      <p>UserID : {data.userId}</p>
      <p>Title : {data.title}</p>
      <p>completed : {data.completed ? "true" : "false"}</p>
    </>
  );
};

function App() {
  return (
    <ErrorBoundary fallback={<p>Somthing went wrong!</p>}>
      <Suspense fallback={<p>Loading...</p>}>
        <DataComponent />
      </Suspense>
    </ErrorBoundary>
  );
}

export default App;
```

<br/>

### use Hook 조건부 사용

아래 코드처럼 use Hook를 통해 조건부 렌더링할 수 있습니다.

```jsx
import { use, Suspense, useEffect, useState } from "react";

const fetchData = fetch("https://jsonplaceholder.typicode.com/todos/1")
  .then((response) => {
    return response.json();
  })
  .catch((error) => console.log(error));

const DataComponent = () => {
  let data;
  const [flag, setFlag] = useState(false);

  useEffect(() => {
    setTimeout(() => {
      setFlag(true);
    }, 2000);
  }, []);

  if (flag) {
    data = use(fetchData);
  }
  
  return (
    <>
      {data ? (
        <>
          <p>ID : {data.id}</p>
          <p>UserID : {data.userId}</p>
          <p>Title : {data.title}</p>
          <p>completed : {data.completed ? "true" : "false"}</p>
        </>
      ) : (
        <>{"data none"}</>
      )}
    </>
  );
};

function App() {
  return (
    <Suspense fallback={<p>Loading...</p>}>
      <DataComponent />
    </Suspense>
  );
}

export default App;
```

<br/>

### use Hook를 사용하여 Context 참조하기

context가 `use`에 전달되면 `useContext`와 유사하게 작동합니다. `useContext`는 컴포넌트의 최상위 수준에서 호출해야 하지만 `use`는 `if`와 같은 조건문이나 `for`과 같은 반복문 내부에서 호출할 수 있습니다. `use`는 유연하므로 `useContext`보다 선호됩니다.

<br/>

**style.css**

```css
.dark {
  background-color: black;
  color: white;
}

.light {
  background-color: white;
  color: black;
}
```

<br/>

**App.jsx**

```jsx
import { createContext, useState, use } from "react";
import "./style.css";

// ThemeContext 생성
const ThemeContext = createContext("light");

function ThemeTest({ toggleTheme }) {
  const [flag, setFlag] = useState(false);
  let theme = "";

  const toggleFlag = () => {
    setFlag((prev) => !prev);
  };
  
  // use()를 통해 Context를 조건문 처리
  if (flag) {
    theme = use(ThemeContext);
  }

  return (
    <>
	    {/* 테마 변경 버튼 활성화 버튼 */}
      <button onClick={toggleFlag}>
        {flag ? "Hide Theme Button" : "show Theme Button"}
      </button>
      <p className={theme}>Hello</p>
      {/* 버튼을 클릭하면 테마 전환 */}
      {theme && (
        <button onClick={toggleTheme}>
          {theme === "light" ? "🌕 Dark Mode" : "🌞 Light Mode"}
        </button>
      )}
    </>
  );
}

function App() {
  const [theme, setTheme] = useState("light");

  // 테마 전환 함수
  const toggleTheme = () => {
    setTheme((prevTheme) => (prevTheme === "light" ? "dark" : "light"));
  };

  return (
    <ThemeContext.Provider value={theme}>
      <ThemeTest toggleTheme={toggleTheme} />
    </ThemeContext.Provider>
  );
}

export default App;
```

<br/>

### useEffect vs use

- use는 Suspense로 감싸주지 않으면 렌더링 되지 않습니다.
- use는 if문안에 사용할 수 있습니다.
- useEffect는 먼저 렌더링 후 실행되지만 use는 데이터를 받아올때 까지 렌더링을 지연시킵니다.

<br/>

자세한 내용은 https://ko.react.dev/reference/react/use 공식문서 를 참고해주세요.