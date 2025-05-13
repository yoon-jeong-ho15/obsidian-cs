# 공식문서
출처 :
- https://nextjs.org/docs/app/guides/lazy-loading

dynamic은 `React.lazy(`) 와 [[Suspense]]를 합친것과 같다고 한다. 

## ssr : false
그런데 dynamic이 이것과 다른 주요한 특징은 바로 ssr을 해제할 수 있다는 것.
그래서 특정 오류들을 예방할 수 있다. [[ReferenceError-document is not defined]]
그런데 서버 컴포넌트에서는 `ssr:false`를 지정할 수 없다.

React.lazy()나 Suspense를 사용하면 클라이언트 컴포넌트도 기본으로 [[pre-rendering|프리랜더링]] 된다.

그리고 서버컴포넌트를 dynamic으로 불러오는 경우에는, 불러오는 서버컴포넌트 안에 있는 클라이언트 컴포넌트만 천천히 불러와지고 불러오려는 서버컴포넌트 자체는 정적으로 바로 불러온다.





