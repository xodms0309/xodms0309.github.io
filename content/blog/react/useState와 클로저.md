---
title: '리액트 useState와 클로저'
date: 2022-06-01 17:06:00
category: 'react'
draft: false
---

요즘 You Don't Know JS라는 책을 완독하는 스터디를 하고 있는데 이번주엔 호이스팅과 클로저에 대해 읽게 되었다. 클로저에 대해 와닿지가 않았는데 같이 스터디 하시는 분이 리액트의 useState에서도 클로저가 사용된다고 하셨다.😮 그래서 한 번 useState에서의 클로저에 대해 작성해보기로 했다.

### 1. 클로저란?

> Closure makes it possible for a function to have private variables

YDKJS 책에서 클로저는 **함수가 속한 렉시컬 스코프를 기억하여 함수가 렉시컬 스코프 밖에서 실행될 때도 스코프에 접근할 수 있도록 하는 기능**이라고 소개한다.

예시 코드)

```javascript
//출처: YDKJS 1권
function foo() {
  var a = 2
  function bar() {
    console.log(a)
  }
  return bar
}

var baz = foo()
baz() // 2
```

이 코드를 설명해보자면,

- bar()는 foo()의 렉시컬 스코프에 접근할 수 있다.
- 이 코드는 bar을 참조하는 함수 객체를 반환한다.
- foo() 실행→ 반환값(bar()함수)를 baz에 대입→ baz() 함수 호출

→ bar()이 실행되었지만 **함수가 선언된 렉시컬 스코프 밖에서 실행됨**

- **bar()는 foo()에 대한 렉시컬 스코프 클로저를 가지고, foo()는 bar()이 나중에 참조할 수 있도록 스코프를 살려둔다.**

→ bar()이 해당 스코프에 참조를 가지는 것을 **클로저**라고 한다.

#### 클로저의 역할

- **은닉화**: 외부에서 변수를 함부로 접근할 수 없게하는 JAVA의 private효과와 비슷하다.
- 전역변수 남용을 방지할 수 있다.

### 2. useState

```js
const [state, setState] = useState(initialValue)
```

- `useState`는 리액트의 함수 컴포넌트에서 변경되는 상태의 값을 관리할 수 있도록 도와주는 hook이다.
- `state`가 변경되었음을 알기 위해서는 이전 `state`의 값을 알고 있어야 하는데, 이 때 클로저가 사용된다.

```js
const React = (function() {
  let hooks = [] //state를 담아두는 배열, 상태들은 순서대로 저장된다.
  let idx = 0 //state를 구별해주는 key

  function useState(initialValue) {
    const state = hooks[idx] || initialValue
    //setState는 비동기적으로 작동하기 때문에 idx값이 계속해서 0으로 바뀌는 것을 방지해줘야 함 !
    //_idx에 idx값을 freeze한다.
    const _idx = idx
    const setState = newValue => {
      hooks[_idx] = newValue
    }
    idx++
    return [state, setState]
  }
  return { useState }
})()
```

코드 출처: [Getting Closure on React Hooks by Shawn Wang | JSConf.Asia 2019](https://www.youtube.com/watch?v=KJP1E-Y-xyo)

- `useState`가 실행될 때 함수가 선언된 렉시컬 스코프 밖에서 실행되고 스코프가 유지되어 `hooks`와 `idx`를 참조할 수 있게 된다.
- 리액트는 `state`들의 이전 상태를 접근하기 위해 `state`를 `useState` 외부에 저장해두고, `useState` 내부에서만 접근할 수 있게끔 한다.

- `state`는 `useState`에서 반환된 값을 사용하고, `state`를 바꾸는 것도 `setState`를 통해서만 할 수 있다.

> 💡 ![](https://velog.velcdn.com/images/taeeeeun/post/2447f5dc-826d-41bc-8fa6-11340e462fa4/image.png)
>
> - 리액트는 **hook이 호출되는 순서에 의존하기 때문에** 조건문, 반복문에서 hook을 호출하면 안된다.
> - 어떤 경우에는 `useState` hook이 2번 실행되고, 어떤 경우에는 3번 실행된다면 참조하는 state가 엉망이 될 수 있다.

### 3. 정리 및 느낀점

리액트는 이전 상태의 값을 참조하기 위해 함수 외부에 상태 배열을 저장한다. 그리고 클로저를 통해 해당 값들에 접근하고 변경할 수 있게 한다. 또한, 상태를 참조할 때 배열에 저장된 순서대로 하므로 조건문, 반복문에서는 hook을 호출하면 안된다.

이런걸 알아갈 때마다 내가 리액트를 쓰면서도 리액트에 대해 알고있지 못했다는 생각이 든다. 오늘도 새로운걸 알아간다..❗

#### 참고

[Getting Closure on React Hooks by Shawn Wang | JSConf.Asia 2019](https://www.youtube.com/watch?v=KJP1E-Y-xyo)
[Hook의 규칙](https://ko.reactjs.org/docs/hooks-rules.html#explanation)
https://yeoulcoding.me/m/149
