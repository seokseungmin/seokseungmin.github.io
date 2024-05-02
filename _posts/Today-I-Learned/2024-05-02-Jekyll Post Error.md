---
title: "Jekyll Post Error"
excerpt: "[TIL] Jekyll Post Error"

categories:
  - Today I Learned
tags:
  - [Today I Learned]

permalink: /categories/JekyllPostError/ # url

toc: true
toc_sticky: true

date: 2024-05-02
last_modified_at: 2024-05-02
---

해결 방법
---

다음의 내용들은 기본적으로 지켜야 하는 것이다.<br>
YEAR-MONTH-DAY-title.md 파일 제목 형식을 확인.<br>
포스팅 날짜 맞게 입력했는지 확인.<br>
_posts 폴더에 맞게 위치해 있는지 확인.<br>
카테고리 맞게 입력 했는지, 해당 카테고리 존재하는지 확인.<br>

첫번째 해결방법
---

포스팅 날짜 입력시 본인이 작성하는 날 기준으로 하루 전으로 해보기.<br>
시간이 utc 기준인듯 하다는 댓글이 있었음.<br>
이것으로 본인도 포스팅 문제 해결 하였음. <br>

두번째 해결방법
---

_config.yml 에 future: true 와 페이지 옵션(타이틀, 카테고리 적는곳)에 published :true 를 추가하기.<br>

또 다른 오류발생
---

```
Warning: The github-pages gem can't satisfy your Gemfile's dependencies.
If you want to use a different Jekyll version or need additional dependencies, consider building Jekyll site with GitHub Actions: https://jekyllrb.com/docs/continuous-integration/github-actions/
To use retry middleware with Faraday v2.0+, install `faraday-retry` gem
  Liquid Exception: Liquid syntax error (line 232): Variable '{ {1, 2, 3}' was not properly terminated with regexp: /\}\}/ in /github/workspace/_posts/Java/2024-05-02-연습문제풀이-3.md
/usr/local/bundle/gems/liquid-4.0.4/lib/liquid/block_body.rb:136:in `raise_missing_variable_terminator': Liquid syntax error (line 232): Variable '{ {1, 2, 3}' was not properly terminated with regexp: /\\}\\}/ (Liquid::SyntaxError)
```

오류 발생 원인
---

마크다운 본문 내용에 중괄호 ({) 가 연속으로 2개 들어가면 Jekyll은 텍스트가 아닌 변수로 인식해서 발생하는 오류다.<br>
템플릿 언어 (Template Language)<br>
Jekyll 은 Liquid 라는 템플릿 언어(template language)를 사용한다.<br>
템플릿 언어는 중괄호(curly brace)를 사용해서 변수, 제어문을 표현한다.<br>

```
<!-- 변수 표현 -->
`{ { variable } }`

<!-- 제어문 표현 -->
{% raw %}

{% if user %}
	Hello, { { user.name } }!
{% endif %}
```

오류 상세 분석
---

{ {a1}, {a1,a2}... 에서 중괄호 2개로 시작해서 변수를 인식할 준비를 했으나, 이어지는 값들이 정상적인 변수의 형태로 입력되지 않아 변수를 정상적으로 찾지 못해 오류가 발생했다.<br>

오류 해결 방법
---

Jekyll 에서는 다양한 템플릿 태그를 지원하는데, 그 중에서 중괄호를 그대로 표현해주는 raw 태그가 있다.<br>
중괄호를 그대로 표현하고 싶은 텍스트 범위에 {퍼센트_기호 raw 퍼센트_기호} 중괄호가 포함된 텍스트 {퍼센트_기호 endraw 퍼센트_기호} 로 감싼다.<br>
퍼센트_기호는 % 로 바꿔쓰면 된다.<br>
다음과 같이 작성하면 된다.<br>

```
{퍼센트_기호 raw 퍼센트_기호}  `{ {1}, {1,2,3}, {1,2} }` 이 있다면 `{1}`, `{1,2,3}`, `{1,2}`  {퍼센트_기호 endraw 퍼센트_기호}
```

**참고 블로그:**
- [해결 깃허브 Pages 블로그 만들기 중 Posts 파일 업로드 안됨](https://velog.io/@jurije/%ED%95%B4%EA%B2%B0-%EA%B9%83%ED%97%88%EB%B8%8C-pages-%EB%B8%94%EB%A1%9C%EA%B7%B8-%EB%A7%8C%EB%93%A4%EA%B8%B0-%EC%A4%91-posts-%ED%8C%8C%EC%9D%BC-%EC%97%85%EB%A1%9C%EB%93%9C-%EC%95%88%EB%90%A8)
- [Github 블로그가 표시되지 않는 문제 해결](https://sehooni.github.io/blog/github_blog_not_shown/)
- [Jekyll에서 발생하는 Liquid 구문 오류(중괄호)](https://han-joon-hyeok.github.io/posts/jekyll-liquid-syntax-error-curly-braces/)

**참고 하면 좋을 블로그:**
- [Pi's Lab](https://pi-314.tistory.com/196)

**참고 자료:**
- [stackoverflow](https://stackoverflow.com/questions/24102498/escaping-double-curly-braces-inside-a-markdown-code-block-in-jekyll)
- [jekyllrb](https://jekyllrb.com/docs/liquid/)
