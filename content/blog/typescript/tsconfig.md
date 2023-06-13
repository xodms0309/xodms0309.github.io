---
title: 'tsconfig.json의 설정들'
date: 2023-06-13 13:00:00
category: 'typescript'
draft: false
---

```json
{
	"extends": "../../tsconfig.json",
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
		...
  },
  "include": ["src"],
	"exclude": ["node_modules"],
}
```

프로젝트를 타입스크립트 기반으로 만들게 된다면 위와 같은 `tsconfig.json` 파일을 생성하게 된다. `tsconfig.json` 는 타입스크립트를 자바스크립트로 변환할 때 필요한 컴파일 설정을 가지고 있는 파일이다. 미션을 하면서 이전에 사용했던 값들을 복사해서 사용해왔는데 모르고 쓰는 설정들이 너무 많아서 한 번 정리해보려고 한다.

## extends

extends 속성은 다른 위치의 `tsconfig.json` 파일의 설정을 가져와 쓸 수 있게 하는 속성이다. 보통 파일의 최상단에 적어준다. 이 속성은 한 레포지토리내에 여러 프로젝트들이 있을 때 유용하게 쓸 수 있다. ex) 모노레포

[이전에 한 프로젝트](https://github.com/YAPP-Github/20th-ALL-Rounder-Team-2-Web)에서 모노레포로 서비스와 랜딩페이지를 같이 관리한 적 있었는데, 최상단에 작성해준 `tsconfig.json` 파일을 각각의 프로젝트에서 extends해와 사용했다.

## include

`include` 속성은 프로젝트에서 컴파일할 파일들을 와일드 카드 패턴을 사용하여 지정한다. 하지만 exclude에 명시되어 있다면 include에 명시되어 있어도 무시된다.

## exclude

`exclude` 속성은 컴파일 대상에서 제외할 파일들을 와일드 카드 패턴을 사용하여 지정한다. 주로 `node_modules`나 test파일, 빌드된 파일들을 제외하게 된다.

## compilerOptions

`compilerOptions`는 말 그대로 컴파일 할 파일들에 대해 어떻게 변환할 지 옵션을 정하는 속성이다.

![compilerOptions](https://imgur.com/QOJJEKj.png)

타입스크립트 공식문서를 보면 이렇게 카테고리 별로 정리되어 있다. 정말 많은 옵션들이 있어 주로 사용하는 옵션들에 대해 알아보았다.

## Modules

### baseUrl, paths

```json
{
  "compilerOptions": {
    "baseUrl": "./src",
		"paths":{
			"@components/*": ["components/*"],
		}
}
```

절대경로로 파일을 import할 때 사용하는 속성이다. 이렇게 paths 속성까지 설정해주게 된다면 파일에서 아래와 같이 import문을 작성해줄 수 있다. (CRA 환경에서는 craco를 사용해 설정해야 하는 것으로 알고 있다.)

```jsx
import Button from '@components/Button'
```

절대경로로 작성해도 문제가 없지만 프로젝트 규모가 커질수록 import문이 깔끔해야 알아보기 쉬워지므로 많이 쓰이는 속성이다.

### module (default commonjs)

```jsx
{
  "compilerOptions": {
    "module": "commonjs",
}
```

import문을 컴파일 했을 때 어떤 방식으로 해석할 지 설정하는 속성이다. (ex. commonJS 환경에서는 import문을 읽을 수 없기 때문에 require 구문으로 컴파일 해줘야 한다.)

## Emit

### noEmit

`true` 로 설정하게 되면 babel이나 swc와 같은 트랜스파일러가 타입스크립트 변환을 할 수 있게끔 하는 옵션이라고 한다.

### outDir

```jsx
{
  "compilerOptions": {
    "outDir": "./dist",
}
```

컴파일을 했을 때 어떤 경로에 결과물을 위치시킬지 결정하는 옵션

### downlevelIteration

ES6에서 추가된 for-of loop, 스프레드 연산자, iterator 등을 ES5 환경에서 더 정확하게 사용하기 위한 옵션이다. 이 옵션을 사용하지 않고 그냥 for문으로 변환하게 된다면 발생하는 예외 케이스들이 있다.

```
ex) 😜 이 이모지의 length는 2이다. 만약 이 이모지를 가지고 `for-of` loop를 돌린다면
1번의 iteration이 발생하지만 `for`문을 돌린다면 loop를 2번 돌게 된다.
```

하지만 이런 예외 경우는 매우 적고 iterator를 사용하지 않았다면 굳이 필요하지 않은 속성이라고 한다.

## Type Checking

### strict

기본으로 활성화 되어 있으며 타입 체킹을 담당한다.

### noImplicitAny

암시적인 any에 대한 에러를 발생시킨다. 위의 `strict` 옵션을 활성화하게 되면 자동으로 `noImplicitAny` 또한 활성화 된다.

### strictNullChecks

null, undefined 타입에 대한 엄격한 검사를 진행할지 여부를 결정할 수 있는 옵션이다. 이 속성 역시 strict를 사용하면 기본적으로 활성화된다.

## Language and Environment

### target (default es5)

```json
{
  "compilerOptions": {
    "target": "es5",
}
```

어떤 버전의 자바스크립트 코드로 컴파일 할 지 결정한다. 주로 ES5로 사용되다가 대부분의 브라우저가 ES6를 지원하고 있어 최근에는 ES6로 많이 사용되는 것 같다. 그 중 `ESNext` 로 설정하게 되면 가장 최신 자바스크립트 버전을 의미하게 된다.

### lib

```json
{
  "compilerOptions": {
    "lib": ["dom", "dom.iterable", "es5"],
}
```

컴파일 과정에서 사용되는 JS API와 같은 라이브러리를 지정할 수 있다.

- dom: window, document 와 같은 dom API 접근
- dom.iterable: `forEach`, `entries`등의 DOM 리스트를 순회하는 iterable method들
- es5: 위에서 지정한 버전의 API (es6, esnext 등등 설정할 수 있음)

왜 dom과 dom.iterable이 따로 있을까 궁금해서 찾아봤는데 정확한 이유는 잘 모르겠지만 dom.iterable이 더 뒤에 나온 프로퍼티라 그런 것 같기도 하다.

### jsx

```json
{
  "compilerOptions": {
    "jsx": "react",
}
```

`.tsx` 로 작성된 파일을 어떻게 컴파일 할 지 결정한다.

![JSX속성](https://imgur.com/SMqDbuj.png)

- `react`: `React.createElement()` 를 생성하여 JSX 변환이 필요하지 않고 `.js`로 컴파일한다.
- `preserve`: JSX파일을 `.jsx` 로 컴파일한다.
- `react-native`: JSX 형식을 유지하나 `.js` 로 컴파일한다.
- `react-jsx`: React17에서 소개된 변환법을 사용하며`.js` 로 컴파일됨. 이 설정을 사용하는 것을 추천한다고 한다. (https://legacy.reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html#whats-different-in-the-new-transform)

## 기타

### skipLibCheck

`true` 로 활성화하게 된다면 프로젝트에서 사용된 라이브러리에 대한 타입 체킹을 스킵하게 되어 컴파일 시간을 줄여준다.

### forceConsistentCasingInFileNames

파일 이름의 일관성을 위해 파일명의 대소문자를 구분하는 속성이다.

### allowJS

타입스크립트로 컴파일이 되지만 자바스크립트 파일도 허용을 할 것인지 설정할 수 있는 옵션이다. (`.js` 파일을 `.ts` 에서 import할 수 있게 한다.) 프로젝트 마이그레이션이 일어날 때 사용하면 좋을 것 같다.

## 정리

정말 많은 속성들이 있어 극 일부분만 정리를 하게 되었다. 하지만 정리를 하다 보니 왜 이렇게 설정했지? 싶은 속성들도 있었고 다음부터는 필요한 속성들을 꺼내 쓸 수 있을 것 같은 기분이 든다. 아래 출처에 모든 속성들이 정리되어있으니 프로젝트 세팅할 때 한 번씩 참고하는 것도 좋을 것 같다.

### 출처

https://www.typescriptlang.org/ko/tsconfig
https://yamoo9.gitbook.io/typescript/cli-env/tsconfig <br/>
https://typescript-kr.github.io/pages/compiler-options.html
