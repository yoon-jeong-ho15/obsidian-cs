# yoon-pf
## editor.tsx
### 문구
51:6  Warning: React Hook useEffect has a missing dependency: 'container'. Either include it or remove the dependency array.  react-hooks/exhaustive-deps
### 코드
```tsx
"use client";

import { useEffect, useRef, RefObject } from "react";
import type Quill from "quill";
import "quill/dist/quill.bubble.css";

export default function Editor({ ref }: { ref: RefObject<Quill | null> }) {
  const containerRef = useRef<HTMLDivElement | null>(null);
  const container = containerRef.current;
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
      if (quill && container) {
        if (typeof ref !== "function") {
          ref.current = null;
        }
        container.innerHTML = "";
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
containerRef.current 대신에 container를 사용했더니 다른 경고가 등장.
container 변수를 사용하고있는데 의존성 배열에 들어있지 않다고 하는 경고.
왜 의존성 배열에 넣어야하지?
일단 의존성 배열에 넣으면 `container`변수의 값이 변할때마다 useEffect가 실행되면서 퀼 에디터를 초기화 할 것 아닌가?
그러려면 useEffect 안에서 `container`를 선언하면 된다. 그런데 그러면 처음에 발생했던 경고로 다시 돌아가는 꼴. [['ref.current' will likely have changed]]
#### 해결?
그래서 container를 의존성 배열에 넣을 수는 없으니 useEffect 안에서 container를 선언해야 한다.
그런데 기존처럼 loadQuill()에서 선언할 수 없으니 밖으로 빼서 useEffect에서 선언하고 초기화를 loadQuill()안에서 한다.
그랬더니 일단은 `pnpm lint` 경고는 사라졌다.
