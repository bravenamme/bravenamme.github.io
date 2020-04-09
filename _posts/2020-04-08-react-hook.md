---
layout: post
title:  "리액트 훅(react hook)이란?"
date:   2020-04-08 10:00
author: kapjong
tags:	[react]
---

# 등장 배경
리액트 컴포넌트는 클래스형 컴포넌트(Class Component)와 함수형 컴포넌트(Functional Component)로 나누어 짐니다.  
리엑트 네이티브 개발을 경험 해보니 개발 방식은 기본적으로 함수형 컴포넌트를 주로 하여 코딩이 진행 됨니다.

> 개발 시 화면 상태 변화, App Life Cycle 제어 등에 처리리는 클래스 컨포넌트 사용이 필수로 요구 되었으나.    
> react hooks 처리 등장으로 함수형 컴포넌트에서 클래스 컴포넌트의 작업을 할수 있게 되었습니다.

# 클래스형 컴포넌트 단점
### 1. Logic의 재사용이 어렵습니다.
클래스형 컴포넌트에서는 컴포넌트 자체를 재사용 할 수는 있지만  
Component class 부분적인 API 사용 및 state를 다루는 등의 처리에는 재사용에 제약이 따릅니다.  

### 2. 코드가 길고 복잡해 진다.
constructor, this, binding 등 기본 규칙을 따르지 않는 다면
![](/files/posts/202004/warning.jpg)
경고 메세지가 표현 되기도 하고, 컴파일이 진행 되지 않을 수도 있습니다.  
기본 규칙을 반드시 따라야 하기 때문에 코드가 복잡하고 길어짐니다.  

# 함수형 컴포넌트 Hook 처리
### 1. Hooks 처리시 장점
> * 함수형 컴포넌트에서는 원하는 기능을 함수로 만든 후 필요한 곳에 넣어주기만 하면 되기 때문에 로직의 재사용이 유연해 집니다.
> * 클래스형 컴포넌트가 가지고 있던 복잡성, 재사용성의 단점들까지 해결 됨니다.

### 2. Hook API
기본 Hook
 * useState 
 * useEffect
 * useContext

추가 Hooks
 * useReducer
 * useCallback
 * useMemo
 * useRef
 * useImperativeHandle
 * useLayoutEffect
 * useDebugValue
 
### 3. State Hook 사용 하기 (useState)
화면에 값이 있고 그 값을 갱신하는 함수형 컨포넌트 코드
```react
import React, { useState } from 'react';

function Example() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

# 결론
* Hook API 정확한 이해도가 있어야 React Native 개발 구조를 단순 하고 진행 할수 있습니다.
* 프로젝트 진행 시 기본 Hook, 추가 Hooks 요소를 충분히 이해 하고 설계를 진행 하자!

---
## 참고
 * [리엑트 네이티브 공식](https://reactnative.dev/)
 * [리엑트 한글 번역 페이지](https://ko.reactjs.org/) 