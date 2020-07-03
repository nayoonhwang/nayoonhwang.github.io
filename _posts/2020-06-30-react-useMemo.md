---
layout: post
title:  "React Hooks"
date:   2020-06-30 14:02:22 +0900
categories: React
---

# React

React는 User Interface를 만드는 라이브러리이다. 개발자에게 Strict한 Data flow를 강제한다는 건 리액트의 큰 강점이다. 
Container와 Presentational Component를 통한 명확한 구조, Component들이 state와 prop의 변화에 대응하는 Strict한 Data 흐름을 강제함으로써 좀 더 논리적인 UI logic 을 만들기 쉽다.

React의 기본적인 개념은 UI의 작은 부분들은 state의 변화에 "react" 할 수 있다는 것이다. 기존에는 class를 통해서 이런 흐름을 만들어 냈었다.

~~~javascript
import React, { Component } from "react";

export default class Button extends Component {
  constructor() {
    super();
    this.state = { buttonText: "Click me, please" };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState(() => {
      return { buttonText: "Thanks, been clicked!" };
    });
  }

  render() {
    const { buttonText } = this.state;
    return <button onClick={this.handleClick}>{buttonText}</button>;
  }
}
~~~

예를 들어, User가 버튼을 누르면 => this.setState를 통해서 component의 내부적인 상태가 변경된다. 버튼 내부의 텍스트는 이런 변화에 "react"하고 업데이트 된 텍스트를 받는다.

하지만, React Hook을 이용해 함수형 컴포넌트에서도 위와 같이 상태관리를 할 수 있다. **훨씬 쉽고 간결하게.**

## useState

useState 는 React 자체적으로 제공하는 함수이다. 
~~~javascript
import React, { useState } from "react";
~~~

일단 React 를 쓰면 두개의 값으로 destructure 할 수 있다. 그 '상태'의 실제 값, 상태를 변경하는 함수(state updater) 두 가지를 반환한다.
~~~javascript
const [buttonText, setButtonText] = useState("Click me, please")
~~~

useState로 전달되는 argument는 실제 초기값이고, 그 데이터가 변경될 값이다.

useState를 이용하면 아까 그 Class Component는 이런식으로 바꿔볼 수 있다.
~~~javascript
import React, { useState } from "react";

export default function Button() {
  const [buttonText, setButtonText] = useState("Click me, please");

  return (
    <button onClick={() => setButtonText("Thanks, been clicked!")}>
      {buttonText}
    </button>
  );
}
~~~

## useEffect

useEffect는 기존의 { componentDidMount, componentDidUpdate, componentWillUnmount } 를 한번에 해결할 수 있는 API이다. 

~~~javascript
import React, { useState, useEffect } from "react";

export default function DataLoader() {
  const [data, setData] = useState([]);

  useEffect(() => {
    fetch("http://localhost:3001/links/")
      .then(response => response.json())
      .then(data => setData(data));
  });

  return (
    <div>
      <ul>
        {data.map(el => (
          <li key={el.id}>{el.title}</li>
        ))}
      </ul>
    </div>
  );
}
~~~

DataLoader라는 함수형 컴포넌트를 만들어봤을때 서버에서 response를 받아서 setData를 통해서 data를 받아와서, <li>로 각 데이터를 뿌려주는 로직이다.
	
이때 주의해야할 점은, 이렇게만 useEffect를 쓰게 되면, 컴포넌트가 새로운 prop을 받는다던가, state가 변경될때마다 useEffect 내의 함수가 호출된다는 것이다.

~~~javascript
useEffect(() => {
   fetch("http://localhost:3001/links/")
     .then(response => response.json())
     .then(data => setData(data));
}, []); // << super important array
~~~

이는 이런식으로 두번째 인자에 빈 어레이를 넘기는 방법으로 해결할 수 있는데, 두번째 인자는 dependency, 즉 useEffect가 다시 수행할 지 여부를 결정하는 의존 변수들이 들어간다. 만약 이 배열이 비어있으면, 컴포넌트가 뜰때 딱 한번만 수행될 것이다.

### useEffect 의 Effect cleanup

Timer, listener, 지속적인 Connection들(websocket이나 그런 친구들)은 Javascript memory leak의 가장 흔한 문제 원인들이다.
~~~javascript
useEffect(() => {
  const socket = socketIOClient(ENDPOINT);
  socket.on("FromAPI", data => {
    setResponse(data);
  });
}, []);
~~~
예를 들어 위와 같은 상황에서 component가 DOM 으로부터 unmount할때도 connection이 열린채로 유지된다는 것이 문제다.

~~~javascript
useEffect(() => {
  const socket = socketIOClient(ENDPOINT);
  socket.on("FromAPI", data => {
    setResponse(data);
  });
  
  return () => socket.disconnect();
  
}, []);
~~~
이런식으로 useEffect 내에서 effect를 cleanup 할 수 있는 function을 return 할 수 있다. 이제 connection 은 component가 unmount될때 close될것으로 기대할 수 있다.

## Custom Hook
자주 사용하는 로직들을 묶어서 custom hook을 만들어볼 수 있다. 다른 hook들 처럼 custom hook 은 보통 use로 시작하는 JS 함수이다. 

~~~javascript
import { useState, useEffect } from "react";

export default function useFetch(url) {
  const [data, setData] = useState([]);

  useEffect(() => {
    fetch(url)
      .then(response => response.json())
      .then(data => setData(data));
  }, []);

  return data;
}
~~~


~~~javascript
import React from "react";
import useFetch from "./useFetch";

export default function DataLoader(props) {
  const data = useFetch("http://localhost:3001/links/");
  return (
    <div>
      <ul>
        {data.map(el => (
          <li key={el.id}>{el.title}</li>
        ))}
      </ul>
    </div>
  );
}
~~~


### Async/Await 를 useEffect와 함께 쓰기

javascript async 함수는 항상 promise를 return 한다. 이때 useEffect는 cleanup 함수만 return할 수 있으므로, 아래와 같은 코드는 에러가 발생한다.
~~~javascript
useEffect(async () => {
  const response = await fetch(url);
  const data = await response.json();
  setData(data);
}, []);
~~~
즉, useEffect 내에서는 Promise를 Return할 수 없으므로, async, await 를 useEffect와 함께 쓰기 위해서는,

~~~javascript
import { useState, useEffect } from "react";

export default function useFetch(url) {
  const [data, setData] = useState([]);

  async function getData() {
    const response = await fetch(url);
    const data = await response.json();
    setData(data);
  }

  useEffect(() => {
    getData();
  }, []);

  return data;
}
~~~ 
이런식으로 async 함수를 별도로 만들어서 사용하여 주도록 한다.

## useReducer
useReducer는 또다른 훅인데, React component의 좀 더 복잡한 상태 변화를 다루는데 유용하다. useReducer는 Redux의 reducer, action, dispatch 컨셉들을 가져온 것이다. 

~~~javascript
export function useFetch(endpoint) {
  const [data, dispatch] = useReducer(apiReducer, initialState);

  useEffect(() => {
    dispatch({ type: "DATA_FETCH_START" });

    fetch(endpoint)
      .then(response => {
        if (!response.ok) throw Error(response.statusText);
        return response.json();
      })
      .then(json => {
        dispatch({ type: "DATA_FETCH_SUCCESS", payload: json });
      })
      .catch(error => {
        dispatch({ type: "DATA_FETCH_FAILURE", payload: error.message });
      });
  }, []);

  return data;
}
~~~

~~~javascript
const [data, dispatch] = useReducer(apiReducer, initialState);
~~~
useReducer를 이용하면, data와 action을 dispatching(:특정 목적을 가지고 보내다)할 함수를 얻는다. 이 함수를 호출하면서 action들을 Dispatch시킬 수 있다.

~~~javascript
useEffect(() => {
    // dispatch an action
    dispatch({ type: "DATA_FETCH_START" });

    fetch(endpoint)
      .then(response => {
        if (!response.ok) throw Error(response.statusText);
        return response.json();
      })
      .then(json => {
        // dispatch an action on success
        dispatch({ type: "DATA_FETCH_SUCCESS", payload: json });
      })
      .catch(error => {
        // dispatch an action on error
        dispatch({ type: "DATA_FETCH_FAILURE", payload: error.message });
      });
	  }, []);
~~~

이 Action들은 Reducer 함수에서 다음 상태를 연산하기 위해 사용된다.
일단 시작할때, loading중을 활성화시키는 DATA_FETCH_START를 dispatch 시키고, 성공시, 실패시 각각 필요한 Action을 dispatch시킨다

~~~javascript
mport { useEffect, useReducer } from "react";

const initialState = {
  loading: "",
  error: "",
  data: []
};

function apiReducer(state, action) {
  switch (action.type) {
    case "DATA_FETCH_START":
      return { ...state, loading: "yes" };
    case "DATA_FETCH_FAILURE":
      return { ...state, loading: "", error: action.payload };
    case "DATA_FETCH_SUCCESS":
      return { ...state, loading: "", data: action.payload };
    default:
      return state;
  }
}

export function useFetch(endpoint) {
  const [data, dispatch] = useReducer(apiReducer, initialState);

  useEffect(() => {
    dispatch({ type: "DATA_FETCH_START" });

    fetch(endpoint)
      .then(response => {
        if (!response.ok) throw Error(response.statusText);
        return response.json();
      })
      .then(json => {
        dispatch({ type: "DATA_FETCH_SUCCESS", payload: json });
      })
      .catch(error => {
        dispatch({ type: "DATA_FETCH_FAILURE", payload: error.message });
      });
  }, []);

  return data;
}
~~~

### Conclusion
결론: React hook은 여러모로 prop rendering이나 HOC를 대체할 수 있고, Stateful 한 로직을 공유하는 데 굉장히 유용하다

Reference
- [React Hooks](https://www.valentinog.com/blog/hooks/#your-first-custom-react-hook)
