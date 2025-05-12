# yoon-pf
## editor.tsx
### 에러문
./src/app/more/board/write/editor.tsx
48:22  Warning: The ref value '*containerRef.current*' will likely have changed by the time this effect cleanup function runs. If this ref points to a node rendered by React, copy 'containerRef.current' to a variable inside the effect, and use that variable in the cleanup function.  react-hooks/exhaustive-deps
### 코드
```tsx
"use client";

import { useEffect, useRef, RefObject } from "react";
import type Quill from "quill";
import "quill/dist/quill.bubble.css";

export default function Editor({ ref }: { ref: RefObject<Quill | null> }) {
  const containerRef = useRef<HTMLDivElement | null>(null);
  // const [isClient, setIsClient] = useState(false);

  // useEffect(() => {
  //   setIsClient(true);
  // }, []);

  useEffect(() => {
    if (!containerRef.current) return;
    let quill: Quill | null = null;

    const loadQuill = async () => {
      try {
        const QuillModule = await import("quill");
        const Quill = QuillModule.default;

        const container = containerRef.current;
        if (!container) return;

        const editorContainer = container.appendChild(
          document.createElement("div")
        );

        quill = new Quill(editorContainer, { theme: "bubble" });

        if (typeof ref !== "function") {
          ref.current = quill;
        }
      } catch (error) {
        console.error("Error initializing Quill:", error);
      }
    };

    loadQuill();

    return () => {
      if (quill && containerRef.current) {
        if (typeof ref !== "function") {
          ref.current = null;
        }
        containerRef.current.innerHTML = "";
      }
    };
  }, [ref]);

  return (
    <div className="pt-8 pb-10 overflow-auto h-full">
      <div ref={containerRef} className="ml-20 mr-30 h-full" />
    </div>
  );
}
```
### 설명
클린업 함수는 지금 ref가 변하거나 컴포넌트가 사라질 때 실행된다.
여기서 quill 객체나 containerRef.cuurent 가 null이나 undefined가 아닌 경우에는 ref를 비우고 컨테이너 안에도 innerHTML을 비운다.
여기서 containerRef의 값이 클린업 함수 실행되는 순간에 변했을 수 있기 때문에 경고를 한다는 것인데.
#### quill vs containRef
그러면 `if (quill && containerRef.currnet)`에서 quill은 문제가 왜 없는가?

일단 quill을 보자.
선언은 useEffect안 맨위에서 
초기화는 loadQuill() 실행되면서 

containerRef를 보자.
선언은 에디터 컴포넌트 맨 위에서 
초기화는 loadQull() 실행되면서.
#### container를 loadQuill() 밖으로
지금 loadQuill 안에서 container를 선언하고 그 다음에 `container.appendChild()`로 안에 자식 돔을 넣어주고 거기에 퀼 에디터를 넣어준다.

#### 새로운 경고
[[useEffect has a missing dependency#editor.tsx]]




