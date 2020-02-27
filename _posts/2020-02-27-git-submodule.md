---
layout: post
title:  "Git submodule"
date:   2020-02-27 12:00:22 +0900
categories: git submodule
---

내가 사용하고자(clone) 하는 저장소가 submodule을 포함하고 있는 저장소일때,

일단,
~~~bash
git clone {url}
~~~
만 하게되면 submodule 내부의 코드까지는 당겨와지지 않는다.

submodule까지 모두 pull 하고 싶을때에는 clone 직후에
~~~bash
git submodule update --init --recursive
~~~
를 입력하면 submodule까지 모두 당겨오게 된다. --init tag에서도 알수있듯이 이는 submodule을 가장 처음 당겨올때 사용하는 명령어이고,

한번 당겨온 이후 submodule의 변경사항을 pull하려면,
~~~bash
git submodule update --recursive --remote
~~~
또는
~~~bash
git pull --recurse-submodules
~~~
를 해주면 된다. 처음에 --init 태그가 달린 명령어를 입력하지않으면 아래 두 명령어는 동작하지 않으므로 주의하도록 한다.

참고: http://openmetric.org/til/programming/git-pull-with-submodule/