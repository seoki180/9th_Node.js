* ORM
  ORM(Object-Relation Mapping)은 객체지향 프로그래밍에서 개발자가 SQL query를 직접 작성하지 않고 데이터베이스를 조작할 수 있게 하는 기술이다.
  ORM은 테이블을 코드내의 객체로 매핑시켜 테이블에 정보를 추가하거나 삭제할때 쿼리를 작성하지 않고 일반 JS class 객체에 접근하듯이 조작 할 수 있게 한다.
  ORM을 사용하면 쿼리를 직접 작성하지 않아도 되서 코드가 간결해지고, 스키마 변경에 유연하게 대응할 수 있다.
  대신 쿼리가 너무 복잡해지면 성능문제가 발생 할 수 있고, 디버깅이 복잡해질 수 있다.
* Prisma 문서 살펴보기
  * Prisma Introduction

    1. prisma client를 설정하기 위해선 데이터베이스 연결, 클라이언트 생성 및 적어도 하나 이상의 모델이 포함된 schema 파일이 필요하다.

    ```jsx
    datasource db {
      url      = env("DATABASE_URL")
      provider = "postgresql"
    }

    generator client {
      provider = "prisma-client-js"
    }

    model User {
      id        Int      @id @default(autoincrement())
      createdAt DateTime @default(now())
      email     String   @unique
      name      String?
    }
    ```
    1. node 프로젝트에 primsa cli, client를 설치한다.

    ```jsx
    npm install prisma --save-dev
    npx prisma
    npm install @prisma/client
    ```
    1. 프로젝트에서 실제로 prisma 클라이언트를 객체로 가져온다.

    ```jsx
    import { PrismaClient } from '@prisma/client'

    const prisma = new PrismaClient()
    // use `prisma` in your application to read and write data in your DB
    ```
    1. client를 사용해 데이터베이스에 쿼리를 보낸다.

    ```jsx
    // run inside `async` function
    const newUser = await prisma.user.create({
      data: {
        name: 'Alice',
        email: 'alice@prisma.io',
      },
    })

    const users = await prisma.user.findMany()
    ```
    ![스키마를 이전하거나 스키마가 바뀔때마다 prisma generate를 실행시켜서 업데이트를 해야한다.](attachment:c1feb47c-012b-47e5-af62-a4596b03de3d:image.png)
    스키마를 이전하거나 스키마가 바뀔때마다 prisma generate를 실행시켜서 업데이트를 해야한다.
  * Full text search
    일반적인 DB를 통한 검색 LIKE %keyword%와 달리  “apple”, “apples”, “Apple’s” 같은 변형까지 매칭해줄 수 있게 하는 prisma 내장 기능이다
    FTS(full text search) 기능을 활성화 하면 데이터베이스 열 내의 텍스트를 검색하는 기능을 어플리케이션에 추가 할 수있다.

    1. schema에 검색 기능을 추가하고 prisma generate를 해서 업데이트를 한다.

    ```jsx
    generator client {
      provider        = "prisma-client-js"
    }

    model post {
      id      Int    @unique
      content String
      title   String

      @@fulltext([content])
      @@fulltext([content, title]) //content, title을 검색에 사용하겠다는 의미
    }
    ```
    1. prisma를 사용해 검색을 한다.

    ```jsx
    // All posts that contain the words 'cat' or 'dog'.
    const result = await prisma.posts.findMany({
      where: {
        body: {
          search: 'cat dog',
        },
      },
    })

    // All posts that contain the words 'cat' and not 'dog'.
    const result = await prisma.posts.findMany({
      where: {
        body: {
          search: '+cat -dog',
        },
      },
    })

    // All drafts that contain the words 'cat' and 'dog'.
    const result = await prisma.posts.findMany({
      where: {
        status: 'Draft',
        body: {
          search: '+cat +dog',
        },
      },
    })
    ```
    ![image.png](attachment:13d99ab8-f2db-452d-ba2e-0dfa54145ce4:image.png)
    장점

    * 인덱스기반 검색으로 동작이 빠르고, 정확하다.
    * 의미 기반,키워드 조합이 가능함

    단점

    * MySQL, PostgreSQL의 DB 엔진에서만 사용 가능하다
    * 한국어 형태소 분석이 불가능하다.
* ORM(Prisma)을 사용하여 좋은 점과 나쁜 점
  * 장점
    1. 타입기반 개발이 가능하여 안정성이 좋다( 특히 typescript를 기반으로 하는 프로젝트 할때 좋다)

       ```jsx
       const user = await prisma.user.findUnique({ where: { id: 'abc' } }); // ❌ 문자열 → 에러 발생
       ```
       다음과 같은 코드가 존재한다고 할때 user.id가 number타입이면 primsa는 이 타입 정보를 바탕으로 컴파일 타임에 에러를 검출 할 수 있다.
    2. 자동 마이그레이션, 타입 생성이 가능하다.
       prisma migrate dev 명령어를 사용하면 DB에 변경된 모델을 자동으로 반영하고, 그에 맞는 **타입 정의**도 자동 생성된다.
       예시: User 모델에 새로운 필드 profileImage를 추가하면, 바로 타입이 반영되어 IDE 자동완성 기능까지 가능하다.
       DB 구조와 코드가 항상 **동기화**되며, 코드 변경 → 마이그레이션 → 타입 자동생성이 자연스럽게 연결됩니다
    3. 모델 중심 설계로 유지보수가 쉽다.
    4. 트랜잭션, 관계처리, 유효성 검사등이 추상화 되어 편리하다.
  * 단점
    1. 복잡한 쿼리(여러 JOIN, 집계 등)은 표현하기 어렵다
    2. 쿼리 튜닝과 성능 최적화에는 한계가 있다.
    3. Prisma 문법을 새로 배워야하기에 러닝커브가 존재한다.
* 다양한 ORM 라이브러리 살펴보기
  * ex. Sequelize
    Nodejs 초기부터 사용된 대표적 ORM으로 자료가 많고 사용자가 많다.
    MYSQL, PostgreSQL, MSSQL등 여러 DB를 지원한다. 모델은 클래스로 정의하고 쿼리는 SQL-Like로 메서드 체이닝 방식을 사용한다.
    하지만 타입추론, 자동완성이 제한적이고, 쿼리문법이 추상화 되어 있지 않아 직접 쿼리를 짜는 느낌이 강하다.
  * ex. TypeORM
    데코레이터를 기반으로 엔티티를 정의하는 방식이다. NestJS에서 사용되며 공식문서에서도 TypeORM을 사용한다. 관계설정이 명시적이고 가독성이 우수한 벙밥이다.
    하지만 마이그레이션 안정성이 부족하고 버전간 호환 이슈가 발생할 수 있으며, DB구조가 복잡해질 경우 설정이 번거롭고 디버깅이 어렵다는 단점이 있다.
* 페이지네이션을 사용하는 다른 API 찾아보기
  * [https://developers.kakao.com/docs/latest/ko/daum-search/dev-guide](https://developers.kakao.com/docs/latest/ko/daum-search/dev-guide)
    카카오 Daum 검색 API

    ```jsx
      GET <https://dapi.kakao.com/v2/search/web?query=검색어&page=1&size=10>
    ```
    * **주요 파라미터:**
      * page: 결과 페이지 번호 (1~50)
      * size: 페이지당 결과 수 (1~50)
    * **응답 구조:**
      * meta: 검색 결과에 대한 메타정보 (총 결과 수, 페이지 수 등)
      * documents: 검색 결과 목록
  * [https://developers.coupangcorp.com/hc/en-us/articles/360033645034-Product-list-paging-query](https://developers.coupangcorp.com/hc/en-us/articles/360033645034-Product-list-paging-query)
    쿠팡 상품 목록 조회 API

    ```jsx
      GET <https://api-gateway.coupang.com/v2/providers/seller_api/apis/api/v1/marketplace/seller-products?vendorId=VENDOR_ID&nextToken=NEXT_TOKEN&maxPerPage=10>
    ```
    * **주요 파라미터:**
      * vendorId: 판매자 ID
      * nextToken: 다음 페이지를 조회하기 위한 토큰
      * maxPerPage: 페이지당 최대 결과 수
    * **응답 구조:**
      * nextToken: 다음 페이지 조회를 위한 토큰
      * data: 상품 목록 데이터
