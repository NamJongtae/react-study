# React Design Pattern

## Design Pattern ?

> 애플리케이션을 설계하고 구현하는 데 있어서 일반적으로 효과적인 방법을 정의한 것을 말합니다. 이러한 패턴들은 재사용 가능한 컴포넌트 설계, 상태 관리, 라이프사이클 메서드의 적절한 활용 등 여러 가지 측면에서 도움을 줍니다.

<br/>

## React 주요 Design Pattern

### 1. **Component Composition**

컴포넌트 합성 패턴으로 React의 기본적인 디자인 패턴 중 하나입니다. 작은 컴포넌트들을 조합하여 더 큰 컴포넌트를 만드는 방식이며, 재사용성과 유지 보수성을 높입니다.

```jsx
// 작은 컴포넌트들
function Header() {
  return (
    <header>
      <h1>Welcome to my website</h1>
    </header>
  );
}

function Content() {
  return <p>This is my awesome website content.</p>;
}

function Footer() {
  return <footer>© 2024 My Website</footer>;
}

// 큰 컴포넌트를 작은 컴포넌트들로 구성
function Page() {
  return (
    <div>
      <Header />
      <Content />
      <Footer />
    </div>
  );
}
```

**장점**

- **코드의 재사용성을 높입니다.**
- **각 컴포넌트가 독립적으로 동작하기 때문에 유지 보수성이 좋습니다.**
- **테스트하기 쉽습니다.**

**단점**

- **너무 많은 컴포넌트를 만들면 관리하기 어려울 수 있습니다.**

<br/>

### 2. Container Presenter

Container Presenter 패턴은 컴포넌트를 두 가지 유형으로 나누는 패턴입니다. Container 컴포넌트는 데이터 흐름과 스테이트 관리를 담당하며, Presenter 컴포넌트는 데이터를 화면에 표시하는 역할을 합니다.

```jsx
// Container Component
import React, { useState, useEffect } from 'react';

function UserContainer() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    const fetchUser = async () => {
      const result = await fetchUser();
      setUser(result);
    }
    fetchUser();
  }, []);

  return user ? <UserPresenter user={user} /> : null;
}

// Presenter Component
function UserPresenter({ user }) {
  return <div>{user.name}</div>;
}

export default UserContainer;
```

**장점**

- **관심사의 분리(separation of concerns)를 통해 코드의 가독성과 유지 보수성을 높입니다.**
- **Container와 Presenter의 분리를 통해 재사용성이 좋아집니다.**

**단점**

- **컴포넌트의 수가 많아져 관리하기 어려울 수 있습니다.**

<br/>

### 3. Custom Hook

React의 Hook 기능을 이용하여 로직을 재사용하는 패턴입니다.

**useCounter.js**

```jsx
function useCounter() {
  const [count, setCount] = useState(0);

  const increment = () => {
    setCount(count + 1);
  };

  return { count, increment };
}

export default useCounter;
```

<br/>

**Counter.js**

```jsx
import useCounter from "./useCounter";

function Counter() {
  const { count, increment } = useCounter();

  return (
    <div>
      Count: {count}
      <button onClick={increment}>Increment</button>
    </div>
  );
}
```

**장점**

- **로직의 재사용성을 높입니다.**
- **컴포넌트 내부의 상태 관리 로직을 분리하여 가독성을 높입니다.**

**단점**

- **Hook의 사용 규칙을 준수해야 합니다.**
- **컴포넌트 사이에서 상태를 공유하려면 추가적인 작업이 필요합니다.**

<br/>

### 4. Atomic Pattern

Atomic Design은 인터페이스를 재사용 가능한 컴포넌트들로 분리하는 디자인 패턴입니다. 이 패턴은 Atoms, Molecules, Organisms, Templates, Pages의 5단계로 구성됩니다.

```jsx
// Atom
function Button({ onClick, children }) {
  return <button onClick={onClick}>{children}</button>;
}

// Molecule
function Form({ onSubmit }) {
  return (
    <form onSubmit={onSubmit}>
      <Button type="submit">Submit</Button>
    </form>
  );
}

// Organism
function Header({ onSubmit }) {
  return (
    <header>
      <Form onSubmit={onSubmit} />
    </header>
  );
}
```

**장점**

- **컴포넌트의 재사용성을 높입니다.**
- **인터페이스의 일관성을 유지하는 데 도움이 됩니다.**

**단점**

- **설계 초기 단계에서 많은 시간과 노력이 필요합니다.**
- **각 단계를 정확히 정의하는 것이 어려울 수 있습니다.**
