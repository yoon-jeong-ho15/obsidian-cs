# rollup, cube, grouping sets
## grouping sets
이게 가장 기본형이다.
`group by col1, col2` 는 col1을 기준으로 분류하고 거기서 col2를 기준으로 다시 분류한다
`group by grouping sets (col1, col2)`는 이것과 같다.

여기서 col1 전체에 대한 데이터를 보고싶다면?
`group by grouping sets((col1, col2), col1)`

## rollup
`group by rollup (col1, col2)`는 위의 `grouping sets((col1, col2), col1)`과 비슷한데,
거기서 col1과 col2를 떠나 테이블 전체의 행에 대한 집합연산 데이터를 산출한다.
즉 `group by rollup (col1, col2)` = `group by grouping sets ((col1, col2), (col1), ())`

## cube
`group by cube (col1, col2)`는 col1+col2(col1 기분으로 분류 후에 거기서 다시 col2 기준으로 분류), col1, col2 가 나오고 거기에 모든 행 전체에 대한 데이터를 산출한다.
즉 `group by cube (col1, col2)` = 
`group by grouping sets ((col1,col2), (col1), (col2), ())` 와 같다.
