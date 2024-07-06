## 5주차 발표자료입니다.

강의의 예시 코드를 포함하고 있습니다.

---

## Page Routing

### SSR를 사용하면 좋은 앱: content heavy app

- **Pre-rendering**: HTML을 사전 렌더링(pre-building)하여 클라이언트에게 전달합니다.
- **React**는 사용자 인터랙션에 강점이 있는 프레임워크입니다.
- 로드 후에 React 코드로 **Hydrate**하면 인터랙티브해집니다.
- 사전 렌더링은 초기 로드 시에만 발생하며, 이후는 SPA처럼 동작합니다.

사전 렌더링 방식에는 두 가지가 있습니다:

1. 정적 생성
2. 서버 사이드 렌더링

---

### 정적 생성

모든 콘텐츠를 사전 준비(빌드 타임에)하여 배포합니다.

- 빌드 프로세스에 진행
  - 서버 사이드에서 동작하는 코드를 작성할 수 있습니다. ex) fs 사용
  - **CDN**으로 캐시가 가능합니다.👍
  - Hydrate 이후 React APP으로 동작하여 페이지가 채워져 있는 SPA처럼 동작합니다.
- **getStaticProps**: 사전 렌더링을 위한 정해진 이름의 함수입니다.
  - 클라이언트에 노출되지 않으므로 크리덴셜 등을 안전하게 처리할 수 있습니다.
  - props를 준비합니다.

```tsx
export async function getStaticProps(context) {
  const filePath = path.join(process.cwd(), "data", "dummy.json");
  const jsonData = await fs.readFile(filePath);
  const data = JSON.parse(jsonData);

  return {
    props: {
      products: data.products,
    },
  };
}

function HomePage(props) {
  const { products } = props;

  return (
    <ul>
      {products.map((product) => (
        <li key={product.id}>{product.title}</li>
      ))}
    </ul>
  );
}

export default HomePage;
```

### Icremental Static Generation (ISR, 증분 생성)

빌드 후에도 재배포 없이 페이지를 업데이트할 수 있습니다~

- **Re-generate 시간**이 지나지 않았다면 기존 페이지를 제공합니다.
  - 서버에서 사전 생성되고, 시간이 되면 페이지를 Re-generate해서 캐시합니다.
  - 시간 설정은? **getStaticProps** return에 **revalidate** 키를 추가합니다.

```tsx
export async function getStaticProps(){
  ...
  return {
    props: {
      products: data.products,
    },
    revalidate: 60, // 초 단위
  }
}
```

---

### 정적 생성 예외 처리

- `notFound`와 `redirect` 키를 반환하여 예외를 처리합니다.

```tsx
export async function getStaticProps(context) {
  const data = await getData();

  if (!data) {
    return { redirect: { destination: "/no-data" } };
  }

  if (data.products.length === 0) {
    return { notFound: true };
  }

  return {
    props: {
      products: data.products,
    },
    revalidate: 60, // 페이지가 구축 & 배포된 이후에도 계속 재성성할 수 있도록 설정합니다.
  };
}
```

---

### params 사용하기

- 컴포넌트에서 params를 사용하면? 👉 브라우저에서 실행됩니다.
- getStaticProps는 서버에서 실행된다. 👉 데이터를 사전 랜더링 할 때 사용합니다.

  ```tsx
  export async function getStaticProps(context) {
    const { params } = context;
    const productId = parmas.pid;

    const data = await getData();
    const product = data.products.find((product) => product.id === productId);

    if (!product) {
      return { notFound: true };
    }

    return {
      props: {
        loadedProduct: product,
      },
      revaildate: 60,
    };
  }
  ```

---

### 동적 페이지 SSG를 위한 getStaticPaths

`/[product]`와 같은 디렉토리의 동적 페이지는 default로 사전 렌더링되지 않습니다. 페이지가 얼마나 생성될 지 모르기 때문에, Next.js에게 추가적인 안내가 필요합니다.

`getStaticProps`를 사용하여 Next.js에 사전 렌더링할 것을 알리고, `getStaticPaths`를 페이지에 추가합니다. 동적 페이지 사전 생성을 위한 추가적인 안내가 바로 `getStaticPaths`입니다.

```tsx
export default async function getStaticPaths() {
  return {
    paths: {
      { params: { pid: "p1" } },
      { params: { pid: "p2" } },
    },
    fallback: false
  }
}
```

### **fallback** 옵션

- `{ fallback: blocking }`: 페이지를 서브하기 전에 사전 렌더링이 끝나기를 기다립니다. 응답 지연이 생기지만, 사전 렌더링이 완료된 페이지를 보여줍니다.
- `{ fallback: false }`: 정적 경로에서 미리 렌더링된 페이지만 제공하며, 사전 렌더링되지 않은 페이지에 접근하면 404 페이지가 표시됩니다.
- `{ fallback: true }`: 페이지를 서브하기 전에 사전 렌더링이 끝나기를 기다리지 않습니다. 최초 요청 시에 정적 페이지를 생성하고, 그동안 로딩 상태를 나타내는 컴포넌트를 출력할 수 있습니다.

---

### 빌드

빌드된 파일에서 다음 파일을 확인할 수 있습니다.

- 데이터 프리패치: p1.json // pre patched data file
- 사전 생성된 페이지: p1.html // will hydrate

페이지 사전 생성을 위해서는 시간 비용이 소요됩니다.

- 만약 사전 생성되어야 할 페이지가 많다면? ex) 아마존
- 거의 방문되지 않는 페이지가 있다면? -> 불필요할 수 있다.

---

### paths 하드코딩 지우기

```tsx
export default async function getStaticPaths() {
  const data = await getData();

  const ids = data.products.map((product) => product.id);
  const pathsWithParams = ids.map((id) => ({ params: { pid: id } }));

  return {
    paths: pathsWithParams,
    fallback: false,
  };
}
```

---

### fallback: true 예시

> 사전 랜더링 페이지가 없는 경우 로딩 상태를 어떻게 표기할 것인가? 그리고 동적인 데이터 소스에서 데이터가 없는 페이지를 어떻게 처리할 것인가?

- 로딩 중: fallback 컴포넌트
  - fallback을 true로 설정하면, 백그라운드에서 사전 렌더링이 진행되는 동안 로딩 상태를 나타내는 컴포넌트를 출력할 수 있음
  - 백그라운드: getStaticProps를 호출 -> 데이터 가져옴 -> 페이지 랜더링
- 데이터 없음: `{ notFound: true }`
  - getStaticProps에서 잘못된 데이터를 컴포넌트에 내려준다면? 데이터가 누락된 페이지에서 오류 발생 => 페이지를 표시하지 않고 404 페이지를 반환하도록 설정

```tsx
function ProductDetailPage(props) {
  const { loadedProduct } = props;

  if (!loadedProduct) {
    return <p>Loading...</p>;
  }

  return (
    <>
      <h1>loadedProduct.title</h1>
    </>
  );
}

export async function getStaticProps(context) {
  const { params } = context;
  const productId = parmas.pid;

  const data = await getData();
  const product = data.products.find((product) => product.id === productId);

  if (!product) {
    return { notFound: true };
  }

  return {
    props: {
      loadedProduct: product,
    },
    revaildate: 60,
  };
}

export default async function getStaticPaths() {
  const data = await getData();

  const ids = data.products.map((product) => product.id);
  const pathsWithParams = ids.map((id) => ({ params: { pid: id } }));

  return {
    paths: pathsWithParams,
    fallback: true,
  };
}
```

---

### getStaticProps 실행 시점

> [Next.js 공식문서](https://nextjs.org/docs/pages/building-your-application/data-fetching/get-static-props#when-does-getstaticprops-run)

- `getStaticProps`는 항상 서버에서 실행되며 클라이언트에서는 절대 실행되지 않습니다.
  - `next build` 동안 실행됩니다.
  - `fallback: true`를 사용할 때 백그라운드에서 실행됩니다.
  - `fallback: blocking`을 사용할 때 초기 렌더링 전에 호출됩니다.
  - `revalidate`를 사용할 때 백그라운드에서 실행됩니다.
  - `revalidate()`를 사용할 때 필요에 따라 백그라운드에서 실행됩니다.
- **ISR**과 결합하면, 오래된 페이지가 다시 검증되는 동안 백그라운드에서 최신 페이지가 브라우저에 제공됩니다.
- `getStaticProps`는 정적 HTML을 생성하므로 request(예: 쿼리 매개변수 또는 HTTP 헤더)에 접근할 수 없습니다. 페이지에 대한 요청에 접근해야 하는 경우 `getStaticProps`와 함께 미들웨어를 사용하는 것을 고려해야 합니다.

---

### 정적 생성 with ISR, 그리고 서버 사이드 렌더링

정적 생성과 ISR는 주로 프로젝트를 빌드할 때 사용됩니다. 그러나 모든 요청에 대해 사전 렌더링을 해야 하는 경우도 있습니다. ex) 쿠키에 액세스, 유저 특정 데이터가 필요한 경우

Next.js는 **getServerSideProps** 함수를 제공합니다.

페이지 컴포넌트 파일에 이 함수가 있다면, Next.js는 해당 페이지에 대한 request가 들어올 때마다 이 함수를 실행합니다.

---

### getServerSideProps

`getServerSideProps`는 `getStaticProps`와 형태는 유사하지만, revalidate를 사용하지 않습니다. 모든 요청에 대해 실행되므로 데이터가 매 요청마다 새로고침 되어야 할 때, 즉 정적 생성이 필요하지 않은 경우에 사용합니다.

이 함수는 `context`를 통해 Node.js의 `req`, `res` 타입 객체에 접근할 수 있습니다. `getStaticPaths`는 필요 없습니다. 왜냐하면, 서버에서만 작동하므로 사전 생성할 필요가 없고, 사전 렌더링이 진행되기 때문입니다.

- **사전 생성X, 사전 렌더링O**: 빌드 시, 람다 기호가 있는 페이지들은 사전 생성은 하지 않았지만, 사전 렌더링이 진행된 페이지입니다.

아래는 `getServerSideProps`의 예시입니다:

```tsx
export async function getServerSideProps(context) {
  const { params, req, res } = context;

  return {
    props: {
      userName: params.userName,
      value: params.value,
    },
  };
}
```

---

### 사전 패칭 pre-fetching

빌드 프로세스 동안 서버 사이드에서 데이터를 패칭합니다. ISR이나 SSR를 사용하여 데이터를 업데이트할 수 있습니다.

클라이언트에게 완성된 페이지를 전달하여 UX를 높이거나 SEO를 개선하는 것이 목적입니다.

### 클라이언트 사이드 데이터 패칭의 필요성

- 갱신 주기가 잦은 데이터 ex) 주식 데이터
- 유저 특정 데이터 ex) 주문확인, 프로필
- 부분 데이터: 여러 부분적인 API 콜이 있는 경우, 서버에서 사전 렌더링이 지연될 가능성이 있습니다. ex) 대시보드

#### Client side data fetch 예시

```tsx
import { useEffect, useState } from "react";

export default function LastSalesPage() {
  const [sales, setSales] = useState();
  const [isLoading, setLoading] = useState(false);

  useEffect(() => {
    const fetchSales = async () => {
      setLoading(true);
      try {
        const res = await fetch("https://~~");
        const data = await res.json();

        const transformedSales = [];

        for (const key in data) {
          transformedSales.push({ id: key, username: data[key].username });
        }

        setSales(transformedSales);
      } catch (error) {
        console.error("Failed to fetch sales data:", error);
      } finally {
        setLoading(false);
      }
    };

    fetchSales();
  }, []);

  if (!isLoading) {
    return <>Loading...</>;
  }

  if (!sales) {
    return <>no data</>; // 📸 페이지 스냅샷 여기 => 클라이언트에서 패칭하기 때문에 no data로 사전 랜더링 됨
  }

  return (
    <ul>
      {sales.map((sale) => (
        <li key={sale.id}>{sale.username}</li>
      ))}
    </ul>
  );
}
```

---

### useSWR

Next.js 팀이 개발한 리액트 훅 라이브러리입니다. **stale-while-revalidate** 패턴을 간편하게 사용할 수 있습니다.

- url를 식별자로 사용해서 중복 요청을 보내지 않습니다.
- revaildate, focus시 자동 패칭과 같은 기능을 사용할 수 있습니다.

```tsx
import { useEffect, useState } from "react";
import useSWR from "swr";

export default function LastSalesPage() {
  const [sales, setSales] = useState();
  const { data, error } = useSWR("https://~~");

  useEffect(() => {
    if (data) {
      const transformedSales = [];

      for (const key in data) {
        transformedSales.push({ id: key, username: data[key].username });
      }

      setSales(transformedSales);
    }
  }, [data]);

  if (!data && !sales) {
    return <>Loading...</>; // 📸 페이지 스냅샷 여기 => 클라이언트에서 패칭하기 때문에 Loading...로 사전 랜더링 됨
  }

  if (!sales) {
    return <>no data</>;
  }

  return (
    <ul>
      {sales.map((sale) => (
        <li key={sale.id}>{sale.username}</li>
      ))}
    </ul>
  );
}
```

---

### 사전 패칭 & 클라이언트 사이드 패칭

- **서버 측에서 패칭**: 초기 데이터 / 기존 데이터, 사전 렌더링
- **클라이언트 측에서 패칭**: 업데이트된 데이터, 실시간성

```tsx
import { useEffect, useState } from "react";
import useSWR from "swr";

export default function LastSalesPage(props) {
  const [sales, setSales] = useState(props.sales);
  const { data, error } = useSWR("https://~~");

  useEffect(() => {
    if (data) {
      const transformedSales = [];
      for (const key in data) {
        transformedSales.push({ id: key, username: data[key].username });
      }
      setSales(transformedSales);
    }
  }, [data]);

  if (error) {
    return <>Fail to fetch</>;
  }

  if (!data && !sales) {
    return <>Loading...</>;
  }

  return (
    <ul>
      {sales.map((sale) => (
        <li key={sale.id}>{sale.username}</li>
      ))}
    </ul>
  );
}

export async function getStaticProps() {
  const res = await fetch("https://~~");
  const data = await res.json();

  const transformedSales = [];
  for (const key in data) {
    transformedSales.push({ id: key, username: data[key].username });
  }

  return {
    props: { sales: transformedSales },
    revalidate: 10,
  };
}
```

---

### 프로젝트 데모를 통해 Section 14 살펴보기

코드를 작성하기 전에 고려해야 할 점이 많아집니다!

#### Pre-fetch vs Client-side Fetch

- 검색 엔진 크롤러가 알아야 하는 정보인가? (트래픽을 유도해야 하는가?)
- 짧은 기간에 여러 번 바뀌는가?
- 사용자 특정 데이터인가? 또는 인증이 필요한 페이지인가?
- 부분 요청이 많거나, 응답에 시간이 걸리는 요청을 포함한 페이지인가?

#### 사전 생성 & 사전 렌더링 & 서버 사이드 렌더링

- 모든 요청마다 사전 렌더링이 되어야 하는가?
- 사용자의 요청에 액세스해야 하는가?

#### 데이터 패칭 최적화하기

- 유효성 검사로 업데이트 진행, revalidate
- 사전 생성은 주요 데이터만 ex) feature 이벤트만 revaildate
- 사전 패칭을 사용해서 초기값 설정하기
- 사전 생성할 페이지가 불필요하게 많다면? => 서버 사이드 렌더링
- SEO가 중요하지 않고, 빠른 페이지 서빙이 필요하다면? => 클라이언트 사이드 렌더링

---

### Next.js 앱 최적화

#### 페이지 Head 추가하기

Head 컴포넌트는 JSX 어디에 있든 상관없습니다.

```tsx
import Head from "next/head";

...
return (
  <div>
    <Head>
      <title>{데이터.title}</title>
      <meta name="description" content={데이터.description} />
    </Head>
  </div>
);
```

- **Head**에 default 값 설정하기
  - `\_app.js`는? 애플리케이션 셸
  - `\_document.js`는? DOM 요소를 추가할 때 사용

```tsx
<Layout>
  <Head>
    <meta name="viewport" content="initial-scale=1, width=device-width" />
    <title>페이지 기본 타이틀</title> // Head를 중복으로 작성할 때 같은 요소가 있다면 덮어씁니다
    <meta ... />
  </Head>
</Layout>
```

### Image

이미지의 여러 버전을 동적으로 생성합니다. 요청이 들어올 때마다 최적화됩니다!

- webp, avif, lazy loading
- 필요할 때마다 최적화 => 요청이 있을 때마다!
- width와 height는 이미지 사이즈 => element 사이즈는 CSS로 설정
