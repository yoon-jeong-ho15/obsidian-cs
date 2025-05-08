#  yoon-pf
## socket.io 튜토리얼 코드에서 
### 에러문
The resource from “[http://localhost:3001/node_modules/socket.io/client-dist/socket.io.js](http://localhost:3001/node_modules/socket.io/client-dist/socket.io.js "http://localhost:3001/node_modules/socket.io/client-dist/socket.io.js")” was blocked due to MIME type (“text/html”) mismatch (X-Content-Type-Options: nosniff).
### 코드
```html
<script src="node_modules/socket.io/client-dist/socket.io.js"></script>
```
### 설명
Socket.IO 튜토리얼 :https://socket.io/docs/v4/tutorial/step-3 
이걸 보고 하다, `<script src="/socket.io/socket.io.js"></script>` 이렇게만 해서 socket.io의 자바스크립트 파일을 가져오길래 신기해서 찾아봤다.
그러니까 `<script src="... />`이것도 하나의 요청인것. 
그래서 `<script src="node_modules/socket.io/client-dist/socket.io.js"></script>` 이렇게 해봤더니, 해당 에러가 발생
그래서 `app.use('/node_modules', express.static(join(__dirname, 'node_modules')));`를 추가했다.
