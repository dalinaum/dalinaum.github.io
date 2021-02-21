---
layout: post
title:  "Jekyll 블로그 페이지네이션 설정"
date:   2021-02-11 00:20:00 +0900
categories: tool
---

점차 글을 적게 되면서 싱글 페이지 블로그가 부담이 되었다. 페이지네이션을 넣어 블로그를 운영하기로 결정.

먼저 `config.yml` 파일을 열었다.

```yml
paginate: 7
paginate_path: "/page/:num/"
```

하단에 보일 글을 7개로 설정했다. 10개는 너무 많고 5개는 너무 적은 것 같았다.

페이지 주소는 `/page/2/`의 형식이 되도록 설정했다. 기본은 `/blog/page:num/`를 권하고 있었으나 `/blog/page2/` 이런 형태는 좀 구려 보였다. `blog`도 맘에 안들고 `page2`도 별로다.

내 `index.html` 파일의 내용은 다음과 같다.

```html
---
layout: default
---

<div class="home">

  <h1 class="page-heading">Posts</h1>

  <ul class="post-list">
{% raw %}
    {% for post in site.posts %}
{% endraw %}
      <li>
{% raw %}
        <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span>
{% endraw %}

        <h2>
          <a class="post-link" href="[[ post.url | prepend: site.baseurl ]]">[[ post.title ]]</a>
        </h2>
      </li>
    [% endfor %]
  </ul>

  <p class="rss-subscribe">subscribe <a href="[[ "/feed.xml" | prepend: site.baseurl ]]">via RSS</a></p>

</div>
```

중괄호 `{}` 대신 대괄호 `[]`로 적었는데 소스코드에 적은 것도 Jekyll이 처리해서 적어두었다. 실제로 설정할 때는 중괄호로 적어야 한다.

여기에 `site.posts`를 쓰는 코드를 `paginator.posts` 기반으로 변경했다.

```html
{% raw %}
<ul class="post-list">
{% for post in paginator.posts %}
    <li>
    <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span>

    <h2>
        <a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
    </h2>
    </li>
{% endfor %}
{% endraw %}
</ul>
```

역시 중괄호 대신 대괄호로 적었다. 실제로는 다 중괄호로 바꾸자.

그 아래에 다음 블록을 추가하였다.

```html
[% if paginator.total_pages > 1 %]
<div class="pagination">
[% if paginator.previous_page %]
    <a href="[[ paginator.previous_page_path | relative_url ]]">&laquo; Prev</a>
[% else %]
    <span>&laquo; Prev</span>
[% endif %]

[% for page in (1..paginator.total_pages) %]
    [% if page == paginator.page %]
    <em>[[ page ]]</em>
    [% elsif page == 1 %]
    <a href="[[ '/' | relative_url ]]">[[ page ]]</a>
    [% else %]
    <a href="[[ site.paginate_path | relative_url | replace: ':num', page ]]">[[ page ]]</a>
    [% endif %]
[% endfor %]

[% if paginator.next_page %]
    <a href="[[ paginator.next_page_path | relative_url ]]">Next &raquo;</a>
[% else %]
    <span>Next &raquo;</span>
[% endif %]
</div>
[% endif %]
```

깃헙에 배포하니 제대로 나온다. 간단하네.

![페이지네이션 적용된 블로그]({{ "/assets/4-pagination.png" | absolute_url }}) 페이지네이션이 적용된 블로그
