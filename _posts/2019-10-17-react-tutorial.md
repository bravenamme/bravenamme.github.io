---
layout: post
title:  "React Tutorial"
date:   2019-10-17 10:00
author: chyusee
tags:	[]
---

# React 시작하기


## 환경설정
### 01. Node.js 설치
- [https://nodejs.org/ko/download/](https://nodejs.org/ko/download/) 에서 설치파일 다운로드 후 설치
- `node --version`, `npm --version` 명령어를 실행하여 설치 여부를 확인

### 02. yarn 설치
- [https://yarnpkg.com/en/docs/install#windows-stable](https://yarnpkg.com/en/docs/install#windows-stable) 에서 설치파일 다운로드 후 설치
- 또는 npm을 이용해서 설치 `npm install --global yarn`
- `yarn --version` 명령어를 실행하여 설치 여부를 확인

### 03. create-react-app 설치 
- yarn 이용해서 설치 `yarn global add create-react-app`
- 또는 npm을 이용해서 설치 `npm install -g create-react-app`

## 어플리케이션 생성
- `create-react-app [어플리케이션 명]`

yarn을 이용해서 어플리케이션 생성을 완료하면 아래와 같이 출력됩니다.
![](/files/posts/201910/1017_001.jpg)

- yarn start (npm start) : 어플리케이션 실행
- yarn build (npm run build) : 어플리케이션 배포
- yarn eject (npm run eject) : 어플리케이션 설정

어플리케이션이 존재하는 경로로 이동 후 yarn start를 실행합니다.<br>
실행 완료 후 [http://localhost:3000/](http://localhost:3000/)로 접근하면 아래와 같은 샘플페이지가 보이게 됩니다.

![](/files/posts/201910/1017_002.jpg)


## 프로젝트 구조
<img src="/files/posts/201910/1017_003.jpg"><br>
- public 폴더 : static 자원 및 가상 DOM을 보여줄 index.html이 존재하는 폴더
- src 폴더 : 실제 구현할 소스들이 위치한 폴더

### public/index.html<br>
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="theme-color" content="#000000" />
    <meta
      name="description"
      content="Web site created using create-react-app"
    />
    <link rel="apple-touch-icon" href="logo192.png" />
    <link rel="manifest" href="%PUBLIC_URL%/manifest.json" />
    <title>React App</title>
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
  </body>
</html>
```

### src/index.js<br>
```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import * as serviceWorker from './serviceWorker';

ReactDOM.render(<App />, document.getElementById('root'));

// If you want your app to work offline and load faster, you can change
// unregister() to register() below. Note this comes with some pitfalls.
// Learn more about service workers: https://bit.ly/CRA-PWA
serviceWorker.unregister();
```

### src/App.js<br>
```jsx
import React from 'react';
import logo from './logo.svg';
import './App.css';

function App() {
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.js</code> and save to reload.
        </p>
        <a className="App-link"
          href="https://reactjs.org"
          target="_blank"
          rel="noopener noreferrer">
          Learn React
        </a>
      </header>
    </div>
  );
}

export default App;
```

<u>app.js</u>에서 구현한 app 클래스를<br>
<u>index.js</u>에서 ReactDOM.render(<App />, document.getElementById('root'))를 통해<br>
<u>index.html</u>의 id가 root인 div 내부에 랜더링 


## 컴포넌트 생성
src에 Hello.js, Hello.css 를 추가 합니다.
### src/Hello.js<br>
```jsx
import React from 'react';
import './Hello.css';

function Hello() {
    const name = 'World';
    return (
    <div className="Hello">
      Hello {name}!
    </div>
  );
}

export default Hello;
```

### src/Hello.css<br>
```css
.Hello {
  font-size: 20px;
  font-weight: bold;
}
```

App.js를 변경
### src/App.js<br>
```jsx
import React from 'react';
import './App.css';
import Hello from './Hello';

function App() {
  return (
    <div className="App">
      <Hello/>
    </div>
  );
}

export default App;
```

Hello World 라고 출력되는 것을 확인 할 수 있습니다.

## JSX(Javascript XML) 
- 가독성 및 생산성이 향상
- 컴파일이 빠르고, 변환과정에서 오류를 발생하므로 문법 오류를 줄일 수 있음

### JSX
```jsx
<div>
  <Hello/>
  <br/>
  <a href="www.google.com">Goole</a>
</div>
```
### JSX 미사용
```jsx
React.createElement(
  "div",
  null,
  React.createElement(Hello, null),
  React.createElement("br", null),
  React.createElement("a", {"href":"www.google.com"}, "Goole")
)
```

### Hello.js
```jsx
import React from 'react';
import './Hello.css';

function Hello() {
  function goWorld(welcome) {
    return welcome.first + ' ' + welcome.last;
  };

  const welcome = {
    first: 'React',
    last: 'World'
  };

  return (
    <h1>Hello, {goWorld(welcome)}!</h1>
  );
};

export default Hello;
```

### ButtonEvent.js
```jsx
import React from 'react';

function ButtonEvent() {
  function handleClick() {
    alert("Hello Alert");
  };

  return (
    <button onClick={handleClick}>Click me</button>
  );
};

export default ButtonEvent;
```

### Timer.js
```jsx
import React from 'react';

function Timer() {
  return (
    <h2>Now is {new Date().toLocaleTimeString()}</h2>
  );
};

export default Timer;
```

### App.js
```jsx
import React from 'react';
import './App.css';
import Hello from "./Hello";
import Timer from "./Timer";
import Button from "./ButtonEvent";


function App() {
  return (
    <div className="App">
      <Hello/>
      <Timer/>
      <Button/>
    </div>
  );
}

export default App;
```
![실행 결과](/files/posts/201910/1017_002.jpg)

- 자바스트립트 표현식을 중괄호로 표현 가능<br>
- 요소의 이벤트 처리도 가능

지금까지 간략하게 React에 대해서 알아보았습니다.
다음에는 예제를 구축해 가면서 좀 더 심호 단계를 학습 하고자 합니다.
