# React SSR 구현

## SSR이란?

> SSR(Server-Side Rendering)은 서버에서 HTML 페이지를 렌더링한 후 클라이언트에 전달하는 방식입니다. 이로 인해 초기 로딩 속도가 빨라지고, 검색 엔진 최적화(SEO)에 유리하며, 서버 측에서 데이터를 처리하기 때문에 보안성이 높습니다. 그러나 페이지 이동 시 클라이언트에서 전체 페이지를 다시 로드해야 하므로 느릴 수 있으며, 서버에서 모든 렌더링을 처리하기 때문에 서버 부하가 증가할 수 있습니다.

<br>

### **SSR 장점**

**1 ) 초기 로드 속도 향상**

- **빠른 초기 렌더링** : 서버에서 미리 렌더링된 HTML을 클라이언트로 전송하므로 초기 페이지 로드 속도가 빠릅니다. 사용자는 즉시 콘텐츠를 볼 수 있습니다.
- **SEO(검색 엔진 최적화) :** 검색 엔진 크롤러가 서버에서 렌더링된 완전한 HTML 페이지를 쉽게 인덱싱할 수 있습니다. 이는 SEO 성능을 크게 향상시킵니다.

**2 ) 사용자 경험 개선**

- **첫 번째 콘텐츠 표시(FCP)** : 사용자가 페이지를 요청하면 즉시 콘텐츠가 표시되므로 사용자 경험이 개선됩니다.
- **빠른 TTFB(Time to First Byte)** : 서버에서 렌더링된 콘텐츠는 빠르게 첫 번째 바이트를 전송할 수 있습니다.

**3 ) 보안 강화**

- **클라이언트 코드 노출 감소** : 서버에서 렌더링되므로 클라이언트 측에 노출되는 코드의 양이 줄어들어 보안이 강화될 수 있습니다

**4 ) 초기 데이터 패칭**

- **초기 상태 전송** : 서버에서 초기 상태를 설정하여 클라이언트로 전송할 수 있습니다. 이는 클라이언트 측에서 불필요한 데이터 페칭을 줄여줍니다.

<br>

### **SSR 단점**

**1 ) 서버 부하 증가**

- **서버 리소스 소모** : 모든 요청마다 서버에서 렌더링을 수행하므로 서버 리소스가 많이 소모됩니다. 이는 서버 부하를 증가시키고, 고성능 서버가 필요할 수 있습니다.
- **스케일링 문제** : 트래픽이 많은 경우 서버가 렌더링 작업을 감당하기 어려울 수 있습니다. 이는 스케일링 문제를 야기할 수 있습니다.

**2 ) 복잡한 설정 및 유지보수**

- **구현 복잡성** : SSR을 구현하려면 서버와 클라이언트 모두에서 렌더링 로직을 작성해야 하므로 코드가 복잡해질 수 있습니다.
- **데이터 페칭 복잡성** : 서버와 클라이언트에서 데이터를 동기화하는 작업이 필요하여 데이터 페칭 로직이 복잡해질 수 있습니다.

**3 ) 네트워크 지연**

- **네트워크 지연** : 서버에서 렌더링된 HTML을 클라이언트로 전송하는 과정에서 네트워크 지연이 발생할 수 있습니다. 이는 사용자 경험에 부정적인 영향을 미칠 수 있습니다.

**4 ) 제한된 브라우저 기능**

- **브라우저 전용 기능 제한** : 일부 브라우저 전용 기능(예: `window` 객체나 `document` 객체를 사용하는 기능)을 서버에서 사용할 수 없으므로 이러한 기능을 관리하는 데 어려움이 있을 수 있습니다.

<br>

### React로 SSR 간단히 구현하기

현재 SSR를 기능을 제공해주는 프레임워크 Next.js, Remix 등으로 편리하고, 쉽게 SSR 기능을 사용할 수 있습니다. 이런 프레임워크 없이 React만으로 SSR를 구현해보고, 어떻게 SSR 처리가 흘러가는지 알아보겠습니다.

현재 프론트엔드에서 사용하는 SSR은 자세히 본다면 **Universal Rendering**이라고 불러야합니다.

**SSR과 CSR을 혼합한 방식**이기 때문입니다.

![SSR](https://github.com/user-attachments/assets/da520ab4-4aee-4ffe-a1e9-b4daffd36818)

**Universal Rendering** 에서는 최초 렌더링(SSR)에 필요한 `node/` 파일과 이후 CSR에 필요한 `web/` 파일 2가지 결과물이 필요합니다. 즉, 기존 코드에서 `node/` 파일을 새로 만들어줘야 한다는 뜻입니다.

<br>

**SSR 구현을 위해 필요사항**

- `node/` : SSR에 필요한 마크업을 만드는 코드가 들어있습니다.
- `web/` : 최초 렌더링 이후 hydreate와 CSR에 필요한 파일이 들어있습니다.
- `Rendering Server`: `node/` 를 이용해 HTML을 만들고 렌더링하여 클라이언트에 보내주게 됩니다.

**SSR 구현을 위해 생각해야하는 부분**

**1 ) Webpack 설정**

SSR에서는 `node/`, `web/` 2종류의 트랜스파일링을 해야 합니다.

**2 ) 렌더링용 Express Server 구축**

SSR 처리를 위한 렌더링용 Express Server 구축이 필요합니다.

<br>

**필요한 패키지 설치**

아래 명령어로 필요한 패키지를 설치합니다.

```bash
npm install react react-dom express
npm install -D typescript
npm install -D @types/express @types/node @types/react @types/react-dom
npm install -D webpack webpack-cli ts-loader html-webpack-plugin
```

<br>

**webpack 설정하기**

서버 및 클라이언트 config를 구분해서 생성합니다.

**webpack.client.js**

```jsx
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");

/** @type {import('webpack').Configuration} */
module.exports = {
  mode: "development",
  entry: path.resolve(__dirname, "src/client/main.tsx"),
  resolve: {
    extensions: [".ts", ".js", ".tsx", ".jsx"],
  },
  output: {
    path: path.resolve(__dirname, "dist/client"),
  },
  module: {
    rules: [
      {
        test: /\.(ts|tsx|js|jsx)$/,
        use: "ts-loader",
        exclude: /node_modules/,
      },
    ],
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: "src/client/index.html",
    }),
  ],
};
```

<br>

**webpack.server.js**

```jsx
const path = require("path");

/** @type {import('webpack').Configuration} */
module.exports = {
  target: "node",
  mode: "development",
  entry: path.resolve(__dirname, "src/server/index.tsx"),
  resolve: {
    extensions: [".ts", ".js", ".tsx", ".jsx"],
  },
  output: {
    filename: "[name].js",
    path: path.resolve(__dirname, "dist/server"),
  },
  module: {
    rules: [
      {
        test: /\.(ts|tsx|js|jsx)$/,
        use: "ts-loader",
        exclude: /node_modules/,
      },
    ],
  },
};
```

<br>

**webpack.config.js**

```jsx
const path = require("path");
const clientConfig = require("./webpack.client.js");
const serverConfig = require("./webpack.server.js");

/** @type {import('webpack').Configuration[]} */
module.exports = [
  {
    ...clientConfig,
  },
  {
    ...serverConfig,
  },
];
```

<br>

**client index.tsx 생성**

```jsx
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>React SSR</title>
</head>
<body>
  <div id="root"></div>
</body>
</html>

```

<br>

**client App.tsx 생성**

```jsx
import { Suspense, useState } from "react";

type Todo = {
  content: string;
  id: string;
  isDone: boolean;
};

const App = () => {
  const [todos, setTodos] = useState<Todo[]>([]);

  const toggleDone = (id: string) => {
    const newTodos = [...todos].map((todo) => {
      if (todo.id === id) {
        todo.isDone = !todo.isDone;
        return todo;
      }
      return todo;
    });
    setTodos(newTodos);
  };

  const removeTodo = (id: string) => {
    const newTodos = [...todos].filter((todo) => todo.id !== id);
    setTodos(newTodos);
  };

  const onSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();

    const formData = new FormData(e.currentTarget);
    const value = formData.get("todo-input");

    if (value) {
      const newTodo = {
        content: value as string,
        id: new Date().getTime().toString(),
        isDone: false,
      };
      setTodos((prev) => [...prev, newTodo]);
      e.currentTarget.reset();
    }
  };

  return (
    <>
      <h1>React SSR TodoList</h1>
      <h2>Add to Task.</h2>
      <form onSubmit={onSubmit}>
        <input name="todo-input" />
        <button type="submit">Add</button>
      </form>
      <ul
        style={{
          margin: "10px 0",
          padding: 0,
          display: "flex",
          flexDirection: "column",
          gap: "10px",
        }}
      >
        {todos.map((todo) => (
          <li
            style={{
              margin: 0,
              padding: 0,
              listStyle: "none",
              display: "flex",
              gap: "5px",
              alignItems: "center",
            }}
            key={todo.id}
          >
            <input
              type="checkbox"
              defaultChecked={todo.isDone}
              onClick={() => toggleDone(todo.id)}
            />
            <p
              style={{
                margin: 0,
                padding: 0,
                textDecoration: todo.isDone ? "line-through" : undefined,
                color: todo.isDone ? "rgba(155,155,155)" : undefined,
              }}
            >
              {todo.content}
            </p>
            <button onClick={() => removeTodo(todo.id)}>remove</button>
          </li>
        ))}
      </ul>
    </>
  );
};

export default App;

```

<br>

**client inext.tsx 생성**

```jsx
import ReactDOM from 'react-dom/client';
import App from './App';

ReactDOM.hydrateRoot(document.getElementById('root') as HTMLElement, <App />);
```

`hydrateRoot`는 SSR로 생성된 HTML를 브라우저 DOM node 내부에 리액트 컴포넌트들을 보여질 수 있도록 합니다.

즉, 서버에서 렌더링된 HTML을 읽고 이를 React 컴포넌트와 연결합니다. 

이 과정(hydreate)에서 React는 기존의 HTML 요소에 이벤트 리스너를 부착하고, 상태를 관리할 수 있게 만듭니다.

<br>

**Rendering Server 구축**

```jsx
import express from "express";
import fs from "fs";
import path from "path";
import ReactDOMServer from "react-dom/server";
import App from "../client/App";

const app = express();

// client build index.html 가져오기
const html = fs.readFileSync(
  path.resolve(__dirname, "../client/index.html"),
  "utf-8"
);

app.get("/", (req, res) => {
  // <App /> 컴포넌트 렌더링
  const renderString = ReactDOMServer.renderToString(<App />);

  // root <div>에 내부에 <App /> 컴포넌트 삽입
  res.send(
    html.replace(
      '<div id="root"></div>',
      `<div id="root">${renderString}</div>`
    )
  );
});

// dist/client 폴더에 있는 파일들 제공
app.use("/", express.static("dist/client"));

app.listen(3000, () => {
  console.log("Server is listening on port 3000");
});
```

<br>

**package.json scripts 명령어 추가**

package.json 파일 scripts에 아래 명령어를 추가해줍니다.

```jsx
  "scripts": {
    "build": "webpack --config webpack.config.js",
    "start": "node dist/server/main.js"
 }
```

<br>

**build 및 실행하기**

```bash
npm run bild
npm run start
```

실행 후 개발자 도구를 통해 요소를 살펴보면 SSR이 이루어져 root div안이 채워져 있는 것을 볼 수 있습니다. 
실제 SSR은 이것보다 훨씬 복잡하게 이루어져 있지만 위 내용을 통해 SSR처리가 어떻게 흘러가는지 알 수 있었습니다.

![React_SSR](https://github.com/user-attachments/assets/fb261d38-194e-423e-ab4c-c6d684518f1b)

<br>

### 정리

**SSR(server-side-rendering)** 은 서버에서 HTML 파일을 생성하고 클라이언트로 전달하는 방식입니다. 서버는 클라이언트의 요청에 따라 필요한 데이터를 가져와 HTML 파일을 파싱하여 렌더링합니다. 이후 이 HTML 파일이 클라이언트로 전송되며, 클라이언트 측에서는 전달받은 HTML에 JavaScript를 통해 동적인 기능을 추가하는 `hydrate` 과정을 거칩니다.

### 참고 사이트

https://minoo.medium.com/next-js-처럼-server-side-rendering-구현하기-7608e82a0ab1

https://solo5star.tistory.com/44
