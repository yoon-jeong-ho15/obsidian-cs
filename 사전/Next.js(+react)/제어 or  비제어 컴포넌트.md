# 개념
출처 : 
https://legacy.reactjs.org/docs/uncontrolled-components.html
https://goshacmd.com/controlled-vs-uncontrolled-inputs-react/

넥스트에서 언급되는 부분 : https://nextjs.org/learn/dashboard-app/adding-search-and-pagination
## controlled component
리액트에서 대부분의 경우에 사용하길 추천하는 방법이라고 한다.
form의 데이터가 말 그대로 리액트에 의해 제어된다고 해서 제어 컴포넌트라고 부른다.
```jsx

handleNameChange = (event) => {
	this.setState({ name: event.target.value });
};

<input value={someValue} onChange={handleChange} />
```
위에처럼 useState를 통해 변수들을 관리하는걸 제어 컴포넌트라고 하는듯.

### value 속성
value 속성은 실제 dom에 있는 값에 덮어씌워진다.
`<input value={someValue} onChange={handleChange} />` 여기서 onChange에 state를 바꿔주면서 dom의 해당 input태그 값이 바뀔 때 마다 handleChange를 통해 setState를 하고, 그걸 다시 someValue에 넣어준다면, 
dom의 값이 변경 -> state 변경 -> 그 state를 다시 input의 값으로 입력
## uncontrolled component
반대로 비제어 컴포넌트란 리액트가 아닌 DOM에 의해 관리되는 경우를 말한다. 
"keeps the source of truth in the DOM"
제출시에만 값이 불러와진다.


