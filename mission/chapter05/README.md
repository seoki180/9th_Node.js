미션 수행

https://github.com/seoki180/9th_node_practice/tree/feature/chapter05



1-1. 특정 지역에 가게 추가하기 API

**1-2. 가게에 리뷰 추가하기 API**

* 리뷰를 추가하려는 가게가 존재하는지 검증이 필요합니다.

1-3. 가게에 미션 추가하기 API

**1-4. 가게의 미션을 도전 중인 미션에 추가(미션 도전하기) API**

* 도전하려는 미션이 이미 도전 중이지는 않은지 검증이 필요합니다.


**Controller → Service → Repository → DB로 이어지는 요청 흐름을 정리해 주세요.**


도메인 주도로 개발(store, user, mission 엔드포인트 사용)

라우터 -> 컨트롤러 -> 서비스 -> 모델 -> DB 로 이어지는 서비스 흐름 작성, 


회원가입 API에 비밀번호 해싱 과정을 추가해 주세요.

-> crypto 라이브러리를 사용하여 비밀번호 해싱, salt 생성 과정을 추가함
