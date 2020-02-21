---
layout: post
title:  "프로젝트 마다 git config 다르게 설정하기"
date:   2020-01-08 16:22:22 +0900
categories: git
---

~~~bash
git config user.name=bla
git config user.password=blabla
~~~
의 기본 옵션은 local이다. 즉 각 프로젝트에 git 설정을 거는것이다.

이는 ${project_directory}/.git/config에서 확인할 수 있다.

반대로 
~~~bash
git config --global user.name=bla
~~~
이런식으로 --global 옵션을 주게 되면, 

~~~bash
remote: Permission to enjoymingles/enjoymingles.github.io.git denied to nayoonhwang.
fatal: unable to access 'https://github.com/enjoymingles/enjoymingles.github.io/': The requested URL returned error: 403
~~~

~~~bash
git remote set-url origin https://{UserId}:{Password}@github.com/enjoymingles/enjoymingles.github.io
~~~