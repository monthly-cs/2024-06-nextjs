# 2주차 section4,5,6
2024.06.16 김지나

# Section 4: 라우팅 및 페이지 렌더링 - 심층 분석
## "Not Found" 페이지 표시하기

```
import { noutFound } from 'next/navigation';

export default function DetailPage({parmas}) {
    if(params.id) {
        notFound();
    }
    ...
}
```

`next/navigation`의 `notFound` 메소드를 이용해 not-found.js 페이지를 트리거 할 수 있다.


## 병렬 라우팅
> 병렬 라우팅(Parallel Routing)은 동시에 또는 조건에 따라 동일한 레이아웃에서 하나 이상의 페이지를 렌더링 할 수 있다. 대시보드나 소셜 사이트의 피드와 같은 매우 동적인 앱 색션에서는 병렬 라우팅을 사용하여 복잡한 라우팅 패턴을 구현할 수 있다.


ex) 대시보드, 소셜 사이트 피드 등 매우 동적인 섹션에 유리하다.

병렬 라우팅은 `@폴더명`의 형태의 명명된 슬롯을 사용한다. 슬롯은 공유된 상위 레이아웃에 props로 전달된다. 

```
 export default function ArchiveLayout({
  children,
  archive,
  latest
}) {
  return (
    <>
      {children}
      {arhive}
      {latest}
    </>
  )
}
```

`@archive/[year]`, `@latest` 두개의 병렬구조가 이와 같은 형태로 있을 때 `/archive/2024` 에 접속하면 페이지가 잘 나오지만 그 페이지에서 새로고침을 시도할 시엔 에러페이지로 이동한다. 이는 NextJs가 슬롯 내에서 탐색 유형이 달라지기 때문에 발생한다.

- **Soft Navigation**: 클라이언트 측 Next.js는 부분 렌더링을 수행하여 슬롯 내의 하위 페이지를 변경하는 동시에 다른 슬롯의 활성 하위 페이지가 현재 URL과 일치하지 않더라도 현제 페이지를 유지한다.

- **Hard Navigation**: 전체 페이지 로드(브라우저 새로 고침) 후 Next.js는 현재 URL과 일치하지 않는 슬롯의 활성 상태를 확인할 수 없어 404 에러를 내뱉는다. 이경우 병렬 라우팅을 사용한 슬롯들의 하위 페이지를 일치시키거나 `default.js` 를 사용하면 된다.

## `default.js`
- 초기 로딩 또는 전체 페이지를 다시 로드할 때 일치하지 않는 슬롯에 대한 폴백으로 렌더링할 파일을 정의할 수 있다.
- `default.js` 는 `page.js`를 대체할 수 있다.


## Catch-All 라우트
`[[...segmentName]]`와 같은 형태의 이중 대괄호 폴더를 사용하면 선택적 catch-all 라우트를 만들 수 있다. 하위 페이지들의 모든 세그먼트를 params로 받아볼 수 있다. 

| Route              | Example URL         |   Params |
| -------------------- | -------------------------------------- | ----------------------------------- |
| pages/shop/[...slug].js |/shop/a  | { slug: ['a'] } |
| pages/shop/[...slug].js |/shop/a/b  | { slug: ['a', 'b'] } |


？ `[...segmentName]` 과 `[[...segmentName]]`의 차이점은 `[[...segmentName]]`와 같이 이중 대괄호를 쓸 때는 `선택적 catch-all 라우트`로 하위 세그먼트가 없어도 params로 세그먼트에 접근이 가능하다. (하위 세그먼트가 없을 시에 prams: { slug: [] })

### Throwing Error
- useError를 사용하면 error 핸들링을 할 수 있다.
- error.js는 클라이언트 측의 에[러를 잡을 수 있어야 하기 때문에 무조건 클라이언트 컴포넌트여야 한다.

```
// page.js

  if (selectedYear) throw new Error('Invalid filter.');
```

```
// error.js
'use client'

import { useError } from "next/error";

export default function ErrorPage() {
  const error = useError();
  return <div>{error.message}</div>;
}

```


## Intercepting Route
`()가로챌 페이지 폴더명` - 내부 내비게이션 요청을 가로채기 하여 현재 레이아웃 내에서 다른 부분의 경로를 로드할 수 있다. 이렇게 하면 다른 전환 없이 내용을 표시할 수 있는 장점이 있다.

예를 들어 화면의 상호작용으로 페이지 이동이 일어났을 때는 모달을 띄우고, url 검색이나 새로고침 하여 페이지에 접속했을때는 페이지를 렌더링 할 수 있다.


- `(.)` 같은 레벨의 세그먼트
- `(..)` 한 단계 위의 레벨의 세그먼트
- `(..)(..)` 두 단계 위의 레벨의 세그먼트
- `(...)` root 레벨(app)의 세그먼트



## 라우트 그룹
라우트 그룹을 사용하면 URL에 영향을 주지 않고 관련 경로를 함계 유지하는 그룹을 만들 수 있다. 
`(folderName)` 괄호로 묶은 폴더는 URL에서 생략된다. 

## middleware.js
페이지가 렌더링되기 전에 서버 측에서 실행되는 함수이다. (=== 특정 요청 전 함수 수행 가능)
Request와 Response 객체에 접근할 수 있다.

아래와 같은 상황에서 사용될 수 있다.
- 페이지 렌더링 전에 인증을 확인하거나 요청을 확인
- 요청 데이터를 사전에 처리하거나 특정 API요청을 수행하거나 캐시 관리
- 요청에 대한 응답을 변환하거나 에러 처리

```
// root에 위치한다.
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'
 
// This function can be marked `async` if using `await` inside
export function middleware(request: NextRequest) {
  return NextResponse.redirect(new URL('/home', request.url))
}
 
// See "Matching Paths" below to learn more
export const config = {
  matcher: '/about/:path*',
}
```

## 라우트 핸들러
특정 경로에 대한 사용자 정의 요청 처리기를 생성할 수 있다. 웹의 요청(request)와 응답(response) API를 사용하여 이를 구현하며 app 디렉토리 내에서만 사용이 가능하다.

`route.js` 파일을 통해 구성할 수 있다. 이 파일은 특정 경로에 대한 요청을 처리하기 위한 서버 사이드 로직을 포함하며 각각의 HTTP 요청 메서드(GET, POST 등)에 대한 별도의 함수를 정의할 수 있다.

```
// app/api/route.js
export async function GET(request) {
    // GET 요청을 처리하는 로직
}

export async function POST(request) {
    // POST 요청을 처리하는 로직
}
```

이러한 구조를 통해 각 HTTP 메서드에 맞게 요청을 처리할 수 있으며, 추가적으로 필요한 메서드도 유사한 방식으로 구현 가능하다.

# Section5: 데이터 가져오기 (fetching)
Next 프로젝트 내에 별도의 backend 프로젝트를 설정할 수 있다.

서버 컴포넌트에서 fetch 함수를 쓸 수 있는 이유
- Node.js가 서버 츨 코드를 실행하는 것을 지원한다.
- Next.js가 fetch함수를 확장하여 몇 가지 추가 캐싱 관련 기능을 추가했다. (추후 강의에서 다룸)


# Section 6: 데이터 변이 - 심층 분석
### Server Action
서버액션은 사실 Next의 고유 기능이 아닌 React 자체에서 지원하는 기능이다. 

참고 > https://react.dev/reference/rsc/server-actions
```
async function onSubmit(formData){
    'use server'

const meal = {
    title: formData.get('title)
}
}

...

return (
    ...
    <form action={onSubmit}>
)
```

기존의 form태그의 action은 브라우저에 내장된 기봉 동작이며 일반적으로 URL을 정의했다.
formAction을 지원하는 React 버전을 사용하거나 Next 사용 시 action은 다르게 동작한다. 브라우저의 기본 동작을 막고 양식이 제출될때 정의된 함수를 작동시킨다.


### "use server"는 서버 측 실행을 보장하지 않는다.
"use server" 지시어는 서버 액션이 되어야 할 것은 Next.js에 "말하기만" 하는 것이다. 이는 **코드가 서버에서 실행된다는 것을 의미하거나 보장하지 않는다.**

클라이언트 측에서 절대 실행되어서 안 되는 코드가 있다면 `server-only` 패키지를 사용해야 한다.

## 낙관적 업데이트
> 낙관적 업데이트란 유저에게 더 미려한 경험을 제공하기 위한 접근법

사용자에게 즉시 새로운 상태를 보여주고 
리액트의 useOptimistic hook을 사용하여 낙관적 업데이트를 실행할 수 있다.
즉 서버에서 데이터를 처리하기 전에 클라이언트 측에서 먼서 데이터를 변경하고 서버 업데이트가 끝나면 서버 측 상태와 다시 동기화 하는 것

```
 const [optimisticState, addOptimistic] = useOptimistic(
    state,
    // updateFn
    (currentState, optimisticValue) => {
      // merge and return new state
      // with optimistic value
    }
  );
  ```
  - state: 작업이 대기 중이지 않을 때 초기에 반환될 값
  - updateFn(currentState, optimisticValue): 현재 상태와 addOptimistic에 전달된 낙관적인 값을 취하는 함수로, 결과적인 낙관적인 상태를 반환한다. **순수 함수여야 한다.**     
    - currentState : 기존 게시물 상태
    - optimisticValue: 업데이트에 필요한 데이터

- optimisticState: 결과적인 낙관적인 상태. 작업이 대기 중이지 않을 때는 state와 동일하며, 그렇지 않은 경우 updateFn에서 반환된 값과 동일하다.
- addOptimistic: addOptimistic는 낙관적인 업데이트가 있을 때 호출하는 디스패치 함수. 어떠한 타입의 optimisticValue라는 하나의 인자를 취하며, state와 optimisticValue로 updateFn을 호출한다
