# 240609 week 1. NextJS 핵심 (앱 라우터)

### **App directory 살펴보기**

- NextJS 는 Filesystem-based Routing 을 따른다.
  - NextJS uses files & folders to define routes.
  - Only files & folders inside the "app" folder are considered.
- App 디렉토리 내부에 있는 page.js, layout.js, icon.png 등과 같은 예약어로 된 파일명을 잘 확인하자.
  - `page.js` => 신규 페이지 생성 (예: `app/about/page.js`은 `<your-domain>/about page`을 생성)
  - `layout.js` => 형제 및 중첩 페이지를 감싸는 신규 레이아웃 생성
  - `not-found.js` => ‘Not Found’ 오류에 대한 폴백 페이지(형제 또는 중첩 페이지 또는 레이아웃에서 전달된)
  - `error.js` => 기타 오류에 대한 폴백 페이지(형제 또는 중첩 페이지 또는 레이아웃에서 전달된)
  - `loading.js` => 형제 또는 중첩 페이지(또는 레이아웃)가 데이터를 가져오는 동안 표시되는 폴백 페이지
  - `route.js` => API 경로 생성(즉, JSX 코드가 아닌 데이터를 반환하는 페이지, 예: JSON 형식)

### **NextJS works with React Server Components**

- 기본적으로 NextJS 의 컴포넌트는 기본적으로 서버 컴포넌트이다. (상단에 'use client' 라 써주지 않으면)
- 이는 컴포넌트가 서버 사이드에서 실행된다는 것을 의미.

### NextJS 에서의 Routing

- 라우트로 취급되는 새로운 경로를 app 경로 내에 폴더를 만들어 경로를 추가할 수 있다.
- 그 폴더 안에 page.js 파일을 만들어 새로운 경로의 컴포넌트를 추가해 주어야 한다.
  - 경로 이미지 🛣️
  ![스크린샷 2024-06-09 오후 12.29.29.png](https://github.com/monthly-cs/2024-06-nextjs/assets/69143207/f98b396e-4f37-4284-9d66-1b6fcc7975ed)

### **RootLayout**

- page.js 는 사용자에게 제공되는 페이지
- layout.js 는 그 페이지를 감싸는 포장지 (최상위에 반드시 하나는 있어야 함)
- layout.js 는 개별 경로별로 page.js 와 같이 한 세트로 존재할 수 있음
- **RootLayout 은 html 의 뼈대, 스타일을 잡기 위한 구조**를 정할 수 있음
- RootLayout 에서 head 태그는 불필요. '**metadata**' 라는 **예약어를** 가진 상수에서 메타태그 등을 설정할 수 있음
- RootLayout 의 children : 현재 레이아웃 경로에서 활성상태인 page.js 를 의미

### 동적경로 (Dynamic Routing) 환경 설정 및 경로 매개변수 (params) 사용방법

- Square Bracket 을 사용해 특정 경로 내에 Dynamic Route 로 단 한번에 정의가 가능
- blog/page.js 에서 개별 블로그 게시글이 추가되었을 때 게시글 클릭시 경로 이동
- 다음과 같은 경로로 접근이 되게 하려면 => /blog/learning-nextjs
  - blog/[slug]/page.js 로 동적 경로 추가가 가능
- 동적 경로를 위한 페이지 컴포넌트가 받는 props
  - props : { params : { slug : 'page-1' }, searchParams : {} }
  - params 에는 **url 에 인코딩 된 key** 가 들어 있다.
- 같은 경로 내 하위 경로에 동적 경로와 정적 경로가 동시에 들어 있을 때?
  - **정적 경로를 동적 경로보다 우선시 한다**는 점 참고

### 레이아웃 개념 다시보기

- layout.js : page 의 포장지
- 최상단 RootLayout 외에 이론상 중첩된 레이아웃도 가능
- 특정 경로 페이지에 특화된 Layout 을 구성할 수 있음
  - 이 Layout 또한 RootLayout 의 children 으로 속해 들어간다는 점 참고

### 레이아웃에 커스텀 컴포넌트 추가

- **components** 폴더는 app 폴더 외부에 위치시키는 것을 추천
  - app 디렉토리가 routing 을 다루는 것에만 집중되게
- 일반 img 태그를 사용하는 경우, assets 디렉토리에서 import 해서 가져와서 사용
  - import 한 이미지 객체의 src 속성으로 접근해서 img 태그의 src 속성에 적용해야 함
  - 그러나 next/image 에서 불러오는 NextJS 빌트인 Image 컴포넌트를 사용

### NextJS 디자인 : 옵션 및 CSS 모듈 사용

- NextJS 프로젝트를 스타일링 할 때 사용할 수 있는 여러 가지 방법
  - 루트 상단의 globals.css
  - Tailwind CSS : NextJS 초기 설치시 설치할 것인지 물어봄
  - **CSS modules** : NextJS 에서 지원하는 방식 중 하나, 일반적인 스탠다드 CSS 코드이지만 **CSS 파일에 특정한 이름을 지정함으로서 특정 컴포넌트로 스코핑** 된다.
    - main-header.module.css ➡ 클래스명, 선택자를 이용한 스타일링 코드가 들어 있음
    - main-header.js ➡️ import 한 css 모듈 객체의 className 의 속성값으로 스타일링 적용
      ```jsx
      import classes from "./main-header.module.css";

      export default function MainHeader() {
        return (
          <header className={classes.header}>
            <Link className={classes.logo} href="/">
              <img src={logoImg.src} alt="A plate with food on it" />
              NextLevel Food
            </Link>

            <nav className={classes.nav}>
              <ul>
                <li>
                  <Link href="/meals">Browse Meals</Link>
                </li>
                <li>
                  <Link href="/community">Foodies Community</Link>
                </li>
              </ul>
            </nav>
          </header>
        );
      }
      ```

### NextJS 이미지 컴포넌트를 통한 이미지 최적화

- NextJS 에서 빌트인으로 제공하는 Image 컴포넌트를 사용
- 기본 img 태그를 쓰는 것 보다 더 최적화된 방식으로 이미지를 출력할 수 있게 도와준다.
- 기본 img 태그를 사용할 때와 달리 **import 한 이미지 객체를 src 속성에 그대로** 넣는다.

### NextJS 공식 문서의 Image Optimization 내용

- Next.js 이미지 컴포넌트는 자동 이미지 최적화를 위한 기능으로 HTML <img> 요소를 확장합니다:
- 크기 최적화: WebP 및 AVIF와 같은 최신 이미지 형식을 사용하여 각 디바이스에 적합한 크기의 이미지를 자동으로 제공합니다.
- 시각적 안정성: 이미지가 로드될 때 레이아웃이 자동으로 이동하는 것을 방지합니다.
- 더 빠른 페이지 로드: 기본 브라우저 지연 로딩을 사용하여 이미지가 뷰포트에 들어올 때만 로드되며, 블러업 플레이스홀더(선택 사항)를 사용할 수 있습니다.
- 자산 유연성: 원격 서버에 저장된 이미지도 온디맨드 이미지 크기 조정 가능

### 클라이언트 컴포넌트 vs 서버 컴포넌트

| 구분                           | 리액트 클라이언트 컴포넌트                                                              | Next.js 서버 컴포넌트                                                            |
| ------------------------------ | --------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| 위치                           | 브라우저(클라이언트 측)                                                                 | 서버 측 (SSR) 및 브라우저 (Hydration)                                            |
| 렌더링 시점                    | 클라이언트 측에서 실시간 렌더링                                                         | 서버 측에서 미리 렌더링 후 HTML을 클라이언트에 전송                              |
| 데이터 페칭                    | 클라이언트 측에서 fetch 또는 axios 사용, useEffect 로 렌더링 이후 데이터 요청           | 서버 컴포넌트 내 데이터 직접 Fetching                                            |
| SEO 친화성                     | 낮음 (클라이언트 측에서 렌더링되기 때문에 검색 엔진이 콘텐츠를 읽기 어려움)             | 높음 (서버에서 미리 렌더링되기 때문에 검색 엔진에 친화적), 페이지 소스 보기 참고 |
| 초기 로드 속도                 | 느릴 수 있음 (JavaScript를 다운로드하고 실행해야 하기 때문에)                           | 빠름 (초기 HTML이 서버에서 제공되기 때문에)                                      |
| 코드 분할                      | 클라이언트 측에서 필요한 경우 번들링 및 코드 분할 가능                                  | 서버 측에서 미리 렌더링하여 필요한 경우만 클라이언트로 전송                      |
| 상태 관리                      | 클라이언트 측 상태 관리 라이브러리 사용 (예: Redux, MobX)                               | 서버 측에서 초기 상태를 설정하고 클라이언트로 전달 가능                          |
| 사용 시기                      | - 실시간 상호작용과 동적 콘텐츠가 중요한 경우 - 사용자 이벤트에 즉각 반응해야 하는 경우 | - SEO가 중요한 경우                                                              |
| - 초기 로드 시간이 중요한 경우 |

### 사용 시기

![스크린샷 2024-06-09 오후 12.07.40.png](https://github.com/monthly-cs/2024-06-nextjs/assets/69143207/565f7556-eb63-4b51-a623-ebc7b2fb3a37)

[Rendering: Composition Patterns](https://nextjs.org/docs/app/building-your-application/rendering/composition-patterns)

- **리액트 클라이언트 컴포넌트**
  - 실시간 **사용자 상호작용**이 중요한 경우 (예: 대화형 대시보드, 실시간 업데이트가 필요한 애플리케이션)
  - 페이지 내에서 많은 동적 콘텐츠와 상태 관리가 필요한 경우 (예: 싱글 페이지 애플리케이션)
  - 사용자의 **이벤트에 따라 즉각적으로 반응**해야 하는 경우
- **Next.js 서버 컴포넌트**
  - SEO 최적화가 중요한 경우 (예: 블로그, 마케팅 사이트)
  - 초기 로드 속도가 중요한 경우 (예: 초기 로딩 시간이 중요한 전자 상거래 사이트)
  - 서버 측에서 데이터를 미리 패칭하여 렌더링해야 하는 경우 (예: 사용자 인증이 필요한 페이지, 데이터베이스 연동 페이지)
  - 초기 상태를 서버에서 설정하고 클라이언트로 전달해야 하는 경우

### NextJS 에서 서버 컴포넌트, 클라이언트 컴포넌트 함께 사용하기

- 클라이언트 컴포넌트 내부에서 서버 컴포넌트를 자식 요소로 둘 수 없다.
  ```jsx
  "use client";

  // ❌ This pattern will not work. You cannot import a Server
  // Component into a Client Component
  import ServerComponent from "./ServerComponent";

  export default function ClientComponent() {
    return (
      <>
        <ServerComponent />
      </>
    );
  }
  ```
- 하지만 클라이언트 컴포넌트에 서버 컴포넌트를 children 으로 넘겨줄 수 있다.
  ```jsx
  "use client";

  export default function ClientComponent({ children }) {
    return <>{children}</>;
  }
  ```
  ```jsx
  // ✅ This pattern works. You can pass a Server Component
  // as a child or prop of a Client Component.
  import ClientComponent from "./ClientComponent";
  import ServerComponent from "./ServerComponent";

  // Pages are Server Components by default
  export default function Page() {
    return (
      <ClientComponent>
        <ServerComponent />
      </ClientComponent>
    );
  }
  ```

### 클라이언트 컴포넌트의 효율적 사용
![image](https://github.com/monthly-cs/2024-06-nextjs/assets/69143207/e219596b-8192-4781-b54f-d68931101771)

- 컴포넌트 트리를 **최대한 내려가서 작업**해야 한다.
  - 가령 버튼 하나 때문에 전체를 클라이언트 컴포넌트로 바꿀 필요는 없다라는 것
  - 클라이언트 컴포넌트가 많아지면 페이지 로딩 속도에 영향을 줄 수 있음
- 필요한 컴포넌트만 클라이언트 컴포넌트로 변환한다면 프로젝트 대부분의 트리 컴포넌트는 서버 컴포넌트로 유지되어 그 장점을 잘 살릴 수 있다.
- **큰 틀은 서버 컴포넌트로 개발하고 안에 동적인 컴포넌트를 클라이언트 컴포넌트로 구성**하는 것이 효율적인 컴포넌트 개발 방식이다.

### 백엔드를 위한 better-sqlite3

- Q : 백엔드 학습

### 서버 컴포넌트에서 데이터 페칭

- fetch, useEffect 없이 서버 컴포넌트에서 바로 데이터 페칭 가능
- 호출하는 API 요청이 Promise 를 리턴한다면 서버 컴포넌트 앞에 async 키워드를 붙인다.
- 클라이언트 컴포넌트와 차이
  - 클라이언트 컴포넌트는 기본 useEffect 로 데이터 페칭 하고서 다른 페이지 이동 후 다시 페이지 이동 시 캐싱이 되어 있지 않아 로딩을 다시 한다.
  - NextJS 에서는 사용자가 들어간 페이지들을 모두 캐싱하므로, 첫 데이터 페칭 로딩 이후 다른 페이지 이동 후 다시 페이지 접근 시 로딩 없이 화면을 그대로 보여준다.

### 서버 컴포넌트 데이터 로딩 상태 처리

- app 루트 상단에 loading.js 로 전역 로딩 상태를 보여줄 컴포넌트 처리
  - 혹은 개별 상세 페이지에 특정 loading.js 를 처리해 줄 수 있음

### Suspense & Streamed Response 를 이용한 세분화 로딩 상태 관리

데이터를 불러오는 것으로 인해 데이터와 관련 없는 정적인 컴포넌트도 로딩 시에 함께 보여지지 않는 문제

- 컴포넌트 안에 데이터 페칭을 수행하고 응답받은 데이터로 렌더링할 **특정 엘리먼트를 리턴하는 컴포넌트를 하나 더** 만든다.
- 즉, **하나의 컴포넌트 안에서 데이터를 페칭하는 컴포넌트를 분리해 작성**한다.
- 본래 컴포넌트에서 위의 컴포넌트를 추가해 주고 Suspense 로 감싸준다.
  - 일부 데이터 또는 리소스가 불리울 때까지 로딩 상태를 처리하고 **대체 컨텐츠 (fallback)** 를 표시할 수 있다.
- 코드 참고 🐹
  ```jsx
  // ✅ 원래 MealsPage 컴포넌트 내에 있던 것을 분리함 : 데이터를 Fetching 하기 위한 용도의 Meals 컴포넌트
  async function Meals() {
    const meals = await getMeals();
    return <MealsGrid meals={meals} />;
  }

  export default function MealsPage() {
    return (
      <>
        <header className={classes.header}>
          <h1>
            Delicious meals, created{" "}
            <span className={classes.highlight}>by you</span>
          </h1>
          <p>
            Choose your favorite recipe and cook it yourself. It is easy and
            fun!
          </p>
          <p className={classes.cta}>
            <Link href="/meals/share">Share Your Favorite Recipe</Link>
          </p>
        </header>
        <main className={classes.main}>
          <Suspense fallback={<MealsLoadingPage />}>
            <Meals />
          </Suspense>
        </main>
      </>
    );
  }
  ```

### 오류 처리 방법 (error.js)

- app 경로 최상단에 컴포넌트 정의 : 모든 페이지에 발생될 수 있는 에러에 대비 가능
- 혹은 특정 경로 page 내 error.js 정의 가능
- Error 컴포넌트가 받는 Props 에는 error 라는 속성값이 있어 상세 에러 메시지 확인 등이 가능
- 클라이언트 측에서 오류를 잡아야 되기에 파일 상단에 ‘use client’ 예약어를 명시해 주어야 한다.

### Not found 상태 처리 방법 1. (not-found.js)

- 유저가 올바르지 않은 url 경로를 입력해 페이지를 요청하는 경우
- 마찬가지로 app 경로 최상단 또는 특정 중첩된 경로 내에서 not-found.js 파일 및 컴포넌트 생성
- error.js 와는 다르게 서버 컴포넌트로 동작

### Not found 상태 처리 방법 2. notFound()

- next/navigation 에서 불러오는 notFound
- 사용자가 동적 경로를 잘못 입력했을 때 데이터를 찾아오지 못한 경우 응답 데이터가 undefined.
  - 조건문을 걸어 응답 데이터가 없을 때, notFound() 를 호출하면 해당 경로에서 가장 가까운 not-found.js 파일을 찾아 호출할 수 있게 된다.

### 양식 제출을 위한 서버 액션 소개 및 사용방법

- 기존 익숙했던 방식 (바닐라 리액트)

  - form 태그에 직접 onSubmit 핸들러를 연결해, 상태 관리를 적용한 각각의 input 값들을 받아 백엔드로 요청
  - React Hook Form 같은 기타 서드파티 라이브러리 사용

- 서버 액션 함수 생성
  - 서버 액션 함수를 생성 : 함수 스코프 상단에 ‘use server’ 라 명시
    - 이를 통해 서버 액션을 생성하는데 오직 서버에서만 함수가 실행되게 보장해주는 기능이다.
    - 서버에서만 동작하는 함수
  - form 태그의 action 속성에 액션 함수를 연결
  - form 이 제출되면 NextJS 가 자동으로 요청을 생성하여 웹사이트를 제공하는 NextJS 서버로 보내게 된다.
- 개별 파일에 서버 액션 함수들을 저장
  - 서버 액션이 있는 컴포넌트가 ‘use client’ 지시어를 쓰는 클라이언트 사이드 컴포넌트 일 수 있다.
  - 기본적으로 코드를 분리해 주는 것이 유지 보수에도 좋다.
  - 공통 파일 상단에 ‘use server’ 를 명시해 주면 개별 액션 함수 내부에 ‘use server’ 를 써주지 않아도 된다.

### useFormStatus 를 이용한 양식 제출 상태 관리

- 버튼 쪽 부분만 별도의 컴포넌트로 빼서 클라이언트 컴포넌트를 생성
- 사용자 경험을 개선하기 위해 form 요청이 진행중일 때 버튼의 텍스트나 로딩 효과를 주기 위함
- useFormStatus 의 pending 값을 통해 사용자의 사용자 경험을 향상
  ```jsx
  "use client";

  import { useFormStatus } from "react-dom";

  export default function MealsFormSubmit() {
    const { pending } = useFormStatus();
    return (
      <button type="submit" disabled={pending}>
        {pending ? "Submitting..." : "Share Meal"}
      </button>
    );
  }
  ```

### 서버 액션 응답 및 useFormState 로 작업하기

- 응답하는 서버 액션에서 입력값이 유효하지 않을 때 throw new Error 를 던지면, NextJS 는 가장 가까운 error.js 컴포넌트를 찾아 화면에 렌더링 한다.
- 사용자 경험에는 Form 화면에서 에러 메시지나 얼럿 등이 보여지는 것이 좋다.
- 그래서 서버 액션에서 입력값이 유효하지 않을 때, 아래와 같이 직렬화가 가능한 객체를 리턴하도록 처리한다.

  ```jsx
  // throw new Error("Invalid input");
  return {
    message: "Invalid input.",
  };
  ```

  - **직렬화 (serialization)** : 데이터 스토리지 문맥에서 [데이터 구조](https://ko.wikipedia.org/wiki/%EB%8D%B0%EC%9D%B4%ED%84%B0_%EA%B5%AC%EC%A1%B0)나 [오브젝트](https://ko.wikipedia.org/wiki/%EC%98%A4%EB%B8%8C%EC%A0%9D%ED%8A%B8) 상태를 동일하거나 다른 컴퓨터 환경에 저장(이를테면 [파일](https://ko.wikipedia.org/wiki/%EC%BB%B4%ED%93%A8%ED%84%B0_%ED%8C%8C%EC%9D%BC)이나 메모리 [버퍼](https://ko.wikipedia.org/wiki/%EB%8D%B0%EC%9D%B4%ED%84%B0_%EB%B2%84%ED%8D%BC)에서, 또는 [네트워크](https://ko.wikipedia.org/wiki/%EC%BB%B4%ED%93%A8%ED%84%B0_%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC) 연결 링크 간 전송)하고 나중에 재구성할 수 있는 포맷으로 변환하는 과정이다.
  - 직렬화의 예 : JSON.stringify()
  - 역직렬화의 예 : JSON.parse()

- **useFormState** 훅을 통해 서버의 액션 함수를 연결하는 작업을 진행한다.
  - useFormState 의 첫 번째 인자 : form 이 제출될 때 동작하는 실제 server action
  - useFormState 의 두 번째 인자 : form 의 초기값
  - useFormState 가 리턴하는 state : form 이 처리되기 전, 처리되고 난 후의 상태값
  - useFormState 가 리턴하는 formAction 함수 : form action 에 연결 해 준다.
  - useFormState 훅을 사용하기 위해서는 해당 컴포넌트가 클라이언트 컴포넌트 여야 한다.

### NextJS 캐싱 구축 및 이해

- npm run build
  - nextjs app 이 배포환경에서 동작할 수 있게 준비하는 것
  - 서버에 배포할 수 있는 프로젝트를 만들어 준다.
- npm start
  - 동일한 [localhost:3000](http://localhost:3000) 으로 열리지만 최적화된 코드로 서버가 열린다.
- 캐싱 (캐시 유효성 재확인)
  - npm run build 에서 nextjs 는 실제로 앱에서 사전 생성될 수 있는 모든 페이지를 사전 렌더링하고 생성한다.
  - 즉 기본적으로는 동적 페이지가 아니게 된다.
  - 캐시 유효성 재확인을 위해 사용자가 form 을 제출 한 뒤의 서버 액션에서 **revalidatePath**(’/meals’) 를 적용해 준다.
