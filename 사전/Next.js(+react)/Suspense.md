
Suspense에 대해서는 Next.js 튜토리얼에서 사용해봤다 (https://nextjs.org/learn/dashboard-app/streaming)
```tsx
        <Suspense fallback={<RevenueChartSkeleton />}>
          <RevenueChart />
        </Suspense>
```
이렇게 서스펜스 컴포넌트 안에 불러올 컴포넌트를 배치해서 뻣뻣하지 않은 자연스런 ui를 보여주기 위한 기능.