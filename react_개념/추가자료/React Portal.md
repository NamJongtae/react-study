# React Portal

## React Portal ?

> React 컴포넌트 트리 안에서는 부모-자식 관계를 유지하면서 DOM 트리 상에서는 다른 위치에 렌더링할 수 있는 방법을 제공합니다.

일반적으로 React 컴포넌트는 부모 컴포넌트의 DOM 노드 안에 렌더링되지만, 가끔 이런 구조를 벗어나 DOM 트리의 다른 위치에 렌더링해야 할 때가 있습니다. 예를 들어, 모달, 툴팁, 팝업 등은 렌더링되는 위치에 따라 스타일이나 동작에 영향을 받을 수 있기 때문에, 이들을 DOM 트리의 루트와 같은 다른 위치에 렌더링하는 것이 이상적일 수 있습니다. 이런 경우에 React Portal을 사용할 수 있습니다.

**React에서 `ReactDOM.createPortal(child, container)` API를 제공하여 이를 가능하게 합니다.**

`child` : 포털을 통해 렌더링할 React 컴포넌트입니다. 이는 React 요소, 문자열, 숫자, 배열, fragment 등 React 렌더링이 가능한 모든 타입을 포함할 수 있습니다.

`container` : `child`가 렌더링될 DOM 요소를 나타냅니다. 보통 이는 `document.getElementById('id')` 등의 방법으로 선택한 DOM 노드입니다.

<br/>

### React Portal 사용하기

아래 코드는 React Portal를 사용하여 Modal창을 구현한 것입니다.

**index.html**

먼저 index.html에 모달창이 렌더링될 상위 부모 container 요소를 생성해줍니다.

아래코드에서는 `<div id="modal-root"></div>` 로 생성하였습니다.

```jsx
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="theme-color" content="#000000" />
    <meta
      name="description"
      content="Web site created using create-react-app"
    />
    <link rel="apple-touch-icon" href="%PUBLIC_URL%/logo192.png" />
    <link rel="manifest" href="%PUBLIC_URL%/manifest.json" />
    <title>React App</title>
  </head>
  <body>
    <div id="root"></div>
		{/* 모달이 들어갈 상위 부모 container 추가 */}
		<div id="modal-root"></div>
  </body>
</html>

```

<br/>

**Modal.js**

모달 컴포넌트에서는 React.creatPortal를 통해 Portal를 생성해줍니다.

생성시 두번째 인자에 아까 만든 상위 컨테이너 DOM 객체를 찾을 수 있도록 `document.getElementById("modal-root")` 을 넣어줍니다.

```jsx
import ReactDOM from "react-dom";

const dimStyles = {
  width: "100%",
  position: "fixed",
  backgroundColor: "rgba(0, 0, 0, 0.7)",
  inset: "0",
  zIndex: "999",
};

const wrapperStyle = {
  position: "absolute",
  top: "0",
  left: "0",
  width: "100%",
  height: "100%",
  zIndex: "999",
};

const childrenWrapperStyle = {
  position: "absolute",
  top: "50%",
  left: "50%",
  transform: "translate(-50%, -50%)",
  width: "500px",
  height: "500px",
  backgroundColor: "powderblue",
  zIndex: "999",
  fontSize: "30px",
};

export default function Modal({ children, onClose }) {
  return (
    <>
      {ReactDOM.createPortal(
        <div style={wrapperStyle}>
          <div onClick={onClose} style={dimStyles}></div>
          <div style={childrenWrapperStyle}>{children}</div>
        </div>,
        document.getElementById("modal-root")
      )}
    </>
  );
}
```

<br/>

**app.js**

```jsx
import React, { useState } from "react";
import Modal from "./Modal";

function App() {
  const [isOpenModal, setIsOpenModal] = useState(false);

  const openModal = () => {
    setIsOpenModal(true);
  };

  const closeModal = () => {
    setIsOpenModal(false);
  };

  return (
    <>
      <div>Hello from Portal!</div>
      <button onClick={openModal}>click here</button>
      {isOpenModal && (
        <Modal onClose={closeModal}>
          <div>Portal Modal</div>
        </Modal>
      )}
    </>
  );
}

export default App;
```

개발자 도구 요소를 들어가보면 아래와 같이 Modal이 성공적으로 modal-root 요소 하위에 생성되는 것을 볼 수 있습니다.

![portal](https://github.com/NamJongtae/react-study/assets/113427991/299f64d8-271d-4fbd-8229-13d4ac10c3a5)
