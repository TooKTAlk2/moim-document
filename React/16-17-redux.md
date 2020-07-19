# 16장 리덕스 라이브러리 이해하기

리액트는 리덕스 관련 상태 관리 라이브러리 입니다. 리덕스를 사용하는 것이 유일한 해결책은 아니며 Context API를 통해서도 관리할 수 있습니다. 체계적으로도 관리해주고 개발자 도구 및 미들웨이 기능을 제공해 줍니다.

## 16.1 개념 미리 정리하기

### 16.1.1 액션

상태에 변화가 필요하면 액션이라는 것이 발생합니다. 이 액션 객체는 다음과 같이 이루어져있습니다.

```
{
	type: 'TOGGLE-EVENT'
	checked : false
}
```

액션 객체는 type 필드를 반드시 가지고 있어야합니다. 이 값은 액션의 이름이라고 생각하시면 됩니다. 그 이외의 값들은 업데이트 할 때 참고할 값입니다.

### 16.1.2 액션 생성 함수

액션 생성 함수는 액션 객체를 만들어 주는 함수 입니다.

```jsx
const addTodo = data =>({type:ADD_TODO, data})
```

어떤 변화를 일으켜야 할 때마다 액션 객체를 만들어야 하는데 매번 액션 객체를 직접 작성하기 번거로우니 이를 함수로 관리합니다.

### 16.1.3 리듀서

변화를 일으키는 함수입니다. 액션을 만들어서 발생시키면 리듀서가 현재 상태와 액션 객체를 파라미터로 받아옵니다.

### 16.1.4 스토어

프로젝트에 리덕스를 적용하기 위해 스토어를 만듭니다. 상태와 리듀서을 모아둔 곳이라 생각하면 됩니다. 다만 한개의 프로젝트에는 단일 스토어를 추천합니다. 이외에도 중요한 내장 함수를 지닙니다.

### 16.1.5 디스패치

스토어의 내장함수 중 하나로 '액션을 발생시키는 것'이라고 생각하면됩니다. 이 함수는 dispatch(action) 처럼 액션 객체를 파라미터로 넣어 호출합니다.

### 16.1.6 구독

구독도 내장 함수 중 하나로 이 함수 내에 리스너 함수를 파라미터로 넣어 호출해주면 이 리스너 함수가 액션이 디스패치되어 상태가 업데이트 될때 마다 호출합니다.



## 16.2 리액트 없이 쓰는 리덕스

리덕스는 리액트에 종속된 라이브러리가 아닙니다. 새로 만들어 사용해보겠습니다.

### 16.2.1  Parcel로 프로젝트 만들기

```jsx
yarn global add parcel-bundler
npm install -g parcel-bundler
mkdir 16-reudx-lib-vanilla
cd 16-reudx-lib-vanilla
yarn init -y
```

package.json. 생성

### 16.2.2 UI 및 지정

index.html

```jsx
<html>
<head><link rel="stylesheet" type="text/css" href="index.css"/>
</head>
  <body>
    <div  class="toggle"></div>
    <hr />
    <h1>0<h1>
      <button id="increase">+1</button>
        <button id="decrease">-1</button>
    <script src="./index.js"></script>
  </body>
</html>
```

index.css

```css
.toggle {
  border: 2px solid black;
  width: 64px;
  height: 64px;
  border-radius: 32px;
  box-sizing: border-box;
}

.toggle.active {
  background: yellow;
}

```

index.js

```js
console.log('hello parcel')
```

서버 실행

```
parcel index.html
// 리덕스 설치
yarn add redux
```



### 16.2.3 리덕스 모듈 작성

index.js

```js
import { createStore } from 'redux';
// DOM REFERENCE 만들기
const divToggle = document.querySelector('.toggle');
const counter = document.querySelector('h1');
const btnIncrease = document.querySelector('#increase');
const btnDecrease = document.querySelector('#decrease');
// 액션에 이름 정의
const TOGGLE_SWITCH = 'TOGGLE_SWITCH';
const INCREASE = 'INCREASE';
const DECREASE = 'DECREASE';

//액션 생성 함수 camel notation
const toggleSwitch = () => ({ type: TOGGLE_SWITCH });
const increase = () => ({ type: INCREASE });
const decrease = () => ({ type: DECREASE });

// 초기값 설정
const initialState = { toggle: false, counter: 0 };

// 리듀서 만들기
function reducer(state = initialState, action) {
  switch (action.type) {
    case TOGGLE_SWITCH:
      return { ...state, toggle: !state.toggle };
    case INCREASE:
      return { ...state, counter: state.counter + 1 };
    case DECREASE:
      return { ...state, counter: state.counter - 1 };
    default:
      return state;
  }
}
// 스토어 만들기
const store = createStore(reducer);

// render 함수 만들기
const render = () => {
  // 토글이 참이라면 노란색, 참이 아니라면 삭제
  const state = store.getState(); // 현재상태 호출
  if (state.toggle) {
    divToggle.classList.add('active');
  } else {
    divToggle.classList.remove('active');
  }
  // 카운터 처리
  counter.innerText = state.counter;
};

// 최초 한번 render
render();

// 구독하기 (상태가 바뀔때마다 render 함수를 호출)
store.subscribe(render);

// 액션 발생시키기
divToggle.onclick = () => {
  store.dispatch(toggleSwitch());
};
btnIncrease.onclick = () => {
  store.dispatch(increase());
};
btnDecrease.onclick = () => {
  store.dispatch(decrease());
};

```



## 16.3 리덕스의 규칙

### 16.3.1 단일스토어

단일스토로 관리하셔야 관리가 편하십니다.

### 16.3.2 읽기전용상태

리덕스의 상태는 읽기전용입니다. 불변성을 지켜주셔야합니다.

### 16.3.3 리듀서는 순수한 함수

- 순수한 함수는 이전 상태와 액션 객체를 파라미터로 받습니다.
- 파라미터 외의 값에는 의존하면 안 됩니다.
- 이전 상태는 절대로 건드리지 않고 , 변화를 준 새로운 상태 객체를 만들어 반환합니다.
- 똑같은 파라미터로 호출된 리듀서 함수는 언제나 똑같은 결과 값을 반환합니다.

즉 리듀서 내부에서 랜덤값을 만들거나, Date 함수를 사용하여 현재 시간을 가져오거나 네트워크요청을 한다면 파라미터가 같아도 다른 결과를 만들어 낼 수 있기 때문에 리듀서 내부에서 사용하면 안됩니다. 이러한 작업은 액션을 만드는 과정이나 리덕스 미들웨어에서 처리하시면 됩니다.



# 17 리덕스를 사용하여 리액트 애플리케이션 상태 관리하기

store.dispatch와 store.subscribe를 사용하기 보다는 connect와 Provider를 사용하여 리덕스 관련 작업을 처리합니다.

## 17.1 작업환경 설정

```
 yarn create react-app 17-react-redux
 yarn add redux react-redux redux-devtools-extension redux-actions immer
```

## 17.2 UI 준비

components/Counter.js

```jsx
import React from 'react';

const Counter = ({ number, onIncrease, onDecrease }) => {
  return (
    <div>
      <h1>{number}</h1>
      <button onClick={onIncrease}>+1</button>
      <button onClick={onDecrease}>-1</button>
    </div>
  );
};

export default Counter;

```

components/Todos.js

```jsx
import React from 'react';
import TodoItem from './TodoItem';

const Todos = ({
  input,
  todos,
  onChangeInput,
  onInsert,
  onRemove,
  onToggle
}) => {
  const onSubmit = e => {
    e.preventDefault();
    onInsert(input);
    onChangeInput('');
  };
  const onChange = e => onChangeInput(e.target.value);
  return (
    <div>
      <form onSubmit={onSubmit}>
        <input onChange={onChange} value={input} />
        <button type="submit">등록</button>
      </form>
      {todos.map(todo => (
        <TodoItem
          key={todo.id}
          todo={todo}
          onRemove={onRemove}
          onToggle={onToggle}
        />
      ))}
    </div>
  );
};

export default Todos;

```

components/TodoItem.js

```jsx
import React from 'react';

const TodoItem = ({ todo, onRemove, onToggle }) => {
  const { id, text, done } = todo;
  return (
    <div>
      <input
        type="checkbox"
        onClick={() => onToggle(id)}
        checked={done}
        readOnly={true}
      ></input>
      <span style={{ textDecoration: done ? 'line-through' : 'none' }}>
        {text}
      </span>
      <button onClick={() => onRemove(id)}>삭제</button>
    </div>
  );
};

export default TodoItem;

```

## 17.3 리덕스 관련 코드 작성

Ducks 패턴 (components/modules/containers 디렉터리)로 작성

- 액션 타입 선언,
- 액션 생성 함수 createAction 활용
- 초기 상태 선언
- 리듀서 handleActions 활용

### 17.3.1 리덕스 모듈

modules/counter.js

```jsx
import { createAction, handleActions } from 'redux-actions';

const INCREASE = 'counter/INCREASE';
const DECREASE = 'counter/DECREASE';

export const increase = createAction(INCREASE);
export const decrease = createAction(DECREASE);

const initialState = {
  number: 0
};

const counter = handleActions(
  {
    [INCREASE]: state => ({ number: state.number + 1 }),
    [DECREASE]: (state, action) => ({ number: state.number - 1 })
  },
  initialState
);

export default counter;

```

modules/todos.js

```jsx
import { createAction, handleActions } from 'redux-actions';
import produce from 'immer';
const CHANGE_INPUT = 'todos/CHANGE_INPUT';
const INSERT = 'todos/INSERT';
const TOGGLE = 'todos/TOGGLE';
const REMOVE = 'todos/REMOVE';

export const changeInput = createAction(CHANGE_INPUT, input => input);
let id = 4;
export const insert = createAction(INSERT, text => ({
  id: id++,
  text: text,
  done: false
})); // 필수
export const toggle = createAction(TOGGLE, id => id); // 필수 아님
export const remove = createAction(REMOVE);

const initialState = {
  input: '',
  todos: [
    { id: 1, text: '할 일 1', done: false },
    { id: 2, text: '할 일 2', done: false },
    { id: 3, text: '할 일 3', done: true }
  ]
};

const todos = handleActions(
  {
    [CHANGE_INPUT]: (state, { payload: input }) =>
      produce(state, draft => {
        draft.input = input;
      }),
    [INSERT]: (state, { payload: todo }) =>
      produce(state, draft => {
        draft.todos.push(todo);
      }),
    [TOGGLE]: (state, { payload: id }) =>
      produce(state, draft => {
        const todo = draft.todos.find(todo => todo.id === id);
        todo.done = !todo.done;
      }),
    [REMOVE]: (state, { payload: id }) =>
      produce(state, draft => {
        const index = draft.todos.findIndex(todo => todo.id === id);
        draft.todos.splice(index, 1);
      })
  },
  initialState
);

export default todos;

```

modules/index.js

```jsx
import { combineReducers } from 'redux';
import counter from './counter';
import todos from './todos';
const rootReducer = combineReducers({
  counter,
  todos
});

export default rootReducer;

```

index.js

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import * as serviceWorker from './serviceWorker';
import { createStore } from 'redux';
import rootReducer from './modules';
import { Provider } from 'react-redux';
import { composeWithDevTools } from 'redux-devtools-extension';
const store = createStore(rootReducer, composeWithDevTools());

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
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
import CounterContainer from './containers/CounterContainer';
import TodosContainer from './containers/TodosContainer';
const App = () => {
  return (
    <div>
      <CounterContainer />
      <hr />
      <TodosContainer />
    </div>
  );
};

export default App;

```

### 17.3.2 컨테이너 컴포넌트

containers/CounterContainer.js

```jsx
import React, { useCallback } from 'react';
import Counter from '../components/Counter';
import { increase, decrease } from '../modules/counter';
import { useSelector, useDispatch } from 'react-redux';

const CounterContainer = () => {
  const number = useSelector(state => state.counter.number);
  const dispatch = useDispatch();

  const onIncrease = useCallback(() => dispatch(increase()), [dispatch]);
  const onDecrease = useCallback(() => dispatch(decrease()), [dispatch]);
  return (
    <Counter number={number} onIncrease={onIncrease} onDecrease={onDecrease} />
  );
};
export default React.memo(CounterContainer);

```

TodosContainer.js

```jsx
import React from 'react';
import Todos from '../components/Todos';
import { useSelector } from 'react-redux';
import { changeInput, insert, remove, toggle } from '../modules/todos';
import useActions from '../lib/useActions';
const TodosContainer = () => {
  const { input, todos } = useSelector(state => ({
    input: state.todos.input,
    todos: state.todos.todos
  }));

  const [onChangeInput, onInsert, onRemove, onToggle] = useActions(
    [changeInput, insert, remove, toggle],
    []
  );
  return (
    <Todos
      input={input}
      todos={todos}
      onChangeInput={onChangeInput}
      onInsert={onInsert}
      onRemove={onRemove}
      onToggle={onToggle}
    />
  );
};

export default React.memo(TodosContainer);

```



## 17.5 컨테이너 컴포넌트 만들기

2가지 방식으로 만들 수 있음

### 17.5.1 connect (mapStateToProps,mapDispatchToProps)

containers/CounterContainers.js

```jsx
import React, { useCallback } from 'react';
import Counter from '../components/Counter';
import { increase, decrease } from '../modules/counter';
import {connect} from 'react-redux'
const CounterContainer = ({number,increase,decrease}) => {

  return (
    <Counter number={number} onIncrease={increase} onDecrease={decrease} />
  );
};

export default connect(state=>({number:state.counter.number}),{increase,decrease})(CounterContainer);

```



### 17.5.2 useSelector+useDispatch + React.memo() 사용

containers/CounterContainer.js

```jsx
import React, { useCallback } from 'react';
import Counter from '../components/Counter';
import { increase, decrease } from '../modules/counter';
import { useSelector, useDispatch } from 'react-redux';

const CounterContainer = () => {
  const number = useSelector(state => state.counter.number);
  const dispatch = useDispatch();

  const onIncrease = useCallback(() => dispatch(increase()), [dispatch]);
  const onDecrease = useCallback(() => dispatch(decrease()), [dispatch]);
  return (
    <Counter number={number} onIncrease={onIncrease} onDecrease={onDecrease} />
  );
};
export default React.memo(CounterContainer);

```

containers/TodosContainer.js

```jsx
import React from 'react';
import Todos from '../components/Todos';
import { useSelector } from 'react-redux';
import { changeInput, insert, remove, toggle } from '../modules/todos';
import useActions from '../lib/useActions';
const TodosContainer = () => {
  const { input, todos } = useSelector(state => ({
    input: state.todos.input,
    todos: state.todos.todos
  }));

  const [onChangeInput, onInsert, onRemove, onToggle] = useActions(
    [changeInput, insert, remove, toggle],
    []
  );
  return (
    <Todos
      input={input}
      todos={todos}
      onChangeInput={onChangeInput}
      onInsert={onInsert}
      onRemove={onRemove}
      onToggle={onToggle}
    />
  );
};

export default React.memo(TodosContainer);

```



## 17.6 리덕스 더 편하게 사용하기

### 17.6.1 redux-actions

#### createAction 기능

```js
const MY_ACTION = 'sample/MY_ACTION';
const myAction = createAction(MY_ACTION);
const action = myAction('hello world');
{
  type: MY_ACTION
  payload: 'hello world'
}
const MY_ACTION = 'sample/MY_ACTION';
const myAction = createAction(MY_ACTION, text=>`${text} !`);
const action = myAction('hello world');
{
  type: MY_ACTION
  payload: 'hello world !'
}
```

액션에 필요한 추가 데이터는 payload라는 키 값을 사용합니다. 만약 변형을 넣어 주고 싶다면 두번째 파라미터에 payload를 정의하는 함수를 따로 선언해서 넣어주면 됩니다.

insert는 todo 객체를 안에 넣어주어야하기 때문에 두번 째 파라미터에서 text=>todo객체를 반환하는 함수를 넣어 주어야했습니다. 나머지 함수에는 파라미터를 그대로 반환하는 함수를 넣었지만 이것은 필수가 아닙니다. 그러나 넣어주면 함수의 파라미터로 어떤 값이 필요한지 쉽게 파악할 수 있습니다.

#### handleActions 기능

handleActions로 리듀서를 재작성합니다.  action.payload에 파라미터가 다 담겨 있습니다. 따라서 payload를 조회하여 라듀서를 작성하면 됩니다.



## 17.6.2 Immer 작성

이미 앞에서 작성하였습니다. 객체의 깊이가 깊고 복잡할수록 사용하면 좋습니다. 사용할 때는 새로운 객체를 만들어서 반환하는 것이 아니라 기존 객체를 수정하면 됩니다.

```jsx
const todos = handleActions(
  {
    [CHANGE_INPUT]: (state, { payload: input }) =>
      produce(state, draft => {
        draft.input = input;
      }),
    [INSERT]: (state, { payload: todo }) =>
      produce(state, draft => {
        draft.todos.push(todo);
      }),
    [TOGGLE]: (state, { payload: id }) =>
      produce(state, draft => {
        const todo = draft.todos.find(todo => todo.id === id);
        todo.done = !todo.done;
      }),
    [REMOVE]: (state, { payload: id }) =>
      produce(state, draft => {
        const index = draft.todos.findIndex(todo => todo.id === id);
        draft.todos.splice(index, 1);
      })
  },
  initialState
);
```



## 17.7 Hooks를 사용하여 컨테이너 컴포넌트 만들기

## 17.7 Hooks를 사용하여 컨테이너 컴포넌트 만들기

### 17.7.1 useSelector, useDispatch 

스토어에서 상태와 액션 생성 함수를 받아옵니다.

### 17.7.2 useStore

 스토어에 직접 접근시 사용. 하지만 그럴 상황은 흔치 않으니 사용 할 일이 없으시다고 설명하였습니다.

### 17.7.4 useActions 사용하여도 좋음

액션이 많을 때 useDispatch로 직접 액션함수들을 훅을 사용하여 액션을 디스패치하면됩니다.  리액트 개발자들이 훅에서 제공하려다 굳이 할 필요가 없다고 생각되어 제외하였지만 사용해도 괜찮습니다.

lib/useActions.js

```jsx
import { bindActionCreators } from 'redux';
import { useDispatch } from 'react-redux';
import { useMemo } from 'react';

export default function useActions(actions, deps) {
  const dispatch = useDispatch();
  return useMemo(
    () => {
      if (Array.isArray(actions)) {
        return actions.map(a => bindActionCreators(a, dispatch));
      }
      return bindActionCreators(actions, dispatch);
    },
    deps ? [dispatch, ...deps] : [dispatch]
  );
}

```



### 17.7.5 connect 와 차이점

connect 함수를 사용하여 컨테이너 컴포넌트를 만들었을 경우 , 부모 컴포넌트의 리렌더링에 의한 리렌더링이 방지됩니다. 하지만 useSelector를 사용할때는 이러한 최적화 작업을 하지 못합니다. 따라서 React.memo() 로 컨테이너를 감싸주어야합니다. 