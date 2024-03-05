# React Suspense

## 1. Suspense ?

> 데이터를 불러오는 동안 UI를 잠시 '대기' 상태로 만들어, 비동기 로딩 상황을 좀 더 우아하게 처리할 수 있도록 도와주는 기능입니다.

기본적으로, 컴포넌트에서 필요한 데이터를 불러오는 동안 해당 컴포넌트 렌더링을 '보류'하고, 대신 fallback UI를 보여줍니다. 데이터가 준비되면 실제 컴포넌트를 렌더링하게 됩니다.

**Data-fetching과 React.lazy의 로딩 화면에 사용할 수 있습니다.**

<br/>

## 2. Suspense 사용하기

Suspense는 아래와 같은 구조로 사용할 수 있습니다.

Suspense는 적용할 컴포넌트를 감싼 후 로딩 중에 보여줄 UI를 fallback props로 전달합니다.

```jsx
import { Suspense } from 'react';

function MyComponent() {
  return (
    <Suspense fallback={<p>Loading...</p>}>
      <OtherComponent />
    </Suspense>
  );
```

<br/>

### 1 ) Data-fetching

기본적으로 데이터 요청 중인지 완료되었는지를 Suspense가 알 수 없어 설정을 해주어야 합니다. Suspense를 이용하여 비동기 처리 로딩 UI를 적용하고 싶다면 아래와 같이 코드를 작성해야합니다.

`read()` 함수는 데이터 수신 중에는 `suspender` 변수에 저장되어 있는 API를 호출하는 코드를 반환하고, 데이터 수신이 완료되면 데이터를 반환합니다.

```jsx
function fetchPosts() {
  let posts = null;
  const suspender = fetch(`https://jsonplaceholder.typicode.com/posts?_page=1`)
    .then((response) => response.json())
    .then((data) => {
      setTimeout(() => {
        posts = data;
      }, 3000);
    });
  return {
    read() {
      if (posts === null) {
        throw suspender;
      } else {
        return posts;
      }
    },
  };
}
```

<br/>

**postList.js**

fetchPosts() 함수를 컴포넌트 외부로 빼준이유는 posts 값이 변경되면, 컴포넌트의 리렌더링에 의해 fetchPosts() 함수가 재성성되어 비동기 요청이 무한 반복될 수 있기 때문입니다.

```jsx
function fetchPosts() {
  let posts = null;
  const suspender = fetch(`https://jsonplaceholder.typicode.com/posts?_page=1`)
    .then((response) => response.json())
    .then((data) => {
      setTimeout(() => {
        posts = data;
      }, 3000);
    });
  return {
    read() {
      if (posts === null) {
        throw suspender;
      } else {
        return posts;
      }
    },
  };
}

const resource = fetchPosts();

export default function PostList() {
  const posts = resource.read();
  return (
    <ul>
      {posts.map((post) => {
        return <li key={post.id}>{post.title}</li>;
      })}
    </ul>
  );
}
```

<br/>

### 2 ) React.lazy

> React의 기능 중 하나로, 코드 분할을 쉽게 할 수 있도록 도와주는 유틸리티입니다.

`React.lazy`는 동적 `import()` 구문을 이용해 컴포넌트를 비동기적으로 로드합니다. 이를 통해 애플리케이션의 초기 로딩 시간을 줄일 수 있습니다.

`React.lazy`를 사용하면, 애플리케이션이 필요로 하는 코드만 처음에 로드하고, 나머지 코드는 필요할 때 로드할 수 있습니다. 이렇게 하면 애플리케이션의 초기 로딩 시간을 줄이고, 성능을 향상시킬 수 있습니다.

```jsx
const OtherComponent = React.lazy(() => import("./OtherComponent"));

function MyComponent() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <OtherComponent />
    </Suspense>
  );
}
```

<br/>

## 3 ) React Router와 함께 사용하기

React 공식 문서에는 Router 바로 아래에 Suspense를 위치 시키고, Route로 보여줄 컴포넌트를 React.lazy로 불러오는 것을 권장하고 있습니다.

**app.js**

```jsx
import { Route, Routes } from "react-router-dom";
import { Suspense } from "react";
import Main from "./Main";

export default function App() {
  return (
    <Suspense fallback={<p>loading...</p>}>
      <Routes>
        <Route path="/" element={<Main />} />
      </Routes>
    </Suspense>
  );
}
```

<br/>

**Main.js**

```jsx
import React from "react";
import People from "./People";
const PostList = React.lazy(() => import("./PostList "));

function Main() {
  return (
    <div>
      <header>
        <h1>Main Page</h1>
      </header>
      <PostList />
    </div>
  );
}
```
