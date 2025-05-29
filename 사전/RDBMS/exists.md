`exists()` 와 `not exists()`가 있다.
괄호 안에는 반드시 서브쿼리만 들어갈 수 있으며, 반환값은 T or F.

그래서 where절이나 having절 처럼 조건을 따지는 절에서 사용된다.
`where exists(select .. )` 에서 서브쿼리가 어떤것이든 행을 가지는 경우 True라서 해당 데이터는 조건을 만족한다.

`in()`이나 `not in()`과의 차이점
- in은 서브쿼리의 결과도 메모리에 담기 때문에 성능상 단점이 있다
- exists는 행의 존재 여부만 판단하기 때문에 select의 값이 의미가 없다.

null 처리
- exist는 null도 데이터가 있는것으로 판단하기 때문에 T를 반환.
