## 4주차 발표자료입니다.

공식문서를 정리한 내용입니다.

- **Authentication**: https://nextjs.org/docs/app/building-your-application/authentication
- **Page & Layouts**: https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts

---

## Authentication

> https://nextjs.org/docs/app/building-your-application/authentication

- 인증(Authentication): 사용자가 주장하는 대로 자신이 누구인지 확인하는 과정. 사용자는 사용자 이름과 비밀번호와 같은 자격 증명을 통해 자신의 신원을 증명.

- 세션 관리(Session Management): 사용자의 상태를 추적하고 관리하는 과정. 예를 들어, 사용자가 로그인한 상태를 여러 요청에서 유지할 수 있음

- 권한 부여(Authorization): 사용자가 애플리케이션의 어느 부분에 접근할 수 있는지를 결정하는 과정

### 인증: 로그인 예시

#### 컴포넌트

1. 사용자의 input
2. form이 서버 액션을 호출
3. 검증이 성공하면, 사용자의 인증이 완료된 프로세스가 진행
4. 검증이 실패하면 오류 메시지가 표시

```tsx
import { authenticate } from "@/app/lib/actions";

export default function Page() {
  return (
    <form action={authenticate}>
      <input type="email" name="email" placeholder="Email" required />
      <input type="password" name="password" placeholder="Password" required />
      <button type="submit">Login</button>
    </form>
  );
}
```

#### 서버 액션

- 인증 성공: 보호된 경로에 접근하거나 사용자 정보를 가져오는 등의 추가 작업
- 인증 실패: 에러를 반환합니다.

```tsx
"use server";

import { signIn } from "@/auth";

export async function authenticate(_currentState: unknown, formData: FormData) {
  try {
    await signIn("credentials", formData);
  } catch (error) {
    if (error) {
      switch (error.type) {
        case "CredentialsSignin":
          return "Invalid credentials.";
        default:
          return "Something went wrong.";
      }
    }
    throw error;
  }
}
```

#### 폼 컴포넌트: useFormState, useFormStatus

```tsx
// app/login/page.tsx
"use client";

import { authenticate } from "@/app/lib/actions";
import { useFormState, useFormStatus } from "react-dom";

export default function Page() {
  const [errorMessage, dispatch] = useFormState(authenticate, undefined);

  return (
    <form action={dispatch}>
      <input type="email" name="email" placeholder="Email" required />
      <input type="password" name="password" placeholder="Password" required />
      <div>{errorMessage && <p>{errorMessage}</p>}</div>
      <LoginButton />
    </form>
  );
}

function LoginButton() {
  const { pending } = useFormStatus();

  const handleClick = (event) => {
    if (pending) {
      event.preventDefault();
    }
  };

  return (
    <button aria-disabled={pending} type="submit" onClick={handleClick}>
      Login
    </button>
  );
}
```

### 인가

#### 미들웨어를 이용한 경로 보호

Next.js에서 미들웨어를 사용하면 사용자의 인증 상태와 역할에 따라 애플리케이션의 다른 부분에 대한 접근을 제어할 수 있습니다.

1. 미들웨어 설정

- 프로젝트의 루트 디렉토리에 middleware.ts 또는 .js 파일을 생성합니다.
- 사용자가 특정 경로에 대한 인증 및 권한을 가지고 있는지 확인하는 로직을 구현합니다

2. 보호된 경로 정의하기

- 모든 경로가 인증을 필요로 하지는 않으므로, 미들웨어의 matcher 옵션을 사용하여 인증 검사가 필요하지 않은 경로를 지정합니다.

3. 미들웨어 로직:

- 사용자가 인증되었는지 검증하고, 경로의 권한 또는 역할을 확인하기 위한 로직을 작성합니다.

4. 인가되지 않은 접근 처리:

- 인가되지 않은 사용자를 로그인 페이지나 적절한 오류 페이지로 리디렉션합니다.

```tsx
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  const currentUser = request.cookies.get("currentUser")?.value;

  // 현재 사용자가 로그인한 상태이고, "/dashboard"로 시작하는 경로가 아닌 경우,
  // "/dashboard"로 리디렉션합니다.
  if (currentUser && !request.nextUrl.pathname.startsWith("/dashboard")) {
    return Response.redirect(new URL("/dashboard", request.url));
  }

  // 현재 사용자가 로그인하지 않은 상태이고, "/login"으로 시작하는 경로가 아닌 경우,
  // "/login"으로 리디렉션합니다.
  if (!currentUser && !request.nextUrl.pathname.startsWith("/login")) {
    return Response.redirect(new URL("/login", request.url));
  }
}

// matcher 속성을 사용하여 Middleware가 적용될 경로를 지정
export const config = {
  matcher: ["/((?!api|_next/static|_next/image|.*\\.png$).*)"],
};
```

- 이 예시는 요청 파이프라인 초기에 리디렉션을 처리하기 위해 Response.redirect를 사용하여 효율적으로 중앙 집중화된 접근 제어를 구현합니다.
- 특정 리디렉션 요구 사항에 대해 redirect 함수를 사용하여 Server Components, Route Handlers, 그리고 Server Actions에서 더 많은 제어를 제공할 수 있습니다. 이는 역할 기반 탐색이나 문맥에 맞는 시나리오에 유용합니다.

```tsx
import { redirect } from "next/navigation";

export default function Page() {
  // Logic to determine if a redirect is needed
  const accessDenied = true;
  if (accessDenied) {
    redirect("/login");
  }

  // Define other routes and logic~
}
```

관리자 사용자는 관리자 대시보드로 리디렉션되고, 일반 사용자는 다른 페이지로 보내질 수 있습니다. 이는 역할별 경험과 필요에 따른 조건적 탐색을 위해 중요합니다. ex) 사용자가 프로필 완료를 유도하는 등의 상황

인가를 설정할 때는 앱이 데이터에 접근하거나 변경하는 곳에서 주요 보안 검사가 수행되도록 하는 것이 중요합니다. 미들웨어는 초기 유효성 검사에 유용할 수 있지만, 데이터 보호의 유일한 방어 선이 되어서는 안 됩니다. 데이터 접근 계층(DAL) 내에서 주요 보안 검사가 이루어져야 합니다.

- **서버 액션**: 민감한 작업에 대한 보안 검사는 서버 측 프로세스에 구현
- **라우트 핸들러**: 인가된 사용자만이 접근할 수 있도록 들어오는 요청을 관리하는 보안 조치를 추가
- **데이터 접근 계층 (DAL)**: 데이터베이스와 직접 상호작용하며, 데이터 접근 및 수정의 핵심 상호작용 지점에서는 보안 검사를 수행

### Protecting Server Actions

- 작업을 진행하기 전에 사용자 role를 확인하는 예시

```tsx
// app/lib/actions.ts
"use server";

// ...

export async function serverAction() {
  const session = await getSession();
  const userRole = session?.user?.role;

  // 사용자가 작업을 수행할 권한이 있는지 확인합니다.
  if (userRole !== "admin") {
    throw new Error(
      "Unauthorized access: User does not have admin privileges."
    );
  }

  // 권한이 있는 사용자에게 작업을 진행합니다.
  // ... 작업 구현
}
```

### Protecting Route Handlers

- 인가된 사용자만 접근할 수 있도록 보안

```tsx
// app/api/route.ts
export async function GET() {
  // 사용자 인증 및 역할 검증
  const session = await getSession();

  // 사용자가 인증되었는지 확인합니다.
  if (!session) {
    return new Response(null, { status: 401 }); // 사용자가 인증되지 않음
  }

  // 사용자가 'admin' 역할을 가지고 있는지 확인합니다.
  if (session.user.role !== "admin") {
    return new Response(null, { status: 403 }); // 사용자는 인증되었지만 적절한 권한이 없음
  }

  // 권한이 있는 사용자를 위한 데이터 가져오기
}
```

### Authorization과 서버 컴포넌트

- 민감한 작업에 대한 보안을 강화
  - Server Components에서 자주 사용되는 관행 중 하나는 사용자 역할에 따라 UI 요소를 조건부로 렌더링하는 것

```tsx
// app/dashboard/page.tsx

export default async function Dashboard() {
  const session = await getSession();
  const userRole = session?.user?.role; // session 객체에 'role'이 있다고 가정

  if (userRole === "admin") {
    return <AdminDashboard />; // 관리자용 컴포넌트
  } else if (userRole === "user") {
    return <UserDashboard />; // 일반 사용자용 컴포넌트
  } else {
    return <AccessDenied />; // 권한이 없는 경우 보여줄 컴포넌트
  }
}
```

## Session Management

세션 관리는 사용자가 애플리케이션과 상호작용하는 동안 사용자의 인증된 상태가 애플리케이션 전반에 걸쳐 유지되도록 추적하고 관리하는 것

- 쿠키 기반 세션:

  - 많은 웹 애플리케이션에서 사용되는 방법으로, 세션 식별자를 쿠키에 저장하여 클라이언트 측에서 관리합니다. 클라이언트가 서버에 요청을 보낼 때마다 세션 식별자가 함께 전송되어 사용자를 인증하고 상태를 유지합니다. 이 방법은 구현이 비교적 간단하고 효율적이지만, 쿠키가 노출될 경우 보안 위험이 있을 수 있습니다.

- 데이터베이스 세션:
  - 세션 데이터를 데이터베이스에 저장하고 관리하는 방법입니다. 클라이언트는 세션 식별자를 쿠키에 저장하고, 서버는 데이터베이스에서 해당 세션에 대한 정보를 조회하여 사용자를 인증하고 상태를 유지합니다. 이 방법은 보안성이 높고, 세션 데이터를 더 세밀하게 관리할 수 있으며, 여러 서버 간의 세션 공유도 용이합니다.

### 쿠기 기반 세션

쿠키 기반 세션은 사용자 데이터를 브라우저 쿠키에 암호화된 세션 정보로 저장하여 관리합니다. 사용자가 로그인하면 이 암호화된 데이터가 쿠키에 저장되며, 이후 모든 서버 요청에 이 쿠키가 포함되어 반복적인 서버 쿼리를 최소화하고 클라이언트 측 효율성을 향상시킵니다.

그러나 쿠키는 클라이언트 측 보안 위험에 노출될 수 있기 때문에 민감한 데이터를 보호하기 위해 신중한 암호화가 필요합니다. 쿠키에 저장된 세션 데이터를 암호화하면 쿠키가 탈취되어도 내부 데이터를 보호할 수 있습니다.

또한, 개별 쿠키의 크기는 일반적으로 4KB로 제한되지만, 쿠키 청크(분할) 기법을 사용하여 큰 세션 데이터를 여러 쿠키로 분할할 수 있습니다.

- 서버 액션

```tsx
// app/actions.ts

"use server";

import { cookies } from "next/headers";
import { encrypt } from "@/security";

export async function handleLogin(sessionData) {
  const encryptedSessionData = encrypt(sessionData); // 세션 데이터 암호화

  cookies().set("session", encryptedSessionData, {
    httpOnly: true, // JavaScript를 통해 쿠키 접근 불가
    secure: process.env.NODE_ENV === "production", // 프로덕션 환경에서만 HTTPS 사용
    maxAge: 60 * 60 * 24 * 7, // 1주일
    path: "/", // 전체 사이트에서 접근 가능
  });

  // 쿠키 설정 후 리다이렉트 또는 응답 처리
}
```

- 서버 컴포넌트에서 쿠키에 저장된 세션 데이터 접근하기

```tsx
// app/page.tsx

import { cookies } from "next/headers";

export async function getSessionData(req) {
  const encryptedSessionData = cookies().get("session")?.value;
  return encryptedSessionData
    ? JSON.parse(decrypt(encryptedSessionData))
    : null;
}
```

### 데이터베이스 세션 Database Sessions

데이터베이스 기반 세션 관리는 세션 데이터를 서버에 저장하고, 사용자의 브라우저는 세션 ID만을 받는 방식입니다. 이 세션 ID는 서버 측에 저장된 세션 데이터를 참조하며, 실제 데이터는 포함되지 않습니다. 이 방법은 민감한 세션 데이터를 클라이언트 측 환경에서 멀리 유지하여 클라이언트 측 공격에 노출될 위험을 줄입니다.

그러나 각 사용자 상호 작용마다 데이터베이스 조회가 필요하므로 성능 오버헤드가 증가할 수 있습니다. (세션 데이터 캐싱과 같은 전략을 통해 완화 가능) 또한 데이터베이스에 의존하는 만큼, 세션 관리는 데이터베이스의 성능과 가용성에 따라 신뢰성이 결정됩니다.

- 서버 액션

```tsx
// app/actions.ts

import db from "./lib/db";

export async function createSession(user) {
  const sessionId = generateSessionId(); // 고유한 세션 ID 생성
  await db.insertSession({ sessionId, userId: user.id, createdAt: new Date() });
  return sessionId;
}
```

- 미들웨어 또는 서버 측 로직에서 세션 검색

```tsx
// app/middleware.ts

import { cookies } from "next/headers";
import db from "./lib/db";

export async function getSession() {
  const sessionId = cookies().get("sessionId")?.value;
  return sessionId ? await db.findSession(sessionId) : null;
}
```

### 세션 관리 방식 선택

Next.js에서 쿠키 기반과 데이터베이스 기반 세션 중 선택하는 것은 애플리케이션의 요구 사항에 따라 달라집니다. 쿠키 기반 세션은 간단하고 서버 부하가 적은 작은 애플리케이션에 적합하지만 보안성이 낮을 수 있습니다. 반면, 데이터베이스 기반 세션은 보다 높은 보안성과 확장성을 제공하여 대규모 데이터 중심 애플리케이션에 적합합니다.

NextAuth.js와 같은 인증 솔루션을 사용하면 세션 관리가 간편해지며, 쿠키나 데이터베이스 저장을 선택할 수 있습니다. 개발 프로세스를 단순화할 수 있지만, 선택한 솔루션의 세션 관리 방법을 이해하고 애플리케이션의 보안 및 성능 요구 사항과 일치시키는 것이 중요합니다.

어떤 방법을 선택하든 세션 관리 전략에서 보안!을 우선시해야 합니다. 쿠키 기반 세션의 경우 안전하고 HTTP-Only 쿠키 사용이 중요하며, 데이터베이스 기반 세션의 경우 정기적인 백업과 세션 데이터의 안전한 처리가 필수입니다. 또한, 잘못된 접근을 방지하고 애플리케이션 성능과 신뢰성을 유지하기 위해 세션 만료 및 정리 메커니즘을 구현하는 것이 중요합니다.

---

## 페이지와 레이아웃 Page & Layout

> https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts

### 페이지 (Pages)

**페이지**는 각 경로(route)에서 보여지는 고유한 UI를 정의합니다. 일반적으로 URL 경로에 대응되는 페이지 컴포넌트를 생성합니다.

- **파일 위치 및 형식**: 페이지는 `page.js`, `.jsx`, 또는 `.tsx` 파일에서 정의됩니다.
- **퍼블릭 접근을 위한 page.js 파일**: 예를 들어, `/` 경로에 대한 페이지를 만들기 위해 `app/page.tsx` 파일을 생성합니다.
- **경로 트리의 마지막 노드**: /dashboard/settings 경로에서 settings가 페이지 컴포넌트가 됩니다.
- **서버 컴포넌트**: 기본적으로 서버 측 컴포넌트입니다. 'use client'; 지시자로 클라이언트 측 컴포넌트로 설정할 수도 있습니다.
- **페이지에서 데이터 패치**: Next.js에서 페이지는 렌더링 과정에서 데이터를 가져올 수 있습니다. 이 기능을 통해 페이지는 렌더링 전에 데이터를 사전에 가져오거나 사용자 상호작용에 따라 동적으로 데이터를 가져올 수 있습니다.

```tsx
// `app/page.tsx` is the UI for the `/` URL
export default function Page() {
  return <h1>Hello, Home page!</h1>;
}

// `app/dashboard/page.tsx` is the UI for the `/dashboard` URL
export default function Page() {
  return <h1>Hello, Dashboard Page!</h1>;
}
```

### 레이아웃 (Layouts)

**레이아웃**은 여러 경로에서 공유되는 UI 구조를 정의합니다. 모든 경로에 동일하게 적용되며, 네비게이션 시에도 상태를 유지하고 재렌더링되지 않습니다. 레이아웃 또한 중첩될 수 있습니다.

레이아웃은 기본적으로 layout.js에서 렌더링 중에 자식 레이아웃(있을 경우) 또는 페이지로 채워질 children prop을 받아야 합니다.

- **파일 위치 및 형식**: 레이아웃은 `layout.js`, `.jsx`, 또는 `.tsx` 파일에서 정의됩니다.
- **공유 UI**: 여러 페이지에서 사용하는 공통 UI 요소를 포함할 수 있습니다. 예를 들어, 네비게이션 메뉴나 헤더 등을 정의할 수 있습니다.
- **중첩 가능**: 다른 레이아웃 내에 중첩하여 사용할 수 있습니다. 예를 들어, `/dashboard` 경로에 대한 레이아웃 내에 `/dashboard/settings` 경로에 대한 레이아웃을 중첩할 수 있습니다.
- **서버 컴포넌트**: 기본적으로 서버 측 컴포넌트입니다. 클라이언트 측 컴포넌트로 설정할 수도 있습니다.
- **데이터 페치**: 레이아웃에서도 데이터를 가져올 수 있습니다.

#### 루트 레이아웃 Root Layout

루트 레이아웃은 애플리케이션 디렉토리의 최상위에 정의되며 모든 경로에 적용됩니다. 이 레이아웃은 필수적이며 서버에서 반환되는 초기 HTML을 수정할 수 있도록 <html>과 <body> 태그를 포함해야 합니다.

```tsx
// app/layout.tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        {/* 레이아웃 UI */}
        <main>{children}</main>
      </body>
    </html>
  );
}
```

#### 레이아웃 중첩

기본적으로 폴더 계층 구조 내에서 레이아웃은 중첩될 수 있습니다. 이는 자식 레이아웃을 children prop을 통해 감싸는 방식으로 동작합니다. 특정 경로 세그먼트(폴더) 내에 layout.js를 추가하여 레이아웃을 중첩할 수 있습니다.

### 레이아웃 중요 사항

- 레이아웃은 기본적으로 서버 컴포넌트이지만 클라이언트 컴포넌트로 설정할 수 있습니다.
- 레이아웃에서 데이터를 패칭할 수 있습니다.
- 부모 레이아웃과 자식 간에 데이터를 직접 전달하는 것은 불가능합니다. 하지만 동일한 데이터를 경로에서 여러 번 가져올 경우 React가 자동으로 중복 요청을 처리하므로 성능에 영향을 주지 않습니다.
- 레이아웃은 자신 아래의 경로 세그먼트에 직접 접근할 수 없습니다. 모든 경로 세그먼트에 접근하려면 클라이언트 컴포넌트에서 useSelectedLayoutSegment 또는 useSelectedLayoutSegments를 사용해야 합니다.
- 경로 그룹을 사용하여 특정 경로 세그먼트를 공유된 레이아웃에 포함하거나 제외시킬 수 있습니다.
- 경로 그룹을 사용하여 여러 개의 루트 레이아웃을 생성할 수 있습니다. 예시는 여기에서 확인할 수 있습니다.
- 페이지 디렉토리에서 마이그레이션할 때, 루트 레이아웃은 \_app.js와 \_document.js 파일을 대체합니다.

### 템플릿 (Templates)

템플릿은 각 자식 레이아웃 또는 페이지를 감싸는 점에서 레이아웃과 유사하지만, 다음과 같은 차이가 있습니다!

- 새로운 인스턴스 생성: 템플릿은 각 자식 요소마다 새로운 인스턴스를 생성합니다. 이는 사용자가 템플릿을 공유하는 경로 간에 이동할 때마다 컴포넌트의 새 인스턴스가 마운트되고, DOM 요소가 재생성되며, 상태가 유지되지 않고 효과가 다시 동기화됨을 의미합니다.

- 사용 예: 페이지 조회 로깅과 같은 useEffect나 페이지마다 별도의 피드백 폼을 관리하는 useState에 유용합니다.

- 기본 프레임워크 동작 변경: 레이아웃 안의 Suspense 경계가 처음으로 로드될 때만 대체 콘텐츠를 표시하지만, 템플릿의 경우 각 탐색마다 대체 콘텐츠가 표시됩니다. 이는 기본 프레임워크 동작을 변경할 수 있는 또 다른 이유입니다.

템플릿은 template.js 파일에서 React 컴포넌트를 내보내서 정의할 수 있습니다. children prop을 받아야 합니다.

```tsx
export default function Template({ children }: { children: React.ReactNode }) {
  return <div>{children}</div>;
}
```

- 중첩 관점에서 봤을 때: 레이아웃과 children 사이에 위치합니다.

```tsx
<Layout>
  {/* Note that the template is given a unique key. */}
  <Template key={routeParam}>{children}</Template>
</Layout>
```
