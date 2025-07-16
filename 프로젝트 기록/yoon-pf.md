dfNext.js + supabase 로 진행한 프로젝트
라이브러리 : Quill.js, Auth.js, motion
# 퀼
## delta와 디비 : 직렬화 문제
### 문제 상황
```tsx
const handleSave = () => {
	const content: Delta | null = quillRef.current?.getContents() || null;
	console.log("content : ", content);
	if (title && content) {
		createBoard(title, content);
	}
};
```
페이지에서 저장버튼을 누르면 위의 함수를 실행한다.

여기서 content를 찍어보면 `Object { ops: […] }` 라고 나온다.
그런데 `action.ts`에서 content를 찍어보면 `content :  [Function (anonymous)]`라고 나온다.
```ts
export async function createBoard(title: string, content: Delta | null) {
	const session = await auth();
	const user: User | null = (session?.user as User) || null;
	console.log("content : ", content);
	const { error } = await sql.from("board").insert({
		writer: user?.username,
		title: title,
		content: content,
	});
	if (error) {
		console.error("Error inserting Data : ", error);
	}
}
```
serialization이 문제라고 한다.
클라이언트에서 서버로 객체를 전송할때 자동으로 직렬화를 해서 전달하는데,
객체가 함수를 가지고 있으면 직렬화 과정에서 문제가 발생해서 객체 그대로 전달되지 않는것 같다.
그래서 `stringify()`로 문자열로 만들어 문자열 상태로 저장하고, 조회할때 `JSON.parse()`로 다시 객체로 만들어 화면에 보여준다.
## 뷰어에서 에디터가 두개가 됨.
### 문제상황
처음에 게시글 조회에 들어가면 `<div class="ql-container">`가 두개가 생김.
새로고침하면 하나만 남음.
### 코드
```tsx
"use client";
import Quill from "quill";
import { useRef, useEffect } from "react";
import "quill/dist/quill.bubble.css";

export default function Viewer({ content }: { content: string }) {
  const containerRef = useRef<HTMLDivElement | null>(null);
  const quillRef = useRef<Quill | null>(null);

  useEffect(() => {
    console.log("useEffect()");
    console.log("content:", typeof content, content);
    let quill: Quill | null = null;
    let container: HTMLDivElement | null = null;

    const loadQuill = async () => {
      console.log("loadQuill()");
      if (!containerRef.current) return;
      const QuillModule = await import("quill");
      const Quill = QuillModule.default;

      container = containerRef.current;
      const editorContainer = container.appendChild(
        document.createElement("div")
      );
      quill = new Quill(editorContainer, {
        theme: "bubble",
      });
      quill.enable(false);
      quillRef.current = quill;
      const delta = JSON.parse(content);
      quill.setContents(delta);
    };
    loadQuill();
  });

  return (
    <div className="h-full overflow-auto">
      <div ref={containerRef} className="ml-20 mr-30 px-15 outline-none" />
    </div>
  );
}

```
### 설명
콘솔에도 console.log들이 두번씩 찍힌다. 이게 useEffect가 두번 실행된다는 말인거 같은데. 
찾아보니까 개발환경에서는 useEffect가 두번 실행된다고 한다. [[Strict Mode]]
실제로 vercel의 배포판에서 보니까 게시글이 두번 등장하지 않음.

## editor-wrapper에서 수정시 content 안넘어감
### 문제 상황
editor에서는 db에서 가지고 온 
### 코드
```tsx
"use client";
import Quill from "quill";
import { useEffect, useRef } from "react";
import Editor from "@/app/more/board/write/editor";

export default function EditorWrapper({
  initialValue,
}: {
  initialValue: string;
}) {
  const quillRef = useRef<Quill | null>(null);

  useEffect(() => {
    if (!quillRef.current) return;
    
    try {
      const delta = JSON.parse(initialValue);
      quillRef.current.setContents(delta);
    } catch (error) {
      console.error("Error parsing initialValue", error);
    }
  }, [quillRef.current, initialValue]);

  useEffect(() => {
    const form = document.querySelector("form");
    form?.addEventListener("submit", () => {
      if (quillRef.current) {
        const delta = quillRef.current.getContents();
        const input = document.createElement("input");
        input.type = "hidden";
        input.name = "content";
        input.value = JSON.stringify(delta);
        form.appendChild(input);
      }
    });
  });
  return <Editor ref={quillRef} />;
}

```

### 설명
왜 content가 추가가 안되느냐 하면 이벤트 리스너가  제출 이후에 등록되서 그런다고한다.
문제점 1) 여기에 의존성 배열이 없고 2) 이벤트 리스너를 해제하지 않고 추가하고 3)이벤트 핸들러가 너무 늦게 실행한다.

### 해결
해결책 : content를 담을 input hidden을 넣고, useState로 값을 입력해준다.

# 수파베이스
## 클라이언트 컴포넌트에서 데이터 패칭
[[supabaseUrl is required#yoon-pf]]
게시글 조회 페이지에서 `editor.tsx`의 버튼을 누르면 Link 컴포넌트를 타고 `[id]/edit/page.tsx`로 이동해 거기서 `fetchBoardById()`를 하려고 했다.
문제는 이걸 하려면 이 페이지가 서버 컴포넌트여야 한다는것.
그러나 이건 수정페이지이기 때문에 `useRef`를 사용해 퀼 에디터의 내용물에 접근할 수 있어야했다.
생각해낸 방법:
1. Link 컴포넌트에 props로 첫 페이지에서 가져온 board를 타고 타고 넘기는것. 
	불가능함. 왜냐하면 링크 컴포넌트는 정해진 props만 가질 수 있음.
2. page 안에 다른 클라이언트 컴포넌트(EditorForm)를 만들고 거기에 에디터, 수정하기, 취소하기 버튼을 담는다.
	일단 간단한 방법인거 같은데, 페이지를 그냥 껍데기로만 쓰는거같아서 맘에들지 않는다.
3. 사실 가장 익숙한 방법으로 `action.ts`에서 데이터를 가져오면 되는데, 그러면 똑같은 기능의 함수를 두 개 사용하는것이라 맘에 들지 않는다. 데이터 가져오는건 전부 `data.ts`에서 하고싶다. 
### 클라이언트 아일랜드 패턴?
`/[id]/edit/page.tsx`에서는 이 패턴을 사용해봤다.
page에서 기본적인 ui를 다 가지고 있고,
form태그 안에 넣어두어서 form자체로 내용이 입력되도록. [[제어 or  비제어 컴포넌트]]

# 프리랜더링 오류
[[ReferenceError-document is not defined#yoon-pf]]
`/write/page.tsx`와 `/[id]/page.tsx`에서 다른 방식으로 동적로딩을 구현했다.
왜그랬냐하면, 게시글 조회 페이지는 쿼리를 해야되기때문에 서버컴포넌트여야 했기 때문인데
[[dynamic()]]자체는 서버에서도 사용 가능하지만  ssr:false는 서버 컴포넌트에서 불가능하다고.
그러나 게시글 작성 페이지는 서버 컴포넌트일 필요가 업어서 클라이언트 컴포넌트로 만들고 `ssr:false`으로 동적 로딩을 했다.

# 게시글 작성 후, redirect
### 증상
딱히 증상은 없음. 그냥 콘솔에 에러문구가 뜬다.
게시글 수정한뒤에 redirect하는건 에러가 안뜨는데 작성에서만 뜬다. 왜?

### 에러문
Uncaught (in promise) Error: NEXT_REDIRECT
    getRedirectError redirect.ts:21
    next NextJS
    promise callback\*serverActionReducer server-action-reducer.ts:227
    clientReducer router-reducer.ts:50
    action app-router-instance.ts:211
    runAction app-router-instance.ts:98
    dispatchAction app-router-instance.ts:163
    dispatch app-router-instance.ts:209
    next NextJS
    startTransition react-dom-client.development.js:7842
    dispatch use-action-queue.ts:45
    dispatchAppRouterAction use-action-queue.ts:22
    NextJS 3
    callServer app-call-server.ts:6
    action react-server-dom-turbopack-client.browser.development.js:2715
    createBoard actions.ts:35
    handleSave page.tsx:24
    executeDispatch react-dom-client.development.js:16501
    runWithFiberInDEV react-dom-client.development.js:847
    processDispatchQueue react-dom-client.development.js:16551
    next NextJS
    batchedUpdates$1 react-dom-client.development.js:3262
    dispatchEventForPluginEventSystem react-dom-client.development.js:16705
    dispatchEvent react-dom-client.development.js:20816
    dispatchDiscreteEvent react-dom-client.development.js:20783

# 채팅
Socket.IO라는걸 사용해보려고 했는데, vercel에서는 websocket 서버를 배포할 수 없다고. vercel은 서버리스 환경이라 지속적인 연결을 제공하지 않음.
https://socket.io/how-to/use-with-nextjs
https://nextjs.org/docs/pages/guides/custom-server

그래서 pusher라는 서비스를 이용하거나.
https://vercel.com/guides/deploying-pusher-channels-with-vercel

sse (server sent event) [[Sever-sent events]]
https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events
https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events

swr (stale while revalidate)
## sse
`more-nav.tsx`에서도 세션을 사용하기 위해,
`more/layout.tsx`에서 세션프로바이더를 사용하기로 하고, `chat/page.tsx`에서 세션프로바이더를 뺐더니
[[모듈 팩토리 오류#chatroom-provider]] 에러 발생
그런데 sse도 지속적인 연결을 필요로 하기 때문에 vercel에서 마찬가지로 사용할 수 없었다.

## sse 사용 불가 supabase realtime 사용
sse도 마찬가지로 사용불가.
왜냐하면 SSE도 마찬가지로 서버리스 환경에서는 지속적인 연결을 유지하지 못하기 때문이다.
내 조사가 미흡하고 짧았다.
vercel의 서버리스 함수는 무료판에서는 10초후에 자동으로 종료됨.
https://vercel.com/blog/an-introduction-to-streaming-on-the-web
이 링크를 읽어보면 


# useSession이 데이터를 가져오지 못함
`chat/page.tsx`에서 `await auth()`로 세션을 가져옴.
근데 `chat/chat-box.tsx`에서 `useSession()`은 세션 데이터를 가져오지 못함.

[[NextAuth.js 에러#profile, chat]]

# 버튼 위치 고정
## 버튼 위치 고정1
`chat-box.tsx`에서 전송버튼 위치를 부모 요소에서 `flex items-center`로 위치를 지정해줬음.
근데 그러면 단점이 textarea에 넣는 글이 길어지면서 form의 높이가 늘어나면 버튼의 위치도 form의 가운데가 되면서 점점 올라감.
그래서 textare의 onInput 핸들러에 버튼의 위치를 조절하는 코드를 넣었음
```tsx
 const handleInput = (e: React.FormEvent<HTMLTextAreaElement>) => {
    // Auto-resize logic
    e.currentTarget.style.height = "auto";
    const newHeight = Math.min(e.currentTarget.scrollHeight, 160);
    e.currentTarget.style.height = newHeight + "px";

    // Button positioning logic
    const button = e.currentTarget.nextElementSibling as HTMLButtonElement;
    const isMinHeight = newHeight <= 40; // min-h-10 is approximately 40px

    if (button) {
      if (isMinHeight) {
        // Center the button when at minimum height
        button.style.top = "50%";
        button.style.transform = "translateY(-50%)";
      } else {
        // Position at bottom when growing
        button.style.top = "auto";
        button.style.bottom = "8px"; // Equivalent to bottom-2
        button.style.transform = "none";
      }
    }

    console.log(e.target.value);
  };
```
이 코드는 성공적으로 버튼의 위치를 고정했지만, 처음에 렌더링되면서 지정된 위치와 조금은 달랐음.
그래서 해결하려면 버튼의 첫 위치와 똑같아지게 정밀하게 위의 수치들을 조율하면 되지만 그건 너무 힘들다고 판단.
그래서 클로드에게 물어보니까 `useEffect`훅을 쓰라고 함.
```tsx
useEffect(() => {
    if (textareaRef.current && buttonRef.current) {
      const textarea = textareaRef.current;
      const button = buttonRef.current;
      const form = textarea.parentElement;
      
      // Calculate the exact position where button should be fixed
      const formRect = form.getBoundingClientRect();
      const buttonRect = button.getBoundingClientRect();
      
      // Calculate the top position (distance from top of form to center of button)
      const topPosition = (formRect.height / 2) - (buttonRect.height / 2);
      
      // Fix the button at this exact position
      button.style.top = `${topPosition}px`;
      button.style.transform = 'none'; // No need for transform
    }
  }, []);
```

> [!1]- 그렇다면 만약에 이게 서버 컴포넌트라면 어떻게 해야하나?

### useEffect 가 실행이 안되나?
위의 코드를 넣고 form에서 `flex items-center`를 지워줬더니 버튼이 form 위에 바로 딱 붙었다.
그래서 문제를 확인해보려고 useEffect 안에 `console.log("topPosition : ",topPosition)`을 넣어봤다.
그랬더니 콘솔에 아무것도 찍히지 않았다.
클로드는 `const [isComponentMounted, setIsComponentMounted] = useState(false);` 를 추가해서 `isCompnentMounted`를 위의 의존성 배열에 넣으라고 함.
그래도 안됨.
#### 안되는 코드
```tsx
// Second useEffect for button positioning with dependencies
  useEffect(() => {
    console.log("Button positioning effect running");
    console.log("Textarea ref exists:", !!textareaRef.current);
    console.log("Button ref exists:", !!buttonRef.current);

    if (textareaRef.current && buttonRef.current) {
      // Add a small delay to ensure DOM is fully rendered
      setTimeout(() => {
        const textarea = textareaRef.current;
        const button = buttonRef.current;

        if (!textarea || !button) {
          console.log("Refs no longer valid");
          return;
        }

        const form = textarea.parentElement;
        if (!form) {
          console.log("Form parent not found");
          return;
        }

        // Log all dimensions for debugging
        const formRect = form.getBoundingClientRect();
        const buttonRect = button.getBoundingClientRect();

        console.log("Form dimensions:", formRect);
        console.log("Button dimensions:", buttonRect);

        // Calculate the top position
        const topPosition = formRect.height / 2 - buttonRect.height / 2;
        console.log("topPosition:", topPosition);

        // Fix the button position
        button.style.top = `${topPosition}px`;
        console.log("Button position set");
      }, 100); // Small delay to ensure rendering
    }
  }, [isComponentMounted]); // Run when component is confirmed mounted
```
- 타임아웃을 줘서 ref들이 null이 아니게될 때까지 기다림.
#### 되는 코드
```tsx
 useEffect(() => {
    // 이 조건을 추가하여 두 ref가 모두 존재할 때만 실행합니다
    if (textareaRef.current && buttonRef.current) {
      console.log("Refs now available, positioning button");

      const textarea = textareaRef.current;
      const button = buttonRef.current;
      const form = textarea.parentElement;

      if (!form) {
        console.log("Form parent not found");
        return;
      }

      const formRect = form.getBoundingClientRect();
      const buttonRect = button.getBoundingClientRect();

      console.log("Form dimensions:", formRect);
      console.log("Button dimensions:", buttonRect);

      const topPosition = formRect.height / 2 - buttonRect.height / 2;
      console.log("topPosition:", topPosition);

      button.style.top = `${topPosition}px`;
      console.log("Button position set");
    } else {
      console.log("Refs not yet available");
    }
  }, [textareaRef.current, buttonRef.current, isComponentMounted]);
```
- 의존성 배열에 ref들을 넣어서 컴포넌트 전체가 렌더링된 후에 다시 실행되게 했다.

## 훅 사용 규칙 에러
[[A change in the order of Hooks#chat-box.tsx]]

# NoProfile 컴포넌트 크기 변경 안됨
#tailwind
[[CSS(tailwind) 에러#yoon-pf|코드는 여기 참고]]
`<NoProfile size="..." />` 이렇게 컴포넌트를 하나 만들고 크기만 따로 props로 넘겨주면서 필요할때마다 다른 크기의 컴포넌트를 만들어 활용하려고 했다.
그런데, 6만 제대로 크기가 조절되고 나머지 모든 숫자들은 부모가 가진 `flex` 때문에 꽉 차버리게 됐다.

알고보니 테일윈드가 사용하는 JIT 방식 컴파일링 때문. [[Tailwind#JIT 컴파일러]]
그래서 방법은 
1. safelist를 추가해주거나, 
2. `classname = {{ full? "size-full" : "size-6"}}` 이렇게 조건에 따라 둘 다 가능하도록 적어두면 JIT 컴파일러도 둘 다 생성해두기 때문에 가능하다.
3. 또 (아마 Next.js 튜토리얼에서 본거같은데) `<NoProfile size="md" />` 이렇게 넘기고 NoProfile 컴포넌트 안에서  sm,md,lg,xl,full 이런식으로 분류하고 각각 크기들을 미리 정의해둘 수 있다. 그리고 그 변수를 활용하는것. *매핑* 방식
	- 난 공부를 위해 이 방법을 택했다.
```Tsx
export default function NoProfile({ sizeprop }: { sizeprop?: "full" | "sm" }) {
  const size = {
    sm: "size-6",
    full: "size-full",
  };

  return (
    <svg
      xmlns="http://www.w3.org/2000/svg"
      fill="none"
      viewBox="0 0 24 24"
      strokeWidth={1.5}
      stroke="currentColor"
      className={`${size[sizeprop ? sizeprop : "full"]}`}
    >
      <path
        strokeLinecap="round"
        strokeLinejoin="round"
        d="M17.982 18.725A7.488 7.488 0 0 0 12 15.75a7.488 7.488 0 0 0-5.982 2.975m11.963 0a9 9 0 1 0-11.963 0m11.963 0A8.966 8.966 0 0 1 12 21a8.966 8.966 0 0 1-5.982-2.275M15 9.75a3 3 0 1 1-6 0 3 3 0 0 1 6 0Z"
      />
    </svg>
  );
}
```

# 로그인 
[[NextAuth 라이브러리#yoon-pf]]
## `action.ts`에서 정의한 `authenticate()`
nextjs 튜토리얼 프로젝트를 그대로 가져와서 처음엔 이렇게 별도의 authenticate 함수를 정의하고 사용했다.
```ts
export async function authenticate(
  prevState: string | undefined,
  formData: FormData
) {
  try {
    console.log("authenticate() formData : ", formData);
    await signIn("credentials", formData);
  } catch (error) {
    if (error instanceof AuthError) {
      const authError = error as { type?: string };
      switch (authError.type) {
        case "CredentialsSignin":
          return "Invalid credentials.";
        default:
          return "Something went wrong.";
      }
    }
    throw error;
  }
}
```
여기서 사용되는 `signIn`은 `auth.ts`에서 `NextAuth()`로 만든것.

## `login-form.tsx`에서 사용하는 `signIn()`
그런데 로그인 컴포넌트에서는 정작 다른 함수 사용중. 
```tsx
import { signIn } from "next-auth/react";

  const credentialsAction = (formData: FormData) => {
    signIn("credentials", {
      username: formData.get("username"),
      password: formData.get("password"),
    });
  };
  
    return (
	    <form
	      action={credentialsAction}
```

직접 `next-auth/react`패키지에서 가져온 그대로 사용한다.

> [!1] 그런데 어떻게 내가 `auth.ts`에 적어둔 설정들이 반영되는거지? 
> session, jwt 콜백과 credentials 설정들 등등.
> `/auth/[...nextauth]/route.ts` 에서 handler를 `auth.ts`에서 불러오기 때문에 가능하다.

# reference equality
데이터베이스에서 (채팅방id, 제목, 유저id, 유저명, 프로픽사진) 형식으로 테이블을 가져온다.
이걸 map 자료형에 저장 `map<Chatroom, Chatroomuser[]>`.
그런데 `chatroom={id:string, title: string}` 에서 id와 title이 똑같더라도 다른 객체로 판단해서
`(방1, 사용자들[])` 로 되는게 아니라 `(방1, [사용자1])` `(방1, [사용자2])` 처럼 됐다.

자바에서는 새로운 클래스를 정의하고 거기서 `equals()` 함수를 재정의하면 방의 id와 title이 같다면 같은 객체로 취급해서 `if (chatrooms.has(room)) {chatrooms.get(room)?.push(user);}` 가 제대로 작동할것인데, 자바스크립트에서는 이렇게 value equality 방식을 사용하지 않고 *reference equality* 방식만을 사용하기 때문에 불가능

이걸 해결하기 위해서 map의 구조를 `<key=roomId, value={room, user[]}>` 이렇게 바꾸는 방법이 있고,
내가 채택한 방법처럼 room을 null로 먼저 생성하고 chatrooms 맵에서 가져오는것으로 해결할 수 있다.
```ts
export async function fetchChatrooms(username: string) {
  // console.log("fetchChatrooms username : ", username);
  const { data, error } = await supabase.rpc("get_chatroom_data", {
    p_username: username,
  });
  const chatrooms: ChatroomMap = new Map();
  data.map((obj: Record<string, unknown>) => {
    console.log("obj", obj);

    const roomId = obj.id as string;
    const roomtitle = obj.title as string;

    //채팅방
    let room = null;
    for (const [existingRoom] of chatrooms) {
      if (existingRoom.id === roomId) {
        room = existingRoom;
        break;
      }
    }

    if (!room) {
      room = {
        id: roomId,
        title: roomtitle,
      };
    }

    //유저
    const user = {
      username: obj.username as string,
      id: obj.uid as string,
      profilePic: obj.profile_pic as string,
    };

    if (chatrooms.has(room)) {
      chatrooms.get(room)?.push(user);
    } else {
      chatrooms.set(room, [user]);
    }
  });

  console.log(chatrooms);
  if (error) {
    console.error("Error fetching data : ", error);
    throw new Error("Failed to fetch Chatrooms");
  } else {
    return chatrooms;
  }
}
```

# 새로고침시에 nav의 selectedTab 초기화
# `auth.config.ts` 에서 세션유저가 비어있는 값
## 문제 상황
```Ts
export const authConfig = {
  pages: {
    signIn: "/login",
  },
  callbacks: {
    authorized({ auth, request: { nextUrl } }) {
      const isLoggedIn = !!auth?.user;
      console.log("=== Auth Debug Info ===");
      console.log("auth:", auth);
      console.log("auth?.user:", auth?.user);
      console.log("isLoggedIn:", isLoggedIn);
      console.log("nextUrl.pathname:", nextUrl.pathname);
```
이렇게 했더니 결과가 다음과같이 텅 비어서 나온다.
```
=== Auth Debug Info ===
auth: { user: {}, expires: '2025-08-05T06:54:37.565Z' }
auth?.user: {}
isLoggedIn: true
nextUrl.pathname: /more/chat
```

## 원인
`auth.config.ts`에  jwt, session 콜백이 작성되어 있지 않기 때문에. 이 콜백들은 jwt토큰과 세션을 어떻게 저장할지에 대해 지정하는것인데.
`auth.ts`는 `auth.config.ts`에서 설정을 불러와서 적용한다.
```Ts
import { authConfig } from "./auth.config";

//...

export const { handlers, auth, signIn, signOut } = NextAuth({
  ...authConfig, // <<<< 여기
  providers: [
    Credentials({
      credentials: {
      //...
      
```
그래서 `auth.ts`에 적혀있던 callback 내용들을 `auth.config.ts`는 모르는 상태라서, jwt토큰을 제대로 읽어낼 수 없었던 것.

## 해결
토큰과 세션 콜백을 `auth.config.ts`로 옮겼다.

> [!1]- 그런데 아예 `auth.config.ts`만 사용하지 않는 이유는?
> 애초에 `auth.ts`에 작성된 내용들도 다 한곳에 모아서 거기서 어플리케이션이 사용할 `auth()`,`signIn()`, `signOUt()`등을 내보내면 되지 않은건가?
> Vercel의 Edge Runtime 때문 [[Edge Runtime]] 
> `middleware.ts`에서 사용하는 `Nextauth().auth`는 Edge-safe한 것이라고.



# 왜 다른사람이 로그인 할 때 자꾸 에러가 발생할까?
## 문제상황
그다지 어렵지 않은 계정명(이름)과 비밀번호(생년월일 6자) 사용중이다.
근데 같은 계정도 다른사람이 로그인할 때는 에러가 발생한다고 함. 
내가 시도하면 되는데.