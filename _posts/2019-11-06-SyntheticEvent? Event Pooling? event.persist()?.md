---
published: true
layout: single
title: "SyntheticEvent? Event Pooling? event.persist()?"
comments: true
---

React의 Event Polling과 Javascript event 객체의 persist()에 대해 알아보자

## SyntheticEvent & Event Pooling

최근 작업하던 천명 프로젝트를 진행하면서, event 객체 관련하여 새롭게 알게된 사실이 있어 정리한다.

### ISSUE

React 내부에서 Input들의 값을 받아 State로 설정하는 작업이 필요했다. <br>
평소에 그냥 일반적으로 useState를 이용해서 하는 편인데, 약간 복잡해질 가능성도 있고 useReducer와 친해지고자 이번에는 useReducer를 사용해 보았다.

```javascript
import React, { useReducer, ChangeEvent } from "react";

interface IState {}
type Action = { type: "SET_STATE", event: ChangeEvent<HTMLInputElement> };

const reducer = (state, action) => {
  switch (action.type) {
    case "SET_STATE":
      const { name, value } = action.event.target;
      return { ...state, [name]: value };
  }
};

const Body: React.FC = (): JSX.Element => {
  const [state, dispatch] = useReducer(reducer, initialInfo);

  const setInfo = (event: ChangeEvent<HTMLInputElement>): void => {
    dispatch({ type: "SET_INFO", event });
  };

  return (
    <div>
      <input type="text" name="name" onChange={setInfo} />
    </div>
  );
};
```

위와 같이 평소 useReducer를 사용하던 방식과, useState를 사용하던 방식을 합쳐서 작성했고, 아무런 문제가 없어 보였다.

`하지만...`
![Error](https://raw.githubusercontent.com/gyeol1212/gyeol1212.github.io/master/_posts/assets/1106error.png)

두 번 이상, setInfo 함수가 실행되는 순간, <br>
다음과 같은 Error 메시지가 출력되며

> Uncaught TypeError: Cannot read property 'name' of null

에러가 발생된다.

---

### Why?

먼저 해당 에러 발생 원인을 찾아보았다. <br>

해당 에러는 event 속성을 직접적으로 비동기 함수에 적용하였을 때, 발생한다고 한다.

```javascript
const setInfo = (event: ChangeEvent<HTMLInputElement>): void => {
  // Problem : event 객체 전체를 직접 비동기 함수에 적용!
  dispatch({ type: "SET_INFO", event });
};
```

다음은 React 공식 문서의 합성 이벤트(SyntheticEvent) 관련 부분이다.

![eventPooling](https://raw.githubusercontent.com/gyeol1212/gyeol1212.github.io/master/_posts/assets/eventPooling.png)

이러한 현상을, _이벤트 풀링(Event Pooling)_ 이라 한다.

---

### Solve?

가장 간단한 해결 방법은 필요한 정보를 비동기 할당 전에 선언한 뒤, 전달하는 것이다.

```javascript
const setInfo = (event: ChangeEvent<HTMLInputElement>): void => {
  // dispatch에 넘겨주기 전, name, value 할당
  const { name, value } = event.target;
  dispatch({ type: "SET_INFO", name, value });
};
```

위와 같이 dispatch 함수에, event객체 대신 필요한 변수들을 할당하여 전달해주고,

```javascript
type Action = { type: "SET_INFO", name: string, value: string };

const reducer = (state: ITeller, action: Action): ITeller => {
  switch (action.type) {
    case "SET_INFO":
      return { ...state, [action.name]: action.value };
  }
};
```

reducer 또한 이렇게 바꿔주면, 문제없이 작동한다.

---

### event.persist()

하지만, reducer 내부에서 event 객체의 다른 property에 접근할 필요가 있어진다면 매번 추가로 넘겨주고, 수정해야하는 불편함이 있어 아쉬웠다.

그래서 추가적으로 구글링을 하던 중, _event.persist()_ 라는 것을 알게 되었다.

```javascript
const setInfo = (event: ChangeEvent<HTMLInputElement>): void => {
  // event 객체의 값을 그대로 유지!
  event.persist();
  dispatch({ type: "SET_INFO", event });
};
```

위와같이 dispactch에 event 객체를 넘겨주기 전에, event.persist() 메서드를 실행시키면, event 객체의 값을 그대로 유지하게 된다.

하지만 해당 이벤트 객체가 메모리에 지속적으로 남아 *성능 문제*가 발생할 수 있다. 따라서 이벤트 객체가 꼭 변수로 필요한 경우에만 사용하는 것이 좋을 듯 하다.

### 결론

1. 비동기 함수에서 event를 다루는 것은 주의가 필요하다.
2. 비동기 함수에 넘겨 주기 전, 필요한 것들은 변수에 할당하여 넘겨주자.
3. event.persist()라는 좋아보이는 해결책이 존재하지만, 성능 이슈가 발생할 수 있으니, 꼭 필요한 경우에만 사용하자!
