## Server Components ?

> **Server Components**는 번들링 이전에 클라이언트 앱, SSR 서버와 별도의 환경에서 미리 렌더링 되는 새로운 컴포넌트입니다.

별도의 환경은 React Server Components의 **Server**를 의미합니다. Server Components는 CI 서버에서 빌드 시 한 번 실행 될 수도 있고, 웹 서버를 사용하여 각 요청에 대해 매 번 실행 될 수도 있습니다.

<br/>

### Server 없이 Server Components 사용

**server components 적용 전**

```jsx
// bundle.js
import marked from 'marked'; // 35.9K (11.2K gzipped)
import sanitizeHtml from 'sanitize-html'; // 206K (63.3K gzipped)

function Page({page}) {
  const [content, setContent] = useState('');
  // NOTE: loads *after* first page render.
  useEffect(() => {
    fetch(`/api/content/${page}`).then((data) => {
      setContent(data.content);
    });
  }, [page]);
  
  return <div>{sanitizeHtml(marked(content))}</div>;
}
```

```jsx
// api.js
app.get(`/api/content/:page`, async (req, res) => {
  const page = req.params.page;
  const content = await file.readFile(`${page}.md`);
  res.send({content});
});
```

이 패턴은 사용자가 추가로 75K(gzip)의 라이브러리를 다운로드하고 구문 분석하고 페이지가 로드된 후 데이터를 가져오는 두 번째 요청을 기다려야 함을 의미합니다. 이는 페이지의 수명 동안 변경되지 않는 정적 콘텐츠를 렌더링하기 위해서입니다.

<br/>

**Server Components 적용**

```jsx
// bundle.js

import marked from 'marked'; // Not included in bundle
import sanitizeHtml from 'sanitize-html'; // Not included in bundle

async function Page({page}) {
  // NOTE: loads *during* render, when the app is built.
  const content = await file.readFile(`${page}.md`);
  
  return <div>{sanitizeHtml(marked(content))}</div>;
}
```

렌더링된 출력은 HTML로 서버 측 렌더링(SSR)되어 CDN에 업로드될 수 있습니다. 앱이 로드되면 클라이언트는 원래 `Page`구성 요소나 마크다운을 렌더링하는 데 드는 비용이 많이 드는 라이브러리를 볼 수 없습니다. 클라이언트는 렌더링된 출력만 볼 수 있습니다.

즉, 첫 번째 페이지가 로드되는 동안 콘텐츠가 표시되고, 번들에는 정적 콘텐츠를 렌더링하는 데 필요한 값비싼 라이브러리가 포함되지 않습니다.

<br/>

### Server에서 Server Components 사용

**Server Components**는 페이지 요청 중에 웹 서버에서 실행될 수 있어 API를 구축하지 않고도 데이터 레이어에 액세스할 수 있습니다. 이들은 애플리케이션이 번들되기 전에 렌더링되며, 데이터와 JSX를 **Client Components**에 props로 전달할 수 있습니다.

 **Server Component**가 아니라면 `useEffect`에서 클라이언트에 동적 데이터를 가져오는 것이 일반적입니다.

```jsx
// bundle.js
function Note({id}) {
  const [note, setNote] = useState('');
  // NOTE: loads *after* first render.
  useEffect(() => {
    fetch(`/api/notes/${id}`).then(data => {
      setNote(data.note);
    });
  }, [id]);
  
  return (
    <div>
      <Author id={note.authorId} />
      <p>{note}</p>
    </div>
  );
}

function Author({id}) {
  const [author, setAuthor] = useState('');
  // NOTE: loads *after* Note renders.
  // Causing an expensive client-server waterfall.
  useEffect(() => {
    fetch(`/api/authors/${id}`).then(data => {
      setAuthor(data.author);
    });
  }, [id]);

  return <span>By: {author.name}</span>;
}
```

```jsx
// api
import db from './database';

app.get(`/api/notes/:id`, async (req, res) => {
  const note = await db.notes.get(id);
  res.send({note});
});

app.get(`/api/authors/:id`, async (req, res) => {
  const author = await db.authors.get(id);
  res.send({author});
});
```

<br/>

 **Server Component**를 사용하면 컴포넌트에서 바로 데이터를 읽고 구성 요소에서 렌더링할 수 있습니다.

```jsx
import db from './database';

async function Note({id}) {
  // NOTE: loads *during* render.
  const note = await db.notes.get(id);
  return (
    <div>
      <Author id={note.authorId} />
      <p>{note}</p>
    </div>
  );
}

async function Author({id}) {
  // NOTE: loads *after* Note,
  // but is fast if data is co-located.
  const author = await db.authors.get(id);
  return <span>By: {author.name}</span>;
}
```

번들러는 데이터를 렌더링된 **Server Components**와 동적인 **Client Components**를 하나의 번들로 결합합니다. 선택적으로, 이 번들은 **서버 사이드 렌더링(SSR)**되어 페이지의 초기 HTML을 생성할 수 있습니다. 페이지가 로드될 때, 브라우저는 원래의 Note와 Author 컴포넌트를 보지 않고, 렌더링된 결과만 클라이언트로 전송됩니다.

**Server Components**는 서버에서 다시 데이터를 가져와 동적으로 만들 수 있으며, 서버에서 데이터를 액세스하여 다시 렌더링할 수 있습니다. 이러한 새로운 애플리케이션 아키텍처는 서버 중심의 다중 페이지 애플리케이션(MPA)의 간단한 "요청/응답" 정신 모델과 클라이언트 중심의 단일 페이지 애플리케이션(SPA)의 원활한 상호작용을 결합하여 양쪽의 장점을 제공합니다.

<br/>

### Server Components의 상호작용

**Server Components**는 브라우저로 전송되지 않기 때문에, `useState`와 같은 상호작용 API를 사용할 수 없습니다. Server Components에 상호작용을 추가하려면, `"use client"` 지시어를 사용하여 Client Component와 함께 조합해야 합니다.

다음 예제에서는 `Notes` Server Component가 상태를 사용하여 확장 상태를 전환하는 `Expandable` Client Component를 가져옵니다.

```jsx
// Server Component
import Expandable from './Expandable';

async function Notes() {
  const notes = await db.notes.getAll();
  return (
    <div>
      {notes.map(note => (
        <Expandable key={note.id}>
          <p note={note} />
        </Expandable>
      ))}
    </div>
  );
}
```

```jsx
// Client Component
"use client";

export default function Expandable({children}) {
  const [expanded, setExpanded] = useState(false);
  return (
    <div>
      <button
        onClick={() => setExpanded(!expanded)}
      >
        Toggle
      </button>
      {expanded && children}
    </div>
  );
}
```

<br/>

이 코드에서는 먼저 `Notes`를 Server Component로 렌더링한 후, 번들러에게 `Expandable` Client Component의 번들을 생성하라고 지시합니다. 브라우저에서는 Client Components가 Server Components에서 전달된 결과를 props로 받게 됩니다.

```jsx
<head>
  <!-- Client Components를 위한 번들 -->
  <script src="bundle.js" />
</head>
<body>
  <div>
    <Expandable key={1}>
      <p>this is the first note</p>
    </Expandable>
    <Expandable key={2}>
      <p>this is the second note</p>
    </Expandable>
    <!-- ... -->
  </div> 
</body>
```

<br/>

### Server Components와 비동기 컴포넌트

Server Components는 `async/await`을 사용하는 새로운 방식의 컴포넌트 작성을 도입합니다. 비동기 컴포넌트에서 `await`를 사용할 경우, React는 해당 Promise가 완료될 때까지 렌더링을 중단하고 기다립니다. 이 방식은 서버/클라이언트 경계를 넘어 작동하며, **Suspense**의 스트리밍 지원과 함께 사용할 수 있습니다.

서버에서 Promise를 생성하고, 클라이언트에서 이를 `await`할 수도 있습니다.

```jsx
// Server Component
import db from './database';

async function Page({id}) {
  // Server Component에서 중단됨.
  const note = await db.notes.get(id);
  
  // 주의: 이 부분은 기다리지 않고 시작되며 클라이언트에서 await됩니다.
  const commentsPromise = db.comments.get(note.id);
  return (
    <div>
      {note}
      <Suspense fallback={<p>Loading Comments...</p>}>
        <Comments commentsPromise={commentsPromise} />
      </Suspense>
    </div>
  );
}
```

```jsx
// Client Component
"use client";
import { use } from 'react';

function Comments({ commentsPromise }) {
  // 주의: 서버에서 시작된 Promise가 여기서 이어집니다.
  // 데이터가 준비될 때까지 중단됩니다.
  const comments = use(commentsPromise);
  return comments.map(comment => <p>{comment}</p>);
}
```

이 예시에서 `note`는 페이지 렌더링에 중요한 데이터이므로 서버에서 `await`합니다. `comments`는 우선순위가 낮은 데이터로, 서버에서 Promise를 시작하고 클라이언트에서 `use` API로 기다립니다. 클라이언트에서는 **Suspense**를 사용해 중단되지만, `note`는 렌더링이 차단되지 않습니다.

클라이언트에서는 비동기 컴포넌트가 지원되지 않기 때문에, `use`로 Promise를 기다립니다.

<br/>

### Server Components를 이용해 데이터 가져오기

Server Cimponents를 사용하여 실제 데이터를 가져오는 예시 코드를 작성하겠습니다.

```jsx
// components/Posts.jsx

export default async function Posts() {
  const res = await fetch("https://jsonplaceholder.typicode.com/posts");
  const posts = await res.json();

  return (
    <div>
      {posts.map((post, index) => (
        <div key={post.id}>
          <h2>{`${index + 1}. ${post.title}`}</h2>
          <p>{post.body}</p>
        </div>
      ))}
    </div>
  );
}
```

- `Posts` 컴포넌트는 서버 컴포넌트이기 때문에 클라이언트에서 직접적으로 API 호출을 하지 않습니다. 서버에서 데이터를 받아와 렌더링된 HTML만 클라이언트로 전달합니다.

<br/>

```jsx
// App.jsx

import { Suspense } from "react";
import Postsfrom "./components/Posts";

function App() {
  return (
    <Suspense fallback="loading...">
      <Posts />
    </Suspense>
  );
}

export default App;
```

- 컴포넌트를 사용하여 `Posts`가 로드될 때까지 대기하는 동안 `"loading..."` 메시지를 표시합니다. 서버에서 데이터가 로드되면 해당 부분이 완료된 컴포넌트로 교체됩니다.
- `Posts`는 비동기 함수이므로 `Suspense`로 감싸서 렌더링이 완료될 때까지 대기할 수 있습니다.

<br/>

Server Components와 Client Components에 대한 자세한 설명은 https://www.notion.so/2-Server-Component-72110f0ccdb54a35bc4405b6a8ca0e4b?pvs=21 를 참고해주세요.

<br />

## 참고 사이트
[React 19 - Server Components 공식문서](https://react.dev/reference/rsc/server-components)