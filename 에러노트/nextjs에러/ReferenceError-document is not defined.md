### 에러문
Error occurred *prerendering* page "/more/board/write". Read more: nextjs.org\/docs/messages/prerender-error
ReferenceError: document is not defined
at /vercel/path0/.next/server/chunks/112.js:1:81396
at Array.forEach (<anonymous\>)
at 87731 (/vercel/path0/.next/server/chunks/112.js:1:81384)
at t (/vercel/path0/.next/server/webpack-runtime.js:1:143)
at 418 (/vercel/path0/.next/server/app/more/board/write/page.js:1:296)
at Object.t \[as require] (/vercel/path0/.next/server/webpack-runtime.js:1:143)
at require (/vercel/path0/node_modules/.pnpm/next@15.3.1_react-dom@19.1.0_react@19.1.0__react@19.1.0/node_modules/next/dist/compiled/next-server/app-page.runtime.prod.js:140:10304)
at u (/vercel/path0/node_modules/.pnpm/next@15.3.1_react-dom@19.1.0_react@19.1.0__react@19.1.0/node_modules/next/dist/compiled/next-server/app-page.runtime.prod.js:92:646)
at U (/vercel/path0/node_modules/.pnpm/next@15.3.1_react-dom@19.1.0_react@19.1.0__react@19.1.0/node_modules/next/dist/compiled/next-server/app-page.runtime.prod.js:92:10632)
at W (/vercel/path0/node_modules/.pnpm/next@15.3.1_react-dom@19.1.0_react@19.1.0__react@19.1.0/node_modules/next/dist/compiled/next-server/app-page.runtime.prod.js:92:13339)
Export encountered an error on /more/board/write/page: /more/board/write, exiting the build.
⨯ Next.js build worker exited with code: 1 and signal: null
 ELIFECYCLE  Command failed with exit code 1.
Error: Command "pnpm run build" exited with 1
### 코드
```tsx
//page.tsx
"use client";
import { useState, useRef } from "react"; 
import Editor from "./editor"; 
import Quill, { Delta } from "quill"; 
import { createBoard } from "@/lib/actions"; 

export default function Page() {
	const quillRef = useRef<Quill | null>(null); 
	const [title, setTitle] = useState(""); 
	
	const handleSave = () => { 
		const delta: Delta | null = quillRef.current?.getContents() || null; 
		const content = JSON.stringify(delta); 
		console.log("content : ", content); 
		if (title && content) {
			createBoard(title, content); 
		} 
	}; 
	
	return ( 
	<div className="container mx-auto px-4 py-8 mt-4 bg-gray-200"> 
		<div className="w-full px-10 flex text-3xl items-center relative"> 
			<div> 
				<span>제목</span> 
				<input 
					onChange={(e) => { setTitle(e.target.value); }} 
					className="mx-2 bg-gray-50 rounded-sm py-2
					focus:outline-0 text-center border border-gray-300 shadow" >
				</input> 
			</div> 
			<div 
			className="flex justify-end gap-2 absolute right-3"> 
				<button 
					onClick={handleSave} 
					className="
					bg-blue-500 hover:bg-blue-600 text-white 
					font-medium px-6 py-2 rounded-lg text-lg 
					shadow-md" > 
					작성하기 
				</button> 
			</div> 
		</div> 
		<div className="my-4 bg-white shadow h-190"> 
			<Editor ref={quillRef} /> 
		</div> 
	</div> 
	); 
}
```
```tsx
//editor.tsx
"use client";
  
import { useEffect, useRef, RefObject } from "react";
import Quill from "quill";
import "quill/dist/quill.bubble.css";
  
export default function Editor({ ref }: { ref: RefObject<Quill | null> }) {
	const containerRef = useRef<HTMLDivElement | null>(null);
	  
	useEffect(() => {
		const container = containerRef.current;
		if (container) {
			const editorContainer = container.appendChild(
			container.ownerDocument.createElement("div")
		);
		const quill = new Quill(editorContainer, { theme: "bubble" });
		  
		ref.current = quill;
		  
		return () => {
			ref.current = null;
			container.innerHTML = "";
		};
		}
	}, [ref]);
	return (
		<div className="h-full overflow-auto">
			<div ref={containerRef} className="ml-20 mt-8 mr-30" />
		</div>
	);
}
```
### 설명
`write/page.tsx`를 프리렌더링 할 수 없어서 발생하는 에러. [[pre-rendering]]
페이지를 미리 만들어 둘 때 거기서 불러온 quill 라이브러리가 `document`객체를 사용하는 DOM API를 필요로 한다. (가령 quill 라이브러리 안에서 `document.getElementById` 와 같은 자바스크립트 코드를 사용할 수 있다)
즉 quill을 실질적으로 사용하는 `editor.tsx`에서 퀼 라이브러리를 불러와서 문제가 되는것.
`write/page.tsx`에서도 quill를 불러오지만 여기서 객체를 생성하지는 않았기 때문에 문제가 되지 않았다.

#### quillRef
그런데 애초에 quillRef를 왜 페이지에서 만들어서 전달해야했을까?

#### dynamic
동적 로딩을 구현하는 방법에는 두가지 방법이 있다. 
1. 페이지에서 [[dynamic()]]을 사용해서 하는 방법. 
```tsx
const DynamicEditor = dynamic(() => import('./editor'), {
  ssr: false,
  loading: () => <div className="로딩 상태 UI"></div>
});
```
2. 에디터 컴포넌트에서 하는 동적 로딩
```tsx
import Quill from "quill"; //이 방법이 정적 로딩. 

useEffect(() => {
  const loadQuill = async () => {
    const QuillModule = await import('quill');
    const Quill = QuillModule.default;
    // Quill 초기화...
  };
  loadQuill();
}, []);
```


