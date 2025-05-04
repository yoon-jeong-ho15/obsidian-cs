#typescript #type
https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html

자바스크립트로 매우 큰 프로젝트를 만들 때, 종종 자바스크립트의 자동 타입추론이 제대로 작동하기 힘들 때가 있다.
그래서 타입스크립트는 다른 프로그래밍언어처럼 타입을 명시하는 interface를 정의하고 거기에 맞춰서 객체를 생성하도록 한다.
```ts
interface User {  
	name: string;  
	id: number;
}

const user: User = {
	name:"Hayes",
	id:0
};
```

자바스크립트에서도 이미 boolean, bigint, null, number, string, symbol 같은 기본 자료형이 있지만 타입스크립트에서 추가되는 기본 자료형으로는 다음의 것들이 있다.
- any
- unknown
- never
- void
# type? interface?
```ts
//interface
interface Person {  
	name: string;
	age: number;
} 
function greet(person: Person) {  
	return "Hello " + person.name;
}
//type
type Person = {  
	name: string;  
	age: number;
}; 
function greet(person: Person) {  
	return "Hello " + person.name;
}
```
그럼 인터페이스랑 타입의 차이는 뭔가?
둘 다 객체가 어떤 구조로 구성되어야하는지 정해주는거 아닌가?

## 차이점
### 자동 선언 병합 declaration merging
인터페이스는 똑같은 이름을 가진 두 인터페이스를 자동으로 병합한다.
(만약 두 인터페이스가 이름은 같지만 타입이 다른 속성을 가지고 있는 경우 에러가 발생한다.)
### 유니언 타입 Union type
`type TypeA = number | string;` 이라고 정의하고 어떤 객체에서 `age: TypeA;` 라고 한다면 그건 숫자이거나 문자열일 수 있게 되는것.
그냥 `age: number|string;`으로 해도 물론 되겠지만. 
## 언제 어떤걸 사용하는가
이 부분은 천천히 알아가야 할 듯.
`import type`은 번들 크기 최적화, 성능 향상의 장점이 있다고 하는데.


