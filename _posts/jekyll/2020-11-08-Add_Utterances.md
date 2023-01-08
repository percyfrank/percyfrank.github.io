---
title: "Minimal Mistakes Theme에 Utterances 댓글 기능 추가"
excerpt: "GitHub 계정으로 댓글을 달 수 있는 기능을 구현한다."
categories:
  - Jekyll
tags:
  - Jekyll
toc: true
toc_sticky: true
last_modified_at: 2020-11-08T14:05:00-05:00
---

## 1. Utterances

구글링을 하다 보면 GitHub 계정으로도 댓글을 달 수 있는 블로그들을 자주 볼 수 있다. Utterances라는 오픈 소스를 사용한 것인데, GitHub Issue 기반으로 Comment를 작성하게 해준다. 댓글에 마크다운 문법을 지원하며, 플랫폼을 이전해도 기존 댓글들을 그대로 가져갈 수 있다는 장점이 있다.

Jekyll에서는 Disqus 및 Facebook 등 다양한 댓글 라이브러리를 간편하게 사용할 수 있고, 나 역시 Disqus를 사용했다. 그런데 사람들이 댓글을 달려면 Disqus 회원가입 및 로그인 절차를 거쳐야 하기 때문에 다소 불편하다고 생각했다. Facebook도 좋은 대안이지만 비슷한 이유로 패스.

어차피 개발 블로그라서 대부분의 방문자가 GitHub 계정이 있을테니, Utterances가 가장 유용하다고 생각했다.

<br>

## 2. 설정

[App 설치하기](https://github.com/apps/utterances) 링크로 가서 GitHub App에 Utterances를 설치한다. 댓글이 등록되는 Issue들을 관리할 Repository에 권한을 제공해주면 된다.

**주의**할 점은, 해당 Repository가 반드시 Public이어야 한다. Private이면 타인이 댓글을 볼 수 없다. 나는 그냥 GitHub Page Repository에 권한을 제공했으나, GitHub Page가 Private이라면 별도의 Public Repository에 권한을 제공해야 한다.

> _config.yml

```yml
repository: xlffm3/xlffm3.github.io
# GitHub username/repo-name e.g. "mmistakes/minimal-mistakes"

...

comments:
  provider: "utterances"
  utterances:
    theme: "github-light" # "github-dark"
    issue_term: "pathname" #"url", "title", "og:title"...
```

repository 부분은 본인의 username과 repo-name으로 수정해준다. comments 부분의 provider는 위 예제처럼 똑같이 설정해주고, 밑의 theme과 issue_term은 기호에 맞게 설정하면 된다. issue_term은 블로그 포스팅과 댓글 관리 Issue를 맵핑하는 방식을 의미한다.  

[utterances](https://utteranc.es/) 페이지에서 다양한 theme과 Posting-Issue 맵핑 방식 및 설정 코드 등을 확인할 수 있다. Utterances는 pathname, url, title 등 총 6가지의 방식을 지원하는데, 각 방식마다 설정 코드가 살짝씩 상이하니 참조하길 바란다.

내 블로그는 흰색이기 때문에 light한 테마를 적용했으며, 포스팅의 pathname으로 Issue를 맵핑하는 방식을 선택했다.

<br>

## 3. 주의점

![image](https://user-images.githubusercontent.com/56240505/98458204-76cd9c80-21d1-11eb-9a36-a0936a39d524.png)

Minimal Mistakes Theme에서 포스팅할 때 파일 이름은 ```YYYY-MM-DD-{NAME}.md``` 형식을 따른다. PathName으로 포스팅과 Issue를 맵핑하기 때문에, Issue 이름은 ```디렉토리/{NAME}```가 된다.

만약 누군가가 포스팅에 댓글을 달아 Issue가 열린 상태에서 포스팅의 파일명(```{NAME}```)을 변경해버리면, Issue 맵핑이 끊어져 해당 포스팅에서 기존 댓글들을 더이상 참조할 수 없게 된다.

게시글의 URL이나 제목 등으로 맵핑하는 다른 방식들도 마찬가지이다. URL 및 제목이 변경되면 Issue 맵핑이 끊어지니, 댓글이 달린 상태에서 파일 수정할 때 주의를 기울이자.

<br>

---

## References

* [Minimal Mistakes Configuration](https://mmistakes.github.io/minimal-mistakes/docs/configuration/#utterances-comments)
