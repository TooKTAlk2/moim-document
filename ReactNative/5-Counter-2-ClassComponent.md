# 5 카운터 앱(2) - 클래스 컴포넌트



## 5.1 클래스 컴포넌트

리액트 네이티브의 버전이 0.60으로 업데이트 되면서 **함수형 컴포넌트가 기본 컴포넌트로 변경**되었습니다. 하지만 아직까지 많은 라이브러리, 예제들이 클래스 컴포넌트를 사용하고 있으므로, 클래스 컴포넌트를 다루는 방식을 이해할 필요가 있습니다.

리액트 훅이 나오기 전까지 클래스컴포넌트를 메인으로 사용하였습니다. 그 이유는 함수형 컴포넌트에서는 State를 사용할 수 없었기 때문인데 이 문제때문에 State를 가지는 컴포넌트는 클래스. 컴포넌트로 작성하고 Props만 받아 화면에 표시할 때는 함수형 컴포넌트를 사용했습니다.

리액트 훅이 나오면서 함수형 컴포넌트에서도 state를 사용할 수 있게 되었습니다.



https://ko.reactjs.org/docs/hooks-intro.html#motivation 에 리액트가 왜 함수형 컴포넌트를 선택했고 훅을 도입했는지 설명되어있습니다.



## 5.2 프로젝트 준비

4장의 프로젝트 준비와 내용과 같습니다.

## 5.3 개발

Button 컴포넌트에서는 state를 사용하지 않으니 Counter 컴포넌트를 클래스 컴포넌트로 변경하며 클래스 컴포넌트를 이해해봅니다.

Button 컴포넌트는 4장에서 만든 내용을 그대로 가져 옵니다.

src/components/common/Button.tsx

```tsx
import React from 'react';
import styled from 'styled-components/native';

// Button 컴포넌트를 리액트 네이티브의 TouchableOpacity, Image로 구현합니다.
const Container = styled.TouchableOpacity``;
const Icon = styled.Image``;

// 타입스크립트의 인터페이스 문법을 통해 컴포넌트의 Props의 타입을 지정하여 타입에 대한 에러와 버그를 줄이고, 정확하게 파악할 수 있도록 하였습니다.
// iconName 필수, onPress 미필수
// iconName은 'plus' 또는 'minus'
// onPress no parma -> void 반환형 함수( 아마 버튼 클릭때 사용할 듯)
interface Props {
  iconName: 'plus' | 'minus';
  onPress?: () => void;
}

// Container는 Button 부모로 Button 컴포넌트가 받은 onPress와 props을 연결하여 사용합니다.
// Image 지정할 때는 require로 기본 사이즈의 이미지를 연결하고, 기본 사이즈의 이미지 이외에 2x,3x 크기의 이미지를 가지고 있다면 리액트 네이티브는 해당 단말기 화면 사이즈에 맞는 이미지 사이즈를
// 자동으로 불러와 표시합니다.
function Button({iconName, onPress}: Props) {
  return (
    <Container onPress={onPress}>
      <Icon
        source={
          iconName === 'plus'
            ? require('~/Assets/Images/add.png')
            : require('~/Assets/Images/remove.png')
        }></Icon>
    </Container>
  );
}

export default Button;

```



src/components/screens/Counter.tsx

```tsx
import React from 'react';
import styled from 'styled-components/native';
import Button from '~/components/common/Button';

const Container = styled.SafeAreaView`
  flex: 1;
`;
const TitleContainer = styled.View`
  flex: 1;
  justify-content: center;
  align-items: center;
`;
const TitleLabel = styled.Text`
  font-size: 24px;
`;
const NumberContainer = styled.View`
  flex: 2;
  justify-content: center;
  align-items: center;
`;
const NumberLabel = styled.Text`
  font-size: 24px;
  font-weight: bold;
`;
const ButtonContainer = styled.View`
  flex: 1;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: space-around;
`;
interface Props {
  title?: string;
  initValue: number;
}
// 함수형 컴포넌트와 달리 상태의 타입을 미리 정의합니다.
interface State {
  count: number;
}
class Counter extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props);
    console.log('constructor');

    this.state = {
      count: props.initValue,
    };
  }

  render() {
    const {title} = this.props;
    const {count} = this.state;

    return (
      <Container>
        {title && (
          <TitleContainer>
            <TitleLabel>{title}</TitleLabel>
          </TitleContainer>
        )}
        <NumberContainer>
          <NumberLabel>{count}</NumberLabel>
        </NumberContainer>
        <ButtonContainer>
          <Button
            iconName="plus"
            onPress={() => this.setState({count: count + 1})}
          />
          <Button
            iconName="minus"
            onPress={() => this.setState({count: count - 1})}
          />
        </ButtonContainer>
      </Container>
    );
  }
}

export default Counter;

```



함수형 컴포넌트과 다른점

1. State 타입을 미리 정의

2. useState가 아니라 constructor 함수 내부에서 초기값 설정

3. constructor에서 항상 super(props); 으로 부모 컴포넌트의 생성자 함수를 호출해야 합니다.

4. 생성자 함수에서만 this.state에 값을 할당할수 있습니다.

5. render 함수는 컴포넌트를 렌더링(화면에 표시)할 때 호출됩니다.

6. props,state에 접근할 때 this를 함께 사용합니다.

7. render함수 내부에서 state는 불변값이므로 변경하고자 할때는 this.setState(객체)를 사용합니다.

   







src/App.tsx

```tsx
import React from 'react';
import styled from 'styled-components/native';
import Counter from './Screens/Counter';

const Container = styled.View`
  flex: 1;
  background-color: #eee;
`;

function App() {
  return (
    <Container>
      <Counter title="This is a Counter App" initValue={5} />
    </Container>
  );
}

export default App;

```







## 5.4 라이프 사이클 함수

정우태 : 리액트 다루는 기술에 이 내용이 포함되어 있습니다.

클래스 컴포넌트는 함수형 컴포넌트와 다르게 라이프 사이클 함수들을 가지고 있습니다. 이 라이프 사이클 함수를 잘 이해하면 클래스 컴포넌트를 좀 더 효율적으로 활용할 수 있습니다 다음은 리액트의 모든 라이프 사이클 함수를 적용한 예제입니다.

```tsx
import React from 'react';
import styled from 'styled-components/native';
import Button from '../../components/common/Button';
const Container = styled.SafeAreaView`
  flex: 1;
`;

const TitleContainer = styled.View`
  flex: 1;
  justify-content: center;
  align-items: center;
`;
const TitleLabel = styled.Text`
  font-size: 24px;
`;

const CountContainer = styled.View`
  flex: 2;
  justify-content: center;
  align-items: center;
`;
const CountLabel = styled.Text`
  font-size: 24px;
  font-weight: bold;
`;

const ButtonContainer = styled.View`
  flex: 1;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: space-around;
`;

interface Props {
  title?: string;
  initValue: number;
}
interface State {
  count: number;
  error: boolean;
}

class Counter extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props);
    console.log('constructor');

    this.state = {
      count: props.initValue,
      error: false,
    };
  }

  render() {
    console.log('render');
    const {title} = this.props;
    const {count, error} = this.state;
    return (
      <Container>
        {!error && (
          <>
            {title && (
              <TitleContainer>
                <TitleLabel>{title}</TitleLabel>
              </TitleContainer>
            )}
            <CountContainer>
              <CountLabel>{count}</CountLabel>
            </CountContainer>
            <ButtonContainer>
              <Button
                iconName="plus"
                onPress={() => this.setState({count: count + 1})}
              />
              <Button
                iconName="minus"
                onPress={() => this.setState({count: count - 1})}
              />
            </ButtonContainer>
          </>
        )}
      </Container>
    );
  }

  static getDerivedStateFromProps(nextProps: Props, prevState: State) {
    console.log('getDerivedStateFromProps');

    return null;
  }

  componentDidMount() {
    console.log('componentDidMount');
  }

  shouldComponentUpdate(nextProps: Props, nextState: State) {
    console.log('shouldComponentUpdate');
    return true;
  }

  getSnapshotBeforeUpdate(prevProps: Props, prevState: State) {
    console.log('getSnapshotBeforeUpdate');

    return null;
  }

  componentDidUpdate(prevProps: Props, prevState: State, snapshot: null) {
    console.log('componentDidUpdate');
  }

  componentWillUnmount() {
    console.log('componentWillUnmount');
  }

  componentDidCatch(error: Error, info: React.ErrorInfo) {
    this.setState({
      error: true,
    });
  }
}
export default Counter;

```

### 5.4.1 constructor 

- state 초기값 설정
- 생성자 함수는 해당 컴포넌트가 생성될 때 한 번만 호출됩니다.

클래스 컴포넌트는 클래스이기 때문에  생성자 함수가 존재합니다. 만약 클래스 컴포넌트에서 state의 초기값 설정이 필요없다면 생성자 함수도 생략이 가능합니다. 생성자 함수를 사용할때는 부모 클래스의 생성자 함수를 반드시 호출합니다. => super(props);





### 5.4.2 render

- 렌더링 화면을 정의 합니다.
- 부모로 받은 Props 값의 변경이 되거나 , this.setState로 state 값이 변경될 때마다 호출됩니다.

이 함수에서 직접 this.setState를 호출하여 직접 State 값을 변경할 때는 무한루프에 빠질 위험이 있습니다. 이 예제에서는 touch event와 연결하였기 때문에 터치 이벤트가 발생할 때 this.setState가 호출되므로, 무한 루프에 빠지지 않습니다.

### 5.4.3 getDerivedStateFromProps

- 부모로받은 Props를  자신의 state에 동기화 할 때 사용합니다.
- 컴포넌트 생성될 때 1번 호출됩니다.
- 생성자 함수와 다르게 부모로 받은 Props가 바뀔 때마다 호출됩니다.

state에 설정하고 싶은 값을 이함수에서 반환하게 합니다. 동기화할 state가 없으면 "null"을 반환합니다.

 

```tsx
static getDerivedStateFromProps(nextProps: Props, prevState: State) {
  
  console.log('shouldComponentUpdate');  
  // 이전 state(부모 클래스의 props로부터 할당받은 값)이 현재 새로 받은 props와 다를 때 다시 설정합니다.
  if(nextProps.id !== prevState.id)
    return {value : nextProps.value}
  return null;
  
  }
```



###  5.4.4 componentDidMount

- 처음으로 화면에 클래스 컴포넌트가 표시된 이후, 이 함수가 호출됩니다. (didMount는 마운트 뒤에 라는 뜻이 내포되어있습니다.)
- 부모로받은 Props가 변경되어도, this.setState로 State가 변경되어도 다시 호출되지 않습니다.
- 따라서 render함수와 다르게 이 함수에 this.setState를 직접 호출할 수 있으며, ajax를 통해 받은 데이터를 this.state를 사용하여 state에 할당하기 적합한 함수입니다.

다른 자바스크립트 라이브러리와의 연동을 수행하기에 적합합니다.

### 5.4.5 shouldComponentUpdate

클래스 컴포넌트는 기본적으로 부모로받은 props가 변경되거나, this.setState로 state가 변경되면 리렌더링하게 됩니다. 하지만 Props 또는 State가 변경되어도 리렌더링을 방지하고 싶은 경우 이 함수를 사용하여 렌더링을 제어합니다.

렌더링 : return true

리렌더링 : return false



```tsx
shouldComponentUpdate(nextProps: Props, nextState: State) {
  
	return nextProps.id !== this.props.id;	
}
```

리렌더링을 방지하는 이유는 렌더링을 최적화 하기 위해서입니다. 하나 바뀌었다고 화면 다 바꿀필요가 없기 때문입니다. 리렌더링을 방지하고 바뀐부분만 바꿉니다.

(리렌더링은 자원 소모가 심합니다.)



질문 : 리렌더링과 컴포넌트 생성은 다른 말 인가?



### 5.4.6 getSnapshotBeforeUpdate

- Props 또는 State가 변경되어 화면을 다시 그리기 위해 render 함수가 호출된 후, 실제로 화면이 갱신되기 바로 직전에, 이 함수가 호출됩니다.

- 이 함수가 반환하는 값은 snapshot입니다.
- snapshot은 componentDidUpdate에 세번째 파라미터로 쓰입니다.
- 이 함수를 선언하고 만약 반환값을 반환하지 않는 경우 => warning
- 이 함수 선언하고 componentDidUpdate를 선언하지 않는 경우 => warning

### 5.4.7 componentDidUpdate

- componentDidUpdate는 처음 화면에 표시가 된 후 실행되고 두번 다시 호출되지 않는 함수입니다. 반면에 이 함수는 첫 렌더링에는 실행되지 않지만 props나 state 변경으로 해당 화면의 부분이 갱신 될 때 render 함수 호출 이후 호출되는 함수입니다.
- 잘 안 사용하지만 getSnapshotBeforeUpdate 함수와 함께 스크롤 수동 고정을 위해 사용하기도 합니다.
- render함수와 마찬가지로 이 함수도 직접 this.setState를 호출하여 직접 State 값을 변경할 때는 무한루프에 빠질 위험이 있습니다.

### 5.4.8 componentWillUnmount

- 해당 컴포넌트가 화면에서 완전히 사라진 후 호출되는 함수입니다.
- 이 함수에서는 componentDidMount에 연동된 js 라이브러리를 해제, 타이머를 clearTimeout, clearInterval를 사용하여 해제 합니다.
- 이 함수에서 state값을 변경하기 위한 this.setStat를 호출하면 메모리 누수 등 생긴다. 당연히 쓰면 안됩니다.

### 5.4.9 componentDidCatch 

- 컴포넌트의 렌더링 도중 에러가 발생하면 앱이 비정상적으로 종료합니다.
- try - catch 같은 구문
- 

### 5.4.10 호출 순서

- 컴포넌트가 생성 될 때

Constructor -> getDerivedStateFromProps -> render -> componentDidMount

- 컴포넌트의 Props가 변경될 때

getDerivedStateFromProps -> shouldComponentUpdate -> render -> getSnapshotBeforeUpdate -> componentDidUpdate

- 컴포넌트의 State가 변경될 때

 shouldComponentUpdate -> render -> getSnapshotBeforeUpdate -> componentDidUpdate

- 컴포넌트의 렌더링 중 에러가 발생될 때

componentDidCatch

- 컴포넌트가 언마운트(화면에서 종료 될때)

componentWillUnmount



## 5.5 요약

클래스 컴포넌트는 잊어버리세요. 함수형 컴포넌트를 집중해서 공부하세요. 예제나 라이브러리가 클래스로 구현되어 있을 때 당황하지 말고 다시 보길 바랍니다.