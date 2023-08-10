---
title: '리액트에서 SvgSprite 컴포넌트 사용하기'
date: 2023-08-06 12:12:00
category: 'etc'
draft: false
---

웹 어플리케이션을 만들게 된다면 화면에 아이콘이 들어가는 경우가 많다. 펀잇의 메인페이지에도 많은 아이콘들이 들어간다.

<img width="377" alt="스크린샷 2023-08-06 오후 3 36 35" src="https://github.com/woowacourse-teams/2023-fun-eat/assets/55427367/a613a12f-3c59-4f7f-9bf4-7821426c5fc7">

이런 아이콘들을 모두 이미지로 넣게 된다면 로딩 시간이 걸리게 될 것이고 좋지 않은 사용자 경험을 제공하게 된다. 안 그래도 메인 페이지에서 5개 이상의 api 호출이 일어나고 있는데 아이콘이라도 부담을 덜어주는 것이 좋을 것이다.

<img width="736" alt="스크린샷 2023-08-06 오후 7 19 55" src="https://github.com/woowacourse-teams/2023-fun-eat/assets/55427367/eec6c81a-c88c-425e-be91-24ca03103181">

(이렇게 페이지를 바꿀 때 마다 네비게이션 바의 아이콘 파일들이 다운된다.)

이러한 문제점을 해결해주는 방식이 바로 [[이미지 스프라이트](http://www.tcpschool.com/css/css_basic_imageSprites)](http://www.tcpschool.com/css/css_basic_imageSprites) 기법인데, 이는 여러개의 이미지를 하나로 합쳐 한 번의 요청으로 인해 모든 이미지를 다운받는 방법을 말한다. 이 방법을 svg로 변형한 것이 바로 svg 스프라이트 방식이다.

## Svg Sprite 컴포넌트 만들기

1. 피그마에서 사용할 아이콘들을 svg 파일로 export 해온다.
2. [[svg sprite bot](https://github.com/thomasjbradley/spritebot)](https://github.com/thomasjbradley/spritebot)을 사용하여 svg sprite sheet를 만든다.

<img width="812" alt="스크린샷 2023-08-06 오후 6 51 53" src="https://github.com/woowacourse-teams/2023-fun-eat/assets/55427367/9c5ebda4-13b6-493a-b3d9-6a8dcb967834">

그럼 이렇게 압축도 할 수 있다.

3. svg sprite 컴포넌트 생성

위에서 만든 svg sprite sheet를 살펴보면 아래와 같은 코드가 있을 것이다. 이것을 `SvgSprite` 라는 컴포넌트로 만들어주자.

```tsx
const SvgSprite = () => {
  return (
    <svg xmlns="http://www.w3.org/2000/svg" display="none">
      <symbol id="arrow" viewBox="0 0 8 12">
        <path d="M7.41 10.59L2.83 6l4.58-4.59L6 0 0 6l6 6 1.41-1.41z" />
      </symbol>
      <symbol id="list" viewBox="0 0 18 14">
        <path d="M0 0h18v2H0zm0 6h18v2H0zm0 6h18v2H0z" />
      </symbol>
      {/* 생략 */}
    </svg>
  )
}
```

이렇게 하면 svg 아이콘들을 하나의 파일로 합칠 수 있다.

4. index.tsx에 위의 `SvgSprite` 컴포넌트를 삽입한다.

```tsx
const root = ReactDOM.createRoot(document.getElementById('root') as HTMLElement)
root.render(
  <React.StrictMode>
    <SvgSprite />
  </React.StrictMode>
)
```

이렇게 정적인 스프라이트 컴포넌트를 삽입해주면 한 번의 로딩 만으로 모든 svg 아이콘들을 다운받을 수 있다.

(이 때 `display: none` 속성을 줘서 svg sprite 컴포넌트는 화면에 보이지 않도록 한다.)

## SvgIcon 컴포넌트로 사용하기

그럼 이제 만든 스프라이트 컴포넌트를 사용해보자.

아래는 현재 펀잇팀에서 사용하고 있는 SvgIcon 컴포넌트이다. 이렇게 `width`, `height`, `fill`을 받아오게끔 지정해주면 크기와 색상도 커스터마이징 가능하다.

그리고 가장 중요한 점은 `use` 태그 안의 href에는 위 스프라이트 컴포넌트에 들어가는 `symbol` 의 `id` 값을 전달해줘야 한다. 우리 팀은 타입스크립트를 쓰고 있기 때문에 id 값들의 배열을 `SvgIconVariant` 라는 type으로 만들어서 사용하고 있다.

```tsx
import { theme } from '@fun-eat/design-system';
import type { ComponentPropsWithoutRef, CSSProperties } from 'react';

export const SVG_ICON_VARIANTS = ['recipe', 'list', ...생략] as const;
export type SvgIconVariant = (typeof SVG_ICON_VARIANTS)[number];

interface SvgIconProps extends ComponentPropsWithoutRef<'svg'> {
  variant: SvgIconVariant;
  color?: CSSProperties['color'];
  width?: number;
  height?: number;
}

const SvgIcon = ({ variant, width = 24, height = 24, color = theme.colors.gray4, ...props }: SvgIconProps) => {
  return (
    <svg width={width} height={height} fill={color} {...props}>
      <use href={`#${variant}`} />
    </svg>
  );
};

export default SvgIcon;
```

이렇게 만들면 밖에서 이렇게 사용할 수 있다.

```tsx
<SvgIcon variant="star" width={20} height={20} color={theme.colors.secondary} />
<SvgIcon variant="home" />
```
