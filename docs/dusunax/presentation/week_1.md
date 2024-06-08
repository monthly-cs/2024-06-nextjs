## 1주차 발표자료입니다.

강의를 통해 새롭게 알게된 사항, 궁금함을 채우기 위해 공식 문서를 살펴본 내용, 실제로 적용한 사례를 공유합니다.

# app router 살펴보기

- Next.js는 파일 기반 라우팅입니다.
- 서버 컴포넌트는 랜더되고 HTML로 변환되어 브라우저에 전송됩니다.
- 페이지에 처음 방문하면 백엔드에서 페이지를 랜더합니다. 이후 페이지는 클라이언트 자바스크립트 코드로 업데이트 된다.

## layout

> https://nextjs.org/docs/app/api-reference/file-conventions/layout

- `/app`의 root layout이 필요합니다.
- `<html>`와 `<body>`를 필수로 가져야 합니다.
- header가 없는 이유는? 해당 layoutd의 metadata 객체를 사용해서 Next.js가 `<head>`를 만들기 때문입니다.

## 레이아웃

레이아웃은 웹 페이지의 포장지와 같이 생각할 수 있습니다. 이는 중첩 구조로 이루어집니다.

즉, root 레이아웃이 항상 최상위에 있으며, 그 아래에 중첩 페이지의 레이아웃이 위치합니다. root 레이아웃은 항상 활성화되어 있으므로, 프로젝트 전체에 적용할 코드는 여기 넣으면 됩니다.

## 예약된 이름의 파일

- 알아두는 것이 좋을 것 같아 정리했습니다

| 파일                 | 내용                                                                                                                                                                                                      |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| default.js           | Next.js가 전체 페이지 로드 후 슬롯의 활성 상태를 복구할 수 없을 때 병렬 라우트 내에서 폴백을 렌더링하는 데 사용됨                                                                                         |
| error.js             | 경로 세그먼트에 대한 오류 UI 경계를 정의함. 서버 컴포넌트와 클라이언트 컴포넌트에서 발생하는 예상치 못한 오류를 잡아내고 폴백 UI를 표시함                                                                 |
| instrumentation.js   | 애플리케이션에 모니터링 및 로깅 도구를 통합함. 애플리케이션의 성능과 동작을 추적하고, 프로덕션에서 문제를 디버그함                                                                                        |
| layout.js            | 경로 간에 공유되는 UI를 정의함. 루트 레이아웃은 루트 앱 디렉터리에서 최상위 레이아웃으로, 응용 프로그램의 다른 부분에서 공유되는 UI를 정의함                                                              |
| loading.js           | Suspense를 기반으로 즉각적인 로딩 상태를 생성함. 기본적으로 이 파일은 서버 컴포넌트이지만, "use client" 지시문을 통해 클라이언트 컴포넌트로도 사용할 수 있음                                              |
| middleware.js        | Middleware를 작성하고 요청이 완료되기 전에 서버에서 코드를 실행함. 들어오는 요청에 따라 응답을 재작성, 리디렉션, 요청 또는 응답 헤더 수정, 또는 직접 응답을 수정할 수 있음                                |
| not-found.js         | 경로 세그먼트 내에서 `notFound` 함수가 호출될 때 UI를 렌더링함. 맞춤 UI를 제공함과 동시에, Next.js는 스트리밍 응답에 대해 200 HTTP 상태 코드를 반환하고 비스트리밍 응답에 대해서는 404 상태 코드를 반환함 |
| page.js              | 경로에 고유한 UI를 정의한다. `params`와 `searchParams` props를 포함할 수 있음                                                                                                                             |
| route.js             | Web Request 및 Response API를 사용하여 주어진 경로에 대한 사용자 정의 요청 핸들러를 만들 수 있음                                                                                                          |
| Route Segment Config | 페이지, 레이아웃 또는 라우트 핸들러의 동작을 구성할 수 있는 옵션을 제공함                                                                                                                                 |
| template.js          | 템플릿 파일은 각 자식 레이아웃이나 페이지를 래핑 => 레이아웃이 경로 간에 상태를 유지하는 것과 달리, 템플릿은 내비게이션 시 각 자식에 대한 새로운 인스턴스를 생성한다                                      |
| Metadata Files       | 메타데이터 파일 `favicon`, `icon`, `apple-icon`, `manifest.json`, `robots.txt`, `opengraph-image`, `sitemap.xml`                                                                                          |

### error

> https://nextjs.org/docs/app/api-reference/file-conventions/error

error.js는 브라우저의 오류도 캐치할 수 있어야하기 때문에 `클라이언트 컴포넌트`여야 합니다.
예상치 못한 오류를 캐치할 수 있습니다. Error 객체와 reset 메소드를 매개변수로 가집니다.

- 에러 바운더리 컴포넌트 사용: React에서는 ErrorBoundary 컴포넌트를 사용하여 에러를 처리할 수 있습니다. 이를 사용하면 애플리케이션의 다른 부분에 영향을 미치지 않고 컴포넌트 트리에서 에러를 잡을 수 있습니다. 주로 API 요청과 같은 비동기 작업에서 발생하는 에러를 처리하는 데 사용됩니다.
- 각 페이지 또는 기능에 대한 오류 처리: 각 페이지나 기능에 따라 별도의 에러 처리 로직을 구현할 수 있습니다. 예를 들어, 로그인 페이지에서 발생하는 오류와 회원가입 페이지에서 발생하는 오류를 각각 다르게 처리할 수 있습니다. 이를 통해 사용자 경험을 개선하고 사용자에게 더욱 친숙한 메시지를 제공할 수 있습니다.

## route

동적 경로와 정적 경로가 겹칠 때, 정적 경로는 순서에 상관없이 항상 우선적으로 방문됩니다. 예를 들어, `user/[...slug]`을 `user/info` 위에 작성해도 `user/info`로 이동하는 데 문제가 발생하지 않습니다. 이는 파일 기반 라우팅의 특성 때문입니다. 파일 시스템에서 폴더의 위치를 바꿀 수 없기 때문에, 정적 경로가 동적 경로보다 우선적으로 처리됩니다.

## 이미지 import

다음은 이미지를 동적으로 로드하고 Next.js의 blurDataURL 속성을 활용하여 최적화된 이미지를 렌더링하는 방법에 대해 설명하겠습니다.

```tsx
import burger from "path/to/your/image.jpg"; // 정적 이미지 import
```

Next.js 프로젝트에서 이미지를 import한 후 결과를 출력해보면 다음과 같은 객체 형태로 나옵니다.

![image](https://github.com/dusunax/casecase/assets/94776135/24a3486f-fce7-48ac-ab76-2ac74d5d4ee1)

- src
- height
- width
- blurDataURL
- blurWidth
- blurHeight

Image 컴포넌트를 사용해 이미지를 최적화하기 위한 속성을 자동으로 생성합니다.

Next.js의 최적화 기능을 사용하면, blurDataURL 속성에 입력할 Base64 이미지를 수동으로 생성할 필요 없이 객체의 blurDataURL 속성을 활용할 수 있습니다.

```tsx
// img 태그인 경우
<img src={burger.src}> // image src에는 .src를 입력해야 합니다.

// Image 컴포넌트인 경우
// X 오류
<Image src={burger.src} width={400} height={400} placeholder={"blur"} />

// O 작동
<Image
  src={burger.src}
  width={400}
  height={400}
  placeholder={"blur"}
  blurDataURL={burger.blurDataURL}
/>

// O 너무 잘 작동
<Image src={burger} width={400} height={400} placeholder={"blur"} />
```

## CSS module

Next.js는 CSS Module을 지원하므로 이를 잘 활용하면 컴포넌트 모듈화를 효과적으로 할 수 있습니다.

- 모듈: 스타일 중복과 공통 스타일 관리, 클래스명을 로컬 스코프로 제한 => 충돌할 위험이 없다.
- 유틸리티 클래스: 클래스명 길어지는 현상. `@apply`로 줄이려면 또 스타일 관리 필요. 빠른 작업

### CSS Module vs. Tailwind CSS vs. Styled-components

- CSS Module: 스타일 중복과 공통 스타일 관리에 유용합니다. 클래스명을 로컬 스코프로 제한하여 충돌할 위험이 없습니다.
- 유틸리티 클래스: 빠르고 간편하게 스타일을 적용할 수 있습니다. 클래스명이 길어지는 현상이 있습니다.
- CSS-in-JS 라이브러리: JavaScript를 사용하여 스타일을 작성합니다. 컴포넌트와 스타일을 함께 관리할 수 있습니다.

## RSC (React Server Component) vs Client Component

> 104강 내용

### RSC (React Server Component)

서버에서만 렌더링되는 컴포넌트입니다. 리액트 내장 기능이지만 현재는 Next.js와 같은 프레임워크에서만 사용 가능합니다.

클라이언트에서 실행되는 자바스크립트 코드가 줄어들고, SEO에 이점이 있습니다. (검색 엔진이 소스 코드에서 완성된 컨텐츠를 볼 수 있어 검색 결과에 노출될 가능성이 높아집니다)

### Client Component

클라이언트에서만 작동 가능한 코드 기준의 기준은 무엇일까요?

#### 브라우저 동작

브라우저에서 실행되는 특정 동작이나 기능은 클라이언트에서만 작동 가능합니다. 예를 들어, 웹 페이지가 스크롤될 때 특정 애니메이션이 발생하는 경우 클라이언트에서만 작동할 수 있습니다.

#### 이벤트 핸들러 (eventHandler)

사용자의 상호작용에 반응하는 이벤트 핸들러도 클라이언트에서만 작동 가능합니다. 예를 들어, 버튼을 클릭하면 모달 창이 열리는 것과 같은 유저 상호작용의 경우 클라이언트에서만 작동할 수 있습니다.

## 프로토타입 구축을 위한 간편한 DB

프론트 엔드 개발자가 간편한 데이터베이스를 사용하면 프로토타입을 빠르게 구축할 수 있습니다.
이는 아이디어를 빠르게 시험하고 검증할 수 있어 개발 초기 단계에서 필요한 피드백을 빠르게 받을 수 있습니다.

프론트 엔드 개발자의 관점에서 보면 DB 개발 도중에 익숙하고 빠르게 사용할 수 있는 간편한 데이터베이스를 준비하면 개발 효율성을 높일 수 있습니다.
물론 MSW와 OpenAPI Generator와 같이 모킹 서버나 API 코드 생성을 위한 유용한 도구가 많기 때문에, 간편한 DB의 경우 토이프로젝트, 혹은 프로젝트 초기 단계에 사용할 수 있습니다.

표로 데이터베이스 옵션을 비교해 보겠습니다:

| 옵션           | 설치 및 설정                                                                                                                  | 장점                                                                                                            | 단점                                                         |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| better-sqlite3 | 1. `npm install better-sqlite3`로 패키지 설치 2. 데이터베이스 생성 및 테이블 정의                                             | 로컬에서 간편하게 SQLite 데이터베이스 사용 가능, 빠른 속도와 효율성 제공                                        | 네트워크 기반의 데이터베이스보다는 제한적인 기능 제공        |
| json-server    | 1. `npm install -g json-server`로 패키지 설치 2. JSON 파일 생성 및 데이터 정의 3. `json-server --watch db.json`으로 서버 실행 | 로컬에서 간단하게 RESTful API 구성 가능, JSON 형식으로 데이터 저장 및 검색 가능                                 | 복잡한 쿼리나 관계형 데이터베이스 기능 제공 X                |
| Firebase       | 1. Firebase 프로젝트 설정 2. Realtime Database 또는 Cloud Firestore 생성 및 구성                                              | 클라우드 기반의 실시간 데이터베이스 손쉽게 구축 및 관리 가능, 실시간 업데이트, 인증, 호스팅 등 다양한 기능 제공 | 클라우드 기반이므로 추가 비용 발생 가능 - 네트워크 연결 필요 |

이 표를 통해 각 데이터베이스 옵션의 장단점을 비교하고, 프로젝트에 가장 적합한 옵션을 선택할 수 있습니다. 선택하신 데이터베이스에 대한 더 자세한 설정 및 사용법에 대해 궁금한 점이 있다면 언제든지 물어보세요!

## sqlite

- initdb.js

```tsx
const sql = require("better-sqlite3");
const db = sql("meals.db");

db.prepare(
  `
  CREATE TABLE IF NOT EXISTS foos (
    ...
  ) 
`
).run();

async function initData() {
  const stmt = db.prepare(`
      INSERT INTO foos VALUES (
        ...
      )
  `);

  for (const foo of dummyFoos) {
    stmt.run(foo);
  }
}

initData();
```

- js

```tsx
import sql from "better-sqlite3";

const db = sql("foos.db");

export async function getMeals() {
  return db.prepare("SELECT * FROM foos");
}
```

## dataUrl은 base64 인코딩 형태의 이미지 소스이다.

| 구분    | 내용                                                                                            | 방법                                                                 | 사용                                                                    |
| ------- | ----------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| Base64  | 바이너리 데이터를 텍스트 형식으로 안전하게 전달하기 위해 인코딩하는 방법 (이진 데이터 ⇒ 문자열) | 8비트 바이너리 데이터를 6비트 단위로 나누어 64개의 ASCII 문자로 표현 | 이메일 전송, 데이터 URI, JSON 등 텍스트 기반 전송 매체                  |
| DataURL | URL 스킴 중 하나, 텍스트 형식의 데이터를 직접 포함할 수 있도록 함                               | data:로 시작하고, 미디어 타입과 Base64로 인코딩된 데이터를 포함      | 이미지, 폰트, 동영상 등 바이너리 데이터를 HTML이나 CSS에 포함할 때 사용 |

dataUrl는 이미지를 브라우저에 로드하는 방법 중 하나다. dataURL을 사용하면 이미지를 HTML이나 CSS 파일 내에 직접 포함시킬 수 있다. 이미지 데이터를 인코딩하여 사용 => 작은 아이콘이나 로고와 같은 이미지를 포함할 때 유용

- HTML 이미지 태그 & CSS background로 사용할 수 있습니다.
- JavaScript의 FileReader API로 파일을 읽고 파일의 내용을 다양한 형식으로 변환할 수 있습니다.

```jsx
const file = event.target.files[0];
if (file) {
  const reader = new FileReader(); // 인스턴스 생성

  reader.onload = function (e) {
    // 성공 시 실행할 함수
    const base64String = e.target.result;

    const imagePreview = document.getElementById("imagePreview");
    imagePreview.style.backgroundImage = `url(${base64String})`;
  };

  reader.readAsDataURL(file); // 읽기 시작
}
```

## 서버 액션 server action

함수를 서버에서만 사용된다고 함수 스코프 최상단에 선언하면, 서버에서 동작하는 서버 액션을 작성할 수 있습니다.

```jsx
async function postFoo(formData){
  'use server';

  const foo = {
    title: formData.get("title"),
    description: formData.get("description"),
  }
}

...
<form onSubmit={postFoo}> ... </form>
```

하지만 우리가 서버 액션을 별도의 파일에 보관하는 것을 권장하는 이유는 클라이언트 컴포넌트에서 사용하기 위한 목적과 보안상의 이유가 있습니다. 클라이언트 측 코드와 서버 측 코드를 한 파일에 작성하면 빌드 프로세스에서 완벽히 분리할 수 없으며, 이로 인해 보안 문제가 발생할 수 있습니다.

따라서 서버 액션을 별도의 파일에 보관하여 서버 측 코드와 클라이언트 측 코드를 분리하는 것이 좋습니다. 이렇게 함으로써 클라이언트 측 코드에 서버 측 로직이 노출되지 않습니다.

또한, 서버 액션을 별도의 파일에 보관함으로써 코드의 유지 보수성도 향상됩니다. 서버 측 코드와 클라이언트 측 코드가 분리되어 있으면, 각각의 코드를 독립적으로 관리하고 변경할 수 있으므로 코드의 이해와 수정이 쉬워집니다.

```tsx
'use server';

export default function getFoo(query){...}
export default function postFoo(formData){...}
export default function patchFoo(formData){...}
export default function deleteFoo(fooId){...}
```

## 사전 생성하지 않은 데이터를 보려면?

> https://nextjs.org/docs/app/api-reference/functions/revalidatePath

사전 생성된 데이터가 아닌 실시간으로 업데이트된 데이터를 보고 싶다면, 캐싱된 페이지를 갱신해야 합니다. 이를 위해 Next.js에서는 revalidate 메서드를 사용할 수 있습니다. 이 메서드는 페이지를 다시 렌더링하고 데이터를 업데이트하는 데 사용됩니다.

### 특정 데이터 갱신

특정 데이터를 다시 렌더링하여 새로운 데이터를 가져오려면 해당 페이지, URL, 레이아웃을 인수로 전달합니다.

```tsx
import { revalidatePath } from "next/cache";

revalidatePath("/blog/post-1"); // URL 갱신

revalidatePath("/blog/[slug]", "page"); // page 갱신
revalidatePath("/(main)/post/[slug]", "page");

revalidatePath("/blog/[slug]", "layout"); // 레이아웃 갱신
revalidatePath("/(main)/post/[slug]", "layout");
```

### 전체 경로 갱신

전체 데이터를 다시 업데이트하려면 빈 문자열을 인수로 전달합니다.

```tsx
import { revalidatePath } from "next/cache";
// 전체 페이지를 다시 렌더링하여 데이터를 업데이트합니다.
revalidatePath("/", "layout");
```

## metadata

### 설정 기반 메타데이터 Config-based Metadata

#### 정적 메타 데이터

- https://nextjs.org/docs/app/api-reference/functions/generate-metadata

런타임에 의존하지 않는 메타데이터라면? 정적 메타데이터를 사용합니다.

```tsx
// layout.js 또는 static page.js
// 예약어 export~
export const openGraphImage = { images: ["http://..."] };
export const metadata = {
  title: "Blog",
  description: "",
  openGraph: {
    ...openGraphImage,
    title: "Blog",
  },
};

// Output:
// <title>Blog</title>
// <meta property="og:title" content="Blog" />
```

#### 동적 메타 데이터

- https://nextjs.org/docs/app/building-your-application/optimizing/metadata#dynamic-metadata

generateMetadata 내에서 데이터 패칭이 완료될 때까지 기다린 후, 해당 데이터를 사용하여 페이지의 메타데이터를 생성합니다. 이를 통해 페이지의 `<head>` 태그에 필요한 메타데이터가 포함됩니다. 함수 내부에서 `notFound()`와 `redirect()` 사용해서 에러 핸들링 및 리디렉션할 수 있습니다.

```tsx
// layout.js 또는 static page.js
// 예약어 함수 export~
// 함수 내에서 동적으로 메타데이터를 패칭하는 함수
export async function generateMetadata({ params }) {
  const foo = getFoo(params.fooSlug); // 서버 액션 실행

  if (!foo) {
    notFound(); // foo가 존재하지 않는 경우. UX를 위해 에러가 아닌 404 처리
  }

  return {
    // return 해준다.
    title: foo.title,
    description: foo.summary,
  };
}
```

### 파일 기반 메타 데이터는? File-based metadata는?

- favicon.ico, apple-icon.jpg, and icon.jpg
- opengraph-image.jpg and twitter-image.jpg
- robots.txt
- sitemap.xml

### metadata: favicon 설정하기

> https://nextjs.org/docs/app/api-reference/file-conventions/metadata/app-icons#icon

asset에 icons 폴더를 만들어서 아이콘 사이즈별로 link 태그를 작성하지 않아도 nextJS가 처리할 수 있습니다.

`/app` root 폴더에 예약된 이름의 파일을 통해 favicon.ico와 icon들을 type과 size까지 head에 알아서 작성해줍니다. 굉장히 편합니다.

![](https://velog.velcdn.com/images/dusunax/post/000d93ec-47f3-460a-ac7e-27f62efab4c9/image.png)

다만 /app root에 있어야해서 다음과 같이 지저분해 보이는 단점이 있습니다.

![](https://velog.velcdn.com/images/dusunax/post/1b34d0e5-776f-4243-8358-609862fd4c6d/image.png)

만약
디바이스와 사이즈별로 더 세분화해서 아이콘을 사용해야 하는 경우 다른 방법을 사용하려 합니다.

![](https://velog.velcdn.com/images/dusunax/post/9e19bfc6-8994-47a8-9612-28b35cab6231/image.png)

---

## 감사합니다✨
