---
title: "[React] useState와 useRef 이해하기"
writer: chanho
date: 2025-04-03 10:00:00 +0900
categories: [Frontend, Library]
tags: [frontend, react]
---

# React의 핵심 Hooks, useState와 useRef 이해하기
> React의 `useState`와 `useRef`는 각자의 용도와 특성이 명확히 구분됩니다. 상태 관리가 필요하고 리렌더링이 필요한 경우 `useState`, 리렌더링 없이 값을 관리하거나 DOM 요소를 직접 접근할 때는 `useRef`를 사용하면 됩니다. 이러한 차이를 명확히 이해하기 위해 해당 글을 작성하고자 합니다.

## 1. useState란 무엇인가?
> React에서 가장 많이 쓰이는 Hook 중 하나인 `useState`는 컴포넌트가 상태(state)를 가지도록 도와줍니다. 상태가 변경될 때마다 React가 자동으로 컴포넌트를 다시 렌더링하게 되어 사용자에게 변경된 내용을 즉시 반영할 수 있습니다.

### useState의 사용 방법

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>현재 카운트: {count}</p>
      <button onClick={() => setCount(count + 1)}>증가</button>
    </div>
  );
}
```

: 이 예시에서 `count`가 상태 값이고, `setCount`는 이 값을 변경하는 함수입니다. 버튼을 클릭할 때마다 `setCount`를 통해 상태가 변경되며, React는 이 변화를 감지하여 화면을 리렌더링합니다.

### 흔히 발생하는 실수

useState를 사용할 때 주의해야 하는 점은 상태 값을 직접 변경하면 안 된다는 것입니다.

```jsx
count = count + 1; // 잘못된 사용 (반영되지 못함)
setCount(count + 1); // 올바른 사용 (반영)
```

---

## 2. useRef란 무엇인가?
> `useRef`는 React 컴포넌트에서 DOM 요소를 직접 참조하거나 리렌더링 없이 값을 저장할 때 사용됩니다. 상태 값과 달리, ref에 저장된 값이 변경되어도 컴포넌트는 리렌더링되지 않습니다.

### useRef의 사용 사례

#### (1) DOM 요소 직접 접근

DOM 요소를 바로 접근하여 포커스나 스크롤과 같은 기능을 다룰 때 유용합니다.

```jsx
import { useRef, useEffect } from 'react';

function InputFocus() {
  const inputRef = useRef(null);

  useEffect(() => {
    inputRef.current.focus();
  }, []);

  return <input ref={inputRef} type="text" placeholder="자동 포커스" />;
}
```

#### (2) 값을 저장 (리렌더링 없이)

타이머 ID나 클릭 횟수처럼, 리렌더링 없이 내부적으로만 값을 유지해야 할 때 사용합니다.

```jsx
function Timer() {
  const countRef = useRef(0);

  const handleClick = () => {
    countRef.current += 1;
    console.log('클릭 횟수:', countRef.current);
  };

  return <button onClick={handleClick}>클릭</button>;
}
```

버튼의 숫자는 바뀌지 않습니다. ref 값 변경이 화면에 즉시 반영되려면 별도의 state를 통해 리렌더링을 유발해야 합니다.

### useRef를 활용한 실무 팁

- **DOM 접근:** 직접적인 요소 접근이 필요할 때 사용합니다.
- **비동기 값 유지:** 타이머와 인터벌 ID 관리 시 적합합니다.
- **이전 값 기억:** 이전 상태나 props 값을 기억하고 싶을 때 사용할 수 있습니다.
- **클로저 문제 해결:** 이벤트 핸들러 내에서 최신 값을 유지하는 데에도 유용합니다.

---

## 3. usePrevious - 이전 상태를 기억하는 방법

React에서 상태의 이전 값을 기억하고 싶은 경우가 많습니다. 이때 사용할 수 있는 커스텀 훅이 바로 `usePrevious`입니다.

```jsx
import { useRef, useEffect } from 'react';

function usePrevious(value) {
  const ref = useRef();
  useEffect(() => {
    ref.current = value;
  }, [value]);
  return ref.current;
}

// 사용 예시
function MyComponent({ count }) {
  const previousCount = usePrevious(count);

  return (
    <div>
      <p>현재 카운트: {count}</p>
      <p>이전 카운트: {previousCount}</p>
    </div>
  );
}
```

### 💡 작동 원리
> usePrevious는 리렌더링 사이의 값을 기억하기 위해 useRef를 활용합니다. React는 상태가 변경되면 렌더링을 수행하고, 이후 useEffect가 실행되어 해당 값이 ref.current에 저장됩니다. 다음 렌더링 시점에서 이전 값을 참조할 수 있게 되는 구조입니다.

#### 작동 흐름 요약

1. 초기 렌더링
    - count는 0입니다.
    - ref.current는 아직 값이 없습니다 (undefined).
2. setState 실행 → count: 1로 변경
    - 렌더링 발생: count는 1로 화면에 표시됩니다.
    - 이 시점의 previousCount는 아직 이전 값인 0이 반영됩니다.
    - 이후 useEffect가 실행되어 ref.current가 1로 갱신됩니다.
3. 다시 setState 실행 → count: 2로 변경
    - 렌더링 발생: count는 2로 표시됩니다.
    - previousCount는 직전에 저장된 ref.current 값인 1이 반영됩니다.
    - 이후 useEffect가 실행되어 ref.current가 2로 갱신됩니다.

#### 주의 사항

usePrevious는 내부적으로 ref를 사용하므로, 해당 값이 변경되어도 리렌더링을 발생시키지 않습니다.

따라서 렌더링이 오직 해당 상태의 변경에 의해서만 일어난다는 보장이 없을 경우, 예기치 않게 이전 값과 현재 값이 동일하게 나타날 수 있습니다.

예를 들어, 다른 상태나 외부 요인으로 인해 렌더링이 발생하면 다음과 같은 상황이 생길 수 있습니다:

현재 카운트: 1이전 카운트: 1

이처럼 usePrevious는 정확히 특정 상태 변경에만 반응하는 것이 아니기 때문에, 필요한 경우에는 보조적인 조건이나 분기 처리를 통해 더 명확하게 제어해주는 것이 좋습니다.
