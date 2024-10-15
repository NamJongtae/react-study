## Actions ?

> **Actions**은 비동기 작업을 처리하는 함수이며, 서버에 데이터를 보내거나 데이터베이스를 업데이트하는 등의 비동기 작업을 처리할 때 사용됩니다. React 19에서 도입된 Actions은 이러한 작업을 더 효과적으로 관리할 수 있도록 도와줍니다.

<br/>

### 기존 React의 데이터 처리

기존 react에서는 데이터를 변형한 후 다음 응답을 통해 상태를 업데이트 하는 방법을 사용하였습니다. 이를 위해서 pending state(대기 상태), errors(에러), optimistic updates(낙관적 업데이트), sequential requests(연속 요청) 등을 수동으로 처리해야 했습니다.

```jsx
import { useState } from "react";

const Form = ({ addPost }) => {
  const [isPending, setIsPending] = useState(false);
  const [error, setError] = useState(null);

  const onSumbitForm = async (event) => {
    event.preventDefault(); // 폼 제출 기본 동작 방지
    setIsPending(true);
    setError(null); // 에러 초기화

    const formData = new FormData(event.target); // 폼 데이터 가져오기
    const title = formData.get("title"); // formData에서 제목 가져오기
    const content = formData.get("content"); // formData에서 내용 가져오기
    const newPost = { title, content }; // 새로운 게시물 생성

    try {
      await new Promise((resolve) => {
        setTimeout(resolve, 2000); // 2초 대기 (비동기 작업 시뮬레이션)
      });
      addPost(newPost); // 새 게시물 추가
    } catch (error) {
      setError(error);
    } finally {
      setIsPending(false);
    }
  };

  return (
    <>
      <form onSubmit={onSumbitForm}>
        <input name="title" type="text" placeholder="title" />
        <input name="content" type="text" placeholder="content" />
        <button type="submit" disabled={isPending}>
          {isPending ? "submitting..." : "submit"}
        </button>
        {error && <p>{error}</p>}
      </form>
    </>
  );
};

function App() {
  const [posts, setPosts] = useState([]); // 게시물 목록 상태
  const addPost = (newPost) => {
    // 새 게시물을 추가하는 함수
    setPosts((posts) => [...posts, newPost]);
  };

  return (
    <div>
      <Form addPost={addPost} />
      {posts.map((post, index) => (
        <div key={index}>
          <h2>{post.title}</h2>
          <p>{post.content}</p>
        </div>
      ))}
    </div>
  );
}

export default App;
```

<br/>

### React 19 버전의 데이터 처리

React 19 부터는 트렌지션에서 비동기 함수를 사용하여 pending state(대기 상태), errors(에러), optimistic updates(낙관적 업데이트), sequential requests(연속 요청)를 자동으로 처리할 수 있습니다. 비동기 트랜지션 함수(Actions)은 isPending 상태를 즉시 true로 설정하고, 비동기 요청를 처리하고 isPending 을 false로 전환합니다. 이렇게 하면 데이터가 변경되는 동안 현재 UI를 반응형 및 대화형으로 유지할 수 있습니다.

**비동기 트랜지션을 사용하는 함수들은 "Actions(액션)"이라고 불립니다.**

React 19에서는 아래 네 가지 처리를 위한 네 가지 비동기 트렌지션 함수(Actions)를 사용하는 훅을 제공합니다.

- **peding state** : 액션은 요청이 시작될 때 대기 상태를 제공하며, 최종 상태 업데이트가 완료되면 자동으로 대기 상태가 초기화됩니다.
- **optimistic state** : api 요청이 성공할 것이라고 낙관적으로 보고, 응답을 기다리지 않고 미리 결과값을 노출하기 위한 상태입니다.
- **error handing** : 액션은 에러 처리를 제공하여 요청이 실패할 경우 에러 경계(Error Boundaries)를 표시할 수 있으며, 낙관적 업데이트를 원래 값으로 자동으로 되돌립니다.
- **Forms** : `<form>` 요소는 이제 `action` 및 `formAction` 속성에 함수를 전달하는 것을 지원합니다. `action` 속성에 함수를 전달하면 기본적으로 액션을 사용하며, 제출 후 폼이 자동으로 초기화됩니다.

<br/>

### useTransition

`useTransition`은 상태 업데이트 pending state를 처리하는 훅입니다.

상태 업데이트가 시간이 걸리는 작업일 때, 다른 상태 업데이트가 우선 처리되도록 하여 UI가 끊기지 않도록 합니다. React 19에서는 `useTransition`이 비동기 함수와 함께 사용할 수 있도록 개선되었습니다. `useTransition`은 상태 업데이트를 '트랜지션' 상태로 처리하여, 사용자에게 매끄러운 UI를 제공합니다.
 
<br/>

**useTransition 예시 코드**
```jsx
import { useState, useTransition } from "react";

const Form = ({ addPost }) => {
  const [isPending, startTransition] = useTransition();
  const FormAction = async (formData) => {
    // 폼 제출 시 실행되는 함수
    const title = formData.get("title"); // formData에서 제목 가져오기
    const content = formData.get("content"); // formData에서 내용 가져오기
    const newPost = { title, content }; // 새로운 게시물 생성

    startTransition(async () => {
      await new Promise((resolve) => {
        setTimeout(resolve, 2000); // 2초 대기 (비동기 작업 시뮬레이션)
      });
      addPost(newPost); // 새 게시물 추가
    });
  };

  return (
    <>
      <form action={FormAction}>
        <input name="title" type="text" placeholder="title" />
        <input name="content" type="text" placeholder="content" />
        <button type="submit" disabled={isPending}>
          {isPending ? "submitting..." : "submit"}
        </button>
      </form>
    </>
  );
};

function App() {
  const [posts, setPosts] = useState([]); // 게시물 목록 상태
  const addPost = (newPost) => {
    // 새 게시물을 추가하는 함수
    setPosts((posts) => [...posts, newPost]);
  };

  return (
    <div>
      <Form addPost={addPost} />
      {posts.map((post, index) => (
        <div key={index}>
          <h2>{post.title}</h2>
          <p>{post.content}</p>
        </div>
      ))}
    </div>
  );
}

export default App;
```

<br/>

### useFormStatus

`useFormStatus` 는 React에서 폼의 상태를 추적하는 데 사용되는 훅입니다.

상위 form를  포함하고 있어야 사용이 가능합니다. 즉, form 내부에 렌더링되는 컴포넌트에서 호출되어야 합니다. 상위 form에 대한 상태 정보만 반환하며, 동일한 컴포넌트나 하위 컴포넌트에서 렌더링된 form에 대한 상태정보를 반환하지 않습니다.

```jsx
const [peding, data, method, action] = useFormStatus();
```

<br/>

**useFormStatus 반환값**

- **pending** : 상위 form의 제출 상태를 나타냅니다.
- **data** : 상위 form이 제출하는 데이터가 포함횐 FormData 인터페이스 객체입니다.
- **method** : 상위 form의 메서드를 나타냅니다.
- **action** : 상위 form의 action prop에 전달된 함수에 대한 참조입니다.

<br/>

**useFormStatus 예시 코드**

useFormStatus를 사용하여  pending 상태와 전송되는 FormData를 표시하는 예시 코드입니다.

```jsx
import { useState } from "react";
import { useFormStatus } from "react-dom";

const Button = () => {
  const { pending, data } = useFormStatus();  // form의 대기 상태와 데이터를 가져옴

  return (
    <button type="submit" disabled={pending}>
      {pending ? "submitting..." : "submit"}
    </button>
    {data && (
      <p>
        {`Requesting title: ${data?.get("title")}, content: ${data?.get("content")}`}
      </p>
    )}
  );
};

const Form = ({ addPost }) => {
  const FormAction = async (formData) => {  // 폼 제출 시 실행되는 함수
    const title = formData.get("title");  // formData에서 제목 가져오기
    const content = formData.get("content");  // formData에서 내용 가져오기
    const newPost = { title, content };  // 새로운 게시물 생성

    await new Promise((resolve) => {
      setTimeout(resolve, 2000);  // 2초 대기 (비동기 작업 시뮬레이션)
    });
    addPost(newPost);  // 새 게시물 추가
  };

  return (
    <>
      <form action={FormAction}>
        <input name="title" type="text" placeholder="title" />
        <input name="content" type="text" placeholder="content" />
        <Button />
      </form>
    </>
  );
};

function App() {
  const [posts, setPosts] = useState([]);  // 게시물 목록 상태
  const addPost = (newPost) => {  // 새 게시물을 추가하는 함수
    setPosts((posts) => [...posts, newPost]);
  };

  return (
    <div>
      <Form addPost={addPost} /> 
      {posts.map((post, index) => (
        <div key={index}>
          <h2>{post.title}</h2>
          <p>{post.content}</p>
        </div>
      ))}
    </div>
  );
}

export default App;
```

<br/>

### useActionState

`useActionState` 는 **Action**를 좀 더 편하게 사용하기 위해 도입된 훅입니다.

`useActionState`는 form action 결과에 기반하여 상태를 업데이트할 수 있는 훅입니다.

```jsx
const [state, formAction, isPending] = useActionState(fn, initialState, permalink?);
```

<br/>

**useActionState 매개변수**

- **fn** : form을 제출하거나 버튼을 눌렀을 때 호출되는 함수입니다.
- **initalState** : 초기 상태값입니다.
- **permalink** : 수정하는 고유한 페이지 URL을 포함하는 문자열입니다. 자바스크립트가 완전히 로드되기 전에 폼이 제출될 경우에도 올바르게 동작하도록 하기 위한 URL로, 자바스크립트가 로드된 후에는 이 매개변수가 영향을 미치지 않습니다.

<br/>

**useActionState 반환값**

- **state** : 현재 상태를 나타내며, 초기에는 initialState로 설정한 값을 갖게되며, action 함수가 호출된 이후에는 함수가 반환한 값으로 변경됩니다.
- **formAction** : form action props을 전달되거나 form 내부의 버튼에 formAction prop을 전달 수 있는 새로운 action입니다. formAction 함수에 인자 값으로는 이전 폼 상태 값과 formData 값이 전달됩니다.
- **isPending** : 현재 폼 제출 상태를 나타냅니다.

<br/>

**useActionState 예시 코드**

아래 코드는 useActionState를 사용하여 peding state와 현재의 post 데이터를 나타내는 예시 코드입니다.

```jsx
import { useActionState } from "react";

function App() {
  const FormAction = async (prevPost, formData) => {  // 폼 제출 시 실행되는 비동기 함수
    const title = formData.get("title");  // formData에서 제목 가져오기
    const content = formData.get("content");  // formData에서 내용 가져오기
    const newPost = { title, content };  // 새로운 게시물 객체 생성

    await new Promise((resolve) => {
      setTimeout(resolve, 2000);  // 2초 대기 (비동기 작업 시뮬레이션)
    });
    return newPost;  // 새로운 게시물을 반환
  };

  const [post, action, isPending] = useActionState(FormAction, {  // useActionState 훅 사용
    title: "Default Title",  // 초기 상태
    content: "Default...",  // 초기 상태
  });

  return (
    <>
      <form action={action}>
        <input name="title" type="text" placeholder="title" />
        <input name="content" type="text" placeholder="content" />
        <button type="submit" disabled={isPending}>
          {isPending ? "updatting..." : "update"}
        </button>
        {post && (
          <>
            <h2>{post.title}</h2>
            <p>{post.content}</p>
          </>
        )}
      </form>
    </>
  );
}

export default App;
```

<br/>

### useOptimistic

`useOptimistic` 는 React 19에서 새롭게 도입된 훅으로 사용자가 애플리케이션 내에서 비동기 작업을 수행할 때 최적의 사용자 경험을 제공하기 위해 활용됩니다. 서버 요청이 처리되는 동안 예상되는 결과를 미리 반영하여, 사용자 경험을 향상시킵니다. 이를 통해 느린 네트워크 환경에서 인터페이스가 즉각적으로 반응하는 것처럼 구현할 수 있습니다.

```jsx
 const [optimisticState, addOptimistic] = useOptimistic(state, updateFn);
```

<br/>

**useOptimistic 매개변수**

- **state** : 처음에 반환되는 값과 대기 중인 작업이 없을 때마다 반환되는 값입니다.
- **updateFn(currentState, optimisticValue)** : 현재 상태와 전달된 낙관적 값을 가져와 `addOptimistic`결과 낙관적 상태를 반환하는 함수입니다. 순수 함수여야 하며, `updateFn`두 개의 매개변수(`currentState`, `optimisticValue`)를 받습니다. 반환값은 `currentState` 와  `optimisticValue`의 병합된 값이 됩니다.

<br/>

**useOptimistic 반환값**

- **optimisticState** : 낙관적 업데이트된 상태를 나타냅니다. action이 pending 상태가 아닌 경우와 동일하며, updateFn에서 반환된 값과 동일합니다.
- **addOptimistic** : addOptimistic은 낙관적 업데이트가 있을 때 호출한 dispatching 함수입니다.

<br/>

**useOptimistic 예시 코드**

아래 코드는 `useOptimistic`를 사용하여 낙관적 업데이트를 구현한 예시 코드입니다.

```jsx
import { useOptimistic, useState } from "react";

function App() {
  const [post, setPost] = useState({  // 현재 게시물 상태
    title: "Default Title",
    content: "Default...",
  });
  const [optimisticPost, addOptimisticPost] = useOptimistic(post);  // 낙관적 업데이트 상태
  const [error, setError] = useState(null);  // 에러 상태
  const pending = post !== optimisticPost;  // 현재 상태와 낙관적 상태 비교

  const FormAction = async (formData) => {  // 폼 제출 시 실행되는 함수
    setError(null);  // 에러 초기화
    const title = formData.get("title");  // formData에서 제목 가져오기
    const content = formData.get("content");  // formData에서 내용 가져오기
    const newPost = { title, content };  // 새로운 게시물 생성
    addOptimisticPost(newPost);  // 낙관적 상태로 새 게시물 추가

    try {
      await new Promise((resolve, reject) => {  // 서버 응답 시뮬레이션
        setTimeout(() => {
          const r = Math.random();  // 무작위로 성공 또는 실패 시뮬레이션
          if (r < 0.5) {
            setPost(newPost);  // 성공 시 실제 게시물 업데이트
            resolve(newPost);
          } else {
            reject("Error: Failed to update post!");  // 실패 시 에러 발생
          }
        }, 3000);
      });
    } catch (error) {
      setError(error);  // 에러 상태 저장
    }
  };

  return (
    <>
      <form action={FormAction}>
        <input name="title" type="text" placeholder="title" />
        <input name="content" type="text" placeholder="content" />
        <button type="submit" disabled={pending}>
          {pending ? "updatting..." : "update"}
        </button>
        {optimisticPost && (
          <>
            <h2>{optimisticPost.title}</h2>
            <p>{optimisticPost.content}</p>
          </>
        )}
        {error && <p>{error}</p>}
      </form>
    </>
  );
}

export default App;
```