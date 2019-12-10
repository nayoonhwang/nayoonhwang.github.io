---
layout: post
title:  "string split"
date:   2019-12-10 08:22:22 +0900
categories: split golang
---

Json diff를 뜨고 "\n"으로 split을 했는데 자꾸 예상한 길이보다 1씩 길게 나오는것이었다.
go 내장함수인 strings.Split 요놈이 문제인가 해서 한번 테스트를 해봤다.
~~~go
hello := "안녕하세요\n 반갑습니다." + string(rune(10))
~~~
문제가 되는 문자열의 마지막 Rune이 U+000A, 즉 10이었다. 알고보니 이는 New line(개행문자)의 unicode중 하나였고, Split는 문제가 없었다. string 을 제대로 출력하고 테스트해보기 위해 rune을 활용하면 좋다.

> 참고1: http://pyrasis.com/book/GoForTheReallyImpatient/Unit45/02
golang은 모두 utf8 인코딩 기반이기 때문에, 한글이나 특수문자를 표현하기 위해 utf8 라이브러리 내의 여러가지 함수를 사용해보면 편하다.

다만 

~~~bash
hello := "안녕하세요\n반갑습니다\n"
fmt.Printf("%q", strings.Split(hello, "\n"))
//["안녕하세요", "반갑습니다", ""]
~~~

맨마지막에 저렇게 empty string이 나오는게 싫어서 저렇게 empty string을 제외하고 Split하는 방법에 대해 좀 찾아보니, 아래와 같은 함수를 찾았다. splitFn라는 함수에 separator 를 정의하고 해당 함수를 strings.FieldsFunc 라는 메소드에 파라미터로 넘기는 방식으로 Split을 수행하면 empty string이 무시된다.

~~~go
splitFn := func(c rune) bool {
        return c == '\n'
}
fmt.Printf("%q", strings.FieldsFunc(hello, splitFn))
//["안녕하세요" "반갑습니다"]
~~~