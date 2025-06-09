# 개념
https://react.dev/reference/react/useCallback

`useCallback` is a React Hook that lets you *cache* a *function definition* between *re-renders*.
## cache?
컴포넌트가 다시 렌더링될 때마다 컴포넌트 내부에서 정의된 함수들은 사라졌다가 전부 새로운 함수 인스턴스로 다시 생성된다.
그런데 `useCallback`을 사용하면 함수를 메모리에 저장해두고 다시 렌더링된 컴포넌트에서는 메모리에 저장된 함수를 참조하여 사용한다.
*메모리에 저장하는 것*을 cache라고 부른다.
