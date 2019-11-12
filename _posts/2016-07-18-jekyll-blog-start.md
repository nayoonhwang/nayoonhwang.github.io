---
layout: post
title: "jekyll blog를 처음 시작하며"
date: 2016-07-19
categories: development blog
tags: home work coding matilda38.github.io
---

지킬 블로그는 사실 정말 좋습니다. 사실 정적(static) 블로그에 대한 개념도 정확하지 않고 무슨 소린지는 차차 알아나가야 하겠지만,

일단 1. github pages에서 무료 호스팅을 제공한다는 점. 2. 개발블로그로써 조금 더 적합하다는 점 에서 다른 블로그보다 지킬을 쓰게 되는 것같습니다.

이 블로그를 시작하면서 워낙 컴퓨터 전공자지만 개발에는 입문 수준이라 삽질을 여러번 하였기에 설치 과정을 포스팅 할까 하였으나 저보다 훨씬 더 잘 설명하시는 블로그(nolboo님의 블로그를 추천합니다)가 많기에 기본적 설치는 포스팅하지 않겠습니다. 단지 간단히 설명하자면 jekyll을 설치하고 github_ID.github.io라는 이름으로 프로젝트/repository를 동시에 만든 후에 master branch로 커밋, push 완료 후 조금(생각보다 오래 걸릴수도 있습니다) 기다리시면 완성되어 있을 것입니다. 사실 매우 간단한데 이 과정에서도 저처럼 삽질을 하는 사람이 있더라구요.

개인적으로 힘들었던 부분인 comment 추가 부분에 대해서 말씀드리겠습니다. 이것도 사실 엄청 간단한데 ㅠㅠ 한국어 블로그들 보면서 삽질하다가 결국 영어 블로그 하나로 5분만에 해결했는데요.

댓글 시스템은 두가지가 가능합니다. 1. facebook 2. disqus 1번의 단점은 facebook 로그인을 해야한다는것...그래서 어쩔수 없이 disqus에 가입 한후 install 절차를 수행하였습니다.

universal code를 클릭하면 제공되는 코드를 이용하였는데요. comments.html를 includes 폴더 안에 넣어 두었지요. 물론 comments.html 에는 해당 universal code가 들어있습니다.

일단 config.yml에서 disqus_shortname=제 disqus 계정의 username 을 쓰고
post.html에

{% highlight html linenos=table%}
{% raw %}
{% if site.disqus_shortname %}
  {% include comments.html %}
{% endif %}
{% endraw %}
{% endhighlight %}

이런식으로 추가해주었습니다. 몇시간을 삽질 하던게 이거 한방에 해결되었습니다. ㅠㅠㅠㅠㅠㅠㅠㅠ

아무튼 지킬 화이팅~~.


--추가 (2016.08.15 광복절)

google search console에 내 사이트를 등록할 수 있습니다. 안내하는 몇가지 절차를 걸치면 구글에 제 블로그를 검색했을 때 제대로 검색되게 만들 수 있습니다.

등록을 완료하면 메일도 오구요!

최근에 Search Console에서 사이트를 확인(또는 사이트를 자동으로 확인하는 Blogger나 Google 사이트 도구에서 사이트 생성)한 것으로 Google 시스템에 표시됩니다.

console에 사이트가 등록되면 추가적으로 sitemap을 등록할 수 도 있습니다. sitemap.xml을 등록하면 되는데 jekyll의 경우에는 gem이 존재합니다. 따라서 자동으로 생성해주고, 제 경우에는 sitemap error가 떠서 살펴봤더니 xml의 url에 https://가 빠져있었기 때문이었습니다. 주의하시길 바랍니다!

이제 검색이 되네요! >ㅇ< 밑에는 sitemap.xml에 등록한 내용이 출력되는걸 보실수 있습니다.
