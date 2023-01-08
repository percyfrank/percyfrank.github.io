---
title: "Minimal Mistakes Theme 커스터마이징 관련 유용한 참고 사이트 모음"
excerpt: "Minimal Mistakes Theme 커스터마이징 관련 유용한 참고 사이트들을 정리한다."
categories:
  - Jekyll
tags:
  - Jekyll
date: 2021-01-16
last_modified_at: 2021-01-16
---

## 1. Minimal Mistakes 커스터마이징

Minimal Mistakes 커스터마이징은 크게 두 가지로 분류할 수 있습니다.

* Configuration 파일 설정을 통해 테마가 지원하는 공식 커스터마이징 기능 사용하기.
* HTML, SCSS, Liquid 등을 직접 조작하여 사용자 취향껏 디자인 수정하기.

프론트 개발이 친숙하거나 이전에 Jekyll, Hexo, Hugo 등을 사용해본 경험이 있으시다면 어렵지 않게 정적 블로그를 꾸밀 수 있습니다. 이번 포스팅에서는 Minimal Mistakes 커스터마이징할 때 유용한 참고 사이트들을 정리해보았습니다. 해당 사이트들을 통해 Jekyll 메커니즘(Minimal Mistakes 테마의 디렉터리 구조와 HTML, SCSS, Liquid 등)을 이해한다면, 더욱 개성있는 나만의 블로그를 만드실 수 있습니다. 🙆‍♂️

<br>

## 2. Quick-Start Guide

[Quick-Start Guide](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/)는 Minimal Mistakes 공식 레퍼런스입니다. Minimal Mistakes 테마가 제공하는 공식 기능들에 대한 Configuration 파일 설정 방법이 잘 기술되어 있습니다. 해당 문서만 잘 참고하셔도 블로그 커스터마이징 기능 대부분을 사용할 수 있습니다.

또한 해당 사이트는 Minimal Mistakes 테마를 이용해서 제작되었습니다. Minimal Mistakes 레포지토리의 docs 디렉토리에 위치한 파일들이 실제 사이트를 구성하고 있는 파일들입니다.

![image](https://user-images.githubusercontent.com/56240505/104830351-75031200-58c1-11eb-91ed-36311ee70634.png)

Quick-Start Guide 문서 외에도, 제작자가 해당 사이트에 다양한 종류의 커스터마이징을 적용한 샘플 포스팅을 튜토리얼 느낌으로 많이 올려두었습니다. 커스터마이징을 적용했을 때의 모습을 사진으로 확인해보고, 마음에 들면 docs 디렉토리 내부의 파일 및 설정 등을 참고하여 커스터마이징에 적용하시면 됩니다.

![image](https://user-images.githubusercontent.com/56240505/104831210-0ecebd00-58ca-11eb-9e42-581d900b2724.png)

* 헤더 및 티저 이미지를 포스팅에 삽입하는 커스터마이징 기능들을 비교할 때 많은 도움이 되었습니다.

<br>

## 3. GitHub Pages 블로그 따라하기 시리즈

[GitHub Pages 블로그 따라하기 시리즈](https://devinlife.com/howto/)는 총 17개의 글로 구성되어 있습니다. 사람들이 자주 사용하는 커스터마이징 기능들 중 가장 실용적인 것들을 많이 다루고 있습니다.

Disqus 댓글, Google 검색 엔진, 애드 센스 광고 등 다양한 기능을 어떻게 등록하는지 사진을 통해 회원가입부터 차근차근 알려줍니다. 또한 HTML, SCSS, Liquid 등을 직접 조작하여 폰트를 비롯한 디자인 영역을 수정하는 방법도 설명하고 있습니다. 이 또한 사진과 Commit 코드를 제공하니 쉽게 따라하실 수 있습니다.

특히 각 게시물의 댓글들에도 유용한 정보들(에러 해결법, 다른 방식의 커스터마이징 등)이 많이 있으니 잘 참고하시길 바랍니다.

* 저는 댓글을 통해 TOC에 포함되는 Header 크기 범위 설정 방법을 배웠습니다.

Quick-Start Guide가 영문으로 작성되어 있어 읽기 불편하거나, 양이 너무 방대해서 내가 원하는 커스터마이징 정보를 찾기 어려운 사람들에게 추천합니다.

<br>

## 4. Minimal-Mistakes 테마의 디렉터리 구조

**2절**과 **3절**의 사이트들만 참고해도 어지간한 커스터마이징 기능들을 다 사용할 수 있지만, 블로그 디자인을 극한으로 마개조하고 싶다면 [[Github 블로그] Minimal-Mistakes 테마의 디렉터리 구조](https://ansohxxn.github.io/blog/jekyll-directory-structure/)를 참조하시길 바랍니다.

Jekyll과 Minimal Mistakes 테마의 디렉토리 구조 등을 잘 정리해놓은 글인데, 몰랐던 커스터마이징 기능들과 더불어서 디자인에 대한 인사이트를 많이 배울 수 있었습니다.

* Grid, Collection Layout 등.

![image](https://user-images.githubusercontent.com/56240505/104799528-0ebfb600-5813-11eb-86c4-46828133dbb7.png)

여담으로, 해당 포스팅의 블로그는 제가 본 Minimal Mistakes 블로그들 중 디자인 커스터마이징이 가장 멋있었습니다. SCSS와 Liquid를 많이 공부하시고 다채롭게 수정한것 같은데, 열정에 감탄하면서 블로그를 구경했습니다. 🤩

만약 커스터마이징을 하는데 SCSS와 Liquid 조작이 어렵다면, 해당 블로그 제작자님의 GitHub에 있는 정적 블로그 Repository의 코드를 참고/비교하며 연습해보시길 추천드립니다.

<br>

## 5. Travis-Ci를 통한 Algolia 검색 도구 적용

Minimal Mistakes는 여러가지 검색 도구를 지원합니다. 기본으로 제공되는 검색 도구는 뭔가 검색이 잘 안되는 것 같아, [깃블로그에 검색 기능을 추가해보도록 하자.](https://xinfolab.github.io/blog/blog-maker-4/) 글을 참조하며 Aloglia 검색 도구를 등록했습니다.

버전 차이로 인한 에러 해결법이 잘 명시되어 있습니다.

<br>

## 6. 끝으로

하고자 하는 커스터마이징에 관련된 키워드를 Minimal Mistakes 키워드와 함께 구글링하면, closed된 비슷한 과거 Issues를 쉽게 찾아볼 수 있습니다. Minimal Mistakes 제작자가 대부분 답변을 달기 때문에 커스터마이징할 때 많은 도움이 되었습니다.

* [Include "Back to top" Icon #1731](https://github.com/mmistakes/minimal-mistakes/issues/1731)
* [TOC - specify depth](https://github.com/mmistakes/minimal-mistakes/issues/108)

HTML, SCSS, Liquid, Minimal Mistakes의 디렉토리 구조 등을 파악하고 커스터마이징 연습하는데 **Chrome 개발자 도구**가 정말 큰 도움이 됩니다.

![image](https://user-images.githubusercontent.com/56240505/104806582-07e97100-581c-11eb-9104-405fcb4b5188.png)

> 오른쪽에 위치한 TOC를 어떻게 왼쪽 Side Profile 아래로 위치시킬까?

![image](https://user-images.githubusercontent.com/56240505/104806192-08343d00-5819-11eb-872c-efdf4f04324c.png)

> 개발자 도구를 통해 클래스 등을 알아내고.

![image](https://user-images.githubusercontent.com/56240505/104806203-27cb6580-5819-11eb-9dde-ce1ac427a407.png)

> 매치되는 부분을 찾아서 HTML, SCSS, Liquid를 수정해주면.

![image](https://user-images.githubusercontent.com/56240505/104806215-4b8eab80-5819-11eb-8795-af8704d964e2.png)

> 완성!

저는 미니멈하고 심플한 블로그를 선호해서 그렇게 많이 커스터마이징 하지는 않았습니다. 그래도 커스터마이징을 공부하면서 프론트 개발과 디자인에 대해 흥미를 가질 수 있었던 좋은 계기였습니다. 😄

<br>

---
