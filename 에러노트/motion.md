# Element type is invalid
## yoon-pf
### app/page.tsx
#### 에러문
Error: Element type is invalid: expected a string (for built-in components) or a class/function (for composite components) but got: undefined. You likely forgot to export your component from the file it's defined in, or you might have mixed up default and named imports.
#### 코드
```tsx
import { motion } from "motion/react";

export default function Page() {
  return (
    <>
      <h1>home</h1>
      <motion.ul className="box" />
    </>
  );
}

```
#### 설명
클라이언트 컴포넌트가 아니라서 그렇다고.

그런데 https://motion.dev/docs/react-motion-component#server-side-rendering 여기를 보면 서버사이드 렌더링이 된다고 하는데??

`motion` components are fully compatible with server-side rendering, meaning the initial state of the component will be reflected in the server-generated output.
```tsx
// Server will output `translateX(100px)`
<motion.div initial={false} animate={{ x: 100 }} />
```

This is with the exception of some SVG attributes like `transform` which require DOM measurements to calculate.
##### 해결
```ts
// React
import { motion } from "motion/react"

// React Server Components
import * as motion from "motion/react-client"
```
멍청. 튜토리얼 맨 앞에 나온다. 서버 컴포넌트에서는 `motion/react-client`패키지에서 불러왔어야.
