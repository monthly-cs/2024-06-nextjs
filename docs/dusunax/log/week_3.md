# 240609_Memo

참석자: 두선아, 김지나, 정래한, 임건, 전소연  
발표자: 정래한님 / 김지나님

---

- 이미지 컴포넌트
  - 로더
    - 컬리
    - 페이지마다 필요한 해상도
  - 이미지 저장소에 사용하는 image src
    - 문자열 주소로 직접 입력
  - 자체적 이미지 서버가 있으면 업로드 시점에 size별 이미지 저장 등의 처리가 가능하다.
    - 색상값을 구하기? blur 아니고 하나의 색상 ⇒ 평균색을 구한다
      - 평균색? 이미지 컬러의 바운더리 // 정책
        - #ddd
        - 배치 돌려서 서버리스 ex) 란다로 처리
          - 이미지 ⇒ API 게이트웨이 ⇒ 람다 전달
          - 개별 이미지 업로드x 배치로 이미지를 한 번에 올린다.
  - 캐싱은 어디에서 되는가?
    - \_next
      - 데이터 캐시 보관
        - 껐다 켜도 데이터 캐시 유지
    - 클라이언트는 브라우저에 저장