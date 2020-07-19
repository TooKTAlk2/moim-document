# 21 백엔드 프로그래밍 : Node.js의 Koa 프레임워크



## 21.1 백엔드 소개

- 서버를 만들어 데이터를 공유함
- 데이터를 등록,조회, 업데이트에 대한 규칙이 필요
- 다양한 언어 사용 가능
- NODE.JS 사용

### 21.1.2 Node.js

V8이라는 자바스크립트 엔진 공개

웹브라우저뿐만아니라 서버에서도 자바스크립트를 사용할 수 있는 런타임

### 21.1.3 Koa 사용

- Koa는 미들웨어 기능만 갖추고 있다.
- async /await 문법 정식 지원





## 21.2 작업환경 준비

- 프로젝트 폴더 생성
- ESLint와 Prettier 설정

-git bash

- ```
  $ node --version (버전 확인)
  $ yarn init -y (얀)
  $ cat package.json(열기)
  $ yarn add koa
  $ yarn add --dev eslint // 개발용 의존 모듈로 eslint 설치
  $ yarn run eslint // eslint (적용 뒤에 질문 잇음)
  $ yarn add elint-config.prettier // prettier에서 관리하는 코드 스타일은 ESLint에서 관리 않도록 하는 명령어
  
  ```

.eslintsrc.json

- eslint 설정

.prettierrc

- 


## 21.3 Koa 기본 사용법

### 21.3.1 서버 띄우기

- use method
- listen method

### 21.3.2 미들웨어

```jsx
const Koa = require('koa');
const app = new Koa();


app((ctx,next)=>{
    console.log(ctx.url);
    console.log(1);
    next()
})
app((ctx,next)=>{
    console.log(2);
    console.log(ctx.url);
    next()
})
app((ctx)=>{
    console.log(ctx.url);
})

app.listen(4000,()=>{
    console.log('Listening to port 4000');
})
Listening to port 4000
/
1
2
/
/
/favicon.ico
1
2
/favicon.ico
/favicon.ico
```

#### 

- 쿼리파라미터는 문자열로 비교해야함

```jsx
const Koa = require('koa');

const app = new Koa();

app.use((ctx,next)=>{
    if(ctx.query.authorized !== '1'){
        ctx.status = 401;
        return;
    }
    console.log(url);
    console.log(1);
    next();
})
app.use((ctx,next)=>{
    console.log(2);
    next();
})
app.use(ctx=>{
    console.log(3);
})
Listening to port 4000
/
1
2
3


```



### 21.3.2.1 next함수를 호출하면 promise를 반환함(express랑 다름) 즉 then을 사용할수 있다.

- next함수가 반환하는 promise는 다음에 처리해야할 미들웨어가 끝나면 완료된다.
- next 함수 호출 이후에 then을 사용하여 Promise가 끝난 다음 콘솔에 END를 기록하도록 수 수정

```jsx
const Koa = require('koa');
const app = new Koa();

app.use((ctx,next)=>{
    if(ctx.query.authorized !== '1'){
        ctx.status = 401;
        return;
    }
    console.log(url);
    console.log(1);
    next().then(()=>{
        console.log('END');
    });
})
app.use((ctx,next)=>{
    console.log(2);
    next();
})
app.use(ctx=>{
    console.log(3);
})
Listening to port 4000
/
1
2
3
END
```



#### 21.3.2.2 async/await 사용하기

- Express도 async/await 문법을 사용할 수 있지만, 오류를 처리하는 부분이 제대로 작동 불가능할수 있따.

```jsx
const Koa = require('koa');
const app = new Koa();

app.use(async (ctx,next)=>{
    if(ctx.query.auth ctx.status = 401;
        return;
    }
    console.log(url);
    console.log(1);
    await next();
    console.log(END);
})
app.use((ctx,next)=>{
    console.log(2);
    next();
})
app.use(ctx=>{
    console.log(3);
})
```



## 21.4 nodemon 사용하기

- 서버 코드를 변경할때마다 서버를 재시작 하는 것이 번거로우면 nodemon 이라는 도구를 사용

```
yarn add --dev nodemon 
```

- package.json

```
...
	
	"scripts":{
	"start": "node src",
	"start:dev" : "nodemon --watch src/ src/index.js"
	
	}
```

- start: node src # 재시작이 필요할때

- start:dev :src 디렉터리를 주시하고 있다가 해당 디렉터리 내부의 어떤 파일이 변경되면 이를 감지 후 src.index.js를 재시작



## 21.5 koa-router 사용하기

koa를 사용할 때도 다른 주소로 요청이 들어올 경우 다른 작업을 처리할 수 있도록 라우터를 사용

Koa 자체에 이 기능이 내장되어 있지는 않으므로, koa-router 모듈을 설치

```
yarn add koa-router
```

### 21.5.1 기본사용법

```jsx
const Koa = require('koa');
const Router = require('koa-router');

const app = new Koa();
const router = new Router();
router.get('/',ctx=>{
    ctx.body='홈';
});
router.get('/about',ctx=>{
    ctx.body='소개'
});
// app 인스턴스에 라우터 적용
app.use(router.routes()).use(router.allowedMethods());

app.listen(4000,()=>{
    console.log('Listening to port 4000');
});

```

- router 객체안의 메서드(get)는 Http 메서드를 의미한다.
- router.get의 첫번째 파라미터는 라우트의 경로, 두번째 파라미터에는 해당 라우트를 적용할 미들웨어 함수를 넣는다.



리액트 라우터에 설정했을 때 비슷하다

### 21.5.2 라우트 파라미터와 쿼리

- 라우터의 파라미터를 설정할때는 about/:name으로 콜론을 사용하여 라우트 경로를 설정합니다. 또한 파라미터가 있을수도 있고 없을 수도 있다면 /about/:name? 같은 형식으로 파라미터 이름 뒤에 물음표를 사용합니다.
- URL 쿼리의 경우 예를 들어 /posts/?id=10 같은 형식으로 요청했다면 해당 값을 ctx.query에서 조회 할 수 있습니다. 문자열 형태의 쿼리 문자열을 조회할 때는 ctx.querystring 사용합니다.
- 파라미터와 쿼리는 둘 다 주소를 통해 특정 값을 받아 올 때 사용하지만 용도가 다르다.
- 파라미터는  작업의 카테고리, 고유 id 혹은 이름으로 **특정 데이터를 조회할 때**
- 쿼리는 **옵션에 관련된 정보를 받아 올 때 , 어떤 조건을 만족하는 항목을 보여줄지, 어떤 기준으로 정렬할 지의 정보를 담고 있다.**



### 21.5.3 REST API

REST API는 요청의 종류에 따라 다른 HTTMP METHOD를 사용합니다. HTTP 메소드는 여러 종류가 습니다.

| 메서드 | 설명                                                         |
| ------ | ------------------------------------------------------------ |
| GET    | 데이터를 조회할 때 사용합니다.                               |
| POST   | 데이터를 등록할 때 사용합니다. 인증 작업을 거칠 때도 사용합니다. |
| DELETE | 데이터를 지울 때 사용합니다.                                 |
| PUT    | 데이터를 새 정보로 통째로 교체할 때 사용합니다.              |
| PATCH  | 데이터의 특정 필드를 수정할 때 사용합니다.                   |



블로그 포스트용 REST API

| 종류                                  | 기능                         |
| ------------------------------------- | ---------------------------- |
| POST /posts                           | 포스트 작성                  |
| GET /posts                            | 포스트 목록 조회             |
| GET /posts/:id                        | 특정 포스트 조회             |
| DELETE /posts/:id                     | 특정 포스트 삭제             |
| PATCH OR PUT /posts/:id               | 특정 포스트 업데이트 구현    |
| POST posts/:id/comments               | 특정 포스트에 덧글 등록      |
| GET posts/:id/comments                | 특정 포스트의 댓글 목록 조회 |
| DELETE /posts/:id/comments/:commentId | 특정 포소트의 특정 덧글 삭제 |



### 21.5.4 라우트 모듈화

프로젝트를 진행하다 보면 여러 종류의 라우트를 만들게 된다.

src/api/index.js

```jsx
const Router = require('koa-router');
const api = new Router();

api.get('/test',ctx=>{
	ctx.body = 'test 성공';
	});

// 라우터를 내보냅니다.
module.exports = api;

```



src/index.js

```jsx
const Koa = require('koa');
const Router = require('koa-router');

const api = require('./api'); 

const app = new Koa();
const router = new Router();

// 라우트 적용
router.use('/api',api.routes())

// app 인스턴스에 라우터 적용
app.use(router.routes()).use(router.allowedMethods());

app.listen(4000,()=>{
    console.log('Listening to port 4000');
})
```

####  맥 os 에서 실행중인  node process 찾기

lsof -i : 4000 (연결한 포트 번호 했으니) => PID 찾기

KILL -9 PID넘버  

### 21.5.5 posts 라우트 생성

src/api/posts/index.js

```jsx
const Router = require('koa-router');
const posts = new Router();

const prinitInfo = ctx =>{
    ctx.body = {
        method: ctx.method,
        path:ctx.path,
        params: ctx.params,
    };
};
posts.get('/', printInfo); 
posts.posts('/',printInfo);


module.exports = posts;
```

src/api/index.js

```jsx
const Router = require('koa-router');
const posts = require('./posts');
const api = new Router();

api.use('/test',posts.routes());

// 라우터를 내보냅니다.
module.exports = api;

```



#### 21.5.5.1 Postman의 설치 및 사용

- REST API 요청 테스팅을 쉽게 할 수 있는 프로그램

#### 21.5.5.2 컨트롤러 파일 작성

미들웨어가 너무 길기 때문에 따로 하나의 파일에 넣어서 정리한다. 기능별로 정리하는 것이 아닌 시스템 구조를 다듬는 방향성을 추구하는 것 같다

- koa-bodyparser 미들웨어를 적용
  - 이 미들웨어는 POST/PUT/PATCH 같은 메서드의 REQUEST BODY 에 JSON 형식으로 데이터를 넣어 주면, 이를 파싱하여 서버에서 사용할 수 있게 합니다.
  - yarn add koa-bodyparser

src/index.js

```jsx
const Koa = require('koa');
const Router = require('koa-router');
const bodyParser = require('koa-bodyparser');

const api = require('./api'); 

const app = new Koa();
const router = new Router();

// 라우트 설정
router.use('/api',api.routes())

//라우터 적용 하기 전에 bodyParser 적용
app.use(bodyParser());

// app 인스턴스에 라우터 적용
app.use(router.routes()).use(router.allowedMethods());

app.listen(4000,()=>{
    console.log('Listening to port 4000');
})
```



posts/posts.ctrl.js 만들고

- api 모듈 만들기
- 쓰기,목록읽기, 특정 포스트 읽기, 특정 포스트 삭제, 특정 포스트 대체, 특정 포스트의  특정 필드 변경
- post,patch,put 등을 쓸때는 무조건 router 연결하기전에 bodyParse를 적용해야한다.

posts/index.js 연결  





#### id 값 수정 안됨

1번을 삭제하면

2,3번 그대로 남아 있음

id가 변하지 않음 . 사실 변하지 않아야하는 것도 맞는 것 같음 고유번호니



# 22 mongoose를 이용한 MongoDB 연동 실습



## 22.1 소개

관계형 데이터베이스(RDBMS)

- MySQL, OracleDB, postgreSQL
- 스키마가 고정되어 있다.
- 확장성이 작다. 데이터베이스 서버의 성능을 업그레이드 하는 방식으로 확장해 주어야 했다. 분산 X



MongoDB

- 문서 지향적 NoSQL 데이터 베이스
- 유동적인 스키마
- 분산 처리함



까다로운 조건으로 데이터를 필터링하거나 ACID 특성을 지켜야한다면 RDBM가 유리

빅데이터에선 NoSQL 유리



### 22.1.1 문서란?

문서는 record와 같은 개념이다. 문서의 데이터 구조는 키-값 쌍으로 되어 있습니다.

예를 들어 하나의 문서는

{

"_id" : ObjectId("xxxx").

"username":"neo",

"name": {first: "M.J", last : "Kim"}

}

로 구성됩니다. 문서는 BSON(Binary + Json)형태로 저장됩니다. 새로운 문서를 만들면 _id라는 고윳값을 자동으로 생성하는데, 이값은 머신 아이디, 프로세스 아이디, 순차 번호로 되어 있어 고유함을 보장합니다.



여러 문서가 들어 있는 곳을 컬렉션이라고 합니다. 기존 RDBMS에서는 테이블 개념을 사용하므로 각 테이블마다 같은 스키마를 가지고 있어야 합니다. 

반면 MongoDB에서는 다른 스키마를 가지고 있는 문서들이 한 컬렉션에서 공존할 수 있습니다. 다음 예시를 한번 살펴보세요

{

"_id" : ObjectId("xxxx").

"username":"neo",

"name": {first: "M.J", last : "Kim"}

},

{

"_id" : ObjectId("xxxx").

"username":"neo2",

}



### 22.1.2 MongoDB 구조

하나의 서버안에 여러개의 데이터베이스

하나의 데이터베이스안에 여러개의 컬렉션이

하나의 컬렉션안에 문서들이 있다.

### 22.1.3 스키마 디자인

블로그용 데이터 스키마를 설계한다면

RDBMS에서는 각 포스트, 댓글마다 테이블을 만들어 필요에 따라 JOIN 해서 사용하는 것이 일반적입니다.

NOSQL에서는 그냥 모든 것을 하나의 문서에 넣습니다.문서 내부에 또다른 문서가 위치 할 수 있는데 이것을 서브다큐먼트라고 할 수 있다. 

## 22.2 MongoDB 설치

macOS, windows , LINUX

```
brew tap mongodb/brew
brew install mongodb-community@4.2
brew services start mongodb-community@4.2
```

[Install MongoDB doc on Mac OS](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/)

## 22.3 mongoose 설치 및 적용

mongoose는 Node.js 환경에서 사용하는 MongoDB 기반 ODM 라이브러리

이 라이브러리는 데이터 베이스 문서들을 자바스크립트 객체처럼 사용할 수 있게 해줍니다.

```
$ yarn add mongoose dotenv
```

dotenv는 환경변수들을 파일에 넣고 사용할 수 있게 하는 개발 도구입니다. mongoose를 사용하여 MongoDB에 접속할 때, 서버에 주소나 계정 및 비밀번호가 필요한 경우도 있습니다. 이렇게 민감하거나 환경별로 달라질 수 있는 값은 코드 안에 직접 작성하지 않고, 환경변수로 설정하는 것이 좋습니다. 프로젝트를 github, gitlab에 올릴 때는 .gitignore를 작성하여 환경변수가 들어 있는 파일은 제외시켜주어야 합니다.

### 22.3.1 .env 환경변수 파일 생성

.env

```jsx
PORT=4000
MONGO_URI = mongodb://localhost:27017/blog
```

여기서 blog는 우리가 사용할 데이터베이스의 이름, 최초 실행시 지정한 데이터베이스가 서버에 없다면 자동으로 만들어 주므로 사전에 직접 생성할 필요는 없습니다.



dotenv를 불러와서 config()함수를 호출해준다. Node.js에서는 환경변수는 process.env 값을 통해 조회할 수 있습니다.

-  src/index.js


```jsx
require('dotenv').config();
const Koa = require('koa');
const Router = require('koa-router');
const bodyParser = require('koa-bodyparser');

const {PORT} = process.env;

const api = require('./api'); 

const app = new Koa();
const router = new Router();

// 라우트 설정
router.use('/api',api.routes())

//라우터 적용 하기 전에 bodyParser 적용
app.use(bodyParser());

// app 인스턴스에 라우터 적용
app.use(router.routes()).use(router.allowedMethods());

app.listen(4000,()=>{
    console.log('Listening to port 4000');
})
```



### 22.3.2 mongoose 서버와 데이터베이스 연결

src/index.js

```jsx
require('dotenv').config();
const Koa = require('koa');
const Router = require('koa-router');
const bodyParser = require('koa-bodyparser');
const mongoose = require('mongoose');
const {PORT,MONGO_URI} = process.env;
//eslintrc.json에서 "env"{"node" : true} 해야 process.env 쓸수 잇음

mongoose.connect(MONGO_URI,{useNewUrlParser:true,useFindAndModify : false })
.then(()=>{
    console.log('Connected to MongoDB');
})
.catch(e=>{
    console.error(e);
})

const api = require('./api'); 

const app = new Koa();
const router = new Router();

// 라우트 설정
router.use('/api',api.routes())

//라우터 적용 하기 전에 bodyParser 적용
app.use(bodyParser());

// app 인스턴스에 라우터 적용
app.use(router.routes()).use(router.allowedMethods());

app.listen(4000,()=>{
    console.log('Listening to port 4000');
})

```



## 22.4 esm으로 ES 모듈 import/export문법 사용하기

Node.js v12은 ES 모듈 import/export 문법 지원

package.json

```json
(...)
    "script" : {
        "start": "node src",
        "start:dev" : "nodemon --watch src/ src/index.js"
    }
    "type":"module"
}
```

- require 못 쓴다.
- 해결책 ?

기존 src/index.js 파일의 이름을 main.js로 변경하고, index.js 파일을 새로 생성하여 다음 코드를 작성하자.

... 책 참고

### 내가 한 방법

- package.json

 "scripts": {

  "start": "node -r esm src",

  "start:dev": "nodemon --watch src/ -r esm src/index.js"},



- package.json
  - type : "export"
- yarn add esm 

main.js은 작성하지 않음

- .eslintrc.json
- parserOptions 속성 안에 "sourceType": "module"



### 22.4.1 기존 코드 ES Module 형태로 바꾸기

src/api/posts/posts.ctrl.js

```
exports => export const
```

src/api/posts/index.js

```
require => import 구문
module.exports = posts; => export default posts;
```

src/api/index.js

src/main.js 수정

jsconfig.json

```json
{
  "compilerOptions": {
    "target": "es6",
    "module": "es2015"
  },
  "include": ["src/**/*"]
}

```



## 22. 5 데이터 베이스의 스키마 모델

mongoose는 스키마와 모델이라는 개념이 있는데 혼동하기 쉽습니다.

스키마는 컬렉션이 들어가는 문서 내부의 각 필드가 어떤 형식으로 되어 있는지 정의하는 객체입니다. 이와 달리 모델은 스키마를 사용하여 만드는 인스턴스로 데이터베이스에서 실제 작업을 처리할 수 있는 함수들을 지니고 있는 객체입니다.

스키마는 형상을 잡아주는 객체

모델은 실제 작업을 진행하는 객체



### 22.5.1 스키마 생성

스키마를 만들 때는 mongoose 모듈의 Schema를 사용하여 정의합니다

| 타입                            | 설명                                         |
| ------------------------------- | -------------------------------------------- |
| String                          | 문자열                                       |
| Number                          | 숫자                                         |
| Date                            | 날짜                                         |
| Buffer                          | 파일을 담을 수 있는 버퍼                     |
| Boolean                         | true 또는 false 값                           |
| Mixed(Schema.Types.Mixed)       | 어떤 데이터도 넣을 수 있는 형식              |
| ObjectId(Schema.Types.ObjectId) | 객체 아이디, 주로 다른 객체를 참조할 때 넣음 |
| Array                           | 배열 형태의 값으로 [] 감싸서 사용            |

제가 만약 작가와 책이라는 모델을 만들고 싶다면

| Author | 타입   |
| ------ | ------ |
| name   | String |
| email  | String |



| BookSchema  |               | type               |
| ----------- | ------------- | ------------------ |
| title       |               | String             |
| description |               | String             |
| authors     |               | String             |
| meta        | likes(좋아요) | Number             |
| extra       |               | Schema.Types.Mixed |



### 22.5.2 모델 생성

src/model/posts.js

```jsx
import mongoose from 'mongoose';

const { Schema } = mongoose;
// 스키마 객체
const PostSchema = new Schema({
  title: String,
  body: String,
  tags: [String],
  publishDated: {
    type: Date,
    default: Date.now,
  },
  user: {
    _id: mongoose.Types.ObjectId,
    username: String,
  },
});

// 모델 생성
// model('스키마 이름',스키마 객체)
// 뭘입력하든 복수형태로 그리고 대문자 소문자로
const Post = mongoose.model('Post', PostSchema);
export default Post;

```

데이터베이스는 스키마 이름을 정해주면 그이름의 복수 형태로 데이터베이스에 컬렉션 이름을 만든다

ex 

Post => posts

BookInfo => bookinfos



## 22.6 MongoDB Compass 설치 및 사용

window는 mongodb 설치시 같이 설치

Mac과 리눅스는 홈페이지 가서 설치해야한다.

Community Edition Stable

### connect to host

Hostname : localhost		PORT : 27017 설정후

connect





## 22.7 데이터 생성과 조회

REST API를 학습하면서 임시적으로 자바스크립트 배열을 사용하여 기능을 구현하였습니다. 자바스크립트 배열 데이터는 시스템 메모리 쪽에 위치하기 때문에 서버를 재시작하면 초기화 됩니다. 이번에는 배열 대신에 MongoDB에 데이터를 보존해 보겠습니다.

### 22.7.1 데이터 생성 , 리스트 받기 ,  특정 포스트 읽기, 삭제, 수정

- params에서 받아오는것은 try 밖에
  try안에서 요청 하고 없는지 검증 catch 는 500에러 띄움
  db저장하고 빼오는게 시간 걸리니까 async await 해야함

src/api/posts/posts.ctrl.js

```jsx
import Post from '../../models/post';

export const write = async ctx => {
  const { title, body, tags } = ctx.request.body;

  const post = new Post({
    title,
    body,
    tags,
  });
  try {
    await post.save();
    ctx.body = post;
  } catch (e) {
    ctx.throw(500, e);
  }
};

export const list = async ctx => {
  try {
    const posts = await Post.find().exec();
    ctx.body = posts;
  } catch (e) {
    ctx.throw(500, e);
  }
};

export const read = async ctx => {
  const { id } = ctx.params;
  try {
    const post = await Post.findById(id).exec();
    if (!post) {
      ctx.status = 404;
      return;
    }
    ctx.body = post;
  } catch (e) {
    ctx.throw(500, e);
  }
};

export const remove = async ctx => {
  const { id } = ctx.params;

  try {
    await Post.findByIdAndRemove(id).exec();
    ctx.status = 204; // success for response but there is not show
  } catch (e) {
    ctx.throw(500, e);
  }
};

export const update = async ctx => {
  const { id } = ctx.params;

  try {
    const post = await Post.findByIdAndUpdate(id, ctx.request.body, {
      new: true,
    }).exec();

    if (!post) {
      ctx.status - 404;
      return;
    }
    ctx.body = post;
  } catch (e) {
    ctx.throw(500, e);
  }
};

```





## 22.9 요청 검증

### 22.9.1 ObjectId 검증

- api/posts/post.ctrl.js

```jsx
import mongoose from 'mongoose';


const { ObjectId } = mongoose.Types;

export const checkedObjectId = (ctx, next) => {
  const { id } = ctx.params;
  if (!ObjectId.isValid(id)) {
    ctx.status = 400;
    return;
  }
  return next();
};
```

 



### 22.9.2 Request Body 검증

- joy 설치 
  - 책과 다름 버전이 달라서,



write, update 부분 수정

```jsx
export const write = async ctx => {
  const schema = Joi.object({
    title: Joi.string().required(),
    body: Joi.string().required(),
    tags: Joi.array()
      .items(Joi.string())
      .required(),
  });

  const result = schema.validate(ctx.request.body);
  if (result.error) {
    ctx.status = 400;
    ctx.body = result.error;
    return;
  }

  const { title, body, tags } = ctx.request.body;

  const post = new Post({
    title,
    body,
    tags,
  });
  try {
    await post.save();
    ctx.body = post;
  } catch (e) {
    ctx.throw(500, e);
  }
};



....


export const update = async ctx => {
  const schema = Joi.object({
    title: Joi.string(),
    body: Joi.string(),
    tags: Joi.array().items(Joi.string()),
  });

  const result = schema.validate(ctx.request.body);
  if (result.error) {
    ctx.status = 400;
    ctx.body = result.error;
    return;
  }

  const { id } = ctx.params;

  try {
    const post = await Post.findByIdAndUpdate(id, ctx.request.body, {
      new: true,
    }).exec();

    if (!post) {
      ctx.status - 404;
      return;
    }
    ctx.body = post;
  } catch (e) {
    ctx.throw(500, e);
  }
};

```



## 22.10 PageNation

1Page당 보이는 포스트의 개수를 10개로 지정

포스트의 목록을 보여줄때

- 포스트의 개수가 많으면 느려질 걸 예상
- 포스트의 내용 전체를 보여줄것이 아니라 200글자로 제한

### 22.10.1 가짜데이터 등록하기

src/createFakeData.js

```jsx
import Post from './models/post';

export default function createFakeData() {
  const posts = [...Array(40).keys()].map(i => ({
    title: `포스트 #${i}`,
    body:
      'Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.',
    tags: ['가짜', '데이터'],
  }));
  Post.insertMany(posts, (err, docs) => {
    console.log(docs);
  });
}

```

src/index.js

```jsx
const { PORT, MONGO_URI } = process.env;
mongoose
  .connect(MONGO_URI, { useNewUrlParser: true, useFindAndModify: false })
  .then(() => {
    console.log('Connected to MongoDB');
    createFakeData();
  })
  .catch(e => {
    console.error(e);
  });
```

mongo compass 열어 자동으로  mongoldb 확인



### 22.10.2-6 page

- 역순
- 1page당 10개
- 페이지 기능 구현
- 마지막 페이지 번호 알려주기(커스텀 헤더로 알려주기)

- 바디 200자 제한 (방법 두 가지 )

src/api/posts/post.ctrl.js

```jsx
export const list = async ctx => {
  const page = parseInt(ctx.query.page || '1', 10);
  if (page < 1) {
    ctx.status = 400;
    return;
  }
  try {
    const posts = await Post.find()
      .sort({ _id: -1 })
      .limit(10)
      .skip((page - 1) * 10)
      .exec();
    const postCount = await Post.countDocuments().exec();
    ctx.set('Last-Page', Math.ceil(postCount / 10));
    ctx.body = posts
      .map(post => post.toJSON())
      .map(post => ({
        ...post,
        body:
          post.body.length < 200 ? post.body : `${post.body.slice(0, 200)}...`,
      }));
  } catch (e) {
    ctx.throw(500, e);
  }
};

```

- 조회할 때 lean() 함수를 사용, 처음부터 JSON으로 읽어 올 수 있다.

src/api/posts/post.ctrl.js

```jsx
export const list = async ctx => {
  const page = parseInt(ctx.query.page || '1', 10);
  if (page < 1) {
    ctx.status = 400;
    return;
  }
  try {
    const posts = await Post.find()
      .sort({ _id: -1 })
      .limit(10)
      .skip((page - 1) * 10)
      .lean()
      .exec();
    const postCount = await Post.countDocuments().exec();
    ctx.set('Last-Page', Math.ceil(postCount / 10));
    ctx.body = posts.map(post => ({
      ...post,
      body:
        post.body.length < 200 ? post.body : `${post.body.slice(0, 200)}...`,
    }));
  } catch (e) {
    ctx.throw(500, e);
  }
};
```



# 23 JWT를 통한 회원 인증 시스템 구현하기

## 23.1 이해

세션 기반 인증 vs 토큰 기반 인증의 차이

- 세션 기반 인증
  - 서버가 사용자가 로그인 중임을 기억하는 원리
  - 서버으로부터 브라우저에게 발급된 id 는 브라우저의 쿠키에 저장
  - 서버는 세션 저장소에서 세션을 조회한 후 로그인 여부를 결정
  - 세션 저장소는 메모리, 디스크, 데이터베이스 등을 사용
  - 단점 : 서버를 확장하기 번거로워질 수 있다. 서버의 인스턴스가 많다면 일일이 같은 세션을 공유해야하고 세션 전용 데이터 베이스를 만들어야 할 뿐 아니라 신경 써야 할 것도 많다. 세션 기반 인증이 무조건 좋지 않은 것은 아니다.
- 토큰 기반 인증
  - 토큰은 로그인 이후 서버가 만들어주는 문자열, 이 문자열 안에는 사용자의 로그인 정보가 들어 있고, 해당 정보가 서버에서 발급되었음을 증명하는 서명이 들어 있다.
  - 이 서명데이터는 해싱 알고리즘을 통해 만들어진다.
  - 서버에서 만들어 준 토큰은 서명이 있기 때문에 무결성 보장
  - 장점 : 서버가 확장되어도 서버끼리 사용자의 로그인 상태를 공유 할 필요가 없다.



이 책에서는 토큰 기반 인증 시스템 사용 할 예정



## 23.2 User 스키마/모델 만들기

- 계정명과 비밀번호
- 플레인(아무런 가공도 하지 않은)으로 비밀번호 저장하면 안된다. bcrypt (단방향 해싱 함수의 일종)으로 암호화 저장
- yarn add bcrypt



src/models/users.js

```jsx
import mongoose from 'mongoose';

const {Schema} = mongoose;

const UserSchema = new Schema({
  username:String,
  hashedPassword:String,
});

const User = mongoose.model('User',Userschema);

export default User;
```





### 23.2.1 모델 메서드 만들기

- 인스턴스 메서드

  - 모델을 통해 만든 문서 인스턴스에서 사용할 수 있는 함수

  - ```jsx
    const user = new User({username:dk99521});
    user.setPasswrod('mypassword12')
    ```

- 스태틱 메서드

  - 객체를 안 만들어도 모델에서 바로 사용할 수 있는 메서드

  - ```jsx
    const user = User.findByUsername('dk99521')
    ```



#### 23.2.1.1 인스턴스 메서드 만들기

setPassword

- 파라미터 : 비밀번호
- 기능  hashedPassword 값을 설정

checkPasswrod

- 파라미터 : 비밀번호
- 계정의 비밀번호와 일치하는지 검증

유의 사항

- 화살표 함수가 아닌 function 키워드를 사용하여 구현해야 합니다.
- 함수 내부에서 this 에 접근해야 하기 때문. (여기 this는 문서 인스턴스)

src/models/user.js

```jsx
UserSchema.methods.setPassword = async function(password) {
  const hash = await bcrypt.hash(password, 10);
  this.hashedPassword = hash;
};
UserSchema.methods.checkPassword = async function(password) {
  const result = await bcrypt.compare(password, this.hashedPassword);

  return result;
};
```



#### 23.2.1.2 스태틱 메서드

  findByUsername

- username을 받아 데이터 찾게 해준다.

src/models/user.js

```jsx
UserSchema.statics.findByUsernam = function(username) {
  return this.findOne({ username });
};
```



## 23.3 회원 인증 API 만들기

- src/api/auth/auth.ctrl.js
- src/api/auth/index.js
- src/api/index.js 

### 23.3.1 회원가입 구현하기

- 회원가입시
  -  hashedPassword 부분이 보이지 않도록 데이터를 json으로 바꾸고 delete를 통해 필드를 지움
  -  이 작업을 serialize라는 인스턴스 함수로 만들어 줄 예정

src/api/auth/auth.ctrl.js

```jsx
import User from '../../models/user';
import Joi from '@hapi/joi';

export const register = async ctx => {
  const Schema = Joi.object({
    username: Joi.string()
      .alphanum()
      .min(3)
      .max(20)
      .required(),
    password: Joi.string().required(),
  });

  const result = Schema.validate(ctx.request.body);
  if (result.error) {
    ctx.status = 400;
    ctx.body = result.error;
    return;
  }
  const { username, password } = ctx.request.body;
  try {
    const exists = await User.findByUsername(username);

    if (exists) {
      1;
      ctx.status = 409;
      return;
    }
    const user = new User({
      username,
    });
    await user.setPassword(password);
    await user.save();

    const data = user.toJSON();
    delete data.hashedPassword;

    ctx.body = data;
  } catch (e) {
    ctx.throw(500, e);
  }
};
```

src/api/auth/auth.ctrl.js

```jsx
    const user = new User({
      username,
    });
    await user.setPassword(password);
    await user.save();

    ctx.body = user.serialize();
```

src/models/user.js

```jsx
UserSchema.methods.serialize = function() {
  const data = this.toJSON();
  delete data.hashedPassword;

  return data;
};
```

### 23.3.2  로그인 구현

- Username,password 값이 제대로 전달되지 않으면 에러처리
- findByUsername을 통해 사용자 데이터 찾고, 없다면 에러처리
- setpassword를 통해 비밀번호 검사하고 성공했을 때 계정 정보 처리

```jsx
export const login = async ctx => {
  const { username, password } = ctx.request.body;

  if (!username || !password) {
    ctx.status = 401; // unauthorized
    return;
  }

  try {
    const user = await User.findByUsername(username);
    if (!user) {
      ctx.status = 401;
      return;
    }
    const valid = await user.checkPassword(password);
    if (!valid) {
      ctx.status = 401;
      return;
    }
    ctx.body = user.serialize();
  } catch (e) {
    ctx.throw(500, e);
  }
};

```



## 23.4 토큰 발급 및 검증하기

클라이언트에서 사용자 로그인 정보를 지니고 있을 수 있도록 서버에서 토큰을 발급해 주겠습니다. jwt 토큰을 만들기 위해서는 jsonwebtoken이라는 모듈을 설치해야합니다.

```
yarn add jsonwebtoken
```

### 23.4.1 비밀 키 설정하기

터미널 창에 openssl rand -hex 64 => 랜덤 문자열 생성

생성 후 .env 파일에 설정, 이 비밀키가 외부에 공개되면 절대로 안 됩니다.

.env

```
PORT = 4000
MONGO_URI = mongodb://localhost:27017/blog
JWT_SECRET = 117e877597a8f8e1e092a775753f8e850941aa99fea696ab78d7d489af103f1ad0ea2d95d694488d4997aad905b61018e02e37c7be7be1bdcaa6238570930fcf
```



### 23.4.2 토큰 발급하기

 src/models/user.js

```jsx
UserSchema.methods.generateToken = function() {
  const token = jwt.sign(
    { _id: this.id, username: this.username },
    process.env.JWT_SECRET,
    { expiresIn: '7d' },
  );
  return token;
};
```

- 토큰을 사용하는 방법은 두 가지 
- localStorage나 sessionStorage에 담아 이용
  - 매우 편리하고 구현도 쉬움
  - 악성 스크립트에 취약함 (XSS공격=Cross site scripting) 
- 브라우저의 쿠키에 담아 사용
  - httpOnly라는 속성을 활성화 시키면 자바스크립트를 통해 쿠키를 조회 할 수 없으므로 악성 스크립트로부터 안전
  - CSRF(cross site request Forgery)라는 공격에 취약, 토큰을 쿠키에 담으면 사용자가 서버로 요청을 할 때마다 무조건 토큰이 함께 전달되는 점을 이용해서 사용자가 모르게 원하지 않는 API 요청을 하게 만듭니다. 예를 들어 사용자가 자신도 모르는 상황에서 글을 쓰거나 삭제하거나 탈퇴하게 만든다. 단 CSRF는  CSRF 토큰 사용 및 REFERER 검증 등의 방식으로 제대로 막을 수 있는 반면, XSS는 보안 장치를 적용해 놓아도 개발자가 놓칠 수 있는 다양한 취약점을 통해 공격 받을 수 있다.
  - 이 책에서는 브라우저의 쿠키에 담아 사용하겟다.

src/api/auth/auth.ctrl.js

```jsx
export const register = async ctx => {
  const Schema = Joi.object({
    username: Joi.string()
      .alphanum()
      .min(3)
      .max(20)
      .required(),
    password: Joi.string().required(),
  });

  const result = Schema.validate(ctx.request.body);
  if (result.error) {
    ctx.status = 400;
    ctx.body = result.error;
    return;
  }
  const { username, password } = ctx.request.body;
  try {
    const exists = await User.findByUsername(username);

    if (exists) {
      1;
      ctx.status = 409; // conflict
      return;
    }
    const user = new User({
      username,
    });
    await user.setPassword(password);
    await user.save();

    ctx.body = user.serialize();

    const token = user.generateToken();
    ctx.cookies.set('access_token', token, {
      maxAge: 1000 * 60 * 60 * 24 * 7,
      httpOnly: true,
    });
  } catch (e) {
    ctx.throw(500, e);
  }
};

export const login = async ctx => {
  const { username, password } = ctx.request.body;

  if (!username || !password) {
    ctx.status = 401; // unauthorized
    return;
  }

  try {
    const user = await User.findByUsername(username);
    if (!user) {
      ctx.status = 401;
      return;
    }
    const valid = await user.checkPassword(password);
    if (!valid) {
      ctx.status = 401;
      return;
    }
    ctx.body = user.serialize();
    
    const token = user.generateToken();
    ctx.cookies.set('access_token', token, {
      maxAge: 1000 * 60 * 60 * 24 * 7,
      httpOnly: true,
    });
  } catch (e) {
    ctx.throw(500, e);
  }
};
```



### 23.4.3 토큰 검증하기

src/lib/jwtMiddleware.js

```jsx
import jwt from 'jsonwebtoken';

const jwtMiddleware = (ctx, next) => {
  const token = ctx.cookies.get('access_token');
  if (!token) return next();

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    console.log(decoded);
    ctx.state.user = { _id: decoded._id, username: decoded.username };
    return next();
  } catch (e) {
    return next();
  }
};

export default jwtMiddleware;

```



src/index.js

```jsx
app.use(bodyParser());
app.use(jwtMiddleware);
```

- Router 미들웨어가 적용되기 전에 적용해야함

src/api/auth/auth.ctrl.js-check

```jsx
export const check = async ctx => {
  const { user } = ctx.state;
  if (!user) {
    // 로그인 중이 아님 Unauthorized
    ctx.status = 401;
    return;
  }
  ctx.body = user;
};

```

### 23.4.4 토큰 재발급하기

Src/lib/jwtMiddleware.js

```jsx
import jwt from 'jsonwebtoken';
import User from '../models/user';

const jwtMiddleware = async (ctx, next) => {
  const token = ctx.cookies.get('access_token');
  if (!token) return next();

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    console.log(decoded);
    ctx.state.user = { _id: decoded._id, username: decoded.username };

    const now = Math.floor(Date.now() / 1000);
    if (decoded.exp - now < 3.5 * 60 * 60 * 24) {
      const user = await User.findById(decoded._id);
      const token = user.generateToken();
      ctx.cookies.set('access_token', token, {
        maxAge: 1000 * 60 * 60 * 24,
        httpOnly: true,
      });
    }

    return next();
  } catch (e) {
    return next();
  }
};

export default jwtMiddleware;

```



### 23.4.5 로그아웃 기능 구현

src/api/auth/auth.ctrl.js - logout

```jsx
export const logout = async ctx => {
  ctx.cookies.set('access_token');
  ctx.status = 204;
};

```



## 23.5 posts API에 회원 인증 시스템 구현

- 새 포스트(wrtie)는  로그인을 해야만 작성 할 수 있다.
- 삭제와 수정은 작성자만 할 수 있도록 구현
- 함수를 직접 수정해도 되지만 미들웨어로 관리 
- 각 포스트에 누가 글을 썼는지 알아야 하기 때문에 post schema를 수정해야 함



### 23.5.1 스키마 수정

- RDBMS에서는  foreign key로  id를 참조하게 만들면 되지만
- noSQL은 다 넣어주어야함



src/models/post.js

```jsx
import mongoose from 'mongoose';

const { Schema } = mongoose;

const PostSchema = new Schema({
  title: String,
  body: String,
  tags: [String]a,
  publishedDate: {
    type: Date,
    default: Date.now,
  },
  user: {
    _id: mongoose.Types.ObjectId,
    username: String,
  },
});

const Post = mongoose.model('Post', PostSchema);

export default Post;

```



### 23.5.2 posts 컬렉션 비우기

mongodb에서 posts만 삭제하기

### 23.5.3 로그인 했을 때만 API를 사용할 수 있게 하기

 src/lib/checkLoggedIn.js

```jsx
const checkLoggedIn = (ctx, next) => {
  if (!ctx.state.user) {
    ctx.status = 401; // unauthorized
    return;
  }

  return next();
};

export default checkLoggedIn
```

src/api/posts/index.js

```jsx
import Router from 'koa-router';
import * as postsCtrl from './post.ctrl';
import checkLoggedIn from '../../lib/checkLoggedIn';
const posts = new Router();
posts.get('/', postsCtrl.list);
posts.post('/', checkLoggedIn, postsCtrl.write);

const post = new Router();

post.patch('/', checkLoggedIn, postsCtrl.update);
post.delete('/', checkLoggedIn, postsCtrl.remove);
post.get('/', postsCtrl.read);

posts.use('/:id', postsCtrl.checkedObjectId, post.routes());
export default posts;

```

### 23.5.4 포스트 작성 시 사용자 정보 넣기



src/api/posts/post.ctrl.js-write

```
 const post = new Post({
    title,
    body,
    tags,
    user: ctx.state.user,
  });
```





## 23.5.5 포스트 수정 및 삭제 시 권환 확인하기

작성자만 수정 삭제해야하니 작성자 Id와 user Id가 같은지 확인해야할 예정

특정 포스트를 찾아 올 때 뿐만 아니라 수정 삭제할 때도 포스트를 찾아 와야하니 중복되는 과정임,그래서 특정 포스트를 찾아오는 미들웨어는 checkedObjectId(url parameter의 id  확인) + 특정 id를 가진 포스트를 찾아오는 것 =  getPostById라는 미들웨어를 만듬



 찾아온 post의 user Id와 로그인 된 user.id가 같은지 비교해야함 =  checkOwnPost

Src/api/posts/post.ctrl.js-getPostId,checkOwnPost,read

```jsx
export const getPostById = async (ctx, next) => {
  const { id } = ctx.params;
  if (!ObjectId.isValid(id)) {
    ctx.status = 400;
    return;
  }
  try {
    const post = await Post.findById(id);
    if (!post) {
      ctx.status = 404; // not Found
      return;
    }
    ctx.state.post = post;
    return next();
  } catch (e) {
    ctx.throw(500, e);
  }
};

export const checkOwnPost = (ctx, next) => {
  const { user, post } = ctx.state;
  // mongodb에서 받아온
  if (post.user._id.toString() !== user._id) {
    ctx.status = 403; // forbidden
    return;
  }
  return next();
};

export const read = ctx => {
  ctx.body = ctx.state.post;
};
```

src/api/posts/index.js

```jsx
import Router from 'koa-router';
import * as postsCtrl from './post.ctrl';
import checkLoggedIn from '../../lib/checkLoggedIn';
const posts = new Router();
posts.get('/', postsCtrl.list);
posts.post('/', checkLoggedIn, postsCtrl.write);

const post = new Router();

post.patch('/', checkLoggedIn, postsCtrl.checkOwnPost, postsCtrl.update);
post.delete('/', checkLoggedIn, postsCtrl.checkOwnPost, postsCtrl.remove);
post.get('/', postsCtrl.read);

posts.use('/:id', postsCtrl.getPostById, post.routes());
export default posts;

```



## 23.6 username/tags로 포스트 필터링하기

List API를 수정하여서  url query로 들어오는 username 과 tag들을 객체로 만들어서  find함수와 countDocuments의 인자로 넣어주기

src/api/posts/post.ctrl.js - list

```jsx
export const list = async ctx => {
  const page = parseInt(ctx.query.page || '1', 10);
  if (page < 1) {
    ctx.status = 400;
    return;
  }

  const { username, tag } = ctx.query;

  const query = {
    ...(username ? { 'user.username': username } : {}),
    ...(tag ? { tags: tag } : {}),
  };

  try {
    const posts = await Post.find(query)
      .sort({ _id: -1 })
      .limit(10)
      .skip((page - 1) * 10)
      .lean()
      .exec();
    const postCount = await Post.countDocuments(query).exec();
    ctx.set('Last-Page', Math.ceil(postCount / 10));
    ctx.body = posts.map(post => ({
      ...post,
      body:
        post.body.length < 200 ? post.body : `${post.body.slice(0, 200)}...`,
    }));
  } catch (e) {
    ctx.throw(500, e);
  }
};

```



