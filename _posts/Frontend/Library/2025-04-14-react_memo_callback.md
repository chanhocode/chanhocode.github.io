---
title: "React 성능 최적화: useMemo, React.memo, 그리고 useCallback"
writer: chanho
date: 2025-04-14 09:00:00 +0900
categories: [Frontend, Library]
tags: [frontend, react]
---

> 리액트로 Client를 작성하다보면 불필요한 렌더링과 무거운 연산이 성능 병목이 되는 경우가 있다. 이를 최적화하기 위해 주로 사용하는 방법에는 `useMemo`, `React.memo`, 그리고 `useCallback`을 사용하여 최적화 하는 것이다.

## useMemo

> 계산량이 많은 연산 결과를 메모이제이션하여, 입력값이 바뀔 때만 다시 계산하도록 만드는 리액트의 Hook이다. 예를 들어 정렬이나 필터링 혹은 재귀적으로 큰 연산을 수행해야 할때 ‘useMemo’를 이용해서 값에 대한 최적화를 할 수 있다.

```tsx
const MemoizedValue = useMemo(() => {
	return heavyCalculation(a, b)
}, [a, b])
```

- 무거운 연산을 리렌더링 때마다 다시 실행하지 않도록 결과를 캐싱함
- React 컴포넌트에서 계산 비용이 큰 값을 효율적으로 관리할 수 있게 한다.

### 기본 사용 예시

```tsx
function heavyComputation(num: number) {
  console.log('>> 무거운 연산 실행 <<');
  let result = 0;
  for (let i = 0; i < 1_000_000_000; i++) {
    result += i % num;
  }
  return result;
}

const UseMemoExample = () => {
  const [number, setNumber] = useState(5);
  const [text, setText] = useState('');

  const computedValue = useMemo(() => {
    return heavyComputation(number);
  }, [number]);

  return (
    <div>
      <h2>useMemo 최적화 예제</h2>
      <input
        type="number"
        value={number}
        onChange={(e) => setNumber(parseInt(e.target.value))}
      />
      <input
        type="text"
        value={text}
        onChange={(e) => setText(e.target.value)}
        placeholder="다른 입력창"
      />
      <p>계산 결과: {computedValue}</p>
    </div>
  );
}
```

위 코드는 number가 변경될 때만 연산을 재실행하여, 불필요한 CPU 소비를 줄인다.

` 💡 React함수 컴포넌트는 상태(State)가 변경될 때마다 전체 함수 컴포넌트를 다시 호출한다.`

1. text가 변경됨 → setText() 호출
2. 상태 업데이트 → React가 컴포넌트를 리렌더링
3. <UseMemoExample /> 함수가 **다시 실행됨**
4. heavyComputation(number) 도 **함수 호출 중에 다시 실행됨**

즉, useMemo를 사용하지 않으면 상태가 바뀔때 마다 heavyCompuation을 실행하게 되어 비효율적 이지만, useMemo를 사용하면 캐시된 값을 재사용하고 다시 실행되지 않아 효율적이다.

## **useCallback**

> 동일한 함수를 계속 재사용하고 싶을 때 함수(콜백)을 메모이제이션하여, 함수 객체의 재생성을 방지한다.

```tsx
function Parent() {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    alert('Clicked!');
  }, []);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount((c) => c + 1)}>+1</button>
      <Child onClick={handleClick} />
    </div>
  );
}

const Child = React.memo(({ onClick }) => {
  console.log('Child 렌더링');
  return <button onClick={onClick}>자식 버튼</button>;
});
```

`count` 값이 변경되면 `Parent` 컴포넌트는 리렌더링된다. 하지만 `useCallback`을 사용하면 `handleClick` 함수는 동일한 참조를 유지하게 되어, 매번 새로 생성되지 않는다. 여기서 `Child` 컴포넌트는 `React.memo`로 감싸져 있기 때문에, 전달받은 `props`가 변경되지 않으면 리렌더링되지 않는다. 즉, `onClick` prop이 이전과 동일한 함수 객체이기 때문에 `Child`는 리렌더링되지 않는 것이다. 반대로, `useCallback`을 사용하지 않고 `handleClick` 함수를 일반 함수로 선언하면, `Parent`가 리렌더링될 때마다 새로운 함수 객체가 생성되므로 `onClick` prop도 매번 변경된 것으로 간주된다. 그 결과, `React.memo`로 감싸더라도 `Child`는 매번 리렌더링되게 된다.

### **useCallback 없이 작성했을 때**

```tsx
// 최적화 안 된 예제
function Parent() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount((c) => c + 1)}>+1</button>
      <Child onClick={() => alert('clicked')} />
    </div>
  );
}

const Child = React.memo(({ onClick }) => {
  console.log('Child 렌더링');
  return <button onClick={onClick}>자식 버튼</button>;
});
```

`onClick`이 매번 새로 생성되는 익명 함수이기 때문에 `React.memo`가 무력화되어 `Child`는 매번 렌더링되고 콘솔 로그도 계속 출력된다.

### **useCallback 적용 후**

```tsx
function Parent() {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    alert('clicked');
  }, []);

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount((c) => c + 1)}>+1</button>
      <Child onClick={handleClick} />
    </div>
  );
}

const Child = React.memo(({ onClick }) => {
  console.log('Child 렌더링');
  return <button onClick={onClick}>자식 버튼</button>;
});
```

useCallback을 적용하면 handleClick은 의존성이 없으므로, 최초 한 번만 생성된다. Child는 props가 동일하다고 판단하여 재렌더링되지 않으며 콘솔 로그가 한 번만 찍혀 **확실한 성능 개선**을 체감할 수 있다.

## **React.memo**

> 위, ‘useCallback’을 설명하면서 ‘React.memo’가 나와 이를 부연 설명하고자 한다. useMemo와 React.memo가 이름 때문에 비슷하게 느껴지지만, 적용 대상과 동작 방식이 완전히 다르다. ‘React.memo’는 컴포넌트 자체를 메모이제이션하여, 동일한 props가 주어졌을 때는 재렌더링하지 않도록 한다. 만약 하위 컴포넌트에서 props 변화가 없는데도, 상위가 리렌더링되면 같이 리렌더링되는 문제가 있을수 있는데, 이때 ‘React.memo’를 사용해 불필요한 렌더링을 막을 수 있다.

```tsx
const Child = React.memo(({ label }) => {
  console.log('Child 렌더링');
  return <div>{label}</div>;
});
```

위 코드에서 Child는 label이 바뀌지 않는 한 렌더링되지 않는다. 내부 상태나 다른 props가 없으면, 콘솔 로그가 찍히지 않아 최적화 효과를 확인 할 수 있다.

### **useMemo vs React.memo**

| **구분** | **useMemo** | **React.memo** |
| --- | --- | --- |
| 적용 대상 | **값** (무거운 연산 결과) | **컴포넌트** |
| 반환 | 캐싱된 **값** | 캐싱된 **컴포넌트** |
| 사용 위치 | 컴포넌트 내부 | 컴포넌트 정의 시 (HOC처럼 감싸는 형태) |
| 주 목적 | **연산 최적화** (정렬, 필터링 등) | **리렌더링 최적화** (동일 props 시 렌더 불필요) |

### 사용 예제

- useMemo

```tsx
const sortedList = useMemo(() => {
  return items.sort((a, b) => a.value - b.value);
}, [items]);
```

items가 바뀌지 않으면 정렬을 다시 하지 않음단순히 값 계산을 메모이제이션함

- React.memo

```tsx
const ListItem = React.memo(({ item }) => {
  console.log('ListItem render');
  return <li>{item.label}</li>;
});

...

<ListItem item={item} /> // item이 바뀌지 않으면 재렌더링 X
```

부모 컴포넌트가 리렌더링되더라도 item props가 같다면 ListItem은 렌더되지 않음 컴포넌트 자체를 메모이제이션

---

## 함께 쓰는 예시

```tsx
const filteredList = useMemo(() => {
  return list.filter((item) => item.active);
}, [list]);

const handleClick = useCallback(() => {
  console.log('clicked');
}, []);

const ItemList = React.memo(({ data, onClick }) => {
  return data.map((item) => <div onClick={onClick}>{item.name}</div>);
});
```

- `useMemo`로 **계산된 값 캐싱**
- `useCallback`으로 **콜백 캐싱**
- `React.memo`로 **컴포넌트 전체 리렌더링 방지**

### 💡Tip

- `React.memo`는 **정적 props를 가진 자식 컴포넌트**에 유용하다.
- `useCallback`은 **props로 전달되는 이벤트 핸들러가 재생성되는 것을 막기 위해 사용한다.**
- 단, **불필요한 최적화는 오히려 오버헤드**가 될 수 있으므로, 렌더링 횟수가 눈에 띄게 많은 경우나 복잡한 UI 구성에서만 사용하는 것이 좋다.
    - useMemo와 useCallback 자체도 내부 동작이 있기 때문에, 성능이 확실히 문제가 될 때 사용하는 것이 좋다.

## **실무에서 자주 쓰는 커스텀 훅**

### **useDebounce (지연된 입력값)**

```tsx
import React, { useEffect, useState } from 'react';

const useDebounce = <T,>(value: T, delay: number): T => {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(handler);
  }, [value, delay]);

  return debounced;
};

const SearchInput = (): JSX.Element => {
  const [input, setInput] = useState('');
  const debouncedInput = useDebounce(input, 500);

  useEffect(() => {
    if (debouncedInput) {
      console.log('API 요청: ', debouncedInput);
    }
  }, [debouncedInput]);

  return (
    <input
      placeholder="검색어 입력"
      value={input}
      onChange={(e) => setInput(e.target.value)}
    />
  );
};

export default SearchInput;

```

> 해당 커스텀 훅은 검색창의 검색어 추천 기능 같은 것을 구현할 때 비동기 요청 수를 줄일 수 있는 훅이다. 사용자의 입력이 멈춘 뒤 0.5초 후에만 API를 요청해 서버 부담을 줄일 수 있다. useEffect와 useCallback을 조합해 실제 API 콜백을 최적화할 수도 있다.

### useMemoCompare (깊은 비교 메모이제이션)

```tsx
export const useMemoCompare = <T>(next: T, compare: (a: T, b: T) => boolean): T => {
  const previousRef = useRef<T>(next);
  const isEqual = compare(previousRef.current, next);

  if (!isEqual) {
    previousRef.current = next;
  }

  return useMemo(() => previousRef.current, [isEqual]);
};
```

> 일반적인 useEffect는 얕은 비교만 수행하기 때문에, 객체가 같더라도 주소가 바뀌면 다시 실행되는데 이 커스텀 훅을 통해 방지할 수 있다. 해당 코드는 비교함수를 통해 JSON.stringify 또는 직접 필드 비교가 가능하여 React Query, Zustand와 같이 상태 동기화 시 유용하게 사용할 수 있다.

### useEventCallback (항상 최신 상태를 참조하는 콜백)

```tsx
const useEventCallback = <T extends (...args: any[]) => any>(fn: T): T => {
  const ref = useRef(fn);

  useEffect(() => {
    ref.current = fn;
  }, [fn]);

  return useCallback((...args: any[]) => ref.current(...args), []);
};
```

> 비동기 핸들러나 이벤트 핸들러에서 최신 state/props를 안정적으로 참조하고 싶을 때 사용한다.

---

## 마무리

- **useMemo**: 무거운 연산 결과 캐싱 (값 최적화)
- **React.memo**: 컴포넌트 자체 메모이제이션 (리렌더링 최적화)
- **useCallback**: 동일 함수 객체 재사용 (함수 최적화)

이 세 가지를 상황에 맞게 조합하면, 리액트 애플리케이션의 불필요한 렌더링과 높은 연산 비용을 크게 줄일 수 있다. 다만, 남용보다는 **정말 필요한 부분에만** 적용하는 것이 중요하다.
