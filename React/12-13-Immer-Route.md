# 12 Immer

## 12.1 설치

```
yarn add immer
```

## 12.1.2 immer를 사용하지 않고 불변성을 유지

App.js

```jsx
import React, { useState, useCallback, useRef } from 'react';

const App = () => {
  const nextId = useRef(1);

  const [form, setForm] = useState({ name: '', username: '' });
  // 일부러 복잡한 것을 만든다고 uselessValue 설정함
  const [data, setData] = useState({ array: [], uselessValue: null });

  const onChange = useCallback(e => {
    const { name, value } = e.target;
    setForm(form => ({ ...form, [name]: value }));
  }, []);

  const onSubmit = useCallback(
    e => {
      // 등록
      const info = {
        id: nextId.current,
        name: form.name,
        username: form.username
      };
      setData(data => ({ ...data, array: data.array.concat(info) }));
      // 인풋 초기화
      setForm({ name: '', username: '' });
      nextId.current += 1;

      e.preventDefault();
    },
    [form]
  );

  const onRemove = useCallback(id => {
    setData(data => ({
      ...data,
      array: data.array.filter(info => info.id !== id)
    }));
  }, []);

  return (
    <div>
      <form onSubmit={onSubmit}>
        <input
          value={form.name}
          onChange={onChange}
          name="name"
          placeholder="아이디"
        />
        <input
          value={form.username}
          onChange={onChange}
          name="username"
          placeholder="이름"
        />
        <button type="submit">등록</button>
      </form>
      <ul>
        {data.array.map(info => (
          <li key={info.id} onClick={() => onRemove(info.id)}>
            {info.username} ({info.name})
          </li>
        ))}
      </ul>
    </div>
  );
};

export default App;

```

매우 복잡하다. 책과는 달리 저는 곧바로 함수형업데이트를 설정하여 사용하였습니다.

### 12.1.3 immer 사용법

immer에서 produce를 import 하여 사용합니다.

첫 번째 파라미터는 수정하고 싶은 상태이고, 두 번째 파라미터에는 상태를 어떻게 업데이트할지 정의 하는 함수 `draft=>{draft...}`

- 사용법

```javascript
import produce from 'immer';
const nextState = produce(originalState, staft=>{draft.a.b.c = 1})

```

- immer 적용전

```javascript
setForm({...form,[name] : [value]});

setData({...data,array : data.array.concat(info)})

setData({...data,array:data.array.filter(info=> info.id !== id)})
```

- immer 적용후

```javascript
setForm(produce(form,draft=>{draft[name] = value}));

setData(produce(data.draft=>{draft.array.push(info)}));
setData(produce(data,draft=>{draft.array.splice(draft.array.findIndex(
    info=>info.id === id),1)}))
```



- useState( 함수형 업데이트

  - 예시 코드(함수형)

    ```javascript
    const [number, setNumber] = useState(0);
    const onIncrease = useCallback(()=>setNumber(prevNumber=>prevNumber+1),[]);
    ```

  - 적용 전

    ```javascript
    setForm(produce(form,draft=>{draft[name] = value}));
    
    setData(produce(data.draft=>{draft.array.push(info)}));
    setData(produce(data,draft=>{draft.array.splice(draft.array.findIndex(
        info=>info.id === id),1)}))
    ```

    

  - 적용 후

    ```javascript
    setData(produce(draft=>{draft[name] = value }));
    setForm(produce(draft=>{draft.array.push(info)}));
    setForm(produce(draft=>{draft.array.splice(draft.array.findIndex(
        info=>info.id === id),1)}));
    ```

    



### 12.1.4-5 Immer 적용하기 + 함수형 업데이트

```JSX
import React, { useState, useCallback, useRef } from 'react';
import produce from 'immer';
const App = () => {
  const nextId = useRef(1);

  const [form, setForm] = useState({ name: '', username: '' });
  // 일부러 복잡한 것을 만든다고 uselessValue 설정함
  const [data, setData] = useState({ array: [], uselessValue: null });

  const onChange = useCallback(e => {
    const { name, value } = e.target;
    setForm(
      produce(draft => {
        draft[name] = value;
      })
    );
  }, []);

  const onSubmit = useCallback(
    e => {
      // 등록
      const info = {
        id: nextId.current,
        name: form.name,
        username: form.username
      };
      setData(
        produce(draft => {
          draft.array.push(info);
        })
      );
      // 인풋 초기화
      setForm({ name: '', username: '' });
      nextId.current += 1;

      e.preventDefault();
    },
    [form]
  );

  const onRemove = useCallback(id => {
    setData(
      produce(draft => {
        draft.array.splice(
          draft.array.findIndex(info => info.id === id),
          1
        );
      })
    );
  }, []);

  return (
    <div>
      <form onSubmit={onSubmit}>
        <input
          value={form.name}
          onChange={onChange}
          name="name"
          placeholder="아이디"
        />
        <input
          value={form.username}
          onChange={onChange}
          name="username"
          placeholder="이름"
        />
        <button type="submit">등록</button>
      </form>
      <ul>
        {data.array.map(info => (
          <li key={info.id} onClick={() => onRemove(info.id)}>
            {info.username} ({info.name})
          </li>
        ))}
      </ul>
    </div>
  );
};

export default App;

```

`setData(produce(draft=>{draft.blah.blah ...} ))`

여기서 중괄호를 무조건 붙여야 실행된다. 그리고 Immer을 사용하면 React dev tool의 state에 표시되지 않는다.

# 13장 리액트 라우터로 SPA 개발하기

## 13.1 SPA란?

Single Page Application의 약자로 한 개의 페이지로 이루어진 애플리케이션이라는 말입니다. 전통적인 웹페이지는 다른 페이지로 이동할 때마다 새로운 html을 다시 받아옵니다. 서버측에서 준비했습니다. 사전에 html 파일을 만들어서 제공하거나 데이터에 따라 유동적인 html을 생성해 주는 템플릿 엔진을 사용했기도 했죠.

하지만 이 전통적인 방법은 서버의 성능을 문제(트래픽과 과부하)를 야기합니다. 그래서 리액트나 다른 라이브러리를 사용하여 뷰렌더링을 사용자의 브라우저가 담당하도록 하고 서버와 사용자의 인터랙션이 발생하면 필요한 부분만 자바스크립트를 사용해서 업데이트 해줍니다. 만약 새로운 데이터가 필요하다면 서버 API를 호출해서 필요한 떼이터만 불러와서 새로운 애플리케이션에서 사용할 수도 있습니다.

싱글페이지라고 해서 화면이 한 종류가 아닙니다. SPA의 경우 서버에서 사용자에게 제공하는 페이지는 한 종류이지만, 해당 페이지에서 로딩된 자바스크립트와 현재 사용자 브라우저의 주소 상태에 따라 다양한 화면을 보여줍니다.

다른 주소에 다른 화면을 보여주는 것을 라우팅이라고 합니다. 리액트는 이 기능이 내장되어 있지는 않지만 브라우저 API를 직접 사용하여 이를 관리하거나, 다른 라이브러리를 사용하여 구현합니다.

예를 들어 리액트라우터, 리치라우터, Next.js 등이 있습니다. 리액트 라우터를 사용할 예정입니다. 리액트 라우터는 클라이언트 사이드에서 이루어지는 라우팅을 아주 간단하게 구현할 수 있도록 해줍니다. 더 나아가서 서버 사이드 렌더링을 할 때도 라우팅을 도와주는 컴포넌트를 제공합니다.

### 13.1.1 SPA의 단점

앱의 규모가 커지면 자바스크립트의 파일도 너무 커집니다. 페이지 로딩 시 사용자가 실제로 방문하지 않을 수도 있는 페이지의 스크립트도 불러오기 때문입니다. 이 문제를 코드 스플리팅을 통해 해결 합니다. 리액트 라우터 처럼 브라우저에서 자바스크립트를 사용하여 라우팅을 관리하는 것은 자바스크립트를 실행하지 않는 일반 크롤러에서는 페이지 정보를 제대로 수집해 가지 못하는 단점이 수반됩니다. 이 문제는 서버 사이드 렌더링을 통해 해결 할 수 있습니다.

## 13.2 준비 및 사용법

### 13.2.1-4 기본 준비

`yarn add react-router-dom`

index.js

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import * as serviceWorker from './serviceWorker';
import { BrowserRouter } from 'react-router-dom';
ReactDOM.render(
  <BrowserRouter>
    <App />
  </BrowserRouter>,
  document.getElementById('root')
);

// If you want your app to work offline and load faster, you can change
// unregister() to register() below. Note this comes with some pitfalls.
// Learn more about service workers: https://bit.ly/CRA-PWA
serviceWorker.unregister();

```

App.js

```jsx
import React from 'react';
import { Route } from 'react-router-dom';
import Home from './Home';
import About from './About';

const App = () => {
  return (
    <div>
      <Route path="/" component={Home} exact={true} />
      <Route path="/about" component={About} />
    </div>
  );
};

export default App;

```

exact를 붙여야 /about에 /의  주소에 라우트 된 컴포넌트 Home이 보이지 않습니다.

index에 BroweserRouter로 감싸주고 보여줄 컴포넌트에

`<Route path="주소규칙 component={보여줄 컴포넌트} />"`

를 작성합니다.

Home.js

```jsx
import React from 'react';

const Home = () => {
  return <div>홈페이지입니다.</div>;
};

export default Home;

```

About.js

```jsx
import React from 'react';

const About = () => {
  return <div>소개</div>;
};

export default About;

```

### 13.2.5 Link 컴포넌트 사용하기

이번에는 Route가 아닌 Link 컴포넌트를 사용하겠습니다. 이는 route와 다르게 클릭하면 다른 주소로 이동시켜 주는 컴포넌트입니다. 일반 웹 애플ㄹ케이션은 a 태그를 사용하여 페이지를 전환하는데요. 리액트 라우터를 사용할 때는 이 태그(a)를 직접 사용하면 안 됩니다. 이 태그는 페이지를 전환하는 과정에서 페이지를 새로 불러오기 때문에 애플리케이션의 상태를 모두 날려버리게 됩니다. 렌더링 된 컴포넌트들도 모두 사라지고 다시 처음부터 렌더링 하게 됩니다.

반면 Link 컴포넌트를 사용하여  페이지를  전환하게 된다면 페이지를 새로 불러오지 않고 애플리케이션은 유지한 상태에서 HTML 5 History API를 사용하여 페이지의 주소만 변경해 줍니다.

`<Link to="주소">내용</Link>`

App.js

```jsx
import React from 'react';
import { Route, Link } from 'react-router-dom';
import Home from './Home';
import About from './About';

const App = () => {
  return (
    <div>
      <ul>
        <li>
          <Link to="/">홈</Link>
        </li>
        <li>
          <Link to="/about">소개</Link>
        </li>
      </ul>
      <hr />
      {/* 이 아래에서 보여줍니다. */}
      <Route path="/" component={Home} exact />
      <Route path="/About" component={About} exact />
    </div>
  );
};

export default App;

```

## 13.3 Route 하나에 여러 개의 path 설정하기

App.js

```jsx
import React from 'react';
import { Route, Link } from 'react-router-dom';
import Home from './Home';
import About from './About';

const App = () => {
  return (
    <div>
      <ul>
        <li>
          <Link to="/">홈</Link>
        </li>
        <li>
          <Link to="/about">소개</Link>
        </li>
        <li>
          <Link to="/info"> 정보 소개</Link>
        </li>
      </ul>
      <hr />
      {/* 이 아래에서 보여줍니다. */}
      <Route path="/" component={Home} exact />
      <Route path={['/about', '/info']} component={About} exact />
    </div>
  );
};

export default App;

```

## 13.4 URL 파라미터와 쿼리

주소를 정의할 때 유동적인 값 전달할 때 사용합니다.

- URL PARAMETER
  - /profiles/velopert
  - 특정 아이디와 이름을 사용하여 조회할 때 사용합니다.
  - 라우터의 속성 중 match 객체을 이용하여 값을 전달합니다 `/:username` 
  - Link의 주소에 속성의 값을 사용합니다.
  - match.parms 에서 값을 조회 합니다.

Profile.js

```jsx
import React from 'react';
const data = {
  dk99521: {
    name: '정우태',
    description: '초심자'
  },
  velopert: {
    name: '김민준',
    description: '선생님'
  }
};

const Profile = ({ match }) => {
  const { username } = match.params;
  const profile = data[username];
  // 값을 받아올 때는 에러 처리해주어야 합니다.
  if (!profile) return <div>존재하지 않는 사용자입니다.</div>;

  const { name, description } = profile;
  return (
    <div>
      <h3>
        {username} ({name})
      </h3>
      <p>{description}</p>
    </div>
  );
};

export default Profile;

```



App.js

```jsx
import React from 'react';
import { Route, Link } from 'react-router-dom';
import Home from './Home';
import About from './About';
import Profile from './Profile';

const App = () => {
  return (
    <div>
      <ul>
        <li>
          <Link to="/">홈</Link>
        </li>
        <li>
          <Link to="/about">소개</Link>
        </li>
        <li>
          <Link to="/info"> 정보 소개</Link>
        </li>
        <li>
          <Link to="/profiles/velopert"> velopert 소개</Link>
        </li>
        <li>
          <Link to="/profiles/dk99521"> dk99521 소개</Link>
        </li>
      </ul>
      <hr />
      {/* 이 아래에서 보여줍니다. */}
      <Route path="/" component={Home} exact />
      <Route path={['/about', '/info']} component={About} exact />
      <Route path="/profiles/:username" component={Profile} />
    </div>
  );
};

export default App;

```



- QUERY
  - /about?details=true
  - 어떤 키워드를 검색하거나 페이지에 필요한 옵션을 전달할 때 사용합니다.\
  - location의 search로 조회합니다. 아래는 location 객체의 조회

```jsx
{
  "pathname" :"/about",
  "search":"?detail=true",
  "hash" : ""}
```

객체는 props로 전달되며, 웹 애플리케이션의 현재 주소의 정보를 지니고 있습니다.

이 객체는 `http://localhost:3000/about?detail=true` 로 접근했을 때의 값입니다.

URL 쿼리의 읽을 때는 위 객체가 지닌 값 중 search 값을 확인해햐 합니다. 하지만 이 문자열을 객체형태로 반환해야 사용할 수 있습니다. 이것을 간편히 하여주는 라이브러리를 소개합니다.

`yarn add qs`

About.js

```jsx
import React from 'react';
import qs from 'qs';
const About = ({ location }) => {
  const query = qs.parse(location.search, {
    ignoreQueryPrefix: true // ?와 같은 접두사를 생략합니다.
  });
  // 문자열 true일 때만 반응합니다
  const showDetail = query.detail === 'true';
  return (
    <div>
      <h3>introduction</h3>
      <p>{showDetail ? 'detail 값이 true 입니다.' : 'detail 에러'}</p>
    </div>
  );
};

export default About;

```

App.js

```jsx
import React from 'react';
import { Route, Link } from 'react-router-dom';
import Home from './Home';
import About from './About';
import Profile from './Profile';

const App = () => {
  return (
    <div>
      <ul>
        <li>
          <Link to="/">홈</Link>
        </li>
        <li>
          <Link to="/about/?detail=true">소개</Link>
        </li>
        <li>
          <Link to="/info"> 정보 소개</Link>
        </li>
        <li>
          <Link to="/profiles/velopert"> velopert 소개</Link>
        </li>
        <li>
          <Link to="/profiles/dk99521"> dk99521 소개</Link>
        </li>
      </ul>
      <hr />
      {/* 이 아래에서 보여줍니다. */}
      <Route path="/" component={Home} exact />
      <Route path={['/about', '/info']} component={About} exact />
      <Route path="/profiles/:username" component={Profile} />
    </div>
  );
};

export default App;

```



## 13.5 서브 라우터

서브 라우트는 내부에 또 라우트를 정의하는 것입니다. 라우트 내부에 다시 라우트 컴포넌트를 사용하면 됩니다.

Profiles.js

```jsx
import React from 'react';
import { Link, Route } from 'react-router-dom';
import Profile from './Profile';

const Profiles = () => {
  return (
    <div>
      <h3>사용자 목록</h3>
      <ul>
        <li>
          <Link to="/profiles/velopert"> velopert 소개</Link>
        </li>
        <li>
          <Link to="/profiles/dk99521"> dk99521 소개</Link>
        </li>
      </ul>
      <Route
        path="/profiles"
        exact
        render={() => <div>사용자를 선택해주세요 </div>}
      />
      <Route path="/profiles/:username" component={Profile} />
    </div>
  );
};

export default Profiles;

```

첫 번재 Route를 생성 이유는 선택을 설명하는 페이지가 필요합니다.

component가 있으면 render 속성은 사용하지 못합니다!! 둘 중 하나만 선택 가능합니다.

App.js

```jsx
import React from 'react';
import { Route, Link } from 'react-router-dom';
import Home from './Home';
import About from './About';
import Profiles from './Profiles';

const App = () => {
  return (
    <div>
      <ul>
        <li>
          <Link to="/">홈</Link>
        </li>
        <li>
          <Link to="/about/?detail=true">소개</Link>
        </li>
        <li>
          <Link to="/info"> 정보 소개</Link>
        </li>
        <li>
          <Link to="/profiles">프로필</Link>
        </li>
      </ul>
      <hr />
      {/* 이 아래에서 보여줍니다. */}
      <Route path="/" component={Home} exact />
      <Route path={['/about', '/info']} component={About} exact />
      <Route path="/profiles" component={Profiles} />
    </div>
  );
};

export default App;
```

## 13.6 리액트 라우터 부가 기능

### 13.6.1 history 

History 객체는 match나,location과 함께 전달되는 props 중 하나로 이 객체를 통해 컴포넌트 내에서 구현하는 메서드에서 라우터 API를 호출 할 수 있습니다. 특정 버튼을 눌렀을 때 뒤로 가거나, 로그인 후 화면을 전환하거나, 다른 페이지로 이탈하는 것을 방지할 때  사용합니다.

책에서는 클래스 컴포넌트의 라이프사이클 메서드로 진행하였지만 저는 함수형 컴포넌트와 훅을 사용해서 만들어 보겠습니다.

 

 HistorySample.js

```jsx
import React, { useEffect } from 'react';

const HistorySample = ({ history }) => {
  const handleGoBack = () => {
    history.goBack();
  };
  const handleGoHome = () => {
    history.push('/');
  };

  // unmount 할 때 설정
  useEffect(() => {
    return history.block('정말 떠나실건가요?');
  }, [history]);

  return (
    <div>
      <button onClick={handleGoBack}>뒤로</button>
      <button onClick={handleGoHome}>홈으로</button>
    </div>
  );
};

export default HistorySample;

```

App.js

```
(...)
				<li>
          <Link to="/history">history 예제</Link>
        </li>
        

      <Route path="/history" component={HistorySample} />
      (...)
        
```



### 13.6.2 withRouter

HoC(Higher-order Component) 입니다. 라우트로 사용된 컴포넌트가 아니어도 match location history 객체를 접근 가능하게 해줍니다.

WithRouterSample.js

```jsx
import React from 'react';
import { withRouter } from 'react-router-dom';
const withRouter = ({ location, match, history }) => {
  return (
    <div>
      <h4>location</h4>
      <textarea
        value={JSON.stringify(location, null, 2)}
        rows={7}
        readOnly={true}
      />
      <h4>match</h4>
      <textarea
        value={JSON.stringify(match, null, 2)}
        rows={7}
        readOnly={true}
      />
      <button onClick={() => history.push('/')}>홈으로</button>
    </div>
  );
};

export default withRouter(withRouter);

```

Profiles.js

```jsx
import React from 'react';
import { Link, Route } from 'react-router-dom';
import Profile from './Profile';
import WithRouterSample from './WithRouterSample';
const Profiles = () => {
  return (
    <div>
      <h3>사용자 목록</h3>
      <ul>
        <li>
          <Link to="/profiles/velopert"> velopert 소개</Link>
        </li>
        <li>
          <Link to="/profiles/dk99521"> dk99521 소개</Link>
        </li>
      </ul>
      <Route
        path="/profiles"
        exact
        render={() => <div>사용자를 선택해주세요</div>}
      />
      <Route path="/profiles/:username" component={Profile} />
      <WithRouterSample />
    </div>
  );
};

export default Profiles;

```

Profiles에서 렌더링 하여 준다. 그렇게 해도 profile 컴포넌트에서 보이는데 그 이유는 경로가 겹치기 때문이다.

`https://localhost:3000/profiles/dk99521` 에서 match.params은 제공되지 않은것을 볼 수 있음 이는 렌더링된 라우트에 url parameter가 제공되지 않았기 때문입니다. 그래서 제공된 profile에가서 withRouter와 WithRouterSample을 사용해주어야합니다.

Profile.js

```jsx
import React from 'react';
import WithRouterSample from './WithRouterSample';
import { withRouter } from 'react-router-dom';
const data = {
  dk99521: {
    name: '정우태',
    description: '초심자'
  },
  velopert: {
    name: '김민준',
    description: '선생님'
  }
};

const Praofile = ({ match }) => {
  const { username } = match.params;
  const profile = data[username];
  // 값을 받아올 때는 에러 처리해주어야 합니다.
  if (!profile) return <div>존재하지 않는 사용자입니다.</div>;

  const { name, description } = profile;
  return (
    <div>
      <h3>
        {username} ({name})
      </h3>
      <p>{description}</p>
      <WithRouterSample />
    </div>
  );
};
export default withRouter(Profile);

```

같은 방법으로 About에서도 적용하여 보았습니다.

### 13.6.3 Switch 

Switch 컴포넌트는 여러 Route를 감싸서 그 중 일치하는 단 하나의 라우트만 렌더링 합니다.

Switch를 사용하면 모든 규칙과 일치하지 않을 때 보여 줄 Not found 페이지도 구현할 수 있습니다.

현재 Switch 와 Route 컴포넌트를 하나씩 추가해서 path가 제공되지 않으면 이 라우트가 렌더링 됩니다.

App.js

```jsx
import React from 'react';
import { Route, Link, Switch } from 'react-router-dom';
import Home from './Home';
import About from './About';
import Profiles from './Profiles';
import HistorySample from './HistorySample';

const App = () => {
  return (
    <div>
      <ul>
        <li>
          <Link to="/">홈</Link>
        </li>
        <li>
          <Link to="/about/?detail=true">소개</Link>
        </li>
        <li>
          <Link to="/info"> 정보 소개</Link>
        </li>
        <li>
          <Link to="/profiles">프로필</Link>
        </li>
        <li>
          <Link to="/history">history 예제</Link>
        </li>
      </ul>
      <hr />
      {/* 이 아래에서 보여줍니다. */}
      <Switch>
        <Route path="/" component={Home} exact />
        <Route path={['/about', '/info']} component={About} exact />
        <Route path="/profiles" component={Profiles} />
        <Route path="/history" component={HistorySample} />
        {/* path가 제공되지 않으면 모든 상황에 대해서 렌더링 되는 기능입니다. */}
        <Route
          render={({ location }) => (
            <div>
              <h2>이 페이지는 존재하지 않습니다.</h2>
              <p>{location.params}</p>
            </div>
          )}
        />
      </Switch>
    </div>
  );
};

export default App;

```



### 13.6.4 NavLink

NavLink = Link +  조건부  css

현재 경로와 Link에서 사용하는 경로가 일치하는 경우 특정 스타일을 적용할 수 있는 클래스입니다.

스타일을 적용할때는 activeStyle의 값을 activeClassName 값을 props로 넣어 주면 됩니다.

Profiles.js

```jsx
import React from 'react';
import { Route, NavLink } from 'react-router-dom';
import Profile from './Profile';
import WithRouterSample from './WithRouterSample';
const Profiles = () => {
  const activeStyle = {
    background: 'black',
    color: 'white'
  };
  return (
    <div>
      <h3>사용자 목록</h3>
      <ul>
        <li>
          <NavLink active activeStyle={activeStyle} to="/profiles/velopert">
            velopert 소개
          </NavLink>
        </li>
        <li>
          <NavLink activeStyle={activeStyle} to="/profiles/dk99521">
            dk99521 소개
          </NavLink>
        </li>
      </ul>
      <Route
        path="/profiles"
        exact
        render={() => <div>사용자를 선택해주세요</div>}
      />
      <Route path="/profiles/:username" component={Profile} />
      <WithRouterSample />
    </div>
  );
};

export default Profiles;

```

# 질문 

NavLink의 active 속성을 왜 붙이는지 잘 모르겠네요.



# 14장 외부 API를 연동하여 뉴스 뷰어 만들기

나중에 하는걸로.