# 개념
## UI building blocks
컴포넌트는 *1) HTML + CSS + JS* 를 합쳐놓은 ,*2) 재사용이 가능한* UI 요소다.

리액트 컴포넌트는 기본적으로 자바스크립트 함수이다. 마크업을 생성할 수 있는 함수.
기존에는 주가되는 마크업으로 뼈대를 만들고 자바스크립트 함수를 통해 상호작용 기능을 추가했다면 리액트 컴포넌트는 *자바스크립트 함수*가 주가 되는 요소.

## JSX
일반 js에서 html 문법을 추가한 형식.
js와 별 차이는 없는데 리액트 작성을 훨씬 편하게 해준다.
### Return a single root element
가장 중요한 규칙. return(...)안에 하나의 태그 안에 모든게 다 담겨있어야 한다.
왜?
jsx는 결국 자바스크립트로 컴파일된다. 모든게 다 자바스크립트 함수들로 이루어지게 되는건데, 함수가 한번에 여러 값을 반환할 수 없기 때문이다.

### camelCase most of the things
js로 컴파일되면서 jsx태그 안에 작성한 속성들은 js객체의 속성이 된다.
`<Tag className="...." dataId="...." />` 이게 아래와 같이 된다.
`const className = "....";`
`const dataId = ".....";`
그런데 자바스크립트에서 `data-id`와 같은 변수명(key)를 사용할 수 없기 때문에 카멜케이스로 작성하는 것이고
`const class="...";`리고도 하지 못하는 이유는 class가 자바스크립트 안에서 예약어기 때문에.

## Props
컴포넌트를 호출할 때 전달할 정보들.
전달 받는 자식 컴포넌트를 정의할 때 매개변수에 자신이 전달받을(수 있는) props들에 대해 작성해둬야한다.
`function Avatar({person, size}){...}` 보통은 이렇게 destructuring해서 받는다.
`function Avatar({person, size = 100}){...}` 이렇게 기본값을 줄 수 있다.

### children prop
```jsx
function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

export default function Profile() {
  return (
    <Card>
      <Avatar
        size={100}
        person={{ 
          name: 'Katsuko Saruhashi',
          imageId: 'YfeOqp2'
        }}
      />
    </Card>
  );
}
```
이렇게 Card 컴포넌트 안의 내용물을 그 부모인 Profile에서 작성할 수 있다.

### event handler

## State
왜 일반 변수로는 부족한가?
1. **Local variables don’t persist between renders.** When React renders this component a second time, it renders it from scratch—it doesn’t consider any changes to the local variables.
2. **Changes to local variables won’t trigger renders.** React doesn’t realize it needs to render the component again with the new data.
일반적인 지역변수는 렌더링과 렌더링 사이에 유지되지 않고,
지역변수의 변화가 렌더링을 유발하지 않는다고.

그래서 `useState`훅을 사용해서 state와 setState 함수를 만들어서 setState함수로 state를 변경해줘야 1) 렌더링 사이에 유지되면서 2) 변화가 렌더링을 유발한다.

### Anatomy of `useState()` 
useState훅은 어떻게 여러 state를 구별할까?
훅의 실행 순서로 구분한다.
```jsx
  const [index, setIndex] = useState(0);
  const [showMore, setShowMore] = useState(false);
```
이렇게 두 개의 훅이 있으면 
렌더링마다 동일한 순서로 실행되니까 그 인덱스로 해당 state를 구분한다.
