# 개념
웹 워커란 메인 스레드가 아닌 백그라운드 스레드에서 코드를 실행할 수 있게 해주는 것.

`Worker()` 안에 자바스크립트 파일을 넣어 실행.

worker 안에서 대체로 모든 코드를 수행할 수 있는데 직접적으로 DOM을 조정할수는 없다.
(https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Functions_and_classes_available_to_workers)

메인스레드와 워커 사이에 데이터 전달은 message시스템을 통해서 이루어진다.
	- 양 측 모두 `postMessage()`라는 함수를 통해 메시지를 전달하고, `onmessage`이벤트 핸들러를 통해 전송된 메시지에 대한 응답을 수행한다.
	- he data is copied rather than shared.

## Service Worker 
거의 프록시 서버와 비슷한 활용도
