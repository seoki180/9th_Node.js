* 환경 변수
  프로그램을 개발할때 개발환경과 배포환경이 다를 수 있다(서버, 데이터베이스, 세부설정 등등) 예를 들어 개발환경에서는 디버깅을 편하게 하기 위해서 여러 정보를 로깅하거나 콘솔에 띄우게 되는데, 배포에서는 그럴 필요도 없고, 그러면 안되기 때문에 둘의 환경을 따로 설정해야 한다.
  이럴때 둘의 환경변수파일 (.env)를 설정해 적용시킨다.
  외부코드에 노출되면 안되는 정보를 저장하며 Nodejs 에서는 dotenv 패키지를 활용해 사용한다
  ```jsx
  #hardcoded
  ...
  password = "realpassword"
  ...

  #using env file
  ...
  passwod = process.env.password
  ...
  ```
* CORS
  Cross-Origin Resource Sharing
  브라우저가 자신의 출처가 아닌 다른 출처(도메인, 스킴, 포트)로부터 자원을 로딩하는 것을 혀용하도록 하는 HTTP header기반 메커니즘
  브라우저는 보안상 서로 다른 출처에 바로 요청을 보내는 것을 막는다. 하지만 요즘 웹은 서버랑 프론트가 서로 다른 곳에 위치하기 떄문에 서버에서 cors()를 설정해야 서로 다른 출처에서 데이터를 보내는것을 허락 할 수 있다.
  ```jsx
  ...
  app.use(cors()) // nodejs 서버에서 cors() 허용함
  ... 
  ```
* DB Connection, DB Connection Pool
  DB connection은 데이터베이스와 쿼리를 한번 날릴때마다 새로 DB와 연결을 시도한다.
  따라서 연결시마다 DB의 메모리, 리소스를 많이 사용하게 된다.
  DB connection Pool은 connection을 미리 여러개 만들어놓고 필요할 때마다 연결을 하나씩 빌려 쓰는 방식이다. 이러한 방식으로 연결 오버헤드를 줄일수 있다.
  ```jsx
  const connection = mysql.createConnection({
  ..
  })

  connection.connect()
  ..

  connection.query(query,(err,result)=>{
  ..
  })

  connection.end()
  ...

  --------------------------------------------------------------
  const pool = mysql.createPool({
  	connectionLimit : 10,
  	...
  })

  pool.getConnection((err,connection)=>{
  	connection.query(...)
  	connection.release()
  })
  ```
* 비동기 (async, await)
  코드 실행을 기다리지 않고 다음 작업을 먼저 처리하는 방식이다. async는 비동기 함수를 선언할 때 사용하고 await는 비동기 작업이 끝날 때까지 기다린다.
  DB를 연결하거나 API를 호출할때 작업이 오래걸리기 때문에 동기로 처리하게 되면 데이터가 올때까지 프로그램이 멈춰 다른 코드 실행이 전부 중단 될 수 있다.
  ```jsx
  const data = db.query('SELECT * FROM users'); // 응답 올 때까지 멈춤
  console.log('여기까지 코드가 멈춘다!');

  ---------------------------------------------------------------

  async function fetchData() {
    const data = await db.query('SELECT * FROM users'); // 기다리긴 하지만 코드 전체는 안 멈춤
    console.log('데이터 가져옴:', data);
  }

  fetchData();
  console.log('나는 먼저 실행된다!');
  ```
* try/catch/finally
  js 예외처리 문법으로
  try 구문 안에 에러가 날 수 있는 코드를 실행하고,
  try 안에 에러가 발생하면 catch에서 에러를 처리한다.
  이후 try-catch에서 에러를 잡든 못잡든 finally의 구문을 마지막에 무조건 실행한다.
  try에서 err를 throw한 순간 try블록은 멈추고 catch에서 에러를 처리한다. 문법을 통해 예상치 못한 에러로 중단되는 것을 막고, 에러를 처리해서 서비스가 중단되지 않도록 한다.
