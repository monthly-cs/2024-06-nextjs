## 1주차 - section3 NextJS 핵심 (앱 라우터)
2024.06.09 김지나
# App router
<img width="700" alt="스크린샷 2024-06-09 오전 11 26 32" src="https://github.com/zzinao/test/assets/77870077/42ab26a6-6444-40a5-b799-114015ce08b6">

- NextJS는 `파일 기반 라우팅`을 지원하여 폴더/page.js를 생성하여 url을 지정할 수 있다.

## 동적 라우팅
<img width="477" alt="스크린샷 2024-06-09 오전 11 27 09" src="https://github.com/zzinao/test/assets/77870077/9e69a6e5-1a0b-400a-9a6c-19ff65728bdf">

- `대괄호`를 이용해 `동적 라우팅`을 구현할 수 있다.
- `대괄호` 안의 `params`는 컴포넌트 내에서 `props`로 받을 수 있다.

## layout.js
<img width="704" alt="스크린샷 2024-06-09 오전 11 27 43" src="https://github.com/zzinao/test/assets/77870077/a5819a02-1a12-4fa1-abec-3bb6b790b659">

```jsx
// layout.js 형식
export const metadata = {
	title: 'MextJS',
	description: 'MextJS App'
};

export default function RootLayout({ children }) {
	return (
		<html lang="ko">
			<body>{children}</body>
		</html>
	)
}
```

- page.js를 감싸는 껍데기를 정의
- 최소 1개의 app폴더를 위한 근본된 layout.js가 필요하다.
- `<head>` 가 없는 이유는 `metadata` 라는 변수를 통해 설정되거나 NextJS로 인해 이면에서 자동으로 설정되기 때문이다.

## loading.js

`page.js`와 같은 경로에 `loading.js`를 추가하면 페이지 이동 시 로딩 페이지를 보여줄 수 있다.

**:Suspense & Streamed Response 기능을 이용하여 로딩 세분화 하기**

React의 Suspense 기능을 이용하면 컴포넌트 단위로 로딩을 세분화 시킬 수 있다.

```
async function Meals () {
   const getMeals = await getMeals();

   return <MealsGrid meals={meals}/>
}

export default function MealsPage (){
    return(
        <>
        <Hedader/>
        <Suspense fallback={Loading}>
        <Meals/>
        </Suspense>
        </>
    )
}

```

## error.js

- `error.js`를 생성하면 그 페이지 내에서 발생하는 에러화면을 `error.js`로 대체할 수 있다.
- `error.js`는 클라이언트 컴포넌트여야 한다. NextJs는 페이지가 서버에서 렌더링 된 후에 발생하는 오류를 포함한, 해당 컴포넌트의 모든 오류를 잡을 수 있도록 보장하기 때문이다.

## not-found.js

- `not-found.js`를 생성하면 유저가 잘못된 url을 입력했을 시 그 `not-found.js` 페이지를 보여준다.

## **보호된 파일명**

- `page.js` - 신규 페이지 생성
- `layout.js` - 형제 및 중첩 페이지를 감싸는 신규 레이아웃 생성
- `not-found.js` - Not Found 오류에 대한 fallback 페이지
- `error.js` - 기타 오류에 대한 fallback 페이지
- `loading.js` - 형제 또는 중첩페이지(또는 레이아웃)가 데이터를 가져오는 동안 표시되는 fallback 페이지
- `route.js` - API 경로 생성(즉, jsx 코드가 아닌 데이터를 반환하는 페이지, ex: JSON형식)

# 서버 컴포넌트 ServerComponent
<img width="706" alt="스크린샷 2024-06-09 오전 11 31 05" src="https://github.com/zzinao/test/assets/77870077/07ef1984-d47d-4b38-8ead-53d064dc47d1">

기본적으로 NextJS는 `서버 컴포넌트`로 동작한다.

서버 컴포넌트란 NextJS가 컴포넌트를 서버에 렌더링하여 초기 HTML을 제공해 렌더링 시 변하지 않는 초기 데이터를 미리 세팅한다. 

⇒ 예시로 page.js에 console.log를 추가하면 브라우저 콘솔에 찍히지 않고 터미널에서 찍히는 것을 볼 수 있다.

### 서버 컴포넌트의 동작 순서
<img width="703" alt="스크린샷 2024-06-09 오전 11 31 39" src="https://github.com/zzinao/test/assets/77870077/01bf3480-157f-42c2-afaf-e1cb73be34f4">

리액트 코드 → 서버에서 HTML로 렌더링 → 브라우저로 보내짐

### 클라이언트 컴포넌트
<img width="706" alt="스크린샷 2024-06-09 오전 11 34 03" src="https://github.com/zzinao/test/assets/77870077/438bf5e0-c10a-46b5-8f84-a2851560a7b2">

기존 바닐라 리액트 앱은 JS코드를 가진 단순한 HTML만 서버로 보내고 클라이언트에서 렌더링 되었다(CSR).
- 클라이언트 컴포넌트는 상태, 이펙트 및 이벤트 리스너를 사용할 수 있어 사용자에게 즉각적인 피드백을 제공하고 UI를 업데이트 할 수 있다.
- 브라우저 API에 엑세스할 수 있다.

NextJS에서 클라이언트 컴포넌트를 사용하려면 컴포넌트 상단에 `'use client'` 라는 키워드를 작성해야한다.

```
'use client' // 👈 이 키워드를 작성해야 Next가 CSR을 진행한다.

import { useState } from "react";

export default function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```
### 클라이언트 컴포넌트 vs 서버 컴포넌트
각 컴포넌트 별로 제공되는 기능이 다르기 때문에 필요에 따라 잘 사용해야한다.
 
<img width="727" alt="스크린샷 2024-06-09 오전 11 39 11" src="https://github.com/zzinao/test/assets/77870077/96a16926-9f2d-421e-9c37-6a863e765d47">

### **클라이언트 컴포넌트의 효율적 사용**

일반적으로 컴포넌트 트리를 가능한 아래로 내려서 필요한 컴포넌트만 클라이언트 컴포넌트로 변환하여, 대부분의 컴포넌트가 서버 컴포넌트로 유지되게끔 하여 서버컴포넌트의 이점을 잃지 않도록 한다.

=> 클라이언트 기능이 필요한 컴포넌트를 분리하여 최소한의 컴포넌트만 클라이언트에서 동작하게끔 만들자

## Next/Link

`a 태그`로 페이지 이동은 현재 페이지를 벗어나 서버에서 새 페이지를 받기 때문에 좋은 방법이 아니다. (단일 페이지 애플리케이션이 아님, SPA)

NextJS에서 제공하는 `<Link>`를 사용하면 SPA처럼 사용이 가능하다.

- url을 입력하여 접속하는 경우 -> 서버에서 렌더링 된 완성된 페이지에 접속
- Link를 사용한 페이지 이동 -> SPA에 머무르는 것을 허용하고 클라이언트 측 자바스크립트 코드로 UI를 업데이트 한다.

엄밀히 말하자면 NextJS 페이지의 내용은 서버에 렌더링 되기 전이지만 클라이언트 측 자바스크리브 코드로 클라이언트 사이드에 업데이트 된다.

=> 두 가지 각각의 장점을 모두 얻을 수 있다.

## Next/Image

기본 img 태그보다 최적화된 기능을 제공한다. 자동으로 이미지를 지연 로딩하고 대체 이미지를 설정하는 프로세스 등을 단순화한다.

- `loading = "lazy"` 이미지가 지연로딩되도록 한다. (실제로 보이는 경우에만 로딩)
- 자동으로 width, hegigth 값 부여 (override 가능)
- `srcset` viewport와 웹사이트를 방문하는 기기에 따라 크기가 조정된 이미지가 로딩되도록 보장
- `priority` 이미지가 로딩될때 필요하지 않은 컨텐츠 변경이나 깜빡임이 없도록 하기 위해 우선적으로 로딩

# Server Action으로 Form 제어

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
`'use server'` 키워드를 함수 내에 작성하면 `Server Action`을 생성하는데 이는 form의 mutation(생성, 업데이트, 삭제)를 서버에서만 실행될 수 있도록 보장해주는 기능이다. API 엔드포인트를 작성하지 않고도 컴포넌트 내에서 비동기 하수를 직접 정의할 수 있다.
`async` 와 함께 동작한다.

기존 form 태그의 action 속성에는 path 경로가 들어갔으나 NextJS에서는 Server Action인 함수로 대체될 수 있다.

1)form이 제출되면 NextJS가 요청을 생성하여 2) NextJS 서버로 보내게 된다. 3) 그 뒤 함수가 실행되고 4) 서버 측에서 form을 제어할 수 있게된다.

form내의 데이터를 함수 인자로 받을 수 있고 `get` 메소드를 통해 값을 불러올 수 있다. input 필드는 name으로 구분된다. 

클라이언트 컴포넌트내에서 직접 `Server Action`을 호출할 수 없다.
```
//actions.js
'user server'

// 이 파일에 정의한 함수들은 모두 server action이 된다.
export async function shareMeal () {
    ..
}
```


## useFormState를 통해서 서버 액션 응답 작업하기
> react19부터는 useActionState로 바뀌었다.
`useFormState`를 사용하면 폼의 로딩 상태를 알 수 있다. 이 hook을 사용하기 위해서는 `클라이언트 컴포넌트`여야 하기 때문에 서버액션과 분리해주어야 한다.
```
// 첫 번째 인자: 폼 서버 액션, 두 번째 인자: 응답 값의 초기값
//state를 통해 현재 form의 상태를 반환한다.
const [state, formAction] = useFormState(shareMeal, {message: null})
...
return (
    <form action={formAction}>
)

```

 ## NextJS의 캐싱 구축 및 이해
 개발 환경에서 
 NextJS가 공격적인 캐싱 작업을 하고 배포 환경을 이해 앱을 준비할 때 그리고 동작할 때 거치는 추가 단계가 하나 있다. 
 앱을 빌드할때 NextJS는 실제로 앱에서 사전 생성될 수 있는 모든 페이지를 모두 사전 렌더링하고 생성하여 기본적으로는 동적 페이지가 아니게 된다.
 
 즉 빌드 프로세스에서 모든 데이터를 불러오고 렌더링 하게되어 배포된 직후부터 모든 페이지가 동작하게끔 한다. 그 다음 NextJS는 사전 렌더링된 페이지를 캐싱하여 모든 방문자들에게 제공할 수 있게 한다.  => 이 방식의 단점은 데이터를 다시 패칭하지 않는다는 것이다. 

**페이지의 캐싱을 키우는 법**

`revalidatePath()` 를 사용하면 
NextJS가 특정 path에 속하는 캐시의 유효성 재검사를 하게 한다. (=== 데이터 재검증)

revalidatePath('/meal', 'page') // 현재 페이지만 재검사

revalidatePath('/meal', 'layout') // 중첨된 페이지 모두 재검사

### 동적 메타데이터 추가
동적 라우팅에서는 정적 메타데이터 방식을 사용하지못한다.
```
// 무조건 generateMetaData 이름으로 작성해야한다.
export async function generateMetaData({params}) {
    const meal = getMeal(params.mealSlug);

    // 유효하지 않은 페이지 접속 시 에러처리를 해주어야 한다.
    if(!meal) return notFound();

    return {
        ///metat data 형식
    }
}
```
