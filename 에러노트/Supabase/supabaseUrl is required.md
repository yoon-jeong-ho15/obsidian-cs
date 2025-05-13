# yoon-pf
## /\[id]/edit/page.tsx

### 에러문구
Error: supabaseUrl is required.
### 코드
```tsx
"use client";

import { fetchBoardById } from "@/lib/data";
import { useParams } from "next/navigation";

export default function Page() {
  const params = useParams<{ id: string }>();
  const id = params.id;
  const board = await fetchBoardById(id);
  return (
    <div>
      <span>{id}</span>
      <span>edit page</span>
    </div>
  );
}

```
### 설명
음 일단 클라이언트 컴포넌트에서 supabase를 쓰려고 하니까 안되는것 같다. [[Server Components]]
왜냐하면 `/[id]/page.tsx`에서는 보드 가져오는데 전혀 문제가 발생하지 않았기 때문에.
111222
