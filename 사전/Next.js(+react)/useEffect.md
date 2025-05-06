# 개념
https://react.dev/reference/react/useEffect

## 구조
`useEffect(setup, dependencies?);`
### 셋업
*컴포넌트가 DOM에 추가될 때* 셋업 함수를 실행한다.
	"마운팅"이라고 불린다. 
	1. 리액트가 컴포넌트(함수형 컴포넌트)를 실행하면서 가상 DOM을 만들고 이걸 실제 DOM으로 변화.
	2. 브라우저가 이 DOM을 화면에 그린다.
	3. 그 후에 셋업함수 실행.
그리고 *컴포넌트가 리렌더링 될 때마다* 반복해서 실행한다. (클린업이 있다면 클린업 실행 -> 셋업 실행)
	정확히는 useEffect가 실행되면서 컴포넌트 전체가 리렌더링 됨.
	state가 변경되면서 리렌더링 될 때에는 해당되지 않는다.
([[컴포넌트 생애주기]] 참고)
#### 클린업
`return ()=>{...}` 이라고 정의한다.
의존배열(dependencies)가 바뀌어서 useEffect가 다시 실행될 때마다 먼저 클린업함수가 실행되고 다시 나머지 셋업함수가 실행된다.
아니면 컴포넌트가 언마운트될 때 실행된다.
### 의존 배열
"The list of all *reactive values* referenced inside of the `setup` code."
*reactive values* : props, state, and all variable and functions declared directly inside your component body.
> [!1] props라서 부모로부터 전달받은게 아니면서 컴포넌트 밖에서 정의된 변수나 함수?
> - 외부 라이브러리
> - 모듈 레벨 상수와 함수
> >```jsx
> >// 컴포넌트 외부에 정의된 상수 
> >const API_URL = "https://api.example.com"; 
> >
> >// 컴포넌트 외부에 정의된 함수 
> >function formatDate(date) { 
> >	return new Date(date).toLocaleDateString(); 
> >} 
> >
> >function MyComponent() { 
> >	// 컴포넌트 내에서 외부 상수와 함수 사용 
> >	useEffect(() => {
> > 		fetch(API_URL)... 
> > 	}, []); 
> >	return <div>{formatDate(new Date())}</div>; 
> >}
> >```
> - 리액트 외부의 전역 변수

참고 : [[useEffect has a missing dependency]]. useEffect안에서 사용되고 있고 && reactive value라면 의존성 배열에 포함되어야 한다.

## 용례
### 외부 시스템과 연결할 때
리액트 내부 요소들이 아닌 리액트 외의 요소들이라는 의미에서 외부 시스템들과 상호작용하는 컴포넌트가 있을 수 있다. 가령 외부 api라던가 네트워크처럼.

# 예시
## external system
### 1. chatroom
```jsx
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]);

  return (
    <>
      <label>
        Server URL:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <h1>Welcome to the {roomId} room!</h1>
    </>
  );
}
```

```jsx
export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        Choose the chat room:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? 'Close chat' : 'Open chat'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

### 2. 에니메이션
```jsx
import { useState, useEffect, useRef } from 'react';
import { FadeInAnimation } from './animation.js';

function Welcome() {
  const ref = useRef(null);

  useEffect(() => {
    const animation = new FadeInAnimation(ref.current);
    animation.start(1000);
    return () => {
      animation.stop();
    };
  }, []);

  return (
    <h1
      ref={ref}
      style={{
        opacity: 0,
        color: 'white',
        padding: 50,
        textAlign: 'center',
        fontSize: 50,
        backgroundImage: 'radial-gradient(circle, rgba(63,94,251,1) 0%, rgba(252,70,107,1) 100%)'
      }}
    >
      Welcome
    </h1>
  );
}
```

```js
export class FadeInAnimation {
  constructor(node) {
    this.node = node;
  }
  start(duration) {
    this.duration = duration;
    if (this.duration === 0) {
      // Jump to end immediately
      this.onProgress(1);
    } else {
      this.onProgress(0);
      // Start animating
      this.startTime = performance.now();
      this.frameId = requestAnimationFrame(() => this.onFrame());
    }
  }
  onFrame() {
    const timePassed = performance.now() - this.startTime;
    const progress = Math.min(timePassed / this.duration, 1);
    this.onProgress(progress);
    if (progress < 1) {
      // We still have more frames to paint
      this.frameId = requestAnimationFrame(() => this.onFrame());
    }
  }
  onProgress(progress) {
    this.node.style.opacity = progress;
  }
  stop() {
    cancelAnimationFrame(this.frameId);
    this.startTime = null;
    this.frameId = null;
    this.duration = 0;
  }
}
```

```jsx
export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? 'Remove' : 'Show'}
      </button>
      <hr />
      {show && <Welcome />}
    </>
  );
}
```

### 3. 모달
```jsx
import { useState } from 'react';
import ModalDialog from './ModalDialog.js';

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(true)}>
        Open dialog
      </button>
      <ModalDialog isOpen={show}>
        Hello there!
        <br />
        <button onClick={() => {
          setShow(false);
        }}>Close</button>
      </ModalDialog>
    </>
  );
}
```

```jsx
mport { useEffect, useRef } from 'react';

export default function ModalDialog({ isOpen, children }) {
  const ref = useRef();

  useEffect(() => {
    if (!isOpen) {
      return;
    }
    const dialog = ref.current;
    dialog.showModal();
    return () => {
      dialog.close();
    };
  }, [isOpen]);

  return <dialog ref={ref}>{children}</dialog>;
}
```
