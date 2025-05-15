# 개념
https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollHeight

`overflow` 지정때문에 보이지 않는 내용까지 포함한 엘리먼트 내용의 높이
value 는 *엘리먼트의 내용물이 스크롤바가 필요 없이도 다 보일만큼의 최소높이*와 같다.
"The `scrollHeight` value is equal to the *minimum height the element would require in order to fit all the content in the viewport without using a vertical scrollbar*.'"
> [!1]- 이게 그냥 '엘리먼트의 높이'와 다른점이 뭔지 모르겠음.

높이를 재는 방식은 `clientHeight`를 재는 방법과 똑같다 : 엘리먼트의 padding / X border, margin, horizontal scrollbar

## 다 읽었는지 확인하기
```js
const info = document.getElementById("info");
const toAgree = document.getElementById("agree");
const toNextStep = document.getElementById("next-step");
const veryImportantRead = document.getElementById("very-important-read");

// Check if user has scrolled the element to the bottom
function isRead(element) {
  return (
    Math.abs(element.scrollHeight - element.clientHeight - element.scrollTop) <=
    1
  );
}

function checkScrollToBottom(element) {
  if (isRead(element)) {
    info.innerText = "You have read all text. Agree to continue.";
    toAgree.disabled = false;
  }
}

//checkbox input 요소
toAgree.addEventListener("change", (e) => {
  toNextStep.disabled = !e.target.checked;
});

//div 요소
veryImportantRead.addEventListener("scroll", () => {
  checkScrollToBottom(veryImportantRead);
});

//input submit 요소
toNextStep.addEventListener("click", () => {
  if (toAgree.checked) {
    toNextStep.value = "Done!";
  }
});
```


