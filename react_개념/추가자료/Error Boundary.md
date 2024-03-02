# Error Boundary

## Error Boundary ?

> UI 부분의 오류를 catch하고, fallback UI를 보여주는데 사용되는 React 컴포넌트입니다.

일반적으로 JavaScript에서 오류가 발생하면 전체 애플리케이션이 중단되거나, 사용자에게 오류 메시지를 보여주는 데 그치지만, Error Boundary를 사용하면 오류를 적절히 처리하고 사용자에게 더 나은 사용자 경험을 제공할 수 있습니다.

Error Boundary는 그 아래에 있는 컴포넌트 트리에서 발생하는 오류를 catch하며, 오류 메시지를 로깅하거나, 대체 UI를 렌더링하는 등의 역할을 수행합니다.

<br/>

### Error Boundary 사용하기

Error Boundary는 클래스형 컴포넌트로 구현해야합니다.

리액트에서는 함수형 컴포넌트로 Error Boundary를 제공하지 않습니다.

Error Boundary를 만들려면 클래스형 컴포넌트에서 `componentDidCatch`나 `getDerivedStateFromError` 라이프사이클 메서드를 정의하면 됩니다.

**기존 코드**

```jsx
import { useEffect } from "react";

export default function App() {
  useEffect(() => {
    throw new Error("Oops, something went wrong!");
  }, []);

  return <div>Hello World</div>;
}
```

**코드 실행시 에러가 UI를 가리고 코드 실행이 중단됩니다.**

![error_boundary_before](https://github.com/NamJongtae/react-study/assets/113427991/7abff4d4-f742-4929-b5cf-7ab2239069e2)

<br/>

**ErrorBoundary.js**

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // 다음 렌더링에서 대체 UI를 보여주기 위해 상태를 업데이트합니다.
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // 에러 리포팅 서비스에 에러를 보고할 수도 있습니다.
    logErrorToMyService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // 대체 UI를 렌더링합니다.
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children;
  }
}
```

<br/>

**index.js**

`<ErrorBoundary>` 컴포넌트로 `<App>` 컴포넌트를 감싸줍니다.

```jsx
import React from "react";
import ReactDOM from "react-dom/client";
import "./index.css";
import App from "./App";
import { BrowserRouter } from "react-router-dom";
import ErrorBoundary from "./ErrorBoundary";

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(
  <React.StrictMode>
    <BrowserRouter>
      <ErrorBoundary>
        <App />
      </ErrorBoundary>
    </BrowserRouter>
  </React.StrictMode>
);
```

**Error Boundary 적용 후 에러가 발생 하여도 실행이 중단되지 않으며, UI에는 에러 메세지가 출력됩니다.**

![error_boundary_after](https://github.com/NamJongtae/react-study/assets/113427991/b5bf8cc5-2035-48a7-85e8-ef45bcbc6375)
