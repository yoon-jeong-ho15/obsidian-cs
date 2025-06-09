# 개념
## 비동기 적용
```jsx
const [count, setCount] = useState(0);
// ...
const handleCount = () => {
  console.log("before: ", count);
  setCount(count + 1);
  console.log("after: ", count);
}
```
여기서 before 와 after가 똑같이 찍힐것이다.
버튼을 누름 -> handleCount() 실행 -> before : 0 출력 -> setCount()실행 (상태 업데이트 예약)-> after : 0 출력 -> handleCount() 완료 -> 컴포넌트 리렌더링 -> count  = 1 이 됨 -> 이 count 값을 참고해 ui 업데이트.

