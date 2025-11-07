https://github.com/seoki180/9th_node_practice/tree/feature/chapter06




* 한 번에 여러 번의 DB 작업을 연달아 처리할 때, 중간에 처리가 실패했는데 DB에는 중간까지만 값이 반영되어 있으면 문제가 있을 것 같습니다. 이를 방지하는 기술로는 Transaction이 있는데, Prisma를 이용해 Transaction을 관리하는 방법을 찾아 정리해주세요. 워크북의 실습 프로젝트에서도 적용할 수 있다면 적용해주세요.

Prisma에서 트랜잭션을 관리하는 법은 크게 두가지가 있다.

1. Batch(Sequential) 트랜잭션
   단순히 여러 prisma 쿼리를 묶어 ALL or NOTHING으로 처리한다.
   prisma.$transaction() 에 prisma 쿼리를 배열 형식으로 전달되면 모든 작업이 성공될 때만 커밋이 되고 하나라도 실패 하게 되면 롤백되는 방식으로 트랜잭션을 구현 할 수 있다.

   ```jsx
   await prisma.$transaction([
     prisma.user.create({ data: { name: 'Alice' } }),
     prisma.post.create({ data: { title: 'Hello', userId: 1 } }),
   ]);
   ```

   이 방식은 매우 직관적이고 간단하여 독립된 쿼리 여러개를 동시에 실행할 때 유용하다.

   하지만 이 방식에서는 모든 쿼리가 병렬로 실행되기 때문에 순차적 실행이 보장되지 않는다.

   따라서 순서가 중요한 쿼리의 경우

   예를들어 ) 유저가 가입을 해서 user.id가 생성되고 이후 쿼리에서  이user.id를 사용해야 할때 다음 방법을 사용할 수 있다.
2. Interactive 트랜잭션
   트랙잭션 내부에서 조건분기, 외부API 호출, 흐름제어, 루프분기등 로직이 포함되는 경우 callback 인자로 받는 tx 객체를 통해 모든 쿼리를  실행한다. 이때 timeout, maxWait, isolationLevel등 옵션을 제어할 수 있다.

   ```jsx
   await prisma.$transaction(async (tx) => {
     const user = await tx.user.create({ data: { name: 'Bob' } });
     await tx.post.create({ data: { title: 'New Post', userId: user.id } });
   });
   ```

   ```jsx
   type PrismaOrTx = PrismaClient | Prisma.TransactionClient;

   async function createUserCore(db: PrismaOrTx, input: CreateUserDto) {
     const user = await db.user.create({ data: input.user });
     const profile = await db.profile.create({ data: { ...input.profile, userId: user.id } });
     return { user, profile };
   }

   // 서비스: 트랜잭션 경계 설정
   async function registerUser(input: CreateUserDto) {
     return prisma.$transaction(async (tx) => {
       const result = await createUserCore(tx, input);
       // 필요시 추가 로직...
       return result;
     });
   }

   //tx 객체 전달예제
   ```

   이 함수에서는 tx가 prisma 객체로 사용되고 위에서 아래로 코드의 순차적 실행이 보장된다. 또한 트랜잭션이므로 모든 작업이 성공적으로 수행되어야 커밋이 이루어질 수 있다.

* DB 성능은 서버 개발에서 가장 중요한 부분 중 하나입니다. Prisma를 이용해서 데이터베이스에 질의할 때, 각 SQL 쿼리가 얼마나 오래 소요되는지 로그를 남겨주세요. (쿼리를 실행하기 전의 시간을 측정하고, 쿼리를 실행한 이후의 시간을 측정하여 몇 ms가 소요되었는지 측정하여 `console.log`로 출력할 수 있습니다.) 가능하면 쿼리를 사용하는 부분마다 매번 `console.log`로 출력하기 보다는, 항상 공통으로 자동으로 적용될 수 있는 방법으로 구현해주세요.

  * DB 성능은 서버 개발에서 가장 중요한 부분 중 하나입니다. Prisma를 이용해서 데이터베이스에 질의할 때, 각 SQL 쿼리가 얼마나 오래 소요되는지 로그를 남겨주세요. (쿼리를 실행하기 전의 시간을 측정하고, 쿼리를 실행한 이후의 시간을 측정하여 몇 ms가 소요되었는지 측정하여 `console.log`로 출력할 수 있습니다.) 가능하면 쿼리를 사용하는 부분마다 매번 `console.log`로 출력하기 보다는, 항상 공통으로 자동으로 적용될 수 있는 방법으로 구현해주세요.
    Prisma 공식문서에서는 두가지 방법으로 prisma에서 logging을 구현 할 수 있다고 한다.

    1. Stdout에 로깅(기본값)
    2. 이벤트기반 로깅

    1번 방식은 모든 로그레벨을 stdout으로 출력 가능하게 해준다. 가장 기본적이며 간단한 방법이고 logLevel객체에 이미 모든 로그레벨이 선언 되어 있어 간단하다.

    ```jsx
    const prisma = new PrismaClient({
      log: [
        {
          emit: 'stdout',
          level: 'query',
        },
        {
          emit: 'stdout',
          level: 'error',
        },
        {
          emit: 'stdout',
          level: 'info',
        },
        {
          emit: 'stdout',
          level: 'warn',
        },
      ],
    })
    ```
    2번방식은 prisma.$on() 메서드를 활용해 모든 이벤트를 감지하고 이벤트가 발생하면 직접 사용자가 로그를 출력할 수 있도록 한다.

    ```jsx
    prisma.$on('query', (e) => {
      console.log('Query: ' + e.query)
      console.log('Params: ' + e.params)
      console.log('Duration: ' + e.duration + 'ms')
    })
    ```
* 우리는 흔히 DB 쿼리에서 N+1 문제를 마주할 수 있습니다. 예를 들어, 게시글을 조회하면서 각 게시글에 해당하는 댓글들을 개별적으로 쿼리한다면, 댓글을 조회하는 쿼리가 각 게시글마다 추가로 발생하게 되어 N+1 문제가 발생합니다. 이를 Prisma에서 N+1문제를 해결할 수 있는 방법을 정리해주세요.


  N+1문제란 연관관계에서 발생하는 이슈로, 연관관계가 설정된 엔티티를 조회할때 조회된 데이터 개수만큼 조회 쿼리가 추가로 발생하게 되는데, 이를 N+1문제라 한다.

  데이터가 많아질수록 쿼리 수가 기하급수적으로 증가하게 된다. 특히 ORM에서 많이 발생하게 된다.

  해결 방법 두가지로 나눠서 생각 할 수 있다.

  1. Eager Loading (즉시 로딩)
     데이터를 조회할 때 연관된 모든 객 체의 데이터까지 한 번에 불러 온다. 즉 위에서 에상된 문제인 게시글과 댓글을 한번에 조인해 불러오는 방식을 생각 할 수 있다.

     ```jsx
     const postsWithComments = await prisma.post.findMany({
       include: {
         comments: true, // 관계된 모든 댓글을 함께 가져옴
       },
     });
     ```
     include방식으로 LEFT JOIN을 수행해 한번의 쿼리로 모든 데이터를 받아온다.

     다만 너무 복잡한 관계인 경우에는 eager loading이 오히려 성능저하의 원인이 될 수 있다.
  2. Batch조회 이후 수동 매핑
     include로 처리할 수 없는 복잡한 관계일 경우, 수동 Batch로 해결 할 수 있다.

     ```jsx
     // 1. 게시글 먼저 가져오기
     const posts = await prisma.post.findMany();
     const postIds = posts.map(post => post.id);

     // 2. 댓글을 postId 기준으로 한 번에 가져오기
     const comments = await prisma.comment.findMany({
       where: {
         postId: { in: postIds },
       },
     });

     // 3. postId별로 댓글을 매핑
     const commentMap = new Map();
     comments.forEach(comment => {
       if (!commentMap.has(comment.postId)) {
         commentMap.set(comment.postId, []);
       }
       commentMap.get(comment.postId).push(comment);
     });

     // 4. 게시글에 댓글 붙이기
     const postsWithComments = posts.map(post => ({
       ...post,
       comments: commentMap.get(post.id) || [],
     }));
     ```
     다음과 같은 쿼리로 조회 할 경우 쿼리가 2번만 실행된다.
