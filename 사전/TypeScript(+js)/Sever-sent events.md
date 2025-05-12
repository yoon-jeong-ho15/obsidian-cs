# 개념
출처 :
https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events
https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events

첫줄부터 이 기능을 Web Workers 에서도 사용 가능하다고 하는데.. [[Web Workers API]] 
(왜 나는 이걸 읽어보지 않을 수 없었을까? 읽어보니 정말로 더도 말고 덜도 말고 "웹 워커"에서도 사용할 수 있다는 말이었다.)

## EventSource
자바스크립트 인터페이스.
EventSource 객체는 HTTP 서버와의 지속적인 연결을 가진다. 
이벤트를 `text/event-stream` 형식으로 전달한다.
`EventSource.close()`를 통해 명시적으로 연결을 닫아줘야 연결이 해제된다. 

"the triggered event is the same as the event field value. If no event field is present, then a generic `message` event is fired."

## vs WebSockets
sse는 unidirectional 이다.
uni, 단일한 방향으로 작동한다.
그래서 특히나 클라이언트 -> 서버로 메시지 형식의 데이터를 보낼 필요가 없을 때 사용하기 좋다고. 
"`EventSource` is a useful approach for handling things like social media status updates, news feeds, or delivering data into a [client-side storage](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Client-side_APIs/Client-side_storage) mechanism like [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API) or [web storage](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)."

