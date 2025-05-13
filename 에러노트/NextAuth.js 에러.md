# Client Fetch Error
## yoon-pf
### profile, chat
[[yoon-pf#useSession이 데이터를 가져오지 못함]]
#### 에러문
"ClientFetchError: JSON.parse: unexpected character at line 1 column 1 of the JSON data. Read more at [https://errors.authjs.dev#autherror](https://errors.authjs.dev/#autherror)"
#### 증상
서버 컴포넌트인 `Page`들에서 각각 `const session = await auth()` 를 통해 세션을 가져오는게 가능했다. (이 `auth()`는 `auth.ts`에 있는것)
근데 각 페이지들 안에 있는 클라이언트 컴포넌트들 안에서 `useSession()`을 사용해서 세션 데이터를 가져오려니까 계속 에러가 발생.
#### 설명
[[NextAuth 라이브러리#[...nextauth]/route.ts|NextAuth 라이브러리 링크]]
`api/auth/[...nextauth]/route.ts`가 자동으로 `api/auth/session` 요청도 자동으로 처리해주는것.