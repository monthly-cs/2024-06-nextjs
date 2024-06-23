## 3주차 발표자료입니다.

강의를 통해 새롭게 알게된 사항, 궁금함을 채우기 위해 공식 문서를 살펴본 내용, 실제로 적용한 사례를 공유합니다.

---

## NextJS의 캐싱 유형

### Caching in Next.js

> https://nextjs.org/docs/app/building-your-application/caching

> [!NOTE]  
> 알아 두면 좋은 정보: 이 페이지는 Next.js가 어떻게 작동하는지 이해하는 데 도움을 줍니다. 그러나 Next.js를 사용하여 생산성을 높이기 위해서는 반드시 알아야 하는 지식은 아닙니다. 대부분의 Next.js 캐싱 휴리스틱은 API 사용에 따라 결정되며 최상의 성능을 위한 기본값이 있어 추가 구성 없이도 작동합니다.

| **메커니즘**                        | **내용**             | **위치**   | **목적**                              | **지속 시간**              |
| ----------------------------------- | -------------------- | ---------- | ------------------------------------- | -------------------------- |
| **요청 메모화 Request Memoization** | 함수의 반환 값       | 서버       | React 컴포넌트 트리에서 데이터 재사용 | 요청별 라이프사이클        |
| **데이터 캐시 Data Cache**          | 데이터               | 서버       | 사용자 요청 및 배포 간 데이터 저장    | 지속적 (재검증 가능)       |
| **전체 경로 캐시 Full Route Cache** | HTML 및 RSC 페이로드 | 서버       | 렌더링 비용 감소 및 성능 향상         | 지속적 (재검증 가능)       |
| **라우터 캐시 Router Cache**        | RSC 페이로드         | 클라이언트 | 탐색 시 서버 요청 감소                | 사용자 세션 또는 시간 기반 |

기본적으로, Next.js는 공격적인 캐싱을 진행합니다. route는 정적으로 랜더링되고, 동일 요청은 캐시됩니다.

![image](https://github.com/dusunax/exercises/assets/94776135/273c9d7a-fa6e-4ec2-8259-cb48cea2bbc2)

캐싱은 route의 정적/동적 랜더링 여부, 최초 접근 여부 등에 따라 다르게 동작합니다. 필요에 따라 각 route, 요청의 캐싱을 설정할 수 있습니다.

---

### 📌 요청 기억 (Request Memoization)

리액트는 `fetch API`를 확장해, 동일한 URL과 옵션을 가진 GET 요청을 메모리 내에서 관리하고 재사용합니다. 이를 통해 동일한 요청에 대해 반복적으로 네트워크를 사용하는 것을 줄일 수 있습니다.

<img width="610" alt="image" src="https://github.com/dusunax/exercises/assets/94776135/94e7b1dc-00bd-4d5f-8b28-35e9ce6844dd">

- How Request Memoization Works
  <img width="1209" alt="image" src="https://github.com/dusunax/exercises/assets/94776135/63809a46-1b3e-42ab-885e-6af545692648">
  - 경로를 렌더링하는 동안 특정 요청이 처음 호출되면 결과가 메모리에 없기 때문에 캐시 미스가 발생합니다.
  - 따라서 함수가 실행되고 외부 소스에서 데이터를 가져와 결과를 메모리에 저장합니다.
  - 동일한 렌더링 패스 내에서의 이후 함수 호출은 캐시 히트가 되어 함수 실행 없이 메모리에서 데이터를 반환합니다.
  - 경로 렌더링이 완료되고 렌더링 패스가 끝나면 메모리가 "리셋"되어 모든 요청 메모화 항목이 삭제됩니다.

=> 요청 메모화는 동일한 렌더링 패스 내에서만 유효하므로, 경로가 변경되면 메모화된 데이터는 초기화됩니다. (post/1 => post/2 => post/1로 접근할 때, post/1에 대한 요청은 다시 패칭)

---

### 📌 데이터 캐시 (Data Cache)

Next.js에서 fetch 요청을 수행할 때, 기본 fetch를 오버라이드하여 다양한 캐싱 옵션을 제공합니다. 이를 통해 데이터 캐싱을 제어할 수 있습니다.

- How the Data Cache Works
  <img width="1377" alt="image" src="https://github.com/ChooseTale/Frontend/assets/94776135/a74cc9ef-495c-46b2-81e2-80e508ccf7f4">
  - 렌더링 중에 처음으로 fetch 요청이 호출될 때, Next.js는 데이터 캐시에서 캐시된 응답을 확인합니다.
  - 만약 캐시된 응답이 발견된다면, 이를 즉시 반환하고 메모화합니다.
  - 캐시된 응답이 발견되지 않으면, 데이터 소스로 요청을 보내고 그 결과를 데이터 캐시에 저장하고 메모화합니다.
  - 캐시되지 않은 데이터의 경우 (예: { cache: 'no-store' }), 항상 데이터 소스에서 결과를 가져오고 메모화합니다.
  - 데이터가 캐시되었든 캐시되지 않았든, 동일한 데이터에 대한 중복 요청을 방지하기 위해 요청은 항상 메모화됩니다. 이는 React 렌더링 패스 중에 동일한 데이터에 대해 중복 요청을 피하기 위한 것입니다.

#### 캐시 사용하지 않는 방법

1. **`revalidatePath` 사용**
2. **fetch에 configuration 넘기기**

#### Fetch Configuration

- `fetch("", { cache: "" })`: 캐싱 동작을 명시적으로 제어할 수 있습니다.

  - `force-cache`: 캐싱을 강제합니다.
  - `no-store`: 캐싱하지 않기를 강제합니다.

- `fetch("", { next: {} })`: Next.js의 기본 fetch 옵션을 사용합니다. 이는 타임스탬프 쿼리 등을 사용하지 않아도 캐싱을 제어할 수 있음을 의미합니다.
  ```tsx
  fetch("", { next: { revalidate: 5 } }); // 초 단위의 만료 시간
  ```

#### 예약어

- **파일 단위 캐싱 설정**

  - `export const revalidate = 5;`: 파일 단위로 캐싱 만료 시간을 설정합니다.
  - `export const dynamic = 'force-dynamic';`: 전체 페이지에 대해 `cache = "no-store"`와 동일한 기능을 수행합니다.

- **컴포넌트 단위 캐싱 설정**

  - `unstable_noStore()`: 파일 단위가 아닌 컴포넌트 단위로 캐싱 설정을 할 수 있습니다. 한 파일에 여러 컴포넌트가 있을 때 유용합니다.
    - [Next.js Documentation - unstable_noStore](https://nextjs.org/docs/app/api-reference/functions/unstable_noStore)

  ```tsx
  import { unstable_noStore } from "next";

  const MyComponent = () => {
    unstable_noStore();

    return <div>My Component</div>;
  };
  ```

## 데이터 캐시와 요청 메모화의 비교

### 지속성과 범위:

- 데이터 캐시: 들어오는 요청과 배포 사이에서 지속됩니다. 사용자 세션 및 애플리케이션의 다양한 배포에서 재사용할 수 있는 데이터를 저장합니다.
- 요청 메모화: 단일 요청 또는 렌더링 패스의 수명 동안만 존재합니다. 요청이 완료되거나 렌더링 패스가 종료되면 메모화된 데이터는 메모리에서 삭제됩니다.

### 수명과 사용 방법:

- 데이터 캐시: 사용자 세션 또는 애플리케이션의 다양한 배포 기간 동안 재사용할 데이터를 저장하는 데 사용됩니다. 원본 데이터 소스(예: 데이터베이스, CMS)로의 요청 수를 줄이는 데 기여합니다.
- 요청 메모화: 주로 단일 요청이나 렌더링 패스 내에서 사용됩니다. React 컴포넌트 트리를 렌더링하는 동안 동일한 데이터에 대한 중복 요청을 줄이어 성능을 최적화하는 데 기여합니다.

### 네트워크 요청 감소:

- 요청 메모화: 동일한 렌더링 패스 내에서 중복 요청 수를 줄여 네트워크 경계(예: CDN, Edge 네트워크, 데이터 소스)를 통과하는 요청 수를 감소시킵니다.
- 데이터 캐시: 메모리나 캐싱 레이어에서 직접 캐시된 데이터를 제공함으로써 원본 데이터 소스로의 전체 요청 수를 감소시킵니다. 이는 네트워크 트래픽과 응답 시간을 최적화하는 데 도움을 줍니다.

👉 요청 메모화는 짧은 기간 동안만 유효하며 특정 요청 또는 렌더링 패스에서 사용되고, 데이터 캐시는 장기간 사용될 수 있으며 여러 세션과 배포에서 데이터를 캐시하여 네트워크 요청을 최적화합니다.

---

## 📌 Full Route Cache

- **빌드 시 생성 및 초기화**: 동적 페이지를 제외하고 모든 페이지를 사전 렌더링합니다.
- **데이터 변경 시 대응**:
  1. 캐싱을 비활성화하거나, 캐싱 시간 설정
  2. `revalidatePath()`: 필요할 때 특정 캐시를 재검증

### RevalidateTag

- **RevalidateTag 기능**: 특정 태그를 사용하여 캐시된 데이터를 무효화합니다.
  - [Revalidate Tag - Next.js Documentation](https://nextjs.org/docs/app/api-reference/functions/revalidateTag)
- **사용법**:

  ```tsx
  // 태그 부착 예시
  fetch(url, { next: { tags: ["tag1", "tag2"] } });

  // revalidateTag 함수 사용 예시
  import { NextRequest } from "next/server";
  import { revalidateTag } from "next/cache";

  // 특정 경로의 GET 요청 예시
  export async function GET(request: NextRequest) {
    const tag = request.nextUrl.searchParams.get("tag"); // 요청 URL의 쿼리 파라미터에서 tag 값을 가져옴
    revalidateTag(tag); // 가져온 tag 값을 사용하여 캐시를 무효화
    return Response.json({ revalidated: true, now: Date.now() }); // 성공 여부와 현재 시간 반환
  }
  ```

  - 태그는 256자 이하의 대소문자 구분 문자열이어야 합니다.
  - 여러 태그를 추가해놓고 특정 태그로 한 번에 revalidate할 수 있습니다.

### 커스텀 데이터 소스에 대한 요청 메모화

- **예시: SQLite**와 같은 커스텀 데이터 소스를 사용하는 경우, React의 `cache` 함수를 사용하여 메모화 설정을 합니다.

  ```tsx
  import { cache } from "react";

  export const getMessages = cache(function getMessages() {
    return db.prepare("SELECT * FROM messages").all();
  });
  ```

### 커스텀 데이터 소스에 대한 데이터 캐싱

- **unstable_cache**: `next/cache`의 `unstable_cache`를 사용하여 데이터를 캐싱합니다.

  - [Unstable Cache - Next.js Documentation](https://nextjs.org/docs/app/api-reference/functions/unstable_cache)

  ```tsx
  import { unstable_cache } from "next/cache";
  import { cache } from "react";

  export const getMessages = unstable_cache(
    cache(function getMessages() {
      return db.prepare("SELECT * FROM messages").all();
    }),
    ["messages"], // 캐시 키
    {
      tags: ["msg", "message"], // 태그 설정
    }
  );
  ```

Next.js에서 제공하는 다양한 캐싱 및 재검증 도구를 활용하면 효율적인 데이터 관리를 할 수 있습니다. 무조건 캐시를 날리기보다는 `revalidatePath`와 `revalidateTag` 등의 도구를 사용하여 필요한 부분만 재검증하는 것이 좋습니다.

## 📌 라우터 캐시 Router Cache

Next.js는 사용자 세션 동안 React 서버 컴포넌트 페이로드를 개별 경로 세그먼트로 분할하여 저장하는 인메모리 클라이언트 측 캐시를 가지고 있습니다. 이를 라우터 캐시라고 합니다.

사용자가 경로를 이동할 때 방문한 경로 세그먼트를 캐시하고 사용자 뷰포트에 있는 <Link> 컴포넌트를 기반으로 사용자가 이동할 가능성이 있는 경로를 미리 가져오는 기능을 제공합니다.

이로 인해 사용자에게 다음과 같은 개선된 네비게이션 경험이 제공됩니다:

방문한 경로가 캐시되어 뒤로/앞으로 바로 이동이 가능합니다.
미리 가져오기와 부분 렌더링 덕분에 새로운 경로로의 빠른 네비게이션이 가능합니다.
네비게이션 간에 전체 페이지 새로고침이 없으며, React 상태와 브라우저 상태가 보존됩니다.

---

## Next.js Optimizing

> [Next.js Optimizing Documentation](https://nextjs.org/docs/app/building-your-application/optimizing)

Next.js는 성능 최적화를 위한 다양한 기능을 제공합니다. 이를 활용하면 더 빠르고 효율적인 웹 애플리케이션을 개발할 수 있습니다.

### Image 컴포넌트 최적화

Next.js의 `Image` 컴포넌트는 웹 성능 최적화에 중요한 역할을 합니다.

- **사이즈 최적화**: 이미지를 WebP, AVIF와 같은 최신 포맷으로 변환하여 더 작은 파일 크기로 제공합니다.
- **시각적 안정화**: CLS(누적 레이아웃 이동) 방지를 통해 시각적 안정성을 제공합니다.
- **페이지 로드 최적화**: 레이지 로딩 및 블러 처리 기능을 제공하여 페이지 로드 속도를 개선합니다.
- **유연한 asset 관리**: 이미지 리사이징을 통해 다양한 해상도에서 최적화된 이미지를 제공합니다.

### import한 이미지의 메타데이터

`Image` 컴포넌트로 이미지를 import하면 다음과 같은 메타데이터를 제공합니다:

- **src**: 처리된 이미지의 경로
- **height**: 이미지의 높이
- **width**: 이미지의 너비
- **blurDataURL**: 블러 처리된 이미지의 데이터 URL
- **blurWidth**: 블러 처리된 이미지의 너비
- **blurHeight**: 블러 처리된 이미지의 높이

### Image 컴포넌트와 생성된 img 태그

> [Next.js Image Component API Reference](https://nextjs.org/docs/pages/api-reference/components/image)

`Image` 컴포넌트는 `src`와 `alt` 속성을 필요로 하며, 생성된 `img` 태그의 `loading` 속성은 기본적으로 `lazy`로 설정됩니다. `srcSet` 속성에는 다양한 해상도에 맞춘 이미지 URL이 동적으로 생성되어 제공됩니다. 이미지는 `height`와 `width` 정보를 기반으로 표시됩니다.

- **size 프로퍼티 사용**: 로컬 이미지를 로드할 때는 `width`와 `height` 대신 `sizes` 프로퍼티를 사용하는 것이 좋습니다.

  ```tsx
  <Image src="/me.png" alt="me" sizes="(max-width: 768px) 100vw, 50vw" />
  ```

- **quality 프로퍼티**: 0~100 범위의 값을 사용하여 이미지 품질을 설정할 수 있습니다.
  ```tsx
  <Image src="/me.png" alt="me" quality={75} />
  ```

### 유저가 올린 이미지 vs asset 이미지

- **사전 로드 이미지**: 로고와 같이 항상 사전 로드가 필요한 이미지는 `priority` 키를 추가합니다.

  ```tsx
  <Image src="/logo.png" alt="logo" priority />
  ```

- **fill 프로퍼티**: 유저가 업로드한 이미지의 크기를 알 수 없을 때는 부모 기준으로 크기를 설정하는 `fill` 프로퍼티를 사용합니다. 이 경우, `position`은 `absolute`로 설정됩니다.

  ```tsx
  <Image src={userImageUrl} alt="user-uploaded image" fill />
  ```

- **remotePattern**: 유저가 업로드한 이미지가 있는 스토리지의 도메인을 `next.config.js`에 등록합니다.
  ```js
  module.exports = {
    images: {
      remotePatterns: [
        {
          protocol: "https",
          hostname: "example.com",
          port: "",
          pathname: "/account123/**",
        },
      ],
    },
  };
  ```

### 이미지 로더 및 클라우드 크기 조정

정적 asset 호스트 서비스를 사용할 때, 이미지 로더를 활용하여 온디맨드로 이미지를 조작할 수 있습니다.

- **Cloudinary**: 파일명 앞에 세그먼트를 추가하여 이미지를 변환합니다.
  ```tsx
  <Image
    src="https://res.cloudinary.com/demo/image/upload/w_500,h_500,c_fill/sample.jpg"
    alt="Sample"
  />
  ```
  자세한 내용은 [Cloudinary Transformation Reference](https://cloudinary.com/documentation/transformation_reference)를 참조하세요.

### 메타데이터

- **정적 메타데이터**: 상수를 내보내어 설정합니다.

  ```tsx
  export const metadata = {
    title: "My Page",
    description: "This is my page",
  };
  ```

- **동적 메타데이터**: 함수를 내보내어 설정합니다.

  ```tsx
  export async function generateMetadata() {
    return {
      title: "My Page",
      description: "This is my page",
    };
  }
  ```

- **레이아웃 메타데이터**: 레이아웃이 적용되는 페이지 중 자체 메타데이터를 설정하지 않은 모든 페이지에 적용됩니다.
  ```tsx
  export const metadata = {
    title: "My Site",
    description: "Welcome to my site",
  };
  ```
