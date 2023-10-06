---
title: '펀잇의 SEO 개선하기'
date: 2023-10-06 16:00:00
category: 'frontend'
draft: false
---

구글에 펀잇 혹은 편의점을 검색했을 때 우리 서비스는 노출되지 않는다.. 🥺

검색으로 유입을 모을 수 있을 것이라고 생각되어 SEO 개선을 해보기로 했다.

<img width="1053" alt="스크린샷 2023-10-04 오후 4 08 22" src="https://github.com/woowacourse-teams/2023-fun-eat/assets/55427367/198120ec-057c-40a9-8be9-cf97e68e530a">

## 1. Google Search Console 등록하기

[구글 서치 콘솔](https://search.google.com/search-console/about)을 사용하면 얼마나 검색이 잘 되고 있는지 유입에 대한 정보를 얻음과 동시에 사이트맵 제출을 통해 SEO 개선을 도와준다고 한다. 이를 이용하기 위해서는 우리 서비스를 등록해야 한다.

<img width="1378" alt="스크린샷 2023-10-04 오후 4 45 14" src="https://github.com/woowacourse-teams/2023-fun-eat/assets/55427367/5c010d43-6b5b-4fab-845b-e483c37aad94">

서비스 URL을 등록하기 위해서는 소유권 확인 절차가 필요하다. (아무래도 아무나 서비스 정보에 접근하면 안되니까..)

<img width="782" alt="스크린샷 2023-10-04 오후 4 46 36" src="https://github.com/woowacourse-teams/2023-fun-eat/assets/55427367/6f1d9a50-140a-4696-8def-8aecea2f0560">

도메인을 통해 소유권 확인을 누르게 되면 이런 화면이 나오는데 도메인 제공 업체에 들어가서 해당 레코드를 DNS 설정에 복사하면 된다. (우리는 가비아를 통해 구입했기 때문에 가비아에 등록했다.)

이 설정을 하고 1~2분 후에 확인을 누르면 바로 인증이 완료된다.

<img width="1074" alt="스크린샷 2023-10-06 오후 3 55 57" src="https://github.com/woowacourse-teams/2023-fun-eat/assets/55427367/f1464e9f-4cf7-4549-afb9-697d61aebfa0">

시간이 좀 지나고 확인 하면 노출 수, 클릭수, 평균 게재순위 등을 확인할 수 있다.

<img width="302" alt="스크린샷 2023-10-06 오후 3 57 33" src="https://github.com/woowacourse-teams/2023-fun-eat/assets/55427367/53a05bf5-373c-46aa-8bac-efe0e8271a70">

하단의 표에서는 사용자가 어떤 검색어를 입력해서 노출이 몇 번 되었는지, 해당 키워드로 몇 번 들어왔는지도 확인할 수 있다.

## 2. sitemap 만들기

sitemap은 사이트에 있는 페이지, 동영상 및 기타 파일과 그 관계에 관한 정보를 제공하는 파일로 크롤봇은 해당 파일을 읽고 사이트를 더 효율적으로 크롤링 할 수 있게 된다. sitemap에서는 우리 서비스에서 중요하다고 생각되는 페이지나 파일을 알리는 것이 중요하다.

나는 이 [사이트](https://www.xml-sitemaps.com/)를 이용하여 기본적인 sitemap 구조를 만들었다. 그리고 `urlset` 태그 아래에 `url` 태그를 사용하여 우리 서비스에서 중요한 상품 목록이나 레시피 페이지를 포함해서 최종적인 xml 파일을 만들었다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset
  xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.sitemaps.org/schemas/sitemap/0.9
            http://www.sitemaps.org/schemas/sitemap/0.9/sitemap.xsd">
  <url>
    <loc>https://funeat.site</loc>
    <lastmod>2023-09-25T05:39:39+00:00</lastmod>
  </url>
  <url>
    <loc>https://funeat.site/products/food</loc>
    <lastmod>2023-09-25T05:39:39+00:00</lastmod>
  </url>
  <url>
    <loc>https://funeat.site/products/store</loc>
    <lastmod>2023-09-25T05:39:39+00:00</lastmod>
  </url>
  <url>
    <loc>https://funeat.site/recipes</loc>
    <lastmod>2023-09-25T05:39:39+00:00</lastmod>
  </url>
</urlset>
```

sitemap 작성이 끝나면 아까 등록했던 구글 서치 콘솔에 제출하면 된다.

<img width="946" alt="스크린샷 2023-10-06 오후 3 55 25" src="https://github.com/woowacourse-teams/2023-fun-eat/assets/55427367/9225b56b-8308-47ca-8470-3dc22f0d123c">

## 3. robots.txt 만들기

robots.txt 파일은 크롤러가 사이트에서 액세스할 수 있는 URL을 검색엔진 크롤러에 알려 준다.

`User-agent` 에는 허용할 크롤러 봇들의 목록을 작성할 수 있다. 와일드카드를 사용하여 모든 봇을 허용할 수도 있다.

`Allow` 와 `Disallow` 를 사용하여 크롤링을 허용할 페이지들을 설정할 수 있다.

`Sitemap` 을 통해 sitemap의 위치를 알려준다.

```
User-agent: *
Disallow: /404
Allow: /

# Host
Host: https://funeat.site/

# Sitemaps
Sitemap: https://funeat.site/sitemap.xml
```

## 4. index.html 파일에 메타 태그 사용하기

메타 태그는 웹페이지의 주요 정보를 담고있다. 크롤러봇은 메타태그에서 해당 사이트에 대한 추가적인 정보를 얻게 된다. 또한, 검색 페이지에서 당 서비스에 대한 소개를 간략하게 보여줄 수도 있다.

`title`, `description` , `og` 등의 메타 태그를 작성할 수 있다. 이 중에서 `og`는 오픈 그래프 태그를 의미하는데 링크를 공유할 때 아래와 같이 미리보기로 어떤 내용을 노출할지 결정한다.

<img width="266" alt="스크린샷 2023-10-04 오후 5 05 46" src="https://github.com/woowacourse-teams/2023-fun-eat/assets/55427367/4d21e992-8c22-4e74-8dde-ea2154f2eca9">

```html
<meta property="og:title" content="펀잇" />
```

작성할 때는 이렇게 property를 작성할 때 앞에 `og:` 를 붙여주면 된다.

펀잇은 [이렇게 메타태그가 작성되어 있다.](https://github.com/woowacourse-teams/2023-fun-eat/blob/develop/frontend/public/index.html)

## 정리

전에는 대충 사이트맵, robots.txt 작성 등 추상적으로만 알고 있었는데 어떤 역할을 하는지 알게 되니까 어떤 식으로 작성해야 할 지 더 쉽게 감이 잡혔다. 라이브러리 없이 리액트에서 SEO를 개선할 수 있는 방법을 알아 보았는데 만약 개선이 되지 않는다면 react-helmet 같은 라이브러리도 알아봐야 할 것 같다.

### Reference

https://fe-developers.kakaoent.com/2022/221208-basic-seo-guide/

https://developers.google.com/search/docs/crawling-indexing/sitemaps/overview?hl=ko

https://developers.google.com/search/docs/crawling-indexing/robots/create-robots-txt?hl=ko
