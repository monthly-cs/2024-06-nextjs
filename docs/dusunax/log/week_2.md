# 240616_Memo

참석자: 강석우, 두선아, 김지나, 전소연, 임건, 정래한  
발표자: 전소연님 / 정래한님

---

- 자리
  | 강석우/프론트 | 두선아/프론트 |
  | ------------- | ------------- |
  | 김지나/프론트 | 전소연/프론트 |
  | 임건/백엔드 | 정래한/프론트 |
- 출발 위치: 인천, 의정부, 선정릉, 신설동

---

### 1-전소연님

- rewrite
  - soft navigation
  - hard nviation으로 접근 시, default 필요
  - 병렬 라우팅
    - page
    - default
- catch-all,
  - optional catch-all ⇒ 매개변수가 없는 경우도 처리 (undefined)
- app/news/[…slug]/image
- suspense 사용 경우: 동적 라우트가 이미 해당 페이지에 있는 경우 [[…parmas]]
- 마이크로 서비스: 마이그레이션 예시
  - 회원가입 앱: vue 에서 next14
    - 라이브러리 사용x
      - react-hook-form
      - tanstack query
- 낙관적 업데이트
  - useOptimtstic 예시

### 2-정래한님

- @ ← 요거 지시자
- 모달 warpper 컴포넌트를 클라이언트로!
  - 모달 래퍼 안에 dimmed 영역 (onCick)
- 미들웨어 - auth처리
- 서버 컴포넌트 비동기로 “데코데이터”할 수 있음
- loading
  - 그 전 데이터가 있다면? 그 전 컴포넌트!
- formAction (클라이언트 액션)
- useActionState로 유효성 검사 어떻게 작성할까?
- 모달
  - 형제
  - 감싸는 부모의 형제
- 관심사 & 기능 분리
  - BFF: nest
- 서버 컴포넌트
  - S3 빌드, 정적 리소스, // supabase
    - 버셀 무료 CDN
  - 정적 로딩
- fetch
  - 모노레포 / 유틸 함수
- 사용자 인증
  - 서버 컴포넌트 cookies
