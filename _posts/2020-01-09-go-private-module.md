---
layout: post
title:  "GOPRIVATE 설정하여 private repo 에 올려진 library 참조하기"
date:   2020-01-08 16:22:22 +0900
categories: git
---

# Background
Code Reusability, Deduplication의 측면에서 프로젝트마다 공통된 로직들(ex. error handler 또는 일부 middleware, util성 함수들)을 통합하고자 공용 private 라이브러리를 제작하였다. private repository다 보니, 일부 설정을 바꿔주거나, docker의 경우에는 credential을 추가해줘야되는 것들이 몇 가지 있다.

# Process
go.mod에 필요한 라이브러리 github.com/( userId )/( private repo ) v1.0.0 를 추가해보면,
~~~bash
$ go mod tidy
verifying github.com/sktaiflow/mls-api-common@v1.0.0: github.com/sktaiflow/mls-api-common@v1.0.0: reading https://sum.golang.org/lookup/github.com/sktaiflow/mls-api-common@v1.0.0: 410 Gone
~~~

이런 에러가 나온다. private repo 라이브러리를 참조하려고 하면, 기존에 go modules가 라이브러리를 땡겨오는 방식이랑 다르게 적용해야하기 때문이다.
이는 go1.13부터 적용된 GOPRIVATE 환경변수로 해결할 수 있다.
(참조: https://golang.org/cmd/go/#hdr-Module_configuration_for_non_public_modules)

~~~go
go env -w GOPRIVATE=github.com/sktaiflow/{repo_name}
~~~
이런식으로 go 환경변수를 설정해주면, 

> The GOPRIVATE environment variable controls which modules the go command considers to be private (not available publicly) and should therefore not use the proxy or checksum database.

위 공식문서에도 나와있듯, go 커맨드가 해당 레포지토리는 private으로 인식하고 기본 go proxy(proxy.golang.org)나 checksum 데이터베이스(sum.golang.org)를 사용하지 않도록 설정해준다. 

dockerfile에 적용하기

https://github.com/golang/go/issues/28680

- module cache

go mod 