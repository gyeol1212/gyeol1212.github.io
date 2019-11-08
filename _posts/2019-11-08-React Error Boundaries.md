---
published: true
layout: single
title: 'Error Boundaries'
comments: true
---

React의 Error Boundaries에 대해 알아보자

## Error Boundaries

최근 React 공식 문서를 읽다가, 새롭게 알게된 사실이 있어 프로젝트에 적용해 보았다.

### ISSUE

Typescript를 사용하는 경우 많이 줄어들기는 하지만, 컴포넌트 내부에서 Javascript 에러가 발생하여, 리액트 전체의 앱이 망가지는 경우를 많이 겪었을 것이다.

다음과 같은 코드를 보자.

```javascript
// App.tsx
import React from 'react';

const App: React.FC = () => {
  const data = {
    name: 'gyeol',
    age: 25,
  };

  return (
    <div>
      안녕하세요
      <TestComponent data={data} />
    </div>
  );
};
```

```javascript
// TestComponent.tsx
import React from 'react';

interface IProps {
  data: { name: string, age: number };
}

const TestComponent: React.FC<IProps> = ({ data }): JSX.Element => {
  return (
    <div>
      <p>{data.name}입니다</p>
      <p>{data.age}살 입니다</p>
      <p>{data} 랍니다</p>
    </div>
  );
};

export default TestComponent;
```

바로 위의 코드에서, data라는 object를 React child로 사용하였기에 다음과 같은 에러가 발생한다.

![Error](https://raw.githubusercontent.com/gyeol1212/gyeol1212.github.io/master/_posts/assets/1108error.png)

실 유저들이 웹사이트를 이용하다가 이러한 현상이 발생할 경우, 아무런 안내 없이 흰 화면만 마주치게 될 것이다.

이러한 상황을 막기 위해서 React는 새로운 컨셉인 'Error Boundary'를 제공한다.

---

### HOW?

우선 간단하게 사용법부터 보자.

```javascript
// ErrorBoundary.tsx
import React from 'react';

class ErrorBoundary extends React.Component {
  public state = { hasError: false };

  public static getDerivedStateFromError(error: Error) {
    // Update state so the next render will show the fallback UI.
    console.log(error);
    return { hasError: true };
  }

  public componentDidCatch(error: Error, errorInfo: any) {
    // You can also log the error to an error reporting service
    console.log(error, errorInfo);


  }

  public render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return <h1>무언가 잘못되었네요...!</h1>;
    }

    return this.props.children;
  }
}

export default ErrorBoundary;
```

```javascript
// App.tsx
// ...
return (
  <ErrorBoundary>
    <div>
      안녕하세요
      <TestComponent data={data} />
    </div>
  </ErrorBoundary>
);

// ...
```

ErrorBoundary.tsx라는 클래스 컴포넌트에서 getDerivedStateFromError와 coponentDidCatch(error, info)라는 새로운 Lifecycle method를 정의하여 'Error Boundary'를 사용하였다. <br>
아쉽게도 아직 hooks을 기반으로 한 함수형 컴포넌트에서는 사용할 수 없는 듯 하다.

그 후, 일반 컴포넌트와 동일하게 Error Boundary를 처리하고 싶은 컴포넌트들을 감싸서 사용한다. 대부분의 경우, 어플리케이션 전체의 경계를 감싸 유저들에게 에러 메시지를 보여줄 것이다.

![ErrorBoundary](https://raw.githubusercontent.com/gyeol1212/gyeol1212.github.io/master/_posts/assets/ErrorBoundary.png)

주의할 점은, *Error Boundary는 자신의 트리 아래에 있는 컴포넌트의 에러만 잡을 수 있다는 점*이다. 즉, 본인을 포함하는 에러는 잡아내지 못한다. Error Boundary에서 에러가 발생한 경우, 그 에러는 그 위의 가장 가까운 에러 경계로 전파된다.

이러한 특성을 이용하여, 한 어플리케이션 내부에서 여러개의 Error Boundary를 중첩하여 사용할 수 있다. 특정 기능들을 하나의 Boundary로 묶어, 기타 기능은 원활하게 돌아가도록 할 수도 있고, 최상위 라우트 컴포넌트를 감싸서 '무언가 잘못되었다' 라는 메시지를 유저들에게 보여줄 수 있다.

### 예외

Error Boundary는 다음과 같은 Error는 캐치하지 못한다

- 이벤트 핸들러
- 비동기 코드(ex. setTimeout, requestAnimationFrame)
- SSR

### 정리

1. React Application 전체를 감싸는 에러 처리가 필요하다면 Error Boundary 컴포넌트를 사용하자.
2. Error Boundary 컴포넌트 내부의 Lifecycle에 대해 시간이 나면 공부해보자.
