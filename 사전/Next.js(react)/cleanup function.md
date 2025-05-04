[[useEffect]]에서 사용되는 것이다.
useEffect의 setup함수 매개변수에 들어가는것.
`return ()=>{...}` 이라고 정의한다.
의존배열(dependencies)가 바뀌어서 useEffect가 다시 실행될 때마다 먼저 클린업함수가 실행되고 다시 나머지 셋업함수가 실행된다.
아니면 컴포넌트가 언마운트될 때 실행된다.

