



# 6장 TodoList App - Context와  AsyncStorage

# 리액트에서 데이터를 다루는 방법으로 **Prop**s와 **State**, 그리고 **Context**가 존재합니다.

Props와 State, 그리고 Context는 **리액트 네이티브가 동작하는 도중**에 데이터를 다루는 데 사용됩니다. 즉 다시 말하면 앱이 종료되거나 다시 실행되면 Props와 State, Context에 있던 데이터는 사라지게 됩니다. 이처럼 앱을 종료하거나 다시 실행해도 데이터가 사라지지 않게 유지하기 위해 AsyncStorage를 사용하여 데이터를 앱 내에 저장하는 방법에 대해서도 알아봅니다.



## 6.1 Context API

리액트에서 Props와 State는 부모 컴포넌트와 자식 컴포넌트 또는 한 컴포넌트 안에서 데이터를 다루기 위해 사용됩니다. 이 Props와 State를 사용하면 부모 컴포넌트에서 자식 컴포넌트, 즉 위에서 아래로 **한쪽 방향**으로만 데이터가 흘러가게 됩니다.

![image-20200721001308430](/Users/neo/Library/Application Support/typora-user-images/image-20200721001308430.png)



만약 다른컴포넌트에서 흐르고 있는 데이터를 사용하고 싶거나 다른 컴포넌트에서 사용하는 데이터를 데이터 흐름에 넣고 싶은 경우가 발생한다면 어떻게 될까요?

![image-20200721001649981](/Users/neo/Library/Application Support/typora-user-images/image-20200721001649981.png)



우리가 생각할 수 있는 방법으로는 데이터는 위에서 아래로, 한쪽 방향으로 흐르게 되므로, 사양하고 싶은 데이터와 이 데이터를 사용할 위치에 **공통 컴포넌트**(Root)에 state를 만들고 사용하고자 하는 데이터를 Props를 전달하여 이 문제를 해결 할 수 있습니다.

![image-20200721001720211](/Users/neo/Library/Application Support/typora-user-images/image-20200721001720211.png)

하지만 [그림 6-3]과 같이 컴포넌트 사이에 공유되는 데이터를 위해 매번 공통 부모 컴포넌트를 수정하고 모든 컴포넌트에 Props를 전달하여 데이터를 사용하는 과정은 매우 비효율적입니다. 이런 문제를 해결하기 위해 리액트에선는 Flux라는 개념을 도입하였고 그에 걸맞는  Context API를 제공하기 시작했습니다.

- [Flux에관한자세한내용](https://reactjs.org/blog/2014/05/06/flux.html)

![image-20200721001815937](/Users/neo/Library/Application Support/typora-user-images/image-20200721001815937.png)

 Context는 부모 컴포넌트로부터 자식 컴포넌트로 전달되는 데이터의 흐름과는 상관없이, 전역적으로 사용되는 데이터를 다룹니다. 전역 데이터를 Context에 저장한 후, 필요한 컴포넌트에서 해당 데이터를 불러와 사용합니다.



Context를 사용하기 위해서 Context API를 사용하여 Context의 Provider, Consumer를 생성합니다.  Context에 저장된 데이터를 사용하기 위해서는 [그림6-5]와 같이 **공통 부모 컴포넌트(Root)에 Context의 Provider를 사용하여 데이터를 제공**합니다. **데이터를 사용하려는 컴포넌트에서 Consumer을 사용**하여 데이터를 사용합니다.





![image-20200721001841067](/Users/neo/Library/Application Support/typora-user-images/image-20200721001841067.png)



## 6.2 AsyncStorage

props,state,context는 휘발성입니다. 이 데이터는 메모리에서만 존재하며, 사실은 물리적으로 데이터를 저장하지 않습니다. 따라서 데이터들을 API를 통해 서버에 저장하여 사용하거나, 앱 내에 저장하여 사용하는 경우가 많습니다.



AsyncStorage는

- 앱 내에서 간단하게 데이터를 저장할 수 있는 저장소입니다. 
- 웹에서 사용하는 windows.localStorage와 매우 유사합니다.
- Key-value 저장소로서 간단하게 앱 내의 데이터를 저장하기 위한 목적으로 사용합니다.
- 현재(version>0.59)을 기점으로 리액트네이티브에서 분리되어 커뮤니티 라이브러리로 분리되었습니다.(deprecated)
- "react-native-community/react-native-async-storage"를 사용합니다.



## 6.3 프로젝트 준비

3.2장에서 준비하였던 프로젝트 준비과정을 재반복 합니다.

```
$ npx react-native init TodoList --template react-native-template-typescript
$ cd TodoList
$ npm install --save styled-components
$ npm install --save-dev @types/styled-components
$ npm install --save-dev babel-plugin-root-import
```

babel.config.js

```js
module.exports = {
  presets: ['module:metro-react-native-babel-preset'],
  plugins: [
    ['babel-plugin-root-import', {rootPathPrefix: '~', rootPathSuffix: 'src'}],
  ],
};


```

타입스크립트도 설정합니다. 하지만 3.2.1 과 같은 이유로 사용하지 못합니다.

tsconfig.json

![image-20200707224058260](/Users/neo/Desktop/moim/Document/ReactNative/img/image-20200707224058260.png)

이렇게 설정한 다음 src 폴더를 생성하고 App.tsx를 src 폴더로 옮깁니다.

![image-20200707224430396](/Users/neo/Desktop/moim/Document/ReactNative/img/image-20200707224430396.png)

위 그림처럼 코드를 수정합니다.



## 6.4 개발

아래의 내용은 Context API의 사용법을 이해하기 위해 리스트 앱을 세분화하였다.

![image-20200721001915448](/Users/neo/Library/Application Support/typora-user-images/image-20200721001915448.png)



### 6.4.1 AsyncStorage 설치 및 설정

데이터 앱 내에 저장할 예정입니다. 0.59 이전의 리액트 네이티브를 사용한다면 이 부분은 건너 뛰어도 됩니다. 0.60 이후 버전은 아래명령어를 실행하여 AsyncStorage를 설치합니다.

```
$ npm install --save @react-native-community/async-storage
```

설치가 완료되면  iOS에서 필요한 라이브러리를 설치합니다. 안드로이드는 추가 라이브 설치가 필요 없습니다

```
$ cd ios
$ pod install
```

리액트 네이티브 0.60 이후에는 라이브러리가 자동으로 연결됩니다. 하지만 리액트 네이티브의 이전 버전을 사용한다면 다음 명령어를 통해 라이브러리를 연결해 줄 필요가 있습니다.

```
$ react-native link @react-native-community/async-storage
```



이것으로 AsyncStorage를 사용하여 데이터를 앱 내에 저장할 준비가 끝났습니다.



### 6.4.2 Context

 전역 데이터를 저장할 Context를 생성해 보도록 하자. 우선 Context의 데이터 타입을 정의하기 위해 src/context/todolist/@types/index.d.ts 파일을 생성하고 아래와 같이수정합니다.



src/context/@types/TodoListContext.d.ts

```ts
interface ITodoListContext {
  todoList: Array<string>;
  addTodoList: (todo: String) => void;
  removeTodoList: (index: number) => void;
}
```

한 파일내에 타입을 정의하여 사용하는 것이 아니라 다르게 이렇게 다른 파일에 미리 데이터 타입을 정의하면 프로젝트 전반에 걸쳐서 사용하기 편합니다.

src/context/TodoListContex.tsx 파일을 생성하고 다음과 같이 수정합니다.

``` tsx
import React, {createContext, useState, useEffect} from 'react';
import AsyncStorage from '@react-native-community/async-storage';

interface Props {
  children: JSX.Element | Array<JSX.Element>;
}

const TodoListContext = createContext<ITodoListContext>({
  todoList: [],
  addTodoList: (todo: string): void => {},
  removeTodoList: (index: number): void => {},
});
function TodoListContextProvider({children}: Props) {
  const [todoList, setTodoList] = useState<Array<string>>([]);

  const addTodoList = (todo: string): void => {
    const list = [...todoList, todo];
    setTodoList(list);
    AsyncStorage.setItem('todoList', JSON.stringify(list));
  };

  const removeTodoList = (index: number): void => {
    let list = [...todoList];
    list.splice(index, 1);
    setTodoList(list);
    AsyncStorage.setItem('todoList', JSON.stringify(list));
  };

  const initData = async () => {
    try {
      const list = await AsyncStorage.getItem('todoList');
      if (list !== null) {
        setTodoList(JSON.parse(list));
      }
    } catch (e) {
      console.log(e);
    }
  };
  useEffect(() => {
    initData();
  }, []);
  return (
    <TodoListContext.Provider
      value={{
        todoList,
        addTodoList,
        removeTodoList,
      }}>
      {children}
    </TodoListContext.Provider>
  );
}

export {TodoListContextProvider, TodoListContext};

```

#### createContext,useState,useEffect

createContext로 Context를 생성하고, useState로 생성한 State 데이터를 Context 안에 저장할 예정입니다. 이렇게 useState로 생성한 State를 저장함으로써 Context의 데이터를 수정할 수 있습니다.

또한 클래스 컴포넌트의 라이프 사이클 함수와 비슷한 역할을 하는 useEffect를 가지고 AsyncStorage에 저장된 데이터를 가져와 설정하도록 할 예정입니다. useEffet 함수에 대해서는 뒤에서 자세히 설명하겠습니다.

#### createContext  

createContext  함수에 초기 값을 할당하여 Context를 생성할 수 있습니다. 이 때 src/context/@types.TodoListContext.d.ts에 정의한 타입을 사용하여 Context 데이터 타입을 지정하였습니다.

```tsx
const TodoListContext = createContext<ITodoListContext>({
  todoList: [],
  addTodoList: (todo: string): void => {},
  removeTodoList: (index: number): void => {},
});
```

할 일 리스트 앱의 Context를 생성하기 위해 createContext 함수의 초깃값으로

-  todoList : 문자열 배열
- addTodoList : 초기값은 빈배열, 아무 기능하지 않는 빈 배열
- removeTodoList : 초기값은 빈배열, 아무 기능하지 않는 빈 배열

을 할당 했습니다.

실제 구현은 Context의 Provider 컴포넌트에서 할 예정입니다.

```tsx
function TodoListContextProvider({children}: Props) {
  const [todoList, setTodoList] = useState<Array<string>>([]);

...
  return (
    <TodoListContext.Provider
      value={{
        todoList,
        addTodoList,
        removeTodoList,
      }}>
      {children}
    </TodoListContext.Provider>
  );
}
```

Context를 사용하기 위해서는 우선 공통 부모 컴포넌트에서 Context의 Provider를 사용합니다. 공통 부모 컴포넌트에서 Provider를 사용하기 위해서는 Context의 Provider 컴포넌트를 만들고 공통부모 컴포넌트로써 사용합니다.

TodoListContextProvider는 Context의 프로바이더 컴포넌트로써, 공통 부모 컴포넌트의 부모 컴포넌트가 될 예정입니다. 따라서 자식 컴포넌트를 children 매개변수를 통해 전달 받습니다. 이렇게 전달 받은 자식 컴포넌트(공통 부모 컴포넌트)는 createContext로 생성한 Context의 Provider인 TodoListContext.Provider의 하위에 위치하도록 설정하였습니다.

Context를 사용하기 위해 만든 Provider컴포넌트도 리액트 네이티브 컴포넌트이므로, 컴포넌트 안에서 수정 가능한 데이터를 사용하기 위해서는 useState를 사용해야합니다.

```tsx
  const [todoList, setTodoList] = useState<Array<string>>([]);
```

우리가 만들 Context 데이터는 문자열 배열의 할 일 리스트이고, 이 Context 데이터에 데이터를 추가, 삭제합니다. 따라서 ContextProvider 컴포넌트에서 useState를 이용해 문자열의 할일 리스트를 선언하고, 이 데이터를 setTodoList 함수를 통해 추가하거나 삭제하여 Context 데이터를 다룰 예정입니다.

```tsx
  const addTodoList = (todo: string): void => {
    const list = [...todoList, todo];
    setTodoList(list);
    AsyncStorage.setItem('todoList', JSON.stringify(list));
  };

  const removeTodoList = (index: number): void => {
    let list = [...todoList];
    list.splice(index, 1);
    setTodoList(list);
    AsyncStorage.setItem('todoList', JSON.stringify(list));
  };

```

addTodoList 함수는 할 일 리스트에 할 일을 추가하기 위한 함수입니다. useState로 만든 todoList는 수정할 수 없는 불변값입니다. 따라서, 새로운 list 변수를 생성하여 todoList의 모든 데이터를 넣고(...todoList), 매개변수를 전달받은 todo를 추가하였습니다. 그리고 todoList 함수를 통해 state를 변경합니다. 마지막으로 AsyncStorage의 setItem을 사용하여 데이터를 물리적으로 저장합니다. setItem은 키 값 형태로 데이터를 관리하며 여기는 키 값은 모두 **문자열** 이어야 합니다. 따라서 문자열 배열인 데이터를 JSON.stringfy 함수를 사용하여 **문자열('todolist')**로 변경하여 저장합니다.



removeTodoList 함수는 인덱스를 파리미터로 전달 하여 할 일 리스트에서 해당 할 일을 제거합니다. 할 일 리스트 데이터인 todoList는 state 값이므로 직접 변경이 불가능합니다. 따라서, todoList 를 복사하여 새로운 배열을 생성합니다.  새롭게 복사한 배열에서 전달 받은 매개 변수를 이용하여 삭제하고자 하는 데이터를 제거하고 setTodoList를 사용하여 state에 해당 할 일이 제거된 데이터를 저장하였습니다.  마지막으로 AsyncStorage를 사용하여 물리적으로 저장된 값을 업데이트 합니다.

``` tsx
  const initData = async () => {
    try {
      const list = await AsyncStorage.getItem('todoList');
      if (list !== null) {
        setTodoList(JSON.parse(list));
      }
    } catch (e) {
      console.log(e);
    }
  };
```

initData 함수는 앱이 시작될 때, AsyncStorage에 저장된 데이터를 불러와, Context의 값을 초기화하기 위한 함수입니다. AsyncStorage의 setItem과 getItem은 모두 Promise 함수입니다. 앞서 addTodoList와 removeTodoList 함수에서는 setItem을 한 후 특정한 작업을 하지 않았기 때문에, 비동기로 데이터를 처리하였지만 여기에서는 값을 바로 초기화 하기 위해 async-awiat를 사용하여 동기화 처리 하였습니다.

AsyncStorage에 저장된 값은 문자열이므로 이 데이터를 JSON.parse 함수를 사용하여 문자열 배열로  변경했습니다.

```tsx
useEffect(() => {
    initData();
  }, []);
```

React Hook의 useEffect는 클래스 컴포넌트의 라이프 사이클 함수와 비슷한 역할을 하는 함수입니다. useEffect의 첫 번째 매개변수로 함수를 전달하였고, 그 함수에서 데이터 초기화 함수를 호출 하였습니다. 두 번째 매개변수에는 빈 배열을 전달하여, 이 useEffect가 componentDidMount와 같은 역할을 수행하도록 하였습니다.

```  tsx
export {TodoListContextProvider, TodoListContext};
```

마지막으로, Context를 제공하기 위해 Provider 컴포넌트와 Context를 내보냈습니다.



### 6.4.3 useEffect

 useEffect에 대해서 자세히 알아보도록 하겠습니다.

```tsx
  useEffect(() => {
    initData();
  }, []);
```

useEffect의 첫번째 매개변수에는 함수를 설정하여, useEffect의 역할을 정의합니다. useEffet의 두 번째 매개변수에는 배열을 전달하는데, 이번 예제에서는 빈 배열을 전달하였습니다.

이렇게 두 번째 매개변수에 빈 배열을 전달하면, 클래스형 컴포넌트 componentDidMount와 같은 역할을 수행하도록 하였습니다. 즉 컴포넌트가 처음 화면에 표시된 후, 이 useEffect는 한 번만 호출됩니다.

```tsx
  useEffect(() => {
    initData();
  });
```

위와 같이 componentDidMount와 componentDidUpdate의 역할을 동시에 수행하며, 즉 컴포넌트가 처음 화면에 표시된 후에도 실행되며, props나  state의 변경에 의해 컴포넌트가 리렌더링된 후에도 실행됩니다. (위 예제에서 이렇게 사용하게 된다면 state를 변경할 때마다 init 함수가 실행되는데 그럴 이유가 없습니다.)

```tsx
  useEffect(() => {
    ...
    return ()=>{
    
    }
  });
```

useEffect 함수의 역할을 정의하는 첫 번째 매개변수 함수는 함수를 반환할 수 있습니다. 이 반환하는 함수는 componentWillUnmount와 같은 역할을 합니다. 즉, 컴포넌트가 화면에서 사라진 후, 이 함수가 호출되며, componentWillUnmount와 마찬가지로 라이브러리와 연동을 해제하거나, 타이머를 해제하는데 사용합니다.



useEffect는 이와 같이 클래스 컴포넌트의 라이프 사이클 함수와 비슷한 역할도 하지만, useEffect만의 고유한 기능도 제공합니다.

```tsx
  useEffect(() => {
		...
  }, [todoList]);
```

useEffect의 두 번째 매개변수로 배열을 전달할 수 있습니다. 이 두 번째 매개변수 배열에 특정 변수를 설정하여 전달하면, 모든 Props와 State에 변경에 호출되는 componentDidUpdate와 다르게 전달된 변수가 변경될때만, 이 함수가 호출됩니다.



```tsx
  useEffect(() => {
		...
  }, [todoList]);
  useEffect(() => {
		...
  });
```

또한 useEffect는 클래스 컴포넌트의 라이프 사이클 함수와 다르게 한 컴포넌트에 여러 번 정의하여 사용할 수 있습니다. 따라서 componentDidMount의 역할을 useffect와 특정 변수의 값이 변경될 때, 실행되는 로직에 useEffect를 정의할 수 있습니다.

 



### 6.4.4 최상단 부모 컴포넌트에 Provider 설정

프로바이더는 Context를 공유할 컴포넌트들의 최상단 공통 컴포넌트에 사용합니다. 이 앱의 최상단 공통 컴포넌트인 src/App.tsx에서 사용하도록 설정하겠습니다.

src/App.tsx

```tsx
import React from 'react';
import styled from 'styled-components/native';
import {TodoListContextProvider} from './context/TodoListContext';
import Todo from './screens/Todo';

const Container = styled.View`
  flex: 1;
  background-color: #eee;
`;

function App() {
  return (
    <TodoListContextProvider>
      <Container>
        <Todo />
      </Container>
    </TodoListContextProvider>
  );
}

export default App;

```

앞에서 만든 Context에서 Provider컴포넌트를 불러와, 최상단 공통 부모 컴포넌트에 사용하였습니다.

### 6.4.5 Todo 컴포넌트

할 일 리스트를 화면에 표시하고 관리할 Todo 컴포넌트를 제작해 봅시다. 

src/components/Todo.tsx

```tsx
import React from 'react';
import styled from 'styled-components/native';
import TodoListView from './TodoListView';
import AddTodo from './AddTodo';
const Container = styled.View`
  flex: 1;
`;
interface Props {}
function Todo({}: Props) {
  return (
    <Container>
      <TodoListView />
      <AddTodo />
    </Container>
  );
}

export default Todo;

```

앞서 본 앱 구조에서도 설명했지만, 할 일 리스트 앱은 할 일 리스트를 보여줄 TodoListView컴포넌트와 AddTodo 컴포넌트를 가지고 있습니다.

이 책에서는 한 컴포넌트에 종속되는 컴포넌트는 src/components 폴더에 분리하지 않고, 종속된 컴포넌트 하위 폴더에 생성합니다. 이렇게 관리하면 해당 컴포넌트에 종속된 컴포넌트를 쉽게 찾을 수 있으며 공통 컴포넌트와 구별이 되는 장점이 있습니다. 

#### 참고 폴더 구조의 결정

 이 관점에서 본다면 제 폴더 구조를 비교하니 작가의 방식이 좋아 보입니다.  하지만 저(정우태)는 리액틀를 다루는 기술에서 마지막으로 사용한 폴더 구조를 사용합니다.

저는 이 예제에서는 간단하게 사용하고,



 이 작가의 폴더 구조로 작성하는 것은 비효율적입니다. 

1. 폴더명의 첫 글자를 대문자를 변환해야합니다.
2. 제가 아직 현재 폴더명을 추출하는것이 어려워 snippet으로 자동완성을 하지 못합니다.

사실 작가의 방식으로 snippet을 사용하려면  regression expression 표현식을 이용하여 적용 파일의 현재 폴더명을 추출할 수있어야합니다. 현재 저는  re에 약하므로 시간이 생기면 하는 것으로 진행하겠습니다.

### 6.4.6 TodoListView 컴포넌트

할 일 리스트를 화면에 표시하기 위한 TodoListView.tsx를 만들어 봅니다.

src/components/TodoListView.tsx

```tsx
import React from 'react';
import styled from 'styled-components/native';
import Todo from './Todo';
import Header from './Header';
import TodoList from './TodoList';

const TodoListViewBlock = styled.SafeAreaView`
  flex: 1;
`;

function TodoListView() {
  return (
    <TodoListViewBlock>
      <Header />
      <TodoList />
    </TodoListViewBlock>
  );
}

export default TodoListView;

```

앱 화면 상단에 이름을 표시하기 위한 Header 컴포넌트와 할 일 리스트를 표시할 TodoList 컴포넌트를 가지고 있다.

### 6.4.7 Header 컴포넌트

src/components/Header.tsx

```tsx
import React from 'react';
import styled from 'styled-components/native';

const HeaderBlock = styled.View`
  height: 40px;
  justify-content: center;
  align-items: center;
`;
const TitleLabel = styled.Text`
  font-size: 24px;
  font-weight: bold;
`;

interface Props {}

function Header({}: Props) {
  return (
    <HeaderBlock>
      <TitleLabel>TodoList APP</TitleLabel>
    </HeaderBlock>
  );
}

export default Header;

```

### 6.4.8  Context 데이터를 사용하는 TodoList 컴포넌트

TodoList 컴포넌트는 TodoListView 컴포넌트에서 실제로 할 일 리스트를 표시하는 컴포넌트입니다. 그래서 이 컴포넌트에서는 Context을 사용하여 저장된 할 일 데이터를 화면에 표시합니다.

src/components/TodoList.tsx

```tsx
import React, {useContext} from 'react';
import styled from 'styled-components/native';
import {FlatList} from 'react-native';
import {TodoListContext} from '~/context/TodoListContext';
import TodoItem from './TodoItem';
import EmptyItem from './EmptyItem';

const TodoListBlock = styled(FlatList)``;

interface Props {}

function TodoList({}: Props) {
  const {todoList, removeTodoList} = useContext<ITodoListContext>(
    TodoListContext,
  );
  return (
    <TodoListBlock
      data={todoList}
      keyExtractor={(item, index) => {
        return `todo-${index}`;
      }}
      ListEmptyComponent={<EmptyItem />}
      renderItem={({item, index}) => (
        <TodoItem
          text={item as string}
          onDelete={() => removeTodoList(index)}
        />
      )}
      contentContainerStyle={todoList.length === 0 && {flex: 1}}
    />
  );
}

export default TodoList;

```

함수형 컴포넌트에서 Context를 사용하기 위해서 리액트 훅의 useContext 함수를 불러와 사용하고자 하는 Context를 초기 값으로 설정하고, 해당 Context에서 사용하고자 하는 값들을 불러와 사용할 수 있습니다.



이 예제에서는 TodoListContext를  useContext의 초기 값으로 설정하였고 todoList변수와 removeTodoList함수를 불러와 사용하였습니다.



TodoList 컴포넌트는 리액트 네이티브의 리스트 뷰 중 하나인 FlatList 컴포넌트를 사용하여 만들었습니다. FlatList 컴포넌트는 아래와 같이 Props를 전달하여 사용할 수 있습니다.

```tsx
    <TodoListBlock
      data={todoList}
      keyExtractor={(item, index) => {
        return `todo-${index}`;
      }}
      ListEmptyComponent={<EmptyItem />}
      renderItem={({item, index}) => (
        <TodoItem
          text={item as string}
          oneDelete={() => removeTodoList(index)}
        />
      )}
      contentContainerStyle={todoList.length === 0 && {flex: 1}}
    />
```

- data : 리스트 뷰에서 표시할 데이터의 배열
- keyExtractor: 리액트에서 반복적으로 동일한 컴포넌트를 표시하기 위해서 컴포넌트에 키 값을 설정해야 합니다. 리액트는 이 키 값을 보고 컴포넌트를 구별하는데, 이 키 값을 설정하지 않으면 어떤 컴포넌트를 업데이트 해야 할 지 구별할 수 없기 때문에 예상치 못한 결과를 가져올 수 있습니다. 반복적으로 동일한 컴포넌트를 표시할 때는 키 값을 구분하기 위해 키 값을 설정하며, 설정 하지 않으면 경고를 표시합니다.
- ListEmptyComponent : 주어진 배열에 데이터가 없을 경우 표시되는 컴포넌트입니다.
- renderItem :  `({item,index})=>{ ...}` 파라미터에 비구조화 할당을 합니다. 주어진 배열에 데이터를 사용하여 반복적으로 표시될 컴포넌트 입니다.
- `contentContainerStyle={todoList.length === 0 && {flex: 1}}` : 표시할 데이터가 없는경우 ListEmptyComponent의 컴포넌트가 화면에 표시 됩니다. 하지만 이 컴포넌트도 하나의 리스트 아이템으로 표시되기때문에 전체화면으로 표시되지 않습니다. 이 컴포넌트를 전체 화면으로 표시하기 위해 flex : 1을 설정하였습니다.

리액트 네이티브는 CSS의 flexbox를 사용하여 화면 레이아웃을 설정합니다.



css의 flexbox를 공부하는 곳

[https://developer.mozilla.org/ko/docs/Web/CSS/CSS_Flexible_Box_Layout/Flexbox%EC%9D%98_%EA%B8%B0%EB%B3%B8_%EA%B0%9C%EB%85%90](https://developer.mozilla.org/ko/docs/Web/CSS/CSS_Flexible_Box_Layout/Flexbox의_기본_개념)



### 6.4.9 TodoList의 EmptyItem 컴포넌트

TodoList 컴포넌트에 데이터가 없을 때, 표시할 EmptyItem 컴포넌트를 만들어 보세요.

```tsx
import React from 'react';
import styled from 'styled-components/native';

const EmptyItemBlock = styled.View`
  flex: 1;
  align-items: center;
  justify-content: center;
`;
const Label = styled.Text``;

interface Props {}
function EmptyItem({}: Props) {
  return (
    <EmptyItemBlock>
      <Label>하단에 "+"를 눌러 새로운 할 일을 등록하세요</Label>
    </EmptyItemBlock>
  );
}

export default EmptyItem;

```



### 6.4.10 TodoItem 컴포넌트

TodoList에서 데이터가 있을 때 , 리스트 안의 각 데이터를 표시할 컴포넌트입니다.

components/TodoItem.tsx

```tsx
import React from 'react';
import styled from 'styled-components/native';

const TodoItemBlock = styled.View`
  flex-direction: row;
  background-color: #fff;
  margin: 4px 16px;
  padding: 8px 16px;
  border-radius: 8px;
  align-items: center;
`;
const Label = styled.Text`
  flex: 1;
`;
const DeleteButton = styled.TouchableOpacity``;
const Icon = styled.Image`
  width: 24px;
  height: 24px;
`;
interface Props {
  text: string;
  onDelete: () => void;
}
function TodoItem({text, onDelete}: Props) {
  return (
    <TodoItemBlock>
      <Label>{text}</Label>
      <DeleteButton onPress={onDelete}>
        <Icon source={require('~/assets/images/remove.png')} />
      </DeleteButton>
    </TodoItemBlock>
  );
}

export default TodoItem;

```

부모 컴포넌트(TodoList 컴포넌트)로부터, 할 일 데이터 하나를 전달받아 화면에 표시합니다.

또한 해당 할일 데이터를 지우기 위한 삭제함수 (onDelete : ()=>void)를 전달받아 삭제 아이콘을 선택했을 시, 데이터를 삭제하도록 설정했습니다.

```  TSX
interface Props {
  text: string;
  onDelete: () => void;
}
function TodoItem({text, onDelete}: Props) {
  return (
    <TodoItemBlock>
      <Label>{text}</Label>
      <DeleteButton onPress={onDelete}>
        <Icon source={require('~/Asset/Image/remove.png')} />
      </DeleteButton>
    </TodoItemBlock>
  );
}

```

이것으로 데이터를 표시하기 위한 컴포넌트는 모두 제작했다. 이제 데이터를 추가하기 위한 컴포넌트를 제작하겠습니다.

### 6.4.11 AddTodo 컴포넌트

할일 데이터를 추가하기 위한 AddTodo 컴포넌트를 제작하겠습니다.

src/components/AddTodo.tsx

```tsx
import React, {useState} from 'react';
import AddButton from './AddButton';
import TodoInput from './TodoInput';

interface Props {}

function AddTodo({}: Props) {
  const [showInput, setShowInput] = useState<boolean>(false);

  return (
    <>
      <AddButton onPress={() => setShowInput(true)} />
      {showInput && <TodoInput hideTodoInput={() => setShowInput(false)} />}
    </>
  );
}

export default AddTodo;

```

AddTodo 컴포넌트는 할 일 데이터를 입력받은 TodoInput 컴포넌트와 이 컴포넌트를 표시하기 위한 AddButton 컴포넌트를 가지고 있습니다.

showInput은 할 일을 입력하는 컴포넌트(TodoInput)를 화면에 표시하기 위한 state입니다.

```tsx
  return (
    <>
      <AddButton onPress={() => setShowInput(true)} />
      {showInput && <TodoInput hideTodoInput={()=>setShowInput(false)} />
    </>
  );
}
```

AddButton을 누르면 TodoInput 컴포넌트가 생성됩니다. TodoInput 컴포넌트에 hideTodoInput 속성을 전달하여 할일을 입력 다 한다면 컴포넌트를 숨길 수 있도록 하였습니다.

### 6.4.12 AddButton 컴포넌트

할 일을 입력하는 컴포넌트를 화면에 표시하기 위한 AddButton 컴포넌트를 제작해 보자. 

src/components/AddButton.tsx

```tsx
import React from 'react';
import styled from 'styled-components/native';

const AddButtonBlock = styled.SafeAreaView`
  position: absolute;
  bottom: 0;
  align-self: center;
  justify-content: flex-end;
`;
const ButtonContainer = styled.TouchableOpacity`
  box-shadow: 4px 4px 8px #999;
`;
const Icon = styled.Image``;
interface Props {
  onPress?: () => void;
}
function AddButton({onPress}: Props) {
  return (
    <AddButtonBlock>
      <ButtonContainer onPress={onPress}>
        <Icon source={require('~/assets/images/add.png')} />
      </ButtonContainer>
    </AddButtonBlock>
  );
}

export default AddButton;

```

onPress를 전달 받아 이미지 버튼이 눌러졌을 때 onPress를 활성화할 수 있도록 하였습니다.

### 6.4.13  TodoInput 컴포넌트

src/components/TodoInput.tsx

```tsx
import React from 'react';
import styled from 'styled-components/native';
import Background from './Background';
import TextInput from './TextInput';
import {Platform} from 'react-native';

const TodoInputBlock = styled.KeyboardAvoidingView`
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  justify-content: flex-end;
`;
interface Props {
  hideTodoInput: () => void;
}

function TodoInput({hideTodoInput}: Props) {
  return (
    <TodoInputBlock behavior={Platform.OS === 'ios' ? 'padding' : undefined}>
      <Background onPress={hideTodoInput} />
      <TextInput hideTodoInput={hideTodoInput} />
    </TodoInputBlock>
  );
}

export default TodoInput;

```

할 일 컴포넌트는 화면을 어둡게 처리할 Background 컴포넌트와 할 일 텍스트를 입력받을 TextInput 컴포넌트를 가지고 있습니다.

화면에 표시된 TodoInput 컴포넌트를 숨기기 위해 부모 컴포넌트(AddTodo)로부터 hideTodoInput 함수를 hideTodoInput 함수를 Props 전달 받았습니다.

이 함수는 Background 컴포넌트를 눌렀을 때와 TextInput 컴포넌트에서 텍스트 입력이 완료 되었을 때, 호출하여 컴포넌트를 숨길 예정입니다.

마지막으로 TodoInput에는 KeyboardAvoidingView라는 컴포넌트를 사용했습니다.

이 컴포넌트는 iOS에서 키보드가 활성화되면서 입력창을 가리는 문제를 해결하기 위하여 iOS만 padding 옵션을 주었습니다.

질문 : ios 패딩을 안한다면?

확인 : 입력창이 가려집니다.



### 6.4.14 Background 컴포넌트

TodoInput이 활성화 되었을때 화면을 어둡게 만들어 줍니다

components/Background.tsx

```tsx
import React from 'react';
import styled from 'styled-components/native';

const BackgroundBlock = styled.TouchableWithoutFeedback`
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
`;

const BlackBackground = styled.View`
  background: #000;
  opacity: 0.3;
  width: 100%;
  height: 100%;
`;

interface Props {
  onPress: () => void;
}
function Background({onPress}: Props) {
  return (
    <BackgroundBlock onPress={onPress}>
      <BlackBackground />
    </BackgroundBlock>
  );
}

export default Background;

```

onPress 함수를 통해 해당 뷰 컴포넌트를 선택하면, TodoInput 컴포넌트를 숨기도록 설정하였습니다.

### 6.4.15 Context에 데이터를 추가하는 TextInput 컴포넌트

Context를 사용하여 할 일 데이터를 추가할 TextInput 컴포넌트를 생성하겠습니다.

components/TextInput.tsx

```tsx
import React, {useContext} from 'react';
import styled from 'styled-components/native';
import {TodoListContext} from '~/context/TodoListContext';

const TextInputBlock = styled.TextInput`
  width: 100%;
  height: 40px;
  background-color: #fff;
  padding: 0px 8px;
`;

interface Props {
  hideTodoInput: () => void;
}
function TextInput({hideTodoInput}: Props) {
  const {addTodoList} = useContext<ITodoListContext>(TodoListContext);

  return (
    <TextInputBlock
      autoFocus={true}
      autoCapitalize="none"
      autoCorrect={false}
      placeholder="할 일을 입력하세요"
      returnKeyType="done"
      onSubmitEditing={({nativeEvent}) => {
        addTodoList(nativeEvent.text);
        hideTodoInput();
      }}
    />
  );
}

export default TextInput;
```



## 6.5 결과 확인

```
npm run ios
```

![image-20200726002203812](/Users/neo/Library/Application Support/typora-user-images/image-20200726002203812.png)



각 기능을 잘 확인하세요.

+ AddButton버튼 `+` 으로 TodoInput 컴포넌트 활성화
+ 할일 텍스트를 입력 후 완료시 Context에 추가 후 키보드 AddTodo 컴포넌트 숨기기
+ 어두운 화면 클릭시 되돌아가기
+ 삭제 기능 

## 6.6 요약

Context를 사용하여 전역관리를 해보았습니다.

Context API가 나온 후에도 많은 개발자들이 Redux와 Mobx와 같은 상태 관리 라이브러리를 사용하고 있습니다.  

마지막으로 AsyncStorage는 로그인 이후, 서버로부터 전달받은 토큰을 저장하거나, 정보를 캐싱하는 데 사용하는 등, 많은 곳에서 활용됩니다. 꼭 기억해두세요.



