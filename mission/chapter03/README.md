* [ ] Soft Delete가 무엇인지 찾아보시고 soft delete에는 어떠한 HTTP Method가 들어가면 좋을지 적어주세요
  * 내용들을 간단하게 정리하여 주세요
    삭제방법은 soft와 hard 두가지 방법이 있다.
    서비스를 구현할때 회원이 탈퇴한다는건 매우 큰 문제이다.
    만약 회원이 탈퇴를 결심했다가 나중에 다시 내 계정을 복구하고 싶다면? 이미 DB에서 정보를 날렸다면 이를 복구하긴 쉽지 않을 것이다.
    이때 사용되는 방법이 soft Delete이다.
    soft delete는 실제로 데이터를 지우는 대신 DB에 is_delete 와 같은 플래그 칼럼을 추가해 데이터가 '삭제된것 처럼' 표현하는 방식이다.
    ![img](https://velog.velcdn.com/images/seoki180/post/84b0ad89-e20b-4855-918c-da7d1bf9023c/image.png)
    soft delete는 휴지통에 넣기, hard delete는 휴지통 비우기에 비유 할 수 있다.
    따라서 delete는 다음처럼 구현 할 수 있다.

    1. Hard Delete → DELETE /posts/1
       실제로 DB에서 데이터가 사라지게 된다.
    2. Soft Delete → PATCH 또는 POST + 별도 URI
       데이터를 삭제된 상태로 바꾼다 라는 개념에서 접근하면 PATCH나 POST도 자연스러워 보인다.

    ```
    PATCH /posts/1
    Body: { "deleted": true }

    POST /posts/1/restore
    ```
    혹은 다른 방법도 생각해볼 수 있다.

    1. SOFT DELETE만 구현 -> 일정 시간이 지나면 자동으로 HARD DELETE
       이 방법은 URI도 DELETE /posts1/ 이렇게 하나만 구현해놓고 실제 데이터 삭제는 DB와 서버에서 담당하면 되니 클라리언트 측면에서 신경쓸게 없어진다.
    2. 보관방식
       DELETE /posts1/ 이렇게 보내면 실제로 삭제하지 않고 archive 테이블을 따로 만들어 관리자가 이를 관리하고 추후 삭제하도록 구현한다.
* [ ] 컨트롤 URI에 대해 조사해주시고 어떠할 때 사용이 가능한 지 예시를 들어 설명해주세요.
  * 내용들을 간단하게 정리하여 주세요
    컨트롤 URI 는 리소스를 제어하기 위한 URI이다. 주로 네트워크나 API 설계, IoT, UPnP(Universal Plug and Play), RESTful 시스템 등에서 사용된다.
    일반적인 URI는 리소스를 “조회”하거나 “접근”하는 데 사용되지만,
    Control URI는 리소스를 제어하거나 상태를 변경하기 위한 엔드포인트로 사용된다.
    IoT 에서는 다음과 같이 전구의 불을 끄는 제어가 가능하고

    ```
    POST <http://smart-home.local/device/light123/control>
    Body: { "command": "turnOff" }
    ```
    UPnP에서는

    ```
    Control URL:
    <http://192.168.1.10:1400/MediaRenderer/AVTransport/Control>
    ```
    REST API에서는 다음처럼 유저를 비활성화 시킬수 있다.

    ```
    PATCH /users/123/deactivate
    ```
    ![image.png](attachment:14e76be4-ccfa-4059-9da9-8b35162c02cd:image.png)
* [ ] [https://learn.microsoft.com/ko-kr/azure/architecture/best-practices/api-design](https://learn.microsoft.com/ko-kr/azure/architecture/best-practices/api-design) - 문서를 읽고 주요 내용을 간단히 정리해주세요.
  * 내용들을 간단하게 정리하여 주세요
    Microsoft 웹 API 디자인 모범사례는 RESTful 웹 api를 설계할때 고려할 원칙과 지침을 제공해준다.
    문서에서는 다음과 같이 올바른 API 디자인에 대해 다음과 같이 얘기해준다.

    1. 리소스를 중심으로 API 디자인 구성 + HTTP 메서드 측면에서 API 작업 정의

    > 웹 API가 표시하는 비즈니스 엔터티에 집중해야 합니다. 예를 들어 전자 상거래 시스템에서는 기본 엔터티가 고객과 주문입니다. 주문 정보가 포함된 HTTP POST 요청을 전송하여 주문 만들기를 구현할 수 있습니다. HTTP 응답은 주문이 성공적으로 수행되었는지 여부를 나타냅니다. 가능하다면 리소스 URI는 동사가 아닌 명사를 기반으로 해야 합니다.
    >

    라고 설명하고 있다. 설명에 따르면 URI는 그 자체가 리소스를 표현해야하고, 그 자체가 행위를 나타내선 안된다고 말하고 있다. 즉, 동작(행위)에 대한 내용은 HTTP method로 표현을 하고, endpoint는 명사로만 이루어져야 한다는 것이다.
    ![](https://velog.velcdn.com/images/seoki180/post/336b60cd-ca97-477b-b2af-c2e20ab98c2f/image.png)
    리소스를 다음처럼 customers 단일 명사로만 표현하고 이외의 행동은 HTTP method로 정의하면 된다는 것이다.

    1. 데이터 필터링 및 페이지 매기기
       단일 URI를 통해 리소스를 표현한다면 하위집합만 필요할때도 대량의 데이터를 가져올 수도 있다.
       예를 들어 일정 가격이상의 주문을 찾아야한다고 하면 클라이언트는 /order URI에 접속해 모든 주문을 받아온 다음 프론트엔드에서 값을 필터링해야할것이다.
       이 대신 /orders?minCost=n처럼 API가 URI의 쿼리 문자열에서 필터 전달을 허용할수 있다. 그러면 Web API가 쿼리 문자열의 minCost 매개 변수를 구문 분석 및 처리하고 서버 쪽에서 필터링된 결과를 반환하게 한다.
    2. HATEOAS를 사용하여 관련 리소스 탐색
       REST의 기본 특징중 하나는 URI체계를 알고 있지 않아도 전체 리소스를 탐색할수 있게 해야한다는 것이다. 따라서 필요한 다음 행동(요청)의 정보를 서버가 응답 안에 하이퍼링크 형태로 제공한다.즉, 클라이언트는 처음 엔드포인트만 알고 있어도, 그 응답 안에 포함된 링크들을 따라가면서 필요한 리소스를 탐색하고 조작하게 할 수 있다.

    ```
    {
      "id": 1,
      "name": "Alice",
      "email": "alice@example.com",
      "_links": {
        "self": { "href": "/users/1" },
        "update": { "href": "/users/1", "method": "PUT" },
        "delete": { "href": "/users/1", "method": "DELETE" },
        "orders": { "href": "/users/1/orders" }
      }
    }
    ```
    1. RESTful 웹 API 버전 관리
       > Web API가 정적 상태를 유지할 가능성은 매우 낮습니다. 비즈니스 요구 사항이 변경됨에 따라 자원의 새 컬렉션이 추가될 수 있으므로, 리소스 간의 관계가 변할 수 있으며 리소스 데이터의 구조가 수정될 수 있습니다.(...) 따라서 새 클라이언트 애플리케이션이 새 기능과 리소스의 장점을 이용할 수 있도록 하면서도 기존 클라이언트 애플리케이션이 변경되지 않고 계속 작동할 수 있도록 해야 합니다.
       >

    API자체에도 버전상태관리를 통해 특정버전으로 요청을 지정하여 서비스를 관리할 수 있다.
    상태관리에는 크게

    1. URI 버전 관리
    2. 쿼리 문자열 버전 관리
    3. 헤더 버전 관리
    4. 미디어 형식 버전 관리

    방식으로 할 수 있다.
