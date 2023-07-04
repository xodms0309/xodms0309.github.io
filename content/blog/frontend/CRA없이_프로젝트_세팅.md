---
title: 'CRA없이 웹팩으로 React, Typescript 환경 구축하기'
date: 2023-07-04 20:00:00
category: 'frontend'
draft: false
---

펀잇 프로젝트를 구축하며 CRA를 사용하지 않고 웹팩을 사용하여 프로젝트 세팅을 진행했다. CRA를 사용하면 `npx create-react-app` 명령어를 통해 아주 간편하게 리액트 환경을 구축할 수 있지만 사용하지 않는 모듈까지 포함이 되고 path alias 설정 등 커스터마이징이 어렵다는 단점이 있다. vite로 마이그레이션 하는 움직임들도 많이 보이지만 아직까지는 webpack으로 구현된 환경이 많다는 점, webpack을 알게 된다면 다른 번들러도 사용하기 쉽다는 점에서 webpack을 사용하게 되었다.

[npm trends](https://npmtrends.com/rollup-vs-vite-vs-webpack)
![번들러npmtrends](https://imgur.com/GTqMbZm.png)

### 설정 환경

- yarn
- React v18
- Typescript
- webpack 5
- ts-loader

## 1. 필요한 dependency 설치

dependency와 devDependency를 분리하여 설치하도록 한다.

> dependency: 개발, 런타임, 빌드타임에서 모두 필요한 패키지들 <br/>
> devDependency: 런타임에선 필요 없고 개발, 빌드타임에서만 필요한 패키지

- 리액트

```bash
yarn add react react-dom
```

- 타입스크립트

```bash
yarn add -D typescript @types/react @types/react-dom
```

- 웹팩

```bash
yarn add -D webpack webpack-dev-server webpack-cli
```

- ts loader

```bash
yarn add -D ts-loader
```

그 외 사용하는 라이브러리가 있다면 설치하도록 한다.

## 2. 웹팩 설정

웹팩을 어떻게 설정할지는 팀마다 천차만별로 달라질 것이라고 생각한다. 우리 팀은 우선 `common`, `dev`, `prod` 이렇게 세 파일을 만들어 환경에 따라 다르게 번들링 되게끔 설정했다.

### 펀잇팀의 웹팩 설정

```jsx
//webpack.common.js

const path = require('path')

module.exports = {
  entry: './src/index.tsx',
  output: {
    path: path.join(__dirname, 'dist'),
    filename: 'bundle.js',
    clean: true,
    publicPath: '/',
  },
  resolve: {
    extensions: ['.ts', '.tsx', '.js'],
    alias: {
      '@': path.resolve(__dirname, './src'),
      //프로젝트에서 설정한 alias
    },
  },
  module: {
    rules: [
      {
        test: /\.(ts|tsx|js)$/,
        exclude: /node_modules/,
        use: [
          {
            loader: 'ts-loader',
          },
        ],
      },
      {
        test: /\.(png|jpeg|jpg)$/,
        type: 'asset/resource',
      },
    ],
  },
}
```

```jsx
//webpack.dev.js

const { merge } = require('webpack-merge')
const common = require('./webpack.common.js')
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = merge(common, {
  mode: 'development',
  devtool: 'eval',
  devServer: {
    historyApiFallback: true,
    port: 3000,
    hot: true,
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html',
      minify: false,
      hash: true,
    }),
  ],
})
```

```jsx
//webpack.prod.js

const { merge } = require('webpack-merge')
const common = require('./webpack.common.js')
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = merge(common, {
  mode: 'production',
  devtool: 'hidden-source-map',
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html',
      minify: {
        collapseWhitespace: true,
        removeComments: true,
      },
      hash: true,
    }),
  ],
})
```

### babel loader vs ts-loader

타입스크립트 환경이기 때문에 `babel-loader` 을 사용할지 `ts-loader` 을 사용할지 많은 고민을 했다.

**babel-loader**

- ts 처리를 하기 위해 [@babel/preset-typescript](https://babeljs.io/docs/en/babel-preset-typescript) 필요
- type checking을 하지 않음 (타입 오류가 있더라고 빌드가 됨) → ts-loader보다 속도가 빠름
- 폴리필 환경 설정 가능

**ts-loader**

- type checking을 진행 (타입 오류가 있으면 빌드 실패)

`babel-loader`가 속도는 더 빠르지만 타입 체킹을 하지 않는다는 점과 폴리필이 필요한 IE가 지원 종료되었다는 점을 고려하여 `ts-loader`로 결정하게 되었다.

### dev환경 vs prod환경

**devtool**

웹팩의 [devtool](https://webpack.js.org/configuration/devtool/) 설정을 통해 소스맵을 어떻게 설정할 지 결정할 수 있다. 보통 development 모드에는 리빌드가 가장 빠른 `eval` 옵션을 많이 사용한다. 배포시에는 소스맵을 적용하지 않거나 `source-map` 옵션이나 `hidden-source-map` 옵션을 많이 사용한다. 우리 팀은 소스맵에 참조가 표시되지 않는 `hidden-source-map` 옵션을 선택했다.

**HtmlWebpackPlugin**

development 모드와 production mode에서 `minify`옵션을 다르게 하여 production 환경에서는 주석, 공백을 지우게끔 하여 용량을 줄이고자 했다.

## 3. tsconfig 설정

tsconfig 설정에 대한 설명은 [이 글](https://xodms0309.github.io/typescript/tsconfig/)에 상세히 작성했다.

```jsx
{
  "compilerOptions": {
    "target": "es6",
    "lib": ["dom", "dom.iterable", "esnext"],
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true,
    "jsx": "react-jsx",
    "outDir": "./dist",
    "baseUrl": ".",
    "paths": {
      "@*": ["src/*"]
    }
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

여기서 `ts-loader` 을 사용할 때 `module`을 `commonjs` 로 설정하게 되면 웹팩이 코드 [트리쉐이킹](https://webpack.kr/guides/tree-shaking)이 되지 않는다고 한다.

![ts-loader와 트리쉐이킹](https://imgur.com/TWN7VEO.png)

## 4. 필요한 파일 생성

### public/index.html

```html
<!DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```

### index.tsx & App.tsx

```jsx
//index.tsx

import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root') as HTMLElement);
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

```jsx
//App.tsx

const App = () => {
  return <></>
}

export default App
```

## 5. package.json에 scripts 작성

```json
"scripts": {
    "start": "webpack serve --open --config webpack.dev.js",
    "build": "webpack --config webpack.prod.js",
    "build-dev": "webpack --config webpack.dev.js"
 },
```

이렇게 설정하고 `yarn start` 를 하면 3000포트에서 프로젝트가 실행된다. 🎉

## 마무리

프로젝트 세팅을 하면서 팀원들과 어떤 로더를 사용할지, tsconfig에 어떤 설정들이 있어야할 지, 웹팩에 어떤 내용이 들어가야 할 지 등 많은 고민을 나눴다. 그리고 공식문서를 읽으면서 몰랐던 소스맵이나 트리쉐이킹에 대해서도 알 수 있었다. 프로젝트를 진행하면서 환경 설정에 변경점들이 생길 수 있지만 이렇게 고민을 통해 구축해냈다는 점이 매우 뿌듯하다.
