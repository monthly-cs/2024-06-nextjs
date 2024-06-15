## 2주차 발표자료입니다.

강의를 통해 새롭게 알게된 사항, 궁금함을 채우기 위해 공식 문서를 살펴본 내용, 실제로 적용한 사례를 공유합니다.

---

### 병렬 라우팅 (Parallel Routes) ✨

병렬 라우팅은 서로 다른 경로를 동일한 페이지 내에서 동시에 표시할 수 있게 합니다. 이를 위해 페이지 폴더 내에 `@페이지명` 폴더를 만들어 구성합니다. 병렬 라우트를 사용하면 다음과 같이 구현할 수 있습니다:

#### 병렬 라우팅 레이아웃

```
baking
├── @news/
└── @recipes/
```

```tsx
export default function BakingPage({ news, recipes }) {
  // 두 개의 페이지를 동시에 표시
  return (
    <>
      <section>{news}</section>
      <section>{recipes}</section>
    </>
  );
}
```

### 병렬 및 중첩 라우팅

병렬 라우트와 중첩 라우트를 함께 사용할 수 있습니다. 동일한 페이지 내의 병렬 라우트는 동일한 경로를 가져야 합니다.

#### 예시: /baking/recipeId

- news 페이지: default.js
- recipes 페이지는 [recipeId] 폴더 내의 page.js

```
baking
├── @news/
│   └── default.js // 특정 경로의 콘텐츠가 로드되지 않았을 때
└── @recipes/
    └── [recipeId]/
    └── page.js
```

❗️ 병렬 라우팅을 활용하면 동일한 페이지 내에서 다양한 경로의 콘텐츠를 동시에 표시할 수 있어, 복잡한 네비게이션 구조를 보다 쉽게 관리할 수 있습니다.

---

### Catch-All Route

Catch-All 라우트는 경로 하위의 모든 페이지를 "Catch"할 수 있도록 설정하는 라우트입니다. Next.js에서는 [...slug] 또는 [[...slug]] 형식을 사용하여 구현할 수 있습니다. 예를 들어, `@archive/[[...filter]]` 경로를 사용하면 params.filter를 통해 해당 경로와 일치하는 모든 경로 세그먼트의 배열을 가져올 수 있습니다.

```tsx
// app/archive/[[...filter]]/page.js
export default function ArchivePage({ params }) {
  const { filter } = params;
  return (
    <div>Filters: {filter ? filter.join(", ") : "No filters applied"}</div>
  );
}
// 경로: /baking/recipes/muffin/chocolate
// params.filter: ["recipes", "muffin", "chocolate"]
```

### 에러 Throw

Next.js에서는 페이지에서 에러 핸들링이 필요할 때 에러를 throw하여 error.js를 사용할 수 있습니다. error.js는 클라이언트 컴포넌트로, 클라이언트와 브라우저 오류를 처리할 수 있습니다.

```tsx
// app/page.js
export default function Page() {
  if (someCondition) {
    throw new Error("Something went wrong!");
  }
  return <div>Page Content</div>;
}
```

```tsx
// app/error.js
import { useError } from "next/error";

export default function ErrorComponent() {
  const error = useError();
  return <div>Error: {error.message}</div>;
}
```

---

### Intercepting Route 네비게이션 가로채기

> https://nextjs.org/docs/app/building-your-application/routing/intercepting-routes

Intercepting Route를 사용하면 특정 경로로의 네비게이션을 가로채서 다른 처리를 할 수 있습니다.

```tsx
/(.)image // 유저가 애플리케이션에 네비게이션 했을 때 가로채는 라우트
/image // url로 접근하거나, 새로고침
/(..)image // 상위 라우트
```

레이어 팝업 구현에서 "복사된 링크로 접근하는 경우"에는 풀페이지 처리를 하는 경우가 있는데, 이럴 때 detail을 별개의 parameter로 구현하거나, querystring으로 popup=true 같이 처리할 수 있습니다.

Next.js의 Intercepting Route를 사용하면 모달, 전체 페이지를 함께 보여주는 페이지 구현을 세련되게 처리할 수 있습니다

### 인터셉트와 병렬 라우트와 모달

```
/news
  /@modal
    /(.)image // news/image를 인터셉트!
```

병렬 라우트를 정의할 때 디렉토리 구조는 URL에 영향을 주지 않습니다. 디렉토리 구조는 단순히 파일을 Next.js가 사용할 수 있도록 정리하는 용도입니다. 병렬 라우트를 사용하여 동일 경로에서 여러 컴포넌트를 동시에 렌더링할 수 있습니다.

#### 방법 1: 레이아웃에서 모달과 페이지를 출력

레이아웃을 정의하고, 페이지와 모달을 매개변수로 받아서 출력하는 방법입니다.

```tsx
// app/news/page.js
export default function NewsPage() {
  return <div>News Content</div>;
}

// app/news/@modal/(.)image/page.js
export default function ImageModal() {
  return <div>Image Modal Content</div>;
}

// app/news/layout.js
export default function NewsLayout({ children, modal }) {
  return (
    <>
      {modal}
      {children} // 경로의 page를 children으로 내보내는 레이아웃
    </>
  );
}
```

#### 방법 2: 병렬 라우트로 @modal과 @page 사용

병렬 라우트를 사용하여 같은 경로에서 모달과 페이지를 동시에 나타내는 방법입니다.

```tsx
// app/news/@content/page.js
export default function NewsPage() {
  return <div>News Content</div>;
}

// app/news/@modal/(.)image/page.js
export default function ImageModal() {
  return <div>Image Modal Content</div>;
}

export default function BakingPage({ news, content }) {
  // 두 개의 페이지를 동시에 표시
  return (
    <>
      <section>{news}</section>
      <section>{recipes}</section> // 병렬 라우트를 내보냄
    </>
  );
}
```

#### 클라이언트 에러 처리

위의 예시 코드에서 /@modal/(.)image에서 URL이나 새로고침으로 접근할 경우, 병렬 라우트에 보여줄 페이지가 없으면 클라이언트 에러가 발생할 수 있습니다.

이러한 상황을 처리하기 위해 default.js 또는 page.js를 사용하여 빈 화면을 반환하도록 할 수 있습니다.

```
/news
  /@modal
    /(.)image // news/image를 인터셉트!
    default.js // 인터셉트하지 않았을 때 return null
```

이렇게 하면, /news/image 경로로 URL을 직접 입력하거나 새로고침할 때, 인터셉트되지 않으면 빈 화면을 표시하게 됩니다.

---

### (라우트 그룹)

같은 관심사의 컨텐츠 묶어주는 역할만 하는 것 아니고 layout과 not-found를 분리할 분리 등 유용한 기능을 합니다. "라우트"와 관련된 그룹입니다.

```

app
├── (content)
│ ├── not-found.js
│ └── layout.js
└── (marketing)
└── layout.js

```

### 라우트 핸들러: route.js

> https://nextjs.org/docs/app/building-your-application/routing/route-handlers

라우트 핸들러는 app 디렉토리 내의 route.js 또는 route.ts 파일에 정의됩니다.

HTTP 메서드: GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS을 지원합니다. (지원되지 않는 메서드가 호출되면, 405 Method Not Allowed 반환)

```tsx
export async function GET(request: Request) {}
```

- 캐싱: GET 메서드 사용 시 기본적으로 캐싱됩니다. 캐싱을 비활성화하려면, Request 객체를 사용하거나 다른 HTTP 메서드를 사용합니다.
- 동적 함수: 쿠키, 헤더 등의 동적 함수 사용이 가능합니다.
- 쿠키 및 헤더: next/headers 모듈을 사용하여 쿠키 및 헤더를 읽거나 설정할 수 있습니다.
- revalidate: next.revalidate 옵션을 사용해 캐시된 데이터를 주기적으로 재검증할 수 있습니다.
- 리디렉션: redirect 함수를 사용해 다른 URL로 리디렉션할 수 있습니다.
- 동적 경로: 동적 경로를 사용하여 URL 매개변수를 처리할 수 있습니다.
- CORS 설정: Access-Control-Allow-Origin 등의 CORS 헤더를 설정할 수 있습니다.
- Webhook 처리: 웹훅을 처리할 수 있는 핸들러를 작성할 수 있습니다.

#### GET 요청 핸들러

```tsx
export async function GET(request: Request) {
  const data = await fetchData();
  return Response.json(data);
}
```

#### 쿠키 읽기

```tsx
import { cookies } from "next/headers";

export async function GET(request: Request) {
  const cookieStore = cookies();
  const token = cookieStore.get("token");
  return new Response(`Token: ${token?.value}`);
}
```

#### CORS 헤더 설정

```tsx
export async function GET(request: Request) {
  return new Response("Hello, Next.js!", {
    status: 200,
    headers: {
      "Access-Control-Allow-Origin": "*",
      "Access-Control-Allow-Methods": "GET, POST, PUT, DELETE, OPTIONS",
      "Access-Control-Allow-Headers": "Content-Type, Authorization",
    },
  });
}
```

### 미들웨어

> https://nextjs.org/docs/app/building-your-application/routing/middleware

미들웨어는 앱 전체의 request를 가로채 리디렉션, 블락 등을 처리합니다.
매쳐를 사용해서 매칭되는 요청만 처리할 수도 있습니다.

```tsx
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

// This function can be marked `async` if using `await` inside
export function middleware(request: NextRequest) {
  return NextResponse.redirect(new URL("/home", request.url));
}

// See "Matching Paths" below to learn more
export const config = {
  // 예약어 export
  matcher: "/about/:path*",
};
```

### 서버 컴포넌트와 async 컴포넌트

#### 데이터 로딩 상태 처리 불필요

서버 컴포넌트와 async 컴포넌트를 사용하면 데이터를 받아올 때까지 undefined 상태가 없으며, 별도의 상태 관리를 할 필요가 없습니다.

👉 코드의 복잡성을 줄이고 단순하게 유지할 수 있도록 도와줍니다.  
👉 클라이언트 컴포넌트에서는 로딩 상태를 관리하기 위해 useState와 useEffect를 사용해야 하지만, 서버 컴포넌트에서는 필요하지 않습니다!

#### 서버 측 코드 실행 지원

Node.js는 서버 측 코드를 실행할 수 있는 환경을 제공하며, Next.js는 이를 활용하여 서버 컴포넌트를 효율적으로 실행할 수 있도록 지원합니다.
서버 컴포넌트는 서버에서 데이터를 가져오고 처리한 후, 클라이언트에 전달합니다.

#### Next.js의 fetch 확장

Next.js는 fetch를 확장하여 기본적으로 캐싱을 지원합니다. 반복적인 데이터 요청을 줄이고, 응답 시간을 단축할 수 있습니다. (동일한 요청을 여러 번 수행하는 경우, Next.js는 이전 응답을 캐싱하여 불필요한 네트워크 요청을 방지합니다.)

### 로딩 상태

- sql의 경우처럼, DB요청을 동기적으로 처리하는 경우: 로딩 상태x
- API가 프로젝트 외부에 있는 경우: 로딩 중 fallback
  - loading.js: 예약된 이름의 폴백 컴포넌트, 형제나 자식 페이지에 적용됩니다. 컴포넌트의 promise가 resolve되기를 기다릴 때 적용
  - 또는 Suspense 컴포넌트 사용

---

### formAction

- HTML에서는? submit 시 양식에 url에 자동으로 제출됨

```tsx
<form action="submit-url" method="post">
  ...
</form>
```

- 리액트는? FormData를 매개변수로 받는 함수를 입력 (Canary, 클라이언트 사이드 기능 => 조만간 서버 사이드 기능이 될 예정)

![image](https://github.com/dusunax/javascript/assets/94776135/52bcafa0-120f-4c64-b6ea-7dd17ae8d966)

```tsx
<form action={creatPost}>...</form>
```

### useFormStatus, useFormState

- useFormState의 반환 구조분해
  - 두 번째 요소: 업데이트된 action 함수
  - 첫 번째 요소: state

### 버튼 action

- 버튼에 fromAction 속성에 액션 입력
- form으로 감싸서 action에 액션 입력

### 좋아요 액션

```
toggleLike.bind(null, postId)(someArgument)
// toggleLike 함수를 postId와 함께 호출
// bind 메서드를 사용하여 함수를 바인딩하면, 원본 함수의 첫 번째 매개변수부터 차례대로 바인딩
// 실제 호출 시에 toggleLike 함수가 호출될 때 postId가 첫 번째 매개변수로 전달
// 다른 인수가 있다면 두 번째부터!
```

### 이미지 스토리지 솔루션

- uploadthing
- https://cloudinary.com/
- S3가 가장 저렴하다!

### 데이터 재검증

```tsx
export async function toggleLike(postId) {
  await updatePostLike(postId, userId);
  revaildatePath("/", "layout"); // 전체
  // revaildatePath("/feed"); feed만
}
```

### useOptimistic (Canary)

- optimistic update pattern => 낙관적으로 토글만 하지 말고 롤백도 생각해야 함!

### 프로덕션 환경의 캐싱

- next.js는 프로덕션 환경에서 사전 랜더링과 전체 페이지 캐싱을 통해 성능 최적화를 진행하기 때문에, 빠른 속도로 콘텐츠를 제공할 수 있지만 => 데이터의 실시간 업데이트나 변경 사항을 반영해야 할 때 재검증을 해야하는 점이 있습니다.
- 다음 주에 캐싱에 대해서 계속 진행!
