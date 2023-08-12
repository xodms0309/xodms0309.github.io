---
title: 'React 18 달라진 점'
date: 2022-06-03 17:06:00
category: 'react'
draft: false
---

![](https://velog.velcdn.com/images/taeeeeun/post/d408b822-5a60-40e8-bb80-1ff09dcb2a29/image.png)

> **React 18**에서 가장 중요하게 생각하는 점은 **concurrency(동시성)**이다.

![](https://velog.velcdn.com/images/taeeeeun/post/f55cbe6f-0f5a-4a14-8b7c-323b71c43ad3/image.png)

`non-concurrent` 환경에서는 한 번에 한 가지 일을 할 수 있다. Alice와 채팅을 한 뒤 Bob과 채팅을 할 수 있다.

![](https://velog.velcdn.com/images/taeeeeun/post/64de9f54-67a3-428b-b0e5-001a1a4e4f82/image.png)

`concurrent` 환경에서는 Alice와 채팅을 하면서 동시에 Bob과 채팅할 수 있다.

이처럼 React 18은 `concurrent rendering`을 제공한다. React이제 오래 걸리는 렌더링 중간에도 즉각적인 유저 인터랙션을 제공할 수 있다.

## 자동 배치 (Automatic Batching)

리액트는 원래 이벤트 핸들러 함수에서는 배치를 수행하여 상태 업데이트를 위한 리렌더링을 진행했다. 하지만 `promise`, `setTimeout`, `native event handler`에서는 배치가 이루어지지 않았다.

```jsx
//기존
setTimeout(() => {
  setCount(c => c + 1) //re-render
  setFlag(f => !f) //re-render
}, 1000)

//v18
setTimeout(() => {
  setCount(c => c + 1)
  setFlag(f => !f)
  //re-render at last
}, 1000)
```

v18에서는 자동 배치를 수행하여 마지막에 한 번만 렌더링 되게끔 한다.

- 기존: `count`와 `flag` 상태값을 업데이트 하기 위해 두 번 렌더링 됨
- v18: **배치를 수행하여 한 번만 렌더링 됨**

만약 배치를 수행하고 싶지 않다면? `ReactDom.flushSync()`를 사용하면 된다. (흔하게 사용되진 않을 것이라고 예상된다.)

```jsx
import { flushSync } from 'react-dom'

function handleClick() {
  flushSync(() => {
    setCounter(c => c + 1)
  })
  // React has updated the DOM by now
  flushSync(() => {
    setFlag(f => !f)
  })
  // React has updated the DOM by now
}
```

## Transition

Transition은 `urgent`와 `non-urgent` 업데이트를 구분하기 위한 개념이다.

- **Urgent**: 직접적인 상호작용(타이핑, 클릭 등)
- **Non-Urgent**: 뷰 UI 전환

사용자들이 화면을 볼 때 타이핑이나 클릭된 것의 결과값은 즉각적으로 제공해야 화면이 어색해지지 않으나 뷰 전환 같은 경우 바로 화면이 제공될 것이라고 예상하지 않기 때문에 긴급한 업데이트가 아니다.

- 이전에 `debounce`, `throttle` 기법을 사용하던 것을 명시적으로 업데이트 한 것!

```jsx
import { startTransition } from 'react'

// Urgent: input에 적힌 값을 보여줘야 함
setInputValue(input)

//startTransition으로 감싸졌다면 긴급하지 않은 업데이트로 간주
startTransition(() => {
  //input값에 따른 결과값 제공
  setSearchQuery(input)
})
```

- `startTransition`으로 감싸져있다면 transition update로 간주되며 urgent update가 들어오게 된다면 중지된다.

```jsx
const [isPending, startTransition] = useTransition()
//or
import { startTransition } from 'react'
```

- `useTransition`: pending state와 transition을 사용하기 위한 훅
- `startTransition`: 훅을 사용할 수 없을 때 사용

## Suspense

`Suspense`는 `React.lazy`를 SSR측면에서 사용할 수 있게 한다.

component를 렌더링 하는데 필요한 data fetching이 오래 걸린다면 사용자들은 오랫동안 빈 화면을 보고 있게 된다.

렌더링 준비가 될 때까지 `Suspense`의 `fallback` 컴포넌트로 대체되어 보여준다.

```jsx
<Layout>
  <NavBar />
  <Sidebar />
  <RightPane>
    <Post />
    <Suspense fallback={<Spinner />}>
      <Comments />
    </Suspense>
  </RightPane>
</Layout>
```

`Comments`가 렌더링 되는 동안 나머지 페이지를 보여주고, `Comments`가 표시되는 부분은 `Spinner`로 대체한다.

```jsx
<div>
  {showComments && (
    <Suspense fallback={<Spinner />}>
      <Panel>
        <Comments />
      </Panel>
    </Suspense>
  )}
</div>
```

`showComments`가 `false`→`true`로 변경될 때 `<Comments />`가 data fetching으로 인해 렌더링이 오래 걸리는 상황이다.

이전 버전에서는 `Comments`를 제외한 `Panel`을 DOM에 추가하고, `Comments`가 준비될 때 까지 `display: none`을 주었다.

하지만 v18에서는 이렇게 동작한다.

1. **(New) `Panel`은 DOM에 넣지 않는다**
2. `Spinner`을 DOM에 넣는다
3. `Comment`를 기다린다
4. re-rendering을 시도한다
5. `Spinner`을 DOM에서 제거한다
6. `Panel`을 `Comment`와 함께 DOM에 넣는다.

### 참고

[https://reactjs.org/blog/2022/03/29/react-v18.html](https://reactjs.org/blog/2022/03/29/react-v18.html) 및 연결 문서들

[https://www.freecodecamp.org/news/react-18-new-features/](https://www.freecodecamp.org/news/react-18-new-features/)
