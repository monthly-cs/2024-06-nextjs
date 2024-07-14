## 6주차 발표자료입니다.

- 내용
  - 1. Next.js 배포
  - 2. Authentication

---

# Next.js 배포

## 배포 옵션

### 🔍 일반 빌드

> next build

`next build`의 출력값을 실행하려면 Node.js 서버가 필요합니다. 빌드된 결과물은 React와 달리 정적 호스트에 올려서 실행할 수 없습니다.

이유는 **서버 사이드 기능**을 사용해야 하기 때문입니다. 사전 렌더링, ISR, API 라우트 등의 코드를 실행하려면 Node.js 서버가 필요합니다.

- 사전 생성 및 사전 렌더링은 여전히 빌드 시에 진행됩니다.
  - getStaticProps
- 서버 측에 요청이 들어오면 실행되는 기능
  - getServerSideProps, revalidation, fallback false/blocking

코드가 변경되었거나 revalidation이 필요한 페이지는 앱을 재배포해야 합니다.

### 🔍 전체 정적 빌드

> next export

HTML, CSS, JS로 이루어진 100% 정적 앱입니다. 서버 측 코드가 필요 없을 때만 사용할 수 있습니다.

정적 앱이므로 콘텐츠가 변경될 때마다 재배포해야 합니다.

## 배포 과정

1. 메타데이터, 코드 정리, 의존성 정리
2. 환경 변수 확인
3. 애플리케이션 빌드 및 테스트
4. 배포

### 환경 변수 설정

- next.config.json에서 프로덕션 서버인 경우를 구분해서 설정할 때

```tsx
// next.config.json
const { PHASE_PRODUCTION_SERVER } = require("next/constants");

module.exports = (phase) => {
  // constants.d.ts
  // export declare const PHASE_PRODUCTION_SERVER = "phase-production-server";
  // export declare const PHASE_DEVELOPMENT_SERVER = "phase-development-server";
  if (phase === PHASE_PRODUCTION_SERVER) {
    return {
      env: {
        customKey: "my-prod-value",
      },
    };
  }

  return {
    env: {
      customKey: "my-value",
    },
  };
};
```

- 앱 내에서 다음과 같이 환경 변수를 사용할 수 있습니다.

```tsx
${process.env.customKey}
```

## 빌드 용량 줄이기

라이브러리에서 제공하는 경량 버전을 사용하는 예시:

| 라이브러리 | 전체 버전 | 경량 버전        |
| ---------- | --------- | ---------------- |
| Lodash     | `lodash`  | `lodash-es`      |
| Moment.js  | `moment`  | `day.js`         |
| D3.js      | `d3`      | 개별 모듈 임포트 |
| Three.js   | `three`   | `three-min.js`   |

## 🤖 Next.js는 트리 셰이킹을 하는데 왜 경량 버전을 사용해야 할까? chatGPT에게 물어봤다.

Next.js는 트리 셰이킹(Tree Shaking)을 지원합니다. 트리 셰이킹은 사용되지 않는 코드(dead code)를 제거하여 번들 크기를 줄이는 기술입니다. 이는 Next.js의 빌드 시스템인 Webpack 덕분에 가능합니다. 따라서, 트리 셰이킹을 통해 필요 없는 코드가 자동으로 제거되므로, 경량 버전 라이브러리를 사용하지 않아도 되는 경우가 많습니다.

그러나, 경량 버전을 사용하면 트리 셰이킹 외에도 추가적인 이점을 얻을 수 있습니다:

- **초기 번들 크기 감소**: 트리 셰이킹이 완벽하지 않거나 복잡한 경우, 초기 번들 크기를 줄이는 데 도움이 됩니다.
- **빌드 속도 향상**: 더 작은 라이브러리는 빌드와 번들링 속도를 향상시킬 수 있습니다.
- **명확한 종속성 관리**: 경량 버전을 사용하면 명확하게 필요한 기능만 포함하게 되어, 의도하지 않은 종속성이 포함되는 것을 방지할 수 있습니다.
- **더 빠른 로드 시간**: 특히 모바일 환경이나 네트워크 속도가 느린 환경에서 더 빠른 로드 시간을 제공합니다.

Next.js의 트리 셰이킹과 경량 버전 사용을 결합하면, 두 기술의 장점을 모두 활용할 수 있습니다. 트리 셰이킹이 제거하지 못하는 코드까지 제거하여 최적화된 빌드를 생성할 수 있습니다.

## 🤖 Next.js와 React의 트리 셰이킹 관련 차이점

| 특징              | Next.js                            | React                             |
| ----------------- | ---------------------------------- | --------------------------------- |
| 기본 설정         | 트리 셰이킹 자동 최적화            | 개발자가 빌드 도구 설정 필요      |
| 코드 스플리팅     | 페이지 기반 자동 코드 스플리팅     | 수동 구현 필요                    |
| 빌드 최적화       | 자동 번들 분석 및 최적화           | 개발자가 직접 도구 설정 및 최적화 |
| 라이브러리 임포트 | 대부분 자동으로 필요 부분만 임포트 | 개발자가 주의 깊게 임포트 문 작성 |
| 학습 곡선         | 낮음 (대부분 자동화)               | 높음 (지식과 경험 필요)           |

- webpack.config.js

```tsx
// webpack.config.js 예시
module.exports = {
  mode: "production", // 트리 셰이킹과 최적화를 활성화 (run build 시 기본)
  optimization: {
    sideEffects: false, // 불필요한 코드 제거
  },
};
```

- React 코드 스플리팅

```tsx
const OtherComponent = React.lazy(() => import("./OtherComponent"));

function MyComponent() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <OtherComponent />
    </Suspense>
  );
}
```

Next.js가 많은 부분을 자동화하고 최적화하는 반면, 일반 React 애플리케이션에서는 개발자가 더 많은 부분을 직접 관리해야 함을 알 수 있습니다.

## Vercel에 일반 빌드를 배포하면 일어나는 일

배포 시, 전체 산출물을 Node.js 서버로 옮깁니다. dependencies를 설치하고 `npm start` (next start) 명령어를 실행합니다. 내부 포트 3000을 외부 포트 80으로 포트 포워딩합니다. Vercel은 페이지와 사전 렌더링된 페이지를 구축하여 CDN에 호스팅하며, API 라우트를 온디맨드로 실행되는 서버리스 함수로 전달합니다.

## 전체 정적 빌드를 배포하려면?

`next export`를 사용하여 서버 사이드 코드가 없는 프로젝트를 정적 애플리케이션으로 만듭니다. `npm run build` 명령어를 먼저 실행한 후, `next export`를 실행하여 Node.js 서버가 아닌 정적 호스팅에 올릴 수 있습니다.

---

# 인증

- 인증되었나요? => 정말로?🤔

인증의 위조 여부는 Credentials로 확인할 수 있습니다. 인증 방식에는 Server-side Sessions와 Authentication Tokens이 있습니다.

## Server-side Sessions

서버에 식별자를 보관하고 클라이언트에 전송합니다.

클라이언트는 쿠키에 식별자를 담아서 요청을 보냅니다. 서버는 요청의 식별자를 체크하여 맞는지 확인하고 액세스를 관리합니다.

### 세션의 보안은?

- 연결에는 SSL을 사용하므로 도난 방지
- Cross-site scripting 공격 방지를 위해 JavaScript로 쿠키에 접근할 수 없도록 구성할 수 있습니다 (코드로 접근할 수 없으므로, 요청에 첨부되었을 때 서버에서만 읽도록 합니다).
  - Cross-site scripting 공격이 없다면 안전합니다.

## Authentication Token

서버는 토큰을 생성하고 클라이언트에 전송합니다.

클라이언트는 토큰과 함께 요청을 보냅니다. 서버는 유효한 토큰인지 확인하고 액세스를 관리합니다.

## SPA 개발에서 Session보다 Token을 많이 쓰는 이유

- 페이지는 직접 전달됩니다 (서버에서 페이지를 내려주는 것이 아님).
- SPA의 백엔드 API는 stateless (연결된 클라이언트를 추적하지 않음).
  - 인증된 클라이언트에 permission 전달 후 보호된 리소스에 대한 요청을 나중에 보냄.
  - API는 연결된 클라이언트에 대한 정보를 가지고 있지 않음.

싱글 페이지 애플리케이션은 백엔드에서 클라이언트를 추적하지 않는 stateless이므로, 페이지를 서버에서 내려주는 것이 아니므로 백엔드와 프론트가 분리되어 있습니다. 따라서 식별자를 서버에서 보관하고 액세스를 관리하는 세션 방식보다 JWT를 사용하는 경향이 있습니다. 권한(토큰)을 전달하고 추후에 보호된 리소스에 요청을 보냈을 때 토큰 유효 여부만 판단하는 방식입니다.

## Next.js, Authentication

### next-auth

> https://next-auth.js.org/

API 라우트에서 사용자 로그인을 확인할 수 있고, 컴포넌트에서도 진행할 수 있습니다. JWT 생성 등 서버 사이드 및 클라이언트 사이드의 인증을 구현할 수 있으며, 사용자 DB는 별도로 만들어야 합니다.

OAuth 등 프로바이더를 사용하는 방식도 공식 문서에 상세히 작성되어 있습니다: https://next-auth.js.org/providers/

- NextAuth 외의 인증 솔루션들 [(공식문서 소개)](https://nextjs.org/docs/pages/building-your-application/authentication#examples)
  - Auth0
  - Clerk
  - Kinde
  - Lucia
  - NextAuth.js
  - Supabase
  - Stytch
  - Iron Session

## 예제 살펴보기

### [클라이언트 측] API 요청 함수

```tsx
async function createUser(email, password) {
  const response = await fetch("/api/auth/signup", {
    method: "POST",
    body: JSON.stringify({ email, password }),
    headers: {
      "Content-Type": "application/json",
    },
  });

  const data = await response.json();

  if (!response.ok) {
    throw new Error(data.message);
  }

  return data;
}
```

### [API 라우트] API Catch-All

`/api/auth/[...nextauth].js`와 같이 catch-all 라우트를 사용할 수 있습니다. (NextAuth를 위한 내장 API 라우트)

[NextAuth.js REST API](https://next-auth.js.org/getting-started/rest-api)

```tsx
import NextAuth from "next-auth";

// API는 handler 함수를 export 해야 합니다.
export default NextAuth({
  session: {
    jwt: true, // 데이터베이스를 지정하지 않으면 기본값은 true입니다.
  },
  providers: [
    Providers.Credentials({
      async authorize(credentials) {
        // 로그인 요청이 들어올 때 실행하는 메서드
      },
    }),
  ],
});
```

`[...nextauth].js`에서 callbacks의 리턴 URL을 설정할 수도 있습니다.  
[NextAuth.js Callbacks](https://next-auth.js.org/v3/configuration/callbacks)

```tsx
callbacks: {
  /**
   * @param  {object} user     User object
   * @param  {object} account  Provider account
   * @param  {object} profile  Provider profile
   * @return {boolean|string}  Return `true` to allow sign in
   *                           Return `false` to deny access
   *                           Return `string` to redirect to (eg.: "/unauthorized")
   */
  async signIn(user, account, profile) {
    const isAllowedToSignIn = true;
    if (isAllowedToSignIn) {
      return true;
    } else {
      // 기본 에러 메시지를 표시하려면 false를 반환합니다.
      return false;
      // 또는 URL을 반환하여 리디렉션할 수 있습니다.
      // return '/unauthorized'
    }
  },
};
```

## [클라이언트 측] next-auth 로그인 메서드 사용

next-auth의 `signIn()`의 두 번째 인자 객체에 `redirect: false`를 설정하면 에러 정보가 있는 객체를 반환합니다.

```tsx
const result = await signIn("credentials", {
  redirect: false,
  email,
  password,
});
```

## [클라이언트 측] 활성 세션 관리

```tsx
import { useSession } from "next-auth/client";
const [session, loading] = useSession();

// session 객체에 토큰에 담긴 정보, 만료 시간 등
// 유저가 활성화되어 있다면 세션 자동 연장
// SignOut() 시 session 상태 변화
```

## [서버 사이드]에서 라우트 가드하기

인증 상태를 서버 사이드에서 확인하기 위해서는 매 요청마다 실행되고 context를 사용하기 위해 **서버 사이드 랜더링 함수**인 `getServerSideProps`를 사용합니다.

페이지를 렌더링하기 전에 `getServerSideProps`를 실행하기 때문에, 인증되었을 때만 페이지에 접근할 수 있습니다. 클라이언트 측 `useSession`은 지울 수 있습니다.

```tsx
// 인증이 필요한 페이지
export async function getServerSideProps(context) {
  const session = await getSession({ req: context.req }); // 요청의 req에서 세션을 확인

  if (!session) {
    return {
      redirect: {
        destination: "/auth",
        permanent: false, // 일시적
      },
    };
  }

  return {
    props: { session },
  };
}
```

```tsx
// 인증이 없어야 하는 페이지
useEffect(() => {
  getSession().then((session) => {
    if (session) {
      router.replace("/");
    } else {
      setIsLoading(false);
    }
  });
}, [router]);

if (isLoading) {
  return <p>Loading...</p>;
}

return <AuthForm />;
```

## next-auth의 Provider

```tsx
// _app.js
import { Provider } from "next-auth/client";
...
<Provider session={pageProps.session}>
  <Layout>
    <Component {...pageProps} />
  </Layout>
</Provider>
```

## 서버사이드, API 라우트도 가드해야 한다

```tsx
// api/user/change-password.js
async function handler(req, res) {
  if (req.method !== "PATCH") return;

  const session = await getSession({ req });

  if (!session) {
    res.status(401).json({ message: "로그인 하세요" });
    return;
  }

  // 비밀번호 변경 로직
}

export default handler;
```

- Next.js의 API 라우트를 사용하면 세션 관리, 요청 유효성 검사 및 처리, 오류 처리 등 서버 사이드의 고려사항이 늘어납니다!

---

# 보너스

## Controlled vs Uncontrolled 컴포넌트 패턴

예제에서 `ref`를 사용하는 경우가 많습니다. 개인적으로는 요즘 습관적으로 `state`를 사용했지만, 이번 기회에 다시 한 번 정리해봅니다.

Controlled 컴포넌트는 상태 값을 업데이트하여 관리하고, Uncontrolled 컴포넌트는 DOM 노드를 참조하여 값을 업데이트합니다. 즉각적인 UI 업데이트를 위해서는 `state`가 유리하고, 그렇지 않은 경우 `ref`가 간결할 수 있습니다. 패턴에 따라 장점이 있지만, 통일성을 위해서 프로젝트 내에서 한 가지 패턴을 사용하는 것이 좋을 것 같습니다.

## 비교 표

| **특징**                  | **Controlled Components**                      | **Uncontrolled Components**                           |
| ------------------------- | ---------------------------------------------- | ----------------------------------------------------- |
| **상태 관리**             | 컴포넌트의 상태(state)로 관리                  | DOM에서 직접 관리                                     |
| **값 업데이트**           | `setState`를 사용하여 상태를 업데이트          | `ref`를 사용하여 DOM 노드에 직접 접근하여 값 업데이트 |
| **싱글 소스 오브 트루스** | 상태가 폼 요소의 유일한 진실의 근원            | 상태와 DOM이 각각 별도로 값 관리                      |
| **초기값 설정**           | 상태를 통해 초기값 설정                        | `defaultValue` 속성이나 직접 설정                     |
| **값 읽기**               | 상태 값을 통해 읽음                            | `ref`를 통해 DOM 요소의 값을 직접 읽음                |
| **렌더링 제어**           | 상태 값이 변경될 때마다 컴포넌트가 다시 렌더링 | `ref` 값 변경 시 컴포넌트가 다시 렌더링되지 않음      |
| **유효성 검사**           | 상태 변경 시 검사가능                          | 폼 제출 시 유효성 검사 수행                           |
| **장점**                  | 일관된 상태 관리, 입력 값의 유효성 검사 용이   | 간결한 코드, 초기값 설정 용이                         |
| **단점**                  | 추가 코드 작성                                 | 유효성 검사, 상태 추적이 상대적으로 어려움            |
