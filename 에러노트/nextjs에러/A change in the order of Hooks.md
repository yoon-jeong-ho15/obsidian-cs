# yoon-pf
## chat-box.tsx
### 에러문
React has detected a change in the order of Hooks called by ChatBox. This will lead to bugs and errors if not fixed. For more information, read the Rules of Hooks: https://react.dev/link/rules-of-hooks 
Previous render Next render ------------------------------------------------------ 
1. useContext useContext 
2. useEffect useEffect 
3. useRef useRef 
4. useRef useRef 
5. undefined useEffect
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
### 코드
```tsx
if (status === "loading") {
    return <div>loading</div>;
  }
  if (!session || !session.user) {
    return <div>no session</div>;
  }

  useEffect(() => {
    if (textareaRef.current && buttonRef.current) {
      const textarea = textareaRef.current;
      const button = buttonRef.current;
      const form = textarea.parentElement!;

      // Calculate the exact position where button should be fixed
      const formRect = form.getBoundingClientRect();
      const buttonRect = button.getBoundingClientRect();

      // Calculate the top position (distance from top of form to center of button)
      const topPosition = formRect.height / 2 - buttonRect.height / 2;

      // Fix the button at this exact position
      button.style.top = `${topPosition}px`;
      button.style.transform = "none"; // No need for transform
    }
  }, []);
```
### 설명
일단 `useEffect` 전에 `if`에서 하는 렌더링이 앞에 있어서 문제라는걸 알게됐다.
[[Rules of Hooks]]을 보면 훅을 사용할때 지켜야 할 규칙들이 있고 거기에 내가 범한 오류가 바로 적혀있다.
근데 이유에 대해서는 설명을 해주지 않는다.
그래서 클로드에게 물어봤다. (너무 기대는거 아닐까 싶기도)

일단 "Previous Render"와 "Next Render"가 무얼 의미하는가?
말 그대로 이전 렌더와 다음 렌더를 의미하는데, 그게 구체적으로 무엇이냐면 이 컴포넌트가 렌더링 되는 과정 중에서 이전 렌더링과 다음 렌더링 단계를 의미한다.
위의 코드를 보면 먼저 `session`을 가져오면서 `if (status==="loading")` 일 경우 로딩 메시지를 렌더링 하기로 되어있다. 이게 Previous Render인 것.
그리고 다음에 `status==="authenticated"`가 되면 원래대로 컴포넌트 전체가 렌더링 되고 이게 Next Render이다.
문제가 되는 원인은 이 `ChatBox`라는 컴포넌트는 함수이기 때문에 코드를 위에서 아래로 하나씩 실행하는데 이전 렌더에서는 useEffect를 실행하지 않고 넘어갔다가(`if(){return(...)}`니까 밑에 있는 코드는 실행되지 않음) 다음 렌더에서는 `useEffect`를 실행하려고 하니까 문제가 되는것.
#### 왜 문제가 되는가
React uses an *array-based system, not names*.
리액트 컴포넌트는 사용되는 훅들을 배열에 저장한다. 그래서 배열의 인덱스로 각 훅들을 저장하고 사용하는데 위에서처럼 어떤 경우에는 훅이 사라지고 어떤 경우에는 훅이 생기는 경우에는 배열의 길이가 달라져서 매번 똑같은 훅을 사용한다는 보장성 잃게된다.




