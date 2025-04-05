---
title: "[React] useEffect 와 useLayoutEffect 이해하기"
writer: chanho
date: 2025-04-05 22:30:00 +0900
categories: [Frontend, Library]
tags: [frontend, react]
---

> 리액트를 사용하면서 가장 많이 사용하면서도 라이프사이클 관련해서 내가 원하는 동작이 안나오는 경우가 간혹 있었던 것 같다. 이에 다시 한번 해당 훅을 살펴보면서 useEffect와 useLayoutEffect를 정리하고자 한다.

## 1. React 컴포넌트의 생명주기와 useEffect
> 기존 클래스형 컴포넌트에서는 생명주기를 제어하기 위해 `componentDidMount`, `componentDidUpdate`, `componentWillUnmount` 같은 메서드를 사용했었던 것으로 알고 있다. 현재는 함수형 컴포넌트가 등장하면서 이러한 메서드들은 `useEffect`로 통합되어 주로 사용되고 있다.

| 클래스형 컴포넌트 메서드 | useEffect로 구현 시점 |
|---------------------|-----------------------|
| componentDidMount   | 최초 렌더링 후, 의존성 배열이 빈 배열(`[]`)일 때 실행 |
| componentDidUpdate  | 의존성 배열에 특정 값이 포함되고 그 값이 변경될 때 실행 |
| componentWillUnmount | useEffect의 반환 함수(clean-up 함수)로 구현 |

### 클래스형 컴포넌트 사용 예시

```jsx
import React, { Component } from 'react';

class MyComponent extends Component {
  componentDidMount() {
    console.log('컴포넌트 마운트됨');
  }

  componentDidUpdate(prevProps, prevState) {
    if (this.props.value !== prevProps.value) {
      console.log('props가 변경됨');
    }
  }

  componentWillUnmount() {
    console.log('컴포넌트 언마운트됨');
  }

  render() {
    return <div>{this.props.value}</div>;
  }
}
```

### 실무 활용 예시
- componentDidMount: 초기 데이터 가져오기, 이벤트 리스너 등록 등
- componentDidUpdate: props 또는 state 변화 감지 후 데이터 재호출
- componentWillUnmount: 리소스 정리(이벤트 리스너 제거, 타이머 해제 등)

## 2. useEffect 개요
> `useEffect`는 React의 함수형 컴포넌트에서 부수 효과(side-effect)를 처리하는 데 사용되는 훅이다. 이 훅은 컴포넌트가 렌더링된 후에 실행되며, API 호출, 타이머 설정, 이벤트 등록 등 렌더링 이후의 다양한 작업을 수행할 수 있다.

### 사용법

```jsx
useEffect(() => {
  // side effect 처리
  return () => {
    // clean-up 코드
  };
}, [의존성 배열]);
```

### 생명주기

- 최초 렌더링 후 실행
- 의존성 배열의 값이 변경될 때마다 다시 실행
- 컴포넌트가 언마운트될 때 clean-up 함수 실행

### 예시 (이벤트 등록)

```jsx
useEffect(() => {
  const handleResize = () => {
    console.log(window.innerWidth);
  };
  window.addEventListener('resize', handleResize);

  return () => window.removeEventListener('resize', handleResize);
}, []);
```

## 3. useLayoutEffect란?
> `useLayoutEffect`는 `useEffect`와 비슷하지만, 실행 시점에서 차이가 있다. `useLayoutEffect`는 브라우저가 실제로 화면을 그리기 전에 실행되며, 주로 DOM 측정 또는 즉각적인 레이아웃 수정이 필요한 경우 사용된다.

### useEffect vs useLayoutEffect

| 특징 | useEffect | useLayoutEffect |
|------|-----------|-----------------|
| 실행시점 | 렌더링 후 (페인팅 후) | 렌더링 직후 (페인팅 전) |
| 렌더링 차단 여부 | 차단하지 않음 | 렌더링을 차단함 |
| 사용 예 | API 호출, 이벤트 처리 등 | DOM 측정, 즉각적인 UI 조정 |


## 4. useEffect를 활용한 Custom Hook 패턴
> 커스텀 훅은 특정 기능을 재사용 가능한 형태로 만드는 패턴이다. 일반적으로 state, ref, effect를 결합하여 구성한다.

### 예제: useWindowSize (창 크기 측정)

```jsx
import { useEffect, useState } from 'react';

function useWindowSize() {
  const [size, setSize] = useState({ width: 0, height: 0 });

  useEffect(() => {
    const updateSize = () => {
      setSize({ width: window.innerWidth, height: window.innerHeight });
    };

    window.addEventListener('resize', updateSize);
    updateSize();

    return () => window.removeEventListener('resize', updateSize);
  }, []);

  return size;
}
```

------

현재는 주로 컴포넌트 생명주기 메서드 보다는 함수형 컴포넌트의 useEffect로 대체하여 사용하고 있으며, 다른 Hook들 처럼 커스텀 훅으로 활용하기에도 용이하다. 다만, SSR 환경에서의 useLayoutEffect의 사용은 주의가 필요하다. 이유는 서버에는 DOM이 없기 때문에 Hydration mismatch 문제가 발생할 수 있기 떄문이다. 물론 자주 사용하는 NextJS 같은 프레임워크의 경우 Hook을 `'use client'` 환경에서 사용해야 하기 때문에 이런 걱정은 덜 할 것으로 보이기는 하지만, 이유정도는 함께 알고 있어도 좋을 것으로 보인다.
