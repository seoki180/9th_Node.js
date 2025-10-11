* ES
  ECMA scrpit의 약자로 JS의 표준사양이다. ecma(European Computer Manufacturers Association)에서 1997년부터 표준화되어 관리하기 시작했다. ECMAScript는 구문(Syntax), 타입(Type), 문(statement), 키워드(keyword), 예약어(reserved words), 연산자(operator), 객체(object) 등 언어의 핵심적인 문법과 기능을 정의한다.
* ES6

  * ES6의 주요 변화 및 특징
    ES6는 2015년 발표되어 let, const, arrow function, class, Promise, module을 도입하였다.
    ES5에서 패키지를 불러올때 module이 없었기에 commonJS방식으로 불러왔다.

    ```json
    var fs = require("fs") // commonJS

    <->

    import fs from "fs" // es6
    ..
    {
    	"type" : "module" //package.json에서 명시적으로 module을 사용한다고 해야함
    }
    ```
  * ES6를 중요시 하는 이유
    module을 사용할때 commonJS와는 달리 es6방식은

    1. 정적분석 : import는 compile-time에 분석되어서 사용하지 않는 코드를 런타임전에 제거해 번들 크기를 줄일수 있다.
    2. 비동기 로딩가능 : es6 module은 비동기적으로 로딩되도록 설계되어서 브라우저에서 효율적인 네트워크 처리가 가능하다. 특히 동적으로 import할 수 있어, 코드 분할에 유용하다.
    3. 자체적인 모듈 스코프 : es6 module은 파일마다 자체적으로 모듈스코프를 가져 전역 변수 오염방지에 유리하다.

    모듈 측면 외에도 현대적인 문법(let, const 도입, arrow funciton ⇒ 사용, class 진관적 문법 제공), 비동기 처리 개선(async, await)등으로 안정적이고 구조적인 코드를 작성할 수 있게 되었다.
* ES Module

  ```json
  //commonJS
  function hello(){ //greet.js
      return "hello"
  }

  module.exports = hello
  ----------------------------------------------------------------
  const hello = require("./greet") //index.js

  console.log(hello())
  ```
  commonJS에서는 module.exports - require을 사용한다. require할때는 파일의 확장자를 따로 명시하지 않아도 되는데 이는 require()로 모듈을 불러올 때 Node.js가 내부적으로 자동으로 확장자를 유추해서 찾는 로직을 가지고 있기 때문이다.

  또한 commonJS는 패키지를 불러오는 시점이 런타임이고, 동기방식으로 불러온다.

  ```json
  //ES Module
  export default function hello(){ //greet.mjs
      return "hello"
  }
  ----------------------------------------------------------------
  import hello from "./greet.mjs"; //index.js

  console.log(hello())
  ```
  ES module에서는 export(export default) - import를 사용한다. ESM을 쓰려면 package.json에 "type": "module"을 추가하거나 .mjs 확장자를 사용해야 한다. 또 ES module은 패키지를 불러오는 시점이 컴파일 타임이기 때문에 정적분석이 가능하고, 비동기 방식으로 불러온다.
* 프로젝트 아키텍처

  * 프로젝트 아키텍처가 중요한 이유
    **유지보수성 향상** : 코드가 체계적으로 구성되어 있어 수정 및 확장이 용이.
    **협업 효율성** : 역할 분리가 명확해 팀원 간 협업이 쉬움.
    **테스트 용이성** : 레이어가 나뉘어 있어 단위 테스트가 간단.
    **재사용성 증가** : 공통 로직이 재사용 가능한 구조로 분리됨.
    **스케일링 준비** : 구조적으로 성장 가능한 코드 베이스 구축 가능.
  * Service-Oriented Architecture(Service Layer Pattern)
    ![image.png](attachment:1e301780-c835-4b97-b6b0-9a0ce08196c3:image.png)
    **정의** : 도메인 로직을 Controller, Repository 등과 분리해 Service 레이어에 모아 관리.
    Service Layer Pattern은 코드를 모듈화하고 재사용성을 높여서, 애플리케이션을 쉽게 확장하고 유지보수 하기 위해 사용한다.
    즉, 각 계층이 독립적으로 개발되고, 테스트될 수 있으며, 따라서 애플리케이션 전체의 유지보수성이 향상된다.
    각 계층은 분명 서로 상호작용이 일어나지만, 다른 계층에서 발생하는 일에 대해선 전혀 몰라도 된다(캡슐화)
    **장점** :
    비즈니스 로직과 웹 레이어 분리
    로직 재사용성 증가
    테스트 코드 작성 용이
    **구성 예시** :
    Service: 핵심 비즈니스 로직 처리
    Controller: 요청 처리 및 응답 반환
    Repository/DAO: 데이터 접근 처리
  * MVC 패턴
    ![image.png](attachment:e1a42a59-6083-495e-a76b-7028fed1fb9a:image.png)
    ![image.png](attachment:de742c89-d773-4f69-98e3-0f626292f9e8:image.png)
    **정의** : 사용자 인터페이스, 데이터 처리, 비즈니스 로직을 분리하는 전통적인 구조
    좀 더 쉽고 편리하게 사용할수 있게 만드는 디자인패턴중 하나이다.
    순서는 다음과 같다.
    User가 컨트롤러를 조작한다. → 컨트롤러는 모델에서 데이터를 가져온다 → 가전온 정보를 바탕으로 뷰를 제어해서 User에게 보여준다.
    model, view는 서로에 대해서 어떤 정보도 알지 말아야 한다.
    모든 데이터의 변경과 통제는 controller를 통해서 이뤄져야한다. 즉 애플리케이션의 메인로직을 controller가 담당한다.
    **구성** :
    **Model** : 데이터 구조 및 DB 연동
    **View** : 사용자에게 보여지는 UI
    **Controller** : 사용자 입력 처리 및 흐름 제어
    **장점** :
    역할 분리가 명확
    UI 변경이 로직에 영향 없음
    유지보수 쉬움
    프레임워크에서 mvc를 채용한 기술은 AngularJS, Python Django등이 있다.
  * 그 외 다른 프로젝트 구조
    **Hexagonal Architecture (Ports & Adapters)** :
    의존성을 내부 도메인 중심으로 설계
    외부 시스템(DB, API 등)은 Adapter로 분리
    ![image.png](attachment:bada3d1f-7928-45a7-80b7-d6d8d91a12eb:image.png)
    **Clean Architecture** :
    내부(core)에 가까운 계층일수록 순수 비즈니스 로직만 포함
    외부 프레임워크에 의존하지 않도록 구성
    ![image.png](attachment:50b267ab-4ef8-430b-ae69-d72c5ece0485:image.png)
    **Layered Architecture** :
    일반적인 3계층 구조 (Presentation → Business → Data Access)
    ![image.png](attachment:b4ee2286-8a02-489d-8132-2b71858fd1bf:image.png)
* 비즈니스 로직
  **정의** : 소프트웨어 시스템이 해결하려는 실제 문제(=도메인 문제)를 어떻게 처리할 것인가에 대한 규칙과 흐름. 즉, 이 시스템이 어떤 일을 하고, 그걸 어떤 방식으로 처리할지에 대한 핵심 로직
  **위치** : 보통 Service Layer 혹은 Domain Layer에 위치
  **예시** : 온라인쇼핑몰 서비스에서

  * 결제 시 할인율 계산
  * 재고 수량 확인
  * 포인트 적립
  * 주문 상태 변경

  **중요성** :
  제품의 가치를 결정하는 로직
  변경 가능성이 높아, 잘 분리해두는 것이 중요
* DTO
  Data Transfer Object, 시스템 내부 혹은 시스템간 데이터를 주고받을때 필요한 데이터를 담아서 전달하는 객체
  DB entitiy를 외부에 노출하면 보안/유지보수의 문제가 생기니 객체로 담아 외부에서 볼 수 없게 한다. 또한 프론트엔드에 맞는 형식으로 데이터를 가공해 응답할 수 있도록 해준다.
  Entity와 DTO의 차이점

  | **구분** | **목적**                  | **변경 가능성** | **예시**      |
  | -------------- | ------------------------------- | --------------------- | ------------------- |
  | Entity         | DB와 직접 매핑되는 클래스       | 있음                  | User, Product       |
  | DTO            | 계층 간 or 외부와의 데이터 전달 | 있음                  | UserDTO, ProductDTO |
  |                |                                 |                       |                     |
