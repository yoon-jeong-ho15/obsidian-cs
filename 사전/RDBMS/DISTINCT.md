# 개념
select 절에서만 사용 가능.

from, where, group by, having은 데이터베이스에서 데이터들을 조회해서 테이블(파생 테이블?)을 만든다.
거기서 select를 사용해 최종적으로 결과를 출력한 것이 Result Set.

distinct는 여기서 중복되는 행을 하나로 합쳐서 출력하도록 만드는  함수다.
다음과 같은 문법은 잘못된 명령어다.
- `select distinct(A), B` 
- `select distinct(A,B), C`
- `select distinct(A), distinct(B)`
