# 공식문서
출처 : 
- https://nextjs.org/learn/pages-router/data-fetching-pre-rendering

nextjs는 모든 페이지를 프리렌더링을 해서 미리 만든다고 한다.
이렇게 하는 이유는 성능개선과 seo 때문이라고.

미리 만들어진 html 페이지들은 *최소한의 자바스크립트*만을 가지고 있다가 브라우저가 이 페이지를 불러오는 순간 *그 자바스크립트 코드들로 완전한 페이지*를 불러온다고. 이걸 **hydration** 이라고 부른다.


