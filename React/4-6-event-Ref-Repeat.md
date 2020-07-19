# 4 이벤트 핸들링

Html은 `onClick="alert('abc')"` 처럼 "" 내부의 자바스크립트를 실행하도록 코드를 작성합니다.

## 4.1 리액트의 이벤트 시슷템

### 4.1.1 이벤트를 사용할 때 주의사항

1. 이벤트 이름은 카멜표기법 예를들어 `onClick`
2. 이벤트에 실행할 자바스크립트 코드를 전달하는 것이 아니라, 항수형태의 값을 전달 합니다.

함수를 만들어서 전달해도 되고, 렌더링 부분 외부에 미리 만들어서 전달해도 됩니다.

3. DOM 요소에만 이벤트를 설정할 수 있습니다.

돔에만 적용가능하고 저희가 만든 Component의 props에 적용하는 것은 전달해 줄 뿐입니다. 전달받은 컴포넌트 내부에 DOM이벤트에서는 사용가능하다.

### 4.1.2 이벤트 종류

리액트에서 지원하는 이벤트 종류

facebook.github.io/react/docs/events.html 을 참고하세요



## 4.2 이벤트 핸들링

### 4.2.1 컴포넌트 생성 및 불러오기

#### 4.2.1.1 컴포넌트 생성

EventPractice.js

```jsx
import React, { Component } from 'react';

class EventPractice extends Component {
  render() {
    return (
      <div>
        <h1>이벤트 연습</h1>
      </div>
    );
  }
}

export default EventPractice;

```

App.js

```jsx
import React from 'react';
import EventPractice from './EventPractice';

const App = () => {
  return <EventPractice />;
};

export default App;

```



### 4.2.2 onChange 이벤트 핸들링 하기

#### 4.2.2.1 onChange 이벤트 설정하기

EventPractice.js

```jsx
import React, { Component } from 'react';

class EventPractice extends Component {
  render() {
    return (
      <div>
        <h1>이벤트 연습</h1>
        <input
          type="text"
          placeholder="아무거나 입력해보세요"
          name="message"
          onChange={e => {
            console.log(e.target.value);
          }}
        />
      </div>
    );
  }
}

export default EventPractice;
```

이 때 기록되는 e 객체는 syntheticEvent로 웹 브라우저의 네이티브 이벤트를 감싸는 객체입니다. SyntheticEvent 및 네이티브 이벤트와 달리 이벤트가 끝나고 나면 이벤트가 초기화되므로 정보를 참조할 수 없습니다. 예를 들어, 0.5초 뒤에 e 객체 내부의 모든 값이 비워지게 됩니다. 만약 비동기적으로  e.persist() 함수를 호출해주어야합니다.

#### 4.2.2.2 state에 input 값 담기

```jsx
import React, { Component } from 'react';

class EventPractice extends Component {
  state = {
    message: ''
  };
  render() {
    return (
      <div>
        <h1>이벤트 연습</h1>
        <input
          type="text"
          placeholder="아무거나 입력해보세요"
          name="message"
          value={this.state.message}
          onChange={e => {
            this.setState({ message: e.target.value });
          }}
        />
      </div>
    );
  }
}

export default EventPractice;
```

#### 4.2.2.3 버튼을 누를 때 comment 값을 공백으로 설정

EventPractice.js

```jsx
import React, { Component } from 'react';

class EventPractice extends Component {
  state = {
    message: ''
  };
  render() {
    return (
      <div>
        <h1>이벤트 연습</h1>
        <input
          type="text"
          placeholder="아무거나 입력해보세요"
          name="message"
          value={this.state.message}
          onChange={e => {
            this.setState({ message: e.target.value });
          }}
        />
        <button
          onClick={() => {
            alert(this.state.message);
            this.setState({ message: '' });
          }}
        >
          Click
        </button>
      </div>
    );
  }
}

export default EventPractice;
```

`value={this.state.message}` 의 행위는 input의 value에 state의 값을 할당하는 것이다. 이것은 setState를 통해 input의 value를 제어할수 있도록 하기 위해서입니다.

### 4.2.3 임의 메서드 만들기

4.1.1절의 주의사항에서 "이벤트에 실행할 자바스크립트 코드를 전달하는 것이 아니라, **함수 형태의 값**을 전달합니다"

#### 4.2.3.1 함수를 밖으로 빼내서 만들고 난 후 전달하기

EventPractice.js

```jsx
import React, { Component } from 'react';

class EventPractice extends Component {
  state = {
    message: ''
  };

  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
  }

  handleChange(e) {
    this.setState({ message: e.target.value });
  }
  handleClick = () => {
    alert(this.state.message);
    this.setState({ message: '' });
  };
  render() {
    return (
      <div>
        <h1>이벤트 연습</h1>
        <input
          type="text"
          placeholder="아무거나 입력해보세요"
          name="message"
          value={this.state.message}
          onChange={this.handleChange}
        />
        <button onClick={this.handleClick}>Click</button>
      </div>
    );
  }
}

export default EventPractice;
```

() 함수로 메서드를 생성하면 생성자 함수에서 bind를 하여주어야합니다만 화살표 함수로 생성하면 그렇지 않아도 됩니다. 이는 바벨의 transform-class-properties 문법을 사용합니다.

### 4.2.4 Input 여러 개 다루기

EventPractice.js

```jsx
import React, { Component } from 'react';

class EventPractice extends Component {
  state = {
    username: '',
    message: ''
  };

  handleChange = e => {
    this.setState({ [e.target.name]: e.target.value });
  };
  handleClick = () => {
    alert(this.state.username + ':' + this.state.message);
    this.setState({ username: '', message: '' });
  };
  render() {
    return (
      <div>
        <h1>이벤트 연습</h1>
        <input
          type="text"
          placeholder="유저이름을 입력하세요"
          name="username"
          value={this.state.username}
          onChange={this.handleChange}
        />
        <input
          type="text"
          placeholder="메세지를 입력해보세요"
          name="message"
          value={this.state.message}
          onChange={this.handleChange}
        />
        <button onClick={this.handleClick}>Click</button>
      </div>
    );
  }
}

export default EventPractice;

```

객체 안에서 key를 대괄호[]로 감싸면 key에 해당하는 실제 value가 생성됩니다.

예 [e.target.name] = username

### 4.2.5 onKeyPress 이벤트 핸들링

```jsx
import React, { Component } from 'react';

class EventPractice extends Component {
  state = {
    username: '',
    message: ''
  };

  handleChange = e => {
    this.setState({ [e.target.name]: e.target.value });
  };
  handleClick = () => {
    alert(this.state.username + ':' + this.state.message);
    this.setState({ username: '', message: '' });
  };
  handleKeyPress = e => {
    if (e.key === 'Enter') {
      this.handleClick();
    }
  };
  render() {
    return (
      <div>
        <h1>이벤트 연습</h1>
        <input
          type="text"
          placeholder="유저이름을 입력하세요"
          name="username"
          value={this.state.username}
          onChange={this.handleChange}
        />
        <input
          type="text"
          placeholder="메세지를 입력해보세요"
          name="message"
          value={this.state.message}
          onChange={this.handleChange}
          onKeyPress={this.handleKeyPress}
        />
        <button onClick={this.handleClick}>Click</button>
      </div>
    );
  }
}

export default EventPractice;

```



## 4.3 함수형 컴포넌트로 구현해 보기

```jsx
import React, { useState } from 'react';

const EventPractice = () => {
  const [username, setUsername] = useState('');
  const [message, setMessage] = useState('');

  const onChangeUsername = e => {
    setUsername(e.target.value);
  };

  const onChangeMessage = e => {
    setMessage(e.target.value);
  };

  const handleClick = () => {
    alert(username + ':' + message);
    setUsername('');
    setMessage('');
  };
  const onKeyPress = e => {
    if (e.key === 'Enter') {
      handleClick();
    }
  };
  return (
    <div>
      <h1>이벤트 연습</h1>
      <input
        type="text"
        placeholder="유저를 입력하세요"
        name="username"
        value={username}
        onChange={onChangeUsername}
      />
      <input
        type="text"
        placeholder="메세지를 입력하세요"
        name="message"
        value={message} // value에 들어가는 이 컴포넌트 상태의 값이며
        								// e.target.value가 아니다.
        onChange={onChangeMessage}
        onKeyPress={onKeyPress}
      />

      <button onClick={handleClick}>Click</button>
    </div>
  );
};

export default EventPractice;
```

onChange 함수를 두가지를 구현하였는데 인풋이 많아지면 일일이 선택하기 어려우니까 다르게 바꾸자.

```jsx
import React, { useState } from 'react';

const EventPractice = () => {
  const [form, setForm] = useState({ username: '', message: '' });

  const onChange = e => {
    const nextForm = { ...form, [e.target.name]: e.target.value };
    setForm(nextForm);
  };
  const { username, message } = form;

  const handleClick = () => {
    alert(form.username + ':' + form.message);
    setForm({ username: '', message: '' });
  };
  const onKeyPress = e => {
    if (e.key === 'Enter') {
      handleClick();
    }
  };
  return (
    <div>
      <h1>이벤트 연습</h1>
      <input
        type="text"
        placeholder="유저를 입력하세요"
        name="username"
        value={username}
        onChange={onChange}
      />
      <input
        type="text"
        placeholder="메세지를 입력하세요"
        name="message"
        value={message} // value에 들어가는 이 컴포넌트 상태의 값이지  e.target.value가 아니다.
        onChange={onChange}
        onKeyPress={onKeyPress}
      />

      <button onClick={handleClick}>Click</button>
    </div>
  );
};

export default EventPractice;

```



# 5장 ref: DOM에 이름 달기

html에서 id를 사용하여 DOM에 이름을 다는 것처럼 리액트 프로젝트 내부에서 DOM에 이름을 다는 방법이 있습니다. 바로 ref 입니다. 

리액트의 dom에 id를 달게 되면 같은 id를 가진 DOM이 중복 id를 가진 DOM이 생길수 있어 잘못된 사용방법입니다. 이와달리ref는 전역적으로 작동하지않고 컴포넌트 내부에서만 작동하기 때문에 이런 문제가 생기지 않습니다.

## 5.1 ref는 어떤 상황에서 사용하는 걸까?

dom을 꼭 직접적으로 건들려야 할때 사용합니다. 다음과 같은 상황은 ref가 아닌 state로 제어할 수 있는 문제입니다.

ValidationSample.css

```css
.success {
  background-color: lightgreen;
}
.failure {
  background-color: lightcoral;
}

```

ValidationSample.js

```jsx
import React, { Component } from 'react';
import './ValidationSample.css';
class ValidationSample extends Component {
  state = { password: '', clicked: false, validated: false };

  handleChange = e => {
    this.setState({ password: e.target.value });
  };

  handleClicked = e => {
    this.setState({
      clicked: true,
      validated: this.state.password === '0000'
    });
  };

  render() {
    return (
      <div>
        <input
          type="password"
          value={this.state.password}
          className={
            this.state.clicked
              ? this.state.validated
                ? 'success'
                : 'failure'
              : ''
          }
          onChange={this.handleChange}
        ></input>
        <button onClick={this.handleClicked}>Click</button>
      </div>
    );
  }
}

export default ValidationSample;

```

### 5.1.2 App에서 리렌더링

### 5.1.3 DOM을 꼭 사용해야 하는 상황

state만으로 해결할 수 없는 기능

- 특정 input에 포커스 주기
- 스크롤 박스 조작하기
- canvas 요소에 그림 조작하기



## 5.2 ref 사용

ref 를 사용하는 방법은 2가지 방법이 있습니다.

### 5.2.1 callback function을 통한 ref 설정

ref를 달고자 하는 요소에 ref라는 콜백 함수를 props로 전달해 주면 됩니다. 이 콜백 함수는 ref 값을 파라미터로 전달받습니다. 그리고 함수 내부에서 파라미터로 받은 ref를 컴포넌트의 멤버 변수로 설정해 줍니다.w

`<input ref={ref=>{this.input=ref}} />`

 this.input은 input 요소의 dom을 가르킵니다. ref의 이름을 원하는 것으로 자유롭게 지정할 수 있습니다. DOM 타입과 관계없이 `this.superman = ref`처럼 마음대로 지정해도 됩니다.

### 5.2.2 createRef를 통한 ref 설정

ref를 만드는 또 다른 방법은 리액트에 내장되어있는 createRef라는 함수를 사용하는 것입니다. 

```jsx

class refsample extends Component {
  input = React.createRef();
	handleFocus = () =>{
    this.input.current.focus();
  }	
  render(){
    return (<div><input ref={this.ref}></input></div>)
  }
}
```

input을 멤버 변수로 설정 , ref를 달고자 하는 요소에 ref props로 넣어 주면 ref 설정이 완료됩니다. ref를 설정해 준 DOM에 접근하려면 this.input.current를 조회하면 됩니다. 콜백과 다른 점은 this.input 뒤에 current를 넣어 주어야 한다는 것입니다.

### 5.2.3 적용

클릭을하면 input요소에 커서를 자동으로 옮겨가게 하고 싶다.

ValidationSample.js

```jsx
import React, { Component } from 'react';
import './ValidationSample.css';
class ValidationSample extends Component {
  state = { password: '', clicked: false, validated: false };

  handleChange = e => {
    this.setState({ password: e.target.value });
  };

  handleClicked = e => {
    this.setState({
      clicked: true,
      validated: this.state.password === '0000'
    });
    this.input.focus();
  };

  render() {
    return (
      <div>
        <input
          type="password"
          value={this.state.password}
          className={
            this.state.clicked
              ? this.state.validated
                ? 'success'
                : 'failure'
              : ''
          }
          onChange={this.handleChange}
          ref={ref => (this.input = ref)}
        ></input>
        <button onClick={this.handleClicked}>Click</button>
      </div>
    );
  }
}

export default ValidationSample;

```

## 5.3 컴포넌트에  ref 달기

### 5.3.1 사용법 

`<MyComponent ref={(ref)=>{this.myComponent=ref}} />` 이렇게 하면 MyComponent 내부의 메서드 및 멤버 변수에도 접근 할 수 있습니다. 내부의 ref에도 접근할 수 있습니다.(myComponent.handleClick , myComponent.input )

 Scroll.js

```jsx
import React, { Component } from 'react';

class Scroll extends Component {
  scrollToBottom = () => {
    const { scrollHeight, clientHeight } = this.box;

    this.box.scrollTop = scrollHeight - clientHeight;
  };
  scrollToTop = () => {
    const { scrollHeight, clientHeight } = this.box;
    this.box.scrollTop = clientHeight - scrollHeight;
  };
  render() {
    const style = {
      border: '1px solid black',
      height: '300px',
      width: '300px',
      overflow: 'auto', // 콘텐츠의 양에 따라 스크롤 가능 여부 만들어줍니다.
      position: 'relative' // 다른 엘리먼트와의 관계를 설정하기 편하게 하기 위해 설정 한다.
    };
    const innerStyle = {
      width: '100%',
      height: '650px',
      background: 'linear-gradient(white,black)'
    };

    return (
      // 막대기가 들어가 있는곳
      <div
        style={style}
        ref={ref => {
          this.box = ref;
        }}
      >
        // 내용물
        <div style={innerStyle}></div>
      </div>
    );
  }
}

export default Scroll;

```

App.js

```jsx
import React, { Component } from 'react';
import Scroll from './Scroll';

class App extends Component {
  render() {
    return (
      <div>
        <Scroll
          ref={ref => {
            this.scrollBox = ref;
          }}
        />
        <button onClick={() => this.scrollBox.scrollToBottom()}>
          맨 밑으로
        </button>
        <button onClick={() => this.scrollBox.scrollToTop()}>맨 위로</button>
      </div>
    );
  }
}

export default App;

```

-  this.멤버변수의 이름은 상관없다.

## 5.4 정리

- 컴포넌트 내부에서 DOM에 직접 접근을 해야할 때만 ref 사용해야합니다.
- 데이터를 교류할 때에는 언제나 데이터를 부모<=>자식 흐름으로 교류해야 합니다. 리덕스 혹은 Context API를 사용하여 효율적으로 교류하는 방법을 배웁니다.
- 함수형 컴포넌트는 createRef와 비슷한 useRef를 씁니다.



# 6장 컴포넌트 반복

## 6.1 배열의 map()

### 6.1.1 문법

arr.map(callback,[thisarg])

이 함수의 파라미터는 다음과 같습니다.

- callback : 새로운 배열의 요소를 생성하는 함수로 파라미터는 다음 세가지가 있습니다.
  - currentValue : 현재 처리하고 있는 요소
  - index : 현재 처리하고 있는 요소의 indx 값
  - array : 현재 처리하고 있는 원본배열
- thisArg(선택 항목) : callback 함수 내부에서 사용할 this 레퍼런스

### 6.1.2 예제

생략하겠습니다.

## 6.2 데이터 배열을 컴포넌트 배열로 변환하기 및 key 설정

Iteration.js

```jsx
import React from 'react';

const Iteration = () => {
  const names = ['눈', '비', '얼음', '바람'];
  const nameList = names.map((name, index) => <li key={index}>{name}</li>);
  return (
    <div>
      <ul>
        {names.map((m, i) => (
          <li key={i}>{m}</li>
        ))}
      </ul>
      <ul>{nameList}</ul>
    </div>
  );
};

export default Iteration;

```

##  6.3 추가 제거 기능

```jsx
import React, { useState } from 'react';

const Iteration = () => {
  const [names, setNames] = useState([
    { id: 1, text: '눈' },
    { id: 2, text: '비' },
    { id: 3, text: '얼음' },
    { id: 4, text: '바람' }
  ]);

  const [nextText, setNextText] = useState('');
  const [nextId, setNextId] = useState(5);

  const onChange = e => {
    setNextText(e.target.value);
  };

  const onClick = () => {
    const nextNames = names.concat({ id: nextId, text: nextText }); // 실행1
    setNames(nextNames); // 살행 2
    setNextText(''); // 실행 3
    setNextId(nextId + 1); // 실행 4
  };
  const onRemove = id => {
    const nextNames = names.filter(name => name.id !== id);
    setNames(nextNames); // 살행 2
  };
  const nameList = names.map(name => (
    <li onDoubleClick={() => onRemove(name.id)} key={name.id}>
      {name.text}
    </li>
  ));
  return (
    <div>
      <input value={nextText} onChange={onChange} />
      <button onClick={onClick}>추가</button>
      <ul>{nameList}</ul>
    </div>
  );
};

export default Iteration;

```

참고로 onDoubleClick={onRemove(name.id)}를 하면 nameList가 보이지 않는다. 왜 그렇지는 잘 모르겠으나. ()=>{onRemove(name.id)} 객체 형태로 사용해야하는것을알수잇다.                           