# 개념
https://react.dev/reference/react/useActionState

Call `useActionState` at the *top level of your component* to create component state that is updated [when a form action is invoked](https://react.dev/reference/react-dom/components/form). You pass `useActionState` an existing form action function as well as an initial state, and it returns a new action that you use in your form, along with the latest form state and whether the Action is still pending. The latest form state is also passed to the function that you provided.

- top level of your component?
다른 함수 안이나 조건문, 반복문 내부가 아닌 컴포넌트 함수 본문의 가장 바깥쪽에서 호출해야한다는 의미.

## 용례
폼을 제출하고 반환받은 값을 사용할 수 있다.
기존에는 `fetch`로 폼을 제출하고 서버에서 데이터를 반환받으면 그걸 `useEffect`을 통해 화면에 반영해왔다.
그런데 `useActionState`를 사용하면 한번에 관리할 수 있다고. 


