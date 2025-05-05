Next.js + supabase 로 진행한 프로젝트

# 퀼
## delta와 디비
### 증상
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


# 프리랜더링 오류
[[ReferenceError-document is not defined#yoon-pf]]
`/write/page.tsx`와 `/[id]/page.tsx`에서 다른 방식으로 동적로딩을 구현했다.
왜그랬냐하면, 게시글 조회 페이지는 쿼리를 해야되기때문에 서버컴포넌트여야 했기 때문에.
dynamic() ssr:false는 서버 컴포넌트에서 불가능하다고.
