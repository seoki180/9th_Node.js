* OAuth 2.0
  OAuth (Open Authorization)은 **사용자 비밀번호를 공개하지 않고도 특정 서비스나 애플리케이션에 대한 접근 권한을 위임할 수 있는 표준화된 인증 프레임워크** 이다
  OAuth의 작동방식은 다음과 같다.

  1. 사용자 인증 : 사용자가 자신의 계정(네이버,구글)로 서비스에 로그인 한다
  2. 접근 권한 요청 : 서비스는 계정에 대한 접근권한을 요청한다.
  3. 권한 부여 : 사용자는 접근 권한을 부여한다.
  4. 접근 토큰 발급 : 권한이 부여 되면 서비스는 해당 계정에 대한 토큰을 받는다
  5. 정보 접근 : 서비스는 토큰을 사용해 계정에 대해 접근이 가능해진다.

  ![image.png](attachment:b2328186-ca81-46e4-a848-33a23a2c0b40:image.png)
  구글 OAuth에서 토큰의 크기는  다음과 같다.

  <aside>
  💡
  토큰 크기는 다음과 같은 한도 내에서 달라질 수 있습니다.
  * 승인 코드: 256바이트
  * 액세스 토큰: 2,048바이트
  * 갱신 토큰: 512바이트
    </aside>

  토큰은 두가지로 나뉘며 access token과 refresh token이 있다.

  * Access Token: 클라이언트가 서버 자원에 접근할 때 사용하는  **짧은 생명주기를 가진 토큰** .
  * Refresh Token: access token이 만료되었을 때  **새로운 access token을 발급받기 위해 사용하는 장기 유효 토큰** .

  토큰을 사용하는 이유는 보안 및 여러가지 이유가 있다.

  1. 비밀번호를 외부에 노출시키지 않기 위해
  2. 정해진 자원만 제한적으로 접근 하도록 설정 하기 위해
  3. access, refresh 토큰을 분리시켜 탈취되더라도 피해를 최소화 하기 위해
  4. 인가만 OAuth에서 담당하고, 인증은 Google, Naver같은 인증서버에서 수행하도록 하기 위해
  5. Bearer Token을 통해 요청을 간편하게 하기 위해

  하지만 보안을 위해 여러가지를 추가적으로 고려해야한다.

  * Access Token 탈취 시 누구나 자원 접근 가능 → 반드시 HTTPS 사용해야 함
  * CSRF 방지 위해 state 매개변수 사용
  * Refresh Token은 유출되면 장기 피해 가능 → 안전한 저장 필요
* JWT
  [let’s JWT ](https://www.notion.so/let-s-JWT-28578d13a6f1809ba326c3834c58a77d?pvs=21)
* Bearer Token

  > A bearer token is a string a client presents to access protected resources.
  >

  Bearer token은 요청 헤더의 Authorization에 “Bearer <토큰값>” 형태로 담아 서버 자원 접근을 허용하는 토큰입니다. “소지자(bearer)가 토큰을 들고 있으면 접근 가능”이라는 단순한 신뢰 모델이어서, 토큰 자체가 곧 권한 증명임
  **핵심 개념**

  * **전송 방식** : Authorization: Bearer 토큰값. 서버는 토큰을 검증 후 요청을 처리한다
  * **토큰 종류** : OAuth 2.0/OpenID Connect에서는 주로  **Access Token** (짧은 수명)과  **Refresh Token** (길게 보관, 재발급용)을 사용합니다. 페이지에 나온 것처럼 구글 기준 토큰 크기는 대략 Access 2,048바이트, Refresh 512바이트 정도로 변동될 수 있다.
  * **포맷** : 토큰은 **JWT**일 수도 있고 아닐 수도 있습니다. JWT인 경우 서명된 클레임을 포함해 서버가 무상태로 검증하기 쉽다.

  **보안 포인트**

  * **HTTPS 필수** : 탈취되면 곧바로 남용 가능하므로 암호화된 채널로만 전송해야 한다.
  * **CSRF 대비** : OAuth 2.0 인가 코드 흐름에서는 **state 파라미터**로 요청 위조를 막는다.
  * **보관 위치** : Refresh Token은 장기 피해를 유발할 수 있어 안전한 저장소에 보관하고, 필요시 회전(Token Rotation)과 **블랙리스트/revoke** 전략을 사용한다.
  * **스코프 최소화** : 필요한 자원만 접근하도록 스코프를 제한한다.

  클라이언트 요청 헤더:
  Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9…
  서버 동작:

  * 서명/유효기간/스코프 검증 → 통과 시 자원 제공, 실패 시 401/403 응답.
