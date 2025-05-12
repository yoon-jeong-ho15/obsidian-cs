# 개념
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures

내부와 외부함수의 묶음. 외부함수의 상태에 대해 내부 함수가 참조할 수 있다.
```js
///
function init() {
  var name = "Mozilla"; // name is a local variable created by init
  function displayName() {
    // displayName() is the inner function, that forms a closure
    console.log(name); // use variable declared in the parent function
  }
  displayName();
}
init();

///
function makeFunc() {
  const name = "Mozilla";
  function displayName() {
    console.log(name);
  }
  return displayName;
}

const myFunc = makeFunc();
myFunc();

```
두 코드 모두 똑같은 결과를 내지만, 2번 경우에서는 `displayName`이 실행되기 전에 `myFunc`로 반환된 뒤에 실행된다는 것.
다른 언어들과 달리 자바스크립트에서는 클로저를 형성하기 때문에 가능하다. 
클로저는 lexical environment + 내부 함수의 조합이다.
여기서 lexical environment란 클로저가 생성되면서 같이 생성된 변수들을 의미한다. 
`myFunc`는 `displayName`를 참조한다. 정확히 `makeFunc()`가 실행되는 순간에 만들어진 `displayName`을 참조한다. 그리고 그때 같이 만들어진 `name`변수가 lexical environment이며 이 값을 기억한다.

```js
function makeAdder(x) {
  return function (y) {
    return x + y;
  };
}

const add5 = makeAdder(5);
const add10 = makeAdder(10);

console.log(add5(2)); // 7
console.log(add10(2)); // 12
```

`add5` 는 `function(y) {return 5 + y;}`를 참조하고 있다.
그래서 `add5(2)`를 실행하면 5+2=7.
"In essence, `makeAdder` is a function factory."