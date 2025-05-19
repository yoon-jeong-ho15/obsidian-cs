# yoon-pf
## chatroom-provider
### 에러문
\[Error: Module \[project]/src/app/more/chat/chatroom-provider.tsx \[app-ssr] 
(ecmascript) was *instantiated* 
because it was required from module \[project]/node_modules/.pnpm/next@15.3.1_react-dom@19.1.0_react@19.1.0__react@19.1.0/node_modules/next/dist/esm/build/templates/app-page.js?*page=/more/chat/page*
{ MODULE_0 
=> "\[project]/src/app/*layout.tsx* \[app-rsc] (ecmascript, Next.js server component)", 
MODULE_1 
=> "\[project]/node_modules/.pnpm/next@15.3.1_react-dom@19.1.0_react@19.1.0__react@19.1.0/node_modules/next/dist/client/components/*not-found-error.js* \[app-rsc] (ecmascript, Next.js server component)", 
MODULE_2 
=> "\[project]/node_modules/.pnpm/next@15.3.1_react-dom@19.1.0_react@19.1.0__react@19.1.0/node_modules/next/dist/client/components/*forbidden-error.js* \[app-rsc] (ecmascript, Next.js server component)", 
MODULE_3 
=> "\[project]/node_modules/.pnpm/next@15.3.1_react-dom@19.1.0_react@19.1.0__react@19.1.0/node_modules/next/dist/client/components/*unauthorized-error.js* \[app-rsc] (ecmascript, Next.js server component)", 
MODULE_4 
=> "\[project]/src/app/more/*layout.tsx* \[app-rsc] (ecmascript, Next.js server component)", MODULE_5 
=> "\[project]/src/app/more/*chat/page.tsx* \[app-rsc] (ecmascript, Next.js server component)" } \[app-rsc] (ecmascript) \<locals>, but the *module factory* is not available. It might have been deleted in an *HMR update*.] {
  digest: '1785654218'
}
### 코드
```Tsx
"use client";
import React, { createContext, useContext, useState } from "react";

type ChatroomContextType = {
  selectedChatroom: string | null;
  selectChatroom: (id: string) => void;
};

const ChatroomContext = createContext<ChatroomContextType | null>(null);

export default function ChatroomProvider({
  children,
}: {
  children: React.ReactNode;
}) {
  const [selectedChatroom, setSelectedChatroom] = useState<string | null>(null);

  const selectChatroom = (id: string) => {
    setSelectedChatroom(id);
  };

  return (
    <ChatroomContext.Provider value={{ selectedChatroom, selectChatroom }}>
      {children}
    </ChatroomContext.Provider>
  );
}

export function useChatroom() {
  const context = useContext(ChatroomContext);
  if (context === undefined) {
    throw new Error("useChatroom must be used within a ChatroomProvider");
  }
  return context;
}
```

```tsx
import { auth } from "@/auth";
import { User } from "@/lib/definitions";
import ChatList from "./chat-list";
import ChatroomProvider from "./chatroom-provider";
import { fetchChatrooms, fetchOneChatroom } from "@/lib/data";
import MessageBox from "./message-box";
import MessageForm from "./message-form";

export default async function Page() {
  const session = await auth();
  console.log("chat/page.tsx > session : ", session);
  if (!session || !session.user) {
    return <div>no session</div>;
  }
  const user = session.user as User;
  let chatrooms = null;
  let chatroom = null;
  if (user.username === "윤정호") {
    chatrooms = await fetchChatrooms(user.username);
  } else {
    chatroom = await fetchOneChatroom(user.username);
  }

  return (
    <div className="w-[90%]">
      <h1>chat</h1>
      <ChatroomProvider>
        <div
          className="h-180 flex rounded shadowp-1 container p-1
            bg-gradient-to-r from-blue-400 to-indigo-400"
        >
          <div
            className="w-full h-full bg-white rounded container 
              flex flex-col justify-between shadow"
          >
            <MessageBox user={user} chatroom={chatroom} />
            <MessageForm user={user} chatroom={chatroom} />
          </div>
        </div>
        {user.username === "윤정호" && <ChatList chatrooms={chatrooms} />}
      </ChatroomProvider>
    </div>
  );
}

```
### 설명
