### 증상
socket.io 튜토리얼 : https://socket.io/docs/v4/tutorial/step-7
채팅을 sqlite로 저장했다가 거기서 불러오는데, 자꾸 안됨
그래서 better-sqlite3 을 시도해봤더니마찬가지로 안됨
### 에러문
> Error: Could not locate the bindings file. Tried: → /Users/jeonghoyoon/Documents/Works/VS-Code/socketio-test/node_modules/.pnpm/sqlite3@5.1.7/node_modules/sqlite3/build/node_sqlite3.node
>........
### 코드
```js
import express from "express";
import { createServer } from "node:http";
import { fileURLToPath } from "node:url";
import { dirname, join } from "node:path";
import { Server } from "socket.io";
// import sqlite3 from "better-sqlite3";
import Database from "better-sqlite3";

const app = express();
const server = createServer(app);
const io = new Server(server, { connectionStateRecovery: {} });

const __dirname = dirname(fileURLToPath(import.meta.url));

// open the database file
// const db = await open({
//   filename: "chat.db",
//   driver: sqlite3.Database,
// });
const db = new Database("chat.db");

// create our 'messages' table (you can ignore the 'client_offset' column for now)
// await db.exec(`
//   CREATE TABLE IF NOT EXISTS messages (
//       id INTEGER PRIMARY KEY AUTOINCREMENT,
//       client_offset TEXT UNIQUE,
//       content TEXT
//   );
// `);
db.exec(`
  CREATE TABLE IF NOT EXISTS messages (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      client_offset TEXT UNIQUE,
      content TEXT
  );
`);

app.use("/node_modules", express.static(join(__dirname, "node_modules")));

app.get("/", (req, res) => {
  res.sendFile(join(__dirname, "index.html"));
});

io.on("connection", (socket) => {
  console.log("conneted");
  socket.on("chat message", (msg) => {
    let result;
    try {
      //result = await db.run("insert into messages (content) values (?)", msg);
      const stmt = db.prepare("INSERT INTO messages (content) VALUES (?)");
      result = stmt.run(msg);
    } catch (e) {
      return;
    }
    // io.emit("chat message", msg, result.lastID);
    io.emit("chat message", msg, result.lastInsertRowid);
  });
});

server.listen(3001, () => {
  console.log("server running at http://localhost:3001");
});

```
### 설명
클로드는 계속 노드 버전과 sqlite3의 버전이 맞지 않는다고, 그래서 better-sqlite3을 써보라고 말해줬다.
근데 better-sqlite3도 안되는건 마찬가지였고, 공식문서의 마지막 업데이트 날짜는 25년 3월이었는데 두달 차이로 갑자기 안된다는게 이해할 수 없었다.

알고보니 처음에 `pnpm i sqlite3`을 했을때, 터미널에 떴던 경고문이 있었음.
"Ignored build scripts: sqlite3.
Run "pnpm approve-builds" to pick which dependencies should be allowed to run scripts."
그래서 `pnpm approve-builds`를 실행하고 sqlite3를 선택해서 진행했더니 해결된거같다.
