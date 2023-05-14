---
title: 'React의 virtualDOM'
date: 2023-04-26 17:06:00
category: 'react'
draft: false
---

# Virtual DOM이란?

### 등장 배경

DOM 조작을 통해 웹페이지의 내용을 변경하게 된다면 변경 대상 element를 업데이트하는 과정에서 렌더트리를 갱신하고 UI를 리렌더링 하게 되어 성능 저하가 발생할 수 있다.

### Virtual DOM이란?

실제 DOM 구조와 비슷한, 사용자 인터페이스를 나타내는 자바스크립트 객체

# Virtual DOM의 동작과정

![](https://i.imgur.com/PIFVCYP.png)

1. `state` 나 `props`가 변경되면 변경사항을 반영하여 새로운 Virtual DOM을 생성한다.
2. 현재 Virtual DOM과 이전 Virtual DOM을 비교하여 변경된 부분을 계산한다. (diffing)
   - 따라서 두 개의 Virtual DOM을 가지고 있게 된다.
3. 변경된 부분을 실제 DOM에 반영한다. (reconciliation)
   - 리액트는 O(n) 복잡도를 가진 휴리스틱 알고리즘을 구현했다

## Reconciliation (재조정)

<aside>
💡 Fiber란?
리액트 16버전에서 나온 개념으로 복잡한 스케줄링을 해결하기 위한 하나의 ‘작업 단위’이다. = 리액트 재조정 알고리즘의 스케줄링 가능한 virtual stack (가상의 call stack)

</aside>

1. Render Phase (비동기적)
   - 변경점이 반영된 fiber node를 생성한다. (변경이 없다면 이전 fiber을 재사용한다)
   - 이전과 다르게 중간에 작업을 중단할 수도 있고 재개할 수도 있음
2. Commit Phase (동기적)
   - Render Phase에서 생성한 트리를 실제 DOM에 반영한다.

### Diffing Algorithm

2개의 트리를 비교할 때, 리액트는 root 엘리먼트부터 비교한다.

1. 엘리먼트의 타입이 다른 경우→ 이전 트리를 버리고 완전히 새로운 트리 생성

   ex) `<a>` 태그에서 `<img>` 로 바꾸는 경우

2. 엘리먼트의 타입이 같은 경우→ 두 엘리먼트의 속성을 확인하고 변경된 속성들만 갱신

   ex) className 변경, style 갱신

### Keys

- 자식들이 key를 가지고 있다면, key를 통해 기존 트리와 이후 트리의 자식들이 일치하는지 확인한다.
- key 값은 변하지 않고, 예상 가능하며, 유일한 값이어야 한다.
  - 재배열 되지 않는 경우엔 배열의 index 값도 ok

# 리액트가 Virtual DOM을 사용하는 이유

1. 렌더링 성능 향상
   - 실제 DOM에 변화를 반영하기 전에 가상 DOM에 반영하고, 최종 결과를 DOM에 전달함
   - 레이아웃과 리렌더링이 한 번만 발생
   - 속도가 빠르다는 것은 X 충분히 빠르다 O (새로운 객체 생성 > 브라우저 렌더링)
2. 위의 과정을 수동이 아닌 자동화, 추상화 할 수 있음

### Reference

[https://indepth.dev/posts/1008/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react](https://indepth.dev/posts/1008/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react)
