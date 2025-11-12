* 미들웨어
  ![image.png](attachment:78e54c1e-eefc-48d2-abe2-b406ecb36983:image.png)
  미들웨어는 HTTP요청이 들어온 순간부터 순차대로 수행이 되며 req→처리→res 과정중 중간에 실행되는 함수이다.  미들웨어는 req,res 객체를 처리하거나 next() 메서드로 다음 미들웨어를 호출 할 수 있다.

  1. 요청이 들어오고
  2. 등록된 미들웨어 순서대로 실행된 다음,
  3. next()가 호출된다면 다음 미들웨어로 넘어가고
  4. 응답이 완료되면 종료된다.

  미들웨어는 사진과 같이 체인처럼 연결 되어 있고, 처리 흐름을 제어 할 수 있다.
  app으로 선언한 전역 메서드에 적용 할 수도 있고 router 메서드를 통해 해당 경로에만 적용되는 라우터를 설정 할 수 도 있다.

  ```jsx
  const app = express()
  ..
  app.use(모든 경로에 적용되는 미들웨어)

  ------------------------------------------------------------------------------
  const router = express.Router()
  ..
  router.use(해당 경로에만 적용되는 미들웨어)

  ```

  미들웨어는 크게 4가지로 나뉘며

  1. app 레벨 미들웨어
  2. router 레벨 미들웨어
  3. 에러철리 미들웨어
  4. 내장 미들웨어

  가 있다.
  ![express가 공식적으로 유지보수하는 미들웨어](attachment:d98ff803-9ded-408d-8ea6-07df130c5e68:image.png)
  express가 공식적으로 유지보수하는 미들웨어
* HTTP 상태 코드
  http status code는 클라이언트가 보낸 요청에 대해 서버가 어떤 응답을 했는지 나타내는 표준 규약이다.

  | **분류** | **의미**                 | **예시**                   |
  | -------------- | ------------------------------ | -------------------------------- |
  | **1xx**  | 정보 응답 (Informational)      | 요청을 받았고, 처리 중           |
  | **2xx**  | 성공 (Success)                 | 요청을 성공적으로 받음/처리함    |
  | **3xx**  | 리다이렉션 (Redirection)       | 요청을 완료하려면 추가 동작 필요 |
  | **4xx**  | 클라이언트 오류 (Client Error) | 잘못된 요청                      |
  | **5xx**  | 서버 오류 (Server Error)       | 서버 내부 문제로 요청 처리 실패  |

  전체 응답 코드는 대략 60개 정도 존재하지만 자주 사용되는 코드는 한정적이다.

  그중에서 자주 사용 되는 코드는 다음과 같다

  | **상태 코드**       | **카테고리**   | **의미**            | **설명**                    |
  | ------------------------- | -------------------- | ------------------------- | --------------------------------- |
  | 200 OK                    | ✅ 성공              | 요청 성공                 | GET/POST 요청 정상 처리           |
  | 201 Created               | ✅ 성공              | 리소스 생성됨             | POST로 새 데이터 생성 시          |
  | 204 No Content            | ✅ 성공              | 응답 본문 없음            | DELETE 등 응답 필요 없는 경우     |
  | 400 Bad Request           | ⚠️ 클라이언트 오류 | 잘못된 요청               | 파라미터 오류, JSON 파싱 실패 등  |
  | 401 Unauthorized          | ⚠️ 클라이언트 오류 | 인증 필요                 | 로그인 안됨, 토큰 누락/만료       |
  | 403 Forbidden             | ⚠️ 클라이언트 오류 | 접근 금지                 | 인증은 되었지만 권한 없음         |
  | 404 Not Found             | ⚠️ 클라이언트 오류 | 리소스를 찾을 수 없음     | 잘못된 경로, 존재하지 않는 데이터 |
  | 409 Conflict              | ⚠️ 클라이언트 오류 | 요청 충돌                 | 중복 데이터, 자원 충돌 등         |
  | 422 Unprocessable Entity  | ⚠️ 클라이언트 오류 | 요청은 이해되나 처리 불가 | 유효성 검사 실패 등               |
  | 500 Internal Server Error | ❌ 서버 오류         | 서버 내부 오류            | 예외 처리 실패, 버그 발생 등      |
  | 502 Bad Gateway           | ❌ 서버 오류         | 게이트웨이 오류           | 프록시 서버 → 원서버 통신 실패   |
  | 503 Service Unavailable   | ❌ 서버 오류         | 서비스 불가               | 서버 점검, 과부하 등              |
  | 504 Gateway Timeout       | ❌ 서버 오류         | 응답 지연으로 타임아웃    | 백엔드 서버 응답 지연             |
* 에러 핸들링(Error Handling)
  사용자에게 일관된 응답을 보내고, 내부적 원인을 외부로 노출시키지 않고 안전한 오류를 전달해주도록 하다

  * **커스텀 에러 객체:** 도메인별로 메시지/코드/상태를 담아 던진다
  * **중앙 에러 미들웨어:** 모든 라우트에서 던진 에러가 이곳으로 모여 표준 응답으로 변환한다.
  * **메시지 숨기기 :** Error.msg등의 값을 그대로 전달하게 된다면 서버의 내부 파일구조, 쿼리문등이 외부에 노출될 수 있기에 이를 커스텀하여 전달한다.

  ```cpp
  import 'express';
  import { HttpError } from '../src/middleware/error';

  declare global {
    namespace Express {
      export interface Response {
        resultType: string;
        data: any;
        successResponse(data?: any, message: string): Response;
        failResponse(message: string, error: HttpError): Response;
      }
    }
  }

  export const responseHandler = (
    req: Request,
    res: Response,
    next: NextFunction
  ) => {
    // 성공 응답 헬퍼 메서드
    res.successResponse = (
      data: any = null,
      message: string = '요청이 성공적으로 처리되었습니다.'
    ) => {
      return res.json(
        new ResponseBase({
          resultType: 'SUCCESS',
          data,
          message
        })
      );
    };

    // 실패 응답 헬퍼 메서드
    res.failResponse = (
      message: string = '요청 처리 중 오류가 발생했습니다.',
      error: HttpError
    ) => {
      return res.status(error.status).json(
        new ResponseBase({
          resultType: 'FAIL',
          data: null,
          message: error.message,
          error: error
        })
      );
    };
    next();
  };
  ```
