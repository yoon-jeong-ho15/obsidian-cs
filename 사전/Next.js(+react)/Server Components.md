# 데이터
출처: https://nextjs.org/learn/dashboard-app/fetching-data

데이터를 패치하는데 몇가지 방법이 있다.
1. api layer
	`/api/route.ts`같은 루트핸들러를 만든다. 그리고 거기서 fetch api 사용.
2. 서버 컴포넌트
	말 그대로 서버에서 실행하는 컴포넌트이기 때문에 디비에 직접 접근 가능.
