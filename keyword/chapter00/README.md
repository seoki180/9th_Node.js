* IP
  네트워크 상의  **호스트 식별 주소** . IPv4(32bit)와 IPv6(128bit)
  IPv4, IPv6가 존재한다. IPv4 는 가장 흔하게 접하는 IP 주소 체계로 [xxx.xxx.xxx.xxx](http://xxx.xxx.xxx.xxx) 형태로 존재한다.
  ![image.png](attachment:7f140eeb-6fce-46b7-9262-6f176faccc0b:image.png)
  IP 주소는 네트워크부분과 호스트부분으로 나뉘고 class 혹은 classless 방식으로 이를 구분한다. 각 subnet mask를 통해 class를 분류하며 해당 네트워크부분 아래 독자적인 망을 구축 할 수 있다.
  ![image.png](attachment:7d773a09-254b-42d5-bdbd-16f383946571:image.png)
  IP 주소 subnet mask를 통해 필요한 만큼 네트워크에 주소를 할당하여 사용 가능 하다.
* PORT
  한 IP 내 여러 프로세스/서비스를 구분하는 0~65535 범위의 번호
  0~1023은 이미 할당 되어 있어 사용이 불가능하다 ( 22-ssh, 80-http, 443-https 등)
  한 서버에서 외부로 여러 포트를 열어놓을 경우 보안상의 문제가 발생 할 수 있기에 리버스프록시(nginx, apache)등을 경유하여 내부 서버의 포트로 접근하도록 설정해야 한다.
* CIDR
  **C**lassless **I**nter-**D**omain **R**outing
  IP주소를 할당하고 라우팅 하는 방식
  앞서 설명한 class 방식에서 호스트 수가 늘어나며 할당가능한 IP주소의 부족으로 classless 방식이 도입되어 유연한 할당이 가능해졌다.
  ![image.png](attachment:e50557ba-13d9-4dc8-a7c8-fe1275896e2c:image.png)
  ![image.png](attachment:de8c9fce-996f-4273-84e2-364099de9eef:image.png)
* TCP와 UDP 차이
  ![image.png](attachment:20b6b525-8cb1-4957-840f-38a124d62f5c:image.png)
  OSI 7 layer에서 Transport layer에 존재하는 프로토콜로 패킷을 추적,관리한다.
  TCP는 연결형 서비스로 3-way handshake를 기반으로 연결상태를 항상 유지하고 오류검출, 재연결 시도등이 가능하지만
  ![image.png](attachment:49e4996e-19df-4777-a066-7169ef0fef33:image.png)
  UDP는 비연결성, 비신뢰성 프로토콜로 연결 설정 과정이 없어 전송속도가 빠르고 가벼운 통신에 사용이 된다.
* Web Server와 WAS의 차이
  Web Server(WS)는 클라리언트가 웹브라우저를 통해 요청한 정적콘텐츠(html, 사진 등)을 제공한다. 단순하 HTTP 프로토콜을 통해 전달하기만 한다(apache, nginx, tomcat등)
  WAS는 추가적으로 동적인 데이터를 처리하며, DB와 연결 해 연산을 수행하거나 여러 로직을 수행하게 된다.
  일반적인 경우는 WS와 WAS를 둘 다 사용하여 정적데이터(웹사이트, 이미지)를 WS처리하고 실제 구동이 필요한 로직(DB접근, 로직계산, 상태변경)을 WAS에서 담당하게 되어  각 서버의 부하를 줄인다.
  ![image.png](attachment:717bdd09-f6f6-4374-ae89-2bb6f4ebde64:image.png)
  [](https://velog.io/@moonblue/%EC%9B%B9-%EC%84%9C%EB%B2%84%EC%99%80-WAS%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90)[https://velog.io/@moonblue/웹-서버와-WAS의-차이점](https://velog.io/@moonblue/%EC%9B%B9-%EC%84%9C%EB%B2%84%EC%99%80-WAS%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90)
* HTTP와 HTTPS의 차이
  HTTP 프로토콜이란 Application level 의 상위 프로토콜로 stateless 프로토콜이다.
  req-res로 요청을 처리하며 특정메서드(GET,POST,PUT,DELETE)를 지정해 request를 날리게 되고 서버는 특정응답코드(200,201,404)로 요청에 대한 상태를 알려준다.
  ![image.png](attachment:6044ffdf-f361-4aab-a947-3b02ca215f7a:image.png)
  하지만 HTTP는 header에 암호화 되지 않은 평문데이터를 전송하기때문에 민감한 정보(주민등록번호, 비밀번호)를 주고받을때 중간에 탈취된다면 그대로 정보가 노출 될 수 있다.
  따라서 HTTPS 프로토콜을 도입해 http에 SSL암호화를 추가했다.
  HTTPS 는 두가지 암호화 방식을 사용하는데

  1. 대칭키 암호화 : 클라이언트와 서버가 동일한 키를 사용해 암호화/복호화를 진행함키가 노출되면 매우 위험하지만 연산 속도가 빠르다
  2. 비대칭키 암호화 : 1개의 쌍으로 구성된 공개키와 개인키를 암호화/복호화 하는데 사용함키가 노출되어도 비교적 안전하지만 연산 속도가 느리다
     비대칭키 암호화 동작 방식

     1. A가 B에게 메시지를 보내고 싶다.
     2. B의 **공개키**로 메시지를 암호화한다.
     3. 암호문은 네트워크에 노출돼도, **B의 개인키** 없이는 복호화할 수 없다.
     4. B는 자기 **개인키**로 복호화해서 원문을 얻는다.

     따라서 공개키가 노출되더라도 복호화 할 수 없다.

  HTTPS 는 두가지 암호화 방식을 혼합해 성능과 보안적 이점을 챙긴다.
  추가로 실제 웹사이트에 HTTPS를 등록하고 싶다면 CA를 거쳐 자신의 인증서가 유효하다는 것을 등록해야한다.
  ![image.png](attachment:58304441-31d6-45c6-99cb-a5f50a05c63a:image.png)
  ![image.png](attachment:037b4556-2b24-4a57-8313-793ddcf1be19:image.png)
  ![image.png](attachment:908d75bb-24ef-42a0-9e7c-4a7adf04eb04:image.png)
  ![image.png](attachment:65d94b7f-86db-47b1-8101-908c5a1f04b8:image.png)
