# items-center
## items-center가 자식 div를 안보이게 만듦
### yoon-pf
#### 코드
```tsx
import NoProfile from "public/no-profile";

export function Chat() {
  return (
    <div className="flex flex-col w-1/6 items-center">
      <div className="flex justify-center">
        <NoProfile size="full" />
      </div>
      <span>username</span>
    </div>
  );
}

export default function ChatList() {
  return (
    <div className="bg-gray-200 mt-6 rounded-xl p-4 grid grid-rows-1">
      <Chat />
    </div>
  );
}
///////
export default function NoProfile({ size }: { size: string }) {
  return (
    <svg
      xmlns="http://www.w3.org/2000/svg"
      fill="none"
      viewBox="0 0 24 24"
      strokeWidth={1.5}
      stroke="currentColor"
      className={`size-${size}`}
    >
      <path
        strokeLinecap="round"
        strokeLinejoin="round"
        d="M17.982 18.725A7.488 7.488 0 0 0 12 15.75a7.488 7.488 0 0 0-5.982 2.975m11.963 0a9 9 0 1 0-11.963 0m11.963 0A8.966 8.966 0 0 1 12 21a8.966 8.966 0 0 1-5.982-2.275M15 9.75a3 3 0 1 1-6 0 3 3 0 0 1 6 0Z"
      />
    </svg>
  );
}

```
#### 증상
`Chat`컴포넌트의 최상위 div에 `items-center`를 주자마자 `NoProfile`을 품고있는 div가 안보이게 됐다.
인스펙터로 보면 아예 크기가 0x0 이 되면서 사라짐. 그런데 위치는 username 위에 알맞게 자리잡고는 있다.

#### 해결
하위 div(`NoProfile`을 품고있는)에 크기 지정이 되지 않아서 그런것이었다. 

#### 추가 문제
참고 [[yoon-pf#NoProfile 컴포넌트 크기 변경 안됨]]
