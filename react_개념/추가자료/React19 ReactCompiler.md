## React Compiler ?

![](https://velog.velcdn.com/images/njt6419/post/2c915db3-44be-420d-a68a-1b89f994c535/image.png)

**React Compiler**는 build time에 소스 코드를 훨씬 **최적화(useMemo(), useCallback(), React.Memo 등을 Compiler가 알아서 처리)** 하여 처리를 도와줍니다.

React 19에서는 지금까지 개발자가 스스로 해왔던 것들을 리액트 자체에서 컨트롤 해줘서 더 안정감이 있고 소스코드도 깔끔하게 만들 수 있게 하는 기술들이 도입 되었으며, 그 이유는 **React Compiler** 덕분입니다.

<br/>

### React Compiler 사용하기

**Compiler 사용 가능 여부 확인**

컴파일 가능한 컴포넌트를 확인할 수 있습니다.

```bash
npx react-compiler-healthcheck@experimental
```

<br/>

**react-compiler eslint 설치하기**

```bash
npm install eslint-plugin-react-compiler@experimental
```

<br/>

**eslint config에 추가하기**

```jsx
module.exports = {
    plugins: [
    'eslint-plugin-react-compiler',
    ],
    rules: {
    'react-Compiler/react-compiler': 'error'
    }
}
```

<br/>

**babel-plugin-react-compiler 설치하기**

```bash
npm install babel-plugin-react-compiler@experimental
```

<br/>

**babel config에 추가하기**

```jsx
module.exports = function () {
  return {
    plugins: [
      ['babel-plugin-react-compiler'], // must run first!
      // ...
    ],
  };
};
```

<br/>

**App.jsx에 적용하기**

```jsx
import { useState } from "react";

function App() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState("");

  console.log("App is Redering~");

  const Increment = () => {
    setCount((prev) => prev + 1);
  };

  const onChangeText = (e) => {
    setText(e.target.value);
  };

  return (
    <>
      <p>{count}</p>
      <button onClick={Increment}>Increment</button>
      <p>{text}</p>
      <input value={text} onChange={onChangeText} />
      <Text text={text} />
    </>
  );
}

const Text = ({ text }) => {
  console.log("Text is Rendering~");
  return <p>{text}</p>;
};

export default App;
```

위 코드를 실행시키면 예상되는 상황은 하위 Text 컴포넌트를 최적화 `React.Memo()`로 감싸지 않았기 때문에 상위 컴포넌트가 리렌더링 즉, Count가 변경될 때 마다 리렌더링이 발생할 것으로 예상됩니다. 하지만 실행 시 Count를 증가 시켜도 Text 컴포넌트는 리렌더링이 발생하지 않습니다.

이것은 **react-compiler**가 자동으로 Text 컴포넌트를 최적화 시켜주어 리렌더링이 발생하지 않는것입니다. 

<br/>

### 💡 compilationMode: "annotation”

`compilationMode: "annotation"` 옵션을 사용하여 컴파일러를 "opt-in" 모드로 실행하도록 구성할 수도 있습니다.

**eslint config 수정**

`compilationMode: "annotation"` 옵션을 추가해줍니다.

```jsx
module.exports = function () {
  return {
    plugins: [
      ['babel-plugin-react-compiler', { compilationMode: "annotation" }], // must run first!
      // ...
    ],
  };
};
```

<br/>

**App.js**

컴포넌트에 적용할 최적화 사항을 최상단에 “use …“를 적어줍니다.

```jsx
import { useState } from "react";

function App() {
  "use memo";
  
  const [count, setCount] = useState(0);
  const [text, setText] = useState("");

  console.log("App is Redering~");

  const Increment = () => {
    setCount((prev) => prev + 1);
  };

  const onChangeText = (e) => {
    setText(e.target.value);
  };

  return (
    <>
      <p>{count}</p>
      <button onClick={Increment}>Increment</button>
      <p>{text}</p>
      <input value={text} onChange={onChangeText} />
      <Text text={text} />
    </>
  );
}

const Text = ({ text }) => {
  "use memo";

  console.log("Text is Rendering~");
  return <p>{text}</p>;
};

export default App;

```

<br/>

### 주의 사항

**react-compiler**를 사용하기 위해서는 https://ko.react.dev/reference/rules를 반드시 따라야합니다.

<br/>

**App.js**

아래 코드와 같이 **react-rule**를 지키지 않으면 react-compiler가 적용되지 않아 Text 컴포넌트가 최적화되지 않습니다.

```jsx
import { useState } from "react";

function App() {
  "use memo";
  
  const [count, setCount] = useState(0);
  const [text, setText] = useState("");

  console.log("App is Redering~");

  const Increment = () => {
    count = count + 1; // 🔴 react rule를 지키지 않음
  };

  const onChangeText = (e) => {
    setText(e.target.value);
  };

  return (
    <>
      <p>{count}</p>
      <button onClick={Increment}>Increment</button>
      <p>{text}</p>
      <input value={text} onChange={onChangeText} />
      <Text text={text} />
    </>
  );
}

const Text = ({ text }) => {
  "use memo";

  console.log("Text is Rendering~");
  return <p>{text}</p>;
};

export default App;

```

`react-compiler-healthcheck` 를 사용하여 컴파일 가능한 컴포넌트를 확인할 수 있습니다.

```jsx
npx react-compiler-healthcheck@experimental
```

<br/>

자세한 내용은 https://react.dev/learn/react-compiler 공식문서를 참고해주세요.