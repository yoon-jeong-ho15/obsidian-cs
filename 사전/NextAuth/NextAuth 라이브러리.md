
# 예시
## yoon-pf
### auth.ts
```ts
import type { User } from "@/lib/definitions";
import NextAuth from "next-auth";
import { createClient } from "@supabase/supabase-js";
import { authConfig } from "./auth.config";
import { z } from "zod";
import Credentials from "next-auth/providers/credentials";

const supabase = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_ANON_KEY!
);

async function getUser(username: string): Promise<User | undefined> {
  const { data: user, error } = await supabase
    .from("user")
    .select("*")
    .eq("username", username);

  if (error) {
    console.error("Failed to fetch user", error);
    throw new Error("Failed to fetch user.");
  }
  return user[0];
}

export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
  //세션 토큰은 기본적으로 30일 유지
  callbacks: {
    session: ({ session, token }) => {
      return {
        ...session,
        user: {
          ...session.user,
          id: token.sub,
          username: token.username as string,
          from: token.from as number,
        },
      };
    },
    jwt: ({ token, user }) => {
      if (user) {
        token.username = (user as User).username;
        token.from = (user as User).from;
      }
      return token;
    },
  },
  providers: [
    Credentials({
      async authorize(credentials) {
        const parsedCredentials = z
          .object({
            username: z.string(),
            password: z.string(),
          })
          .safeParse(credentials);

        if (parsedCredentials.success) {
          const { username, password } = parsedCredentials.data;
          const user = await getUser(username);

          if (!user) return null;
          const passwordsMatch = user.password == password ? true : false;

          if (passwordsMatch) return user;
        }
        console.log("Invalid credentials");
        return null;
      },
    }),
  ],
});
```
`NextAuth()`는 매개변수로 `NextAuthConfig`를 받는다.
근데 나는 지금 `...authConfig`, `callbacks`, `providers` 이렇게 세개를 보내는 중.
`...authConfig`는 내가 `auth.config.ts`파일에서 정리해둔걸 불러오는 것,
`callbacks`랑 `providers`는 여기서 다시 정의하면서 `authConfig`랑 합치는 것.

> [!1]- 그렇다면 `auth.config.ts`에 한번에 작성해도 되는가?
> 클로드 [[클로드/NextAuth#auth.config.ts에 모든 설정 입력|NextAuth]]

내가 인자로 넣은것들을 가지고 `auth`, `signIn`, `signOut` 함수를 가져온다.
로그인을 할 때 `login-form.tsx`에서 
`const [errorMessage, formAction] = useActionState(authenticate, undefined);`
를 통해 `action.ts`에 있는 `authenticate()`를 실행. [[useActionState]]
거기서 `await signIn("credentials", formData)`을 실행.
#### 과정
##### next-auth index.ts 
```ts
export interface NextAuthResult{
	//.....
	 signIn: <P extends ProviderId, R extends boolean = true>(
	    provider?: P, 
	    options?:
	      | FormData
	      | ({
	          redirectTo?: string
	          redirect?: R
	        } & Record<string, any>),
	    authorizationParams?:
	      | string[][]
	      | Record<string, string>
	      | string
	      | URLSearchParams
	  ) => Promise<R extends false ? any : never>
	//.....
  }
export default function NextAuth(config):NextAuthResult{
	//.....
	return{
		//...
	    signIn: (provider, options, authorizationParams) => {
	      return signIn(provider, options, authorizationParams, config)
	    },
	    //...
	}
}
```
위를 보면 `signIn`을 다른 파일에서 가져온다는 걸 볼 수 있다.
그리고 그걸 
 `import { signIn, signOut, update } from "./lib/actions.js"`
##### next-auth action.js
```js
export async function signIn(provider, options = {}, authorizationParams, config) {
    const headers = new Headers(await nextHeaders());
    const { redirect: shouldRedirect = true, redirectTo, ...rest } = options instanceof FormData ? Object.fromEntries(options) : options;
    const callbackUrl = redirectTo?.toString() ?? headers.get("Referer") ?? "/";
    const signInURL = createActionURL("signin", 
    // @ts-expect-error `x-forwarded-proto` is not nullable, next.js sets it by default
    headers.get("x-forwarded-proto"), headers, process.env, config);
    if (!provider) {
        signInURL.searchParams.append("callbackUrl", callbackUrl);
        if (shouldRedirect)
            redirect(signInURL.toString());
        return signInURL.toString();
    }
    let url = `${signInURL}/${provider}?${new URLSearchParams(authorizationParams)}`;
    let foundProvider = {};
    for (const providerConfig of config.providers) {
        const { options, ...defaults } = typeof providerConfig === "function" ? providerConfig() : providerConfig;
        const id = options?.id ?? defaults.id;
        if (id === provider) {
            foundProvider = {
                id,
                type: options?.type ?? defaults.type,
            };
            break;
        }
    }
    if (!foundProvider.id) {
        const url = `${signInURL}?${new URLSearchParams({ callbackUrl })}`;
        if (shouldRedirect)
            redirect(url);
        return url;
    }
    if (foundProvider.type === "credentials") {
        url = url.replace("signin", "callback");
    }
    headers.set("Content-Type", "application/x-www-form-urlencoded");
    const body = new URLSearchParams({ ...rest, callbackUrl });
    const req = new Request(url, { method: "POST", headers, body });
    const res = await Auth(req, { ...config, raw, skipCSRFCheck });
    const cookieJar = await cookies();
    for (const c of res?.cookies ?? [])
        cookieJar.set(c.name, c.value, c.options);
    const responseUrl = res instanceof Response ? res.headers.get("Location") : res.redirect;
    // NOTE: if for some unexpected reason the responseUrl is not set,
    // we redirect to the original url
    const redirectUrl = responseUrl ?? url;
    if (shouldRedirect)
        return redirect(redirectUrl);
    return redirectUrl;
}
```
코드를 보면 별 문제나 특별한 점이 없으면 
`redirectUrl`을 반환하거나 거기로 리다이렉트를 해준다.
그런데 이걸 `index.ts`에서 다시 import 하고 다시 정의하고 그걸 다시 반환한다.
근데 `NextAuthResult`의 정의부분에는 `signIn()`은 매개변수 3개를 가지는데,
왜 반환하는건 `_config`까지 포함해 4개를 반환할 수 있는거지?
`NextAuthResult`의 정의에 있는 `signIn()`과 결과적으로 반환하는 `signIn()`이 다르다. 후자는 `action.ts`에서 import한 함수.
참고 : [[클로저 Closure]]
##### 어떻게 `signIn()`이 같은 이름일 수 있나?
 [[셰도잉]] -> X
자바스크립트에서는 객체의 속성 이름과 변수 이름을 다르게 취급한다. "별개의 네임스페이스에 있다."
```js
// 변수로서의 name
const name = "홍길동";

// 객체 속성으로서의 name
const person = {
  name: "김철수",  // 여기서 name은 객체 속성 이름
  sayName() {
    console.log(name);  // 여기서 name은 변수를 참조 (홍길동 출력)
  }
};

console.log(person.name);  // 김철수 (객체 속성 접근)
person.sayName();  // 홍길동 (변수 접근)
```

클로저 개념과 합쳐서 이해하면,
`auth.ts`의 `const {.., signIn} = NextAuth({...});`에서 `signIn`은 `NextAuthResult`의 속성으로서 존재하는 `signIn1()`.
이 `signIn()`은 `next-auth/.../index.ts`애 정의되어있길 `signIn1() return signIn2(...)` 이다.
그래서 로그인할때 `action.ts`의 `authenticate`함수를 실행하면 거기 안에서 `signIn1("credentials", formData)`를 실행한다.
그러면 `return signIn2("credentials", formData, authorizationParams, config)` 이 실행되면서 `action.ts`에 정의되어있는대로 redirect가 실행되는것.

##### authorizationParams 는 없는데?
```ts
type SignInParams = Parameters<NextAuthResult["signIn"]>
export async function signIn(
  provider: SignInParams[0],
  options: SignInParams[1] = {},
  authorizationParams: SignInParams[2],
  config: NextAuthConfig
) {
	//.....
```
`action.ts`의 signIn을 보면 authorizationParams 매개변수가 명시되어 있다. (`config`는 자동으로 넣어져서 괜찮지만)
보면 위에서 `SignInParams` 라는 배열 안에 타입들이 담겨있는거 같다.
`Parameters<T>` 는 T의 매개변수들을 튜플 타입으로 추출.

##### Auth()
서드파티 로그인을 사용하는중이라면 이렇게 url을 통해 해당 서비스에 연결한다.
그런데 credentials를 사용중이라면 "signin" 대신 "callback" 주소로
post 방식 요청을 보낸다. `Auth()`함수에서.
그리고 완료되면 `const cookeiJar = await cookies()` 와 그밑의 코드들을 통해 쿠키와 토큰을 만들어 브라우저에 저장한다.

#### jwt
`auth.ts`의 `authorize`함수에 적힌대로 아이디와 비밀번호가 맞으면 `user` 객체를 반환한다.
이걸 가지고 jwt에 `token`을 만들고 이걸 가지고 `session`에 `user`를 넣는것.

### 클라이언트 signIn

### \[...nextauth]/route.ts
```ts
import { handlers } from "@/auth";
export const { GET, POST } = handlers;
```
이게 없어서 `useSession()` 이 계속 "GET /api/auth/session 404 in 146ms" 404 에러가 뜬것.
[[NextAuth.js 에러#profile, chat]]