* Swagger
  Swagger는 RESTful API를 문서화하고 시각화하며 테스트할 수 있는 툴 세트이자, 그 기반이 되는 OpenAPI Specification(OAS)을 사용하는 대표적인 구현체다.

  | Swagger UI      | 브라우저에서 API 문서를 시각적으로 확인하고 테스트할 수 있는 도구 |
  | --------------- | ----------------------------------------------------------------- |
  | Swagger Editor  | OpenAPI 명세를 YAML 또는 JSON으로 작성할 수 있는 웹 기반 에디터   |
  | Swagger Codegen | OpenAPI 명세를 기반으로 서버/클라이언트 코드를 자동 생성          |
  | Swagger Hub     | API 문서를 협업하고 배포할 수 있는 클라우드 기반 통합 플랫폼      |

  swagger를 사용함으로서


  * 개발자, 프론트엔드, QA, 디자이너 모두가 **하나의 문서를 공유**하여 협업
  * API를 직접 실행해보면서 **빠르게 테스트**
  * 명세 기반으로 **자동화된 문서, 클라이언트, 서버 코드 생성**

  와 같은 장점들이 있다.

  ## 추가정리

  swagger로 문서를 만드는 방법은 크게 두가지로 나뉘는데

  1. 직접 json, yaml 문서 작성
  2. 코드기반문서 생성

  이 있다.
  1번 방법의 경우는 말 그대로 직접 OAS내용을 문서로 작성을 하고 SwaggerUI가 이 파일을 불러와서 문서를 생성하는 방식이다.

  ```jsx
  // openapi.yaml
  paths:
    /users:
      get:
        summary: Get all users
        responses:
          200:
            description: OK
  ```
  ```jsx
  // server.js
  app.use(
    "/docs",
    swaggerUi.serve,
    swaggerUi.setup(undefined, {
      swaggerOptions: { url: "/openapi.yaml" },
    })
  );
  ```
  Swagger Editor ( [https://editor.swagger.io](https://editor.swagger.io) ), Swagger UI등의 툴을 사용한다.
  2번 방법은 또 두가지로 나눌수 있는데
  2-1 Swagger JSDoc 스타일 주석(Open API 2.0, 3.0지원)
  @Swagger annotation을 사용해 JSDoc안에 YAML포맷으로 작성한다. swagger-jsdoc이 이 주석을 파싱해서 문서로 변환시켜준다.

  ```jsx
  /**
   * @swagger
   * /users:
   *   get:
   *     summary: Get all users
   *     responses:
   *       200:
   *         description: OK
   *         content:
   *           application/json:
   *             schema:
   *               type: array
   *               items:
   *                 $ref: '#/components/schemas/User'
   */
  router.get("/users", userController.getAllUsers);
  ```
  2-2. Swagger autogen기반  주석
  swagger-autogen이 소스코드를 기반으로 가볍게 문서를 만들어준다.
* OpenAPI
  open API와 OpenAPI는 둘이 다른 개념이다

  1. open API란 말 그대로 열려있는 API를 뜻한다. 즉, 누구나 사용할 수 있도록 endpoint가 개방되어 있다면 open API인것이다. 예를들어 공공데이터포털의 API혹은 카카오의 open API가 있다.
     [공공데이터 포털](https://www.data.go.kr/)
     [Kakao Developers](https://developers.kakao.com/)
  2. 하지만 여기서 다루고있는 OpenAPI는 전혀 다른 개념으로 REST API를 기계가 읽을 수 있도록 명세화한 JSON 혹은 YAML 형식의 표준 포맷을 의미한다 따라서 OpenAPI Specification(OAS)라고도 부른다.
     <aside>
     💡
     OAS는 RESTful API의 작동 방식을  **개발자들이 쉽게 이해하고 문서화할 수 있도록 도움을 줘요** . API의 구조와 접근 방식을 쉽고 상세하게 설명하기 때문이죠. **JSON이나 YAML 형식으로 작성**되어 있기 때문에 코드나 문서 없이도 전체 구조를 이해하기 쉬워요. 개발 언어에 구애받지도 않고요. API를 사용하기 위한  **인증 방법** , **엔드포인트**와 **요청 및 반환할 수 있는 데이터** 등이 명세에 포함됩니다. 다음과 같은 형식이에요.
     [https://docs.tosspayments.com/resources/glossary/oas](https://docs.tosspayments.com/resources/glossary/oas)
     </aside>

  그럼 REST API에서 API의 구조나 접근을 쉽고 상세하게 설명한다고 하면 swagger와 OpenAPI는 같은 개념으로 보인다. 하지만 둘은 별개의 개념이다.

  > OpenAPI = Specification
  >
  > Swagger = Tools for implementing the specification
  >

  swagger 공식 홈페이지에서 말하기를 OpenAPI는 사양의 이름으로, microsoft, google, IBM등의 조직이 잠여하는 OpenAPI Initiative에 의해 개발되고 발전된다.
  반면 swagger는 OpenAPI specification을 구현하기 위해 가장 잘 알려져있고, 널리 사용되는 도구의 이름인 것이다.
* OpenAPI Component
  **OpenAPI Component**는 OpenAPI Specification(OAS) 문서에서 **반복되는 데이터 구조나 요소를 재사용**하기 위해 정의하는 **공통 리소스 블록**이다.
  주로 components라는 키 아래 정의되며, 요청/응답/스키마 등 여러 곳에서 **참조($ref) 방식**으로 재사용된다.
  OAS를 components를 활용해 구현한다면 다음과 같은 장점이 있다.

  1. 중복 제거 : 같은 스키마를 선언하고 여러 경로에서 사용할 경우 재사용성이 증가되고 타이핑이 줄어든다
  2. 유지보수 편의 : 컴포넌트만 수정하면 연결된 모든 path에 반영된다.
  3. 가독성 향상 : 문서가 간결하고 구조적으로 정리된다.

  ```jsx
  openapi: 3.0.0
  info:
    title: Sample API
    version: 1.0.0
  paths:
    /users:
  .........
                schema:
                  $ref: '#/components/schemas/User'
  .........
  components:
    schemas:
      User:
        type: object
        properties:
          id:
            type: integer
          name:
            type: string
          email:
            type: string
  ```
  백엔드에서 Swagger 자동 생성 도구(e.g., NestJS의 @nestjs/swagger, SpringDoc 등)를 쓸 경우 DTO 클래스 기반으로 components가 자동 생성되기도 한다.
