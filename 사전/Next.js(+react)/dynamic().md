# 공식문서
출처 :
- https://nextjs.org/docs/app/guides/lazy-loading

dynamic은 React.lay() 와 Suspense를 합친것과 같다고 한다. 
Suspense에 대해서는 Next.js 튜토리얼에서 사용해봤다 (https://nextjs.org/learn/dashboard-app/streaming)
```tsx
        <Suspense fallback={<RevenueChartSkeleton />}>
          <RevenueChart />
        </Suspense>
```
이렇게 서스펜스 컴포넌트 안에 불러올 컴포넌트를 배치해서 뻣뻣하지 않은 자연스런 ui를 보여주기 위한 기능.

그런데 dynamic이 이것과 다른 주요한 특징은 바로 ssr을 해제할 수 있다는 것.
그래서 특정 오류들을 예방할 수 있다. [[ReferenceError-document is not defined]]
그런데 서버 컴포넌트에서는 `ssr:false`를 지정할 수 없다.

그리고 서버컴포넌트를 dynamic으로 불러오는 경우에는, 불러오는 서버컴포넌트 안에 있는 클라이언트 컴포넌트만 천천히 불러와지고 불러오려는 서버컴포넌트 자체는 정적으로 바로 불러온다.



