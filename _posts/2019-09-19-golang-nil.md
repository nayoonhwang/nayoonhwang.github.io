---
layout: post
title: "golang nil check"
date: 2019-09-19
categories: development
tags: golang
---

# golang zero value
초기화값이 없는 상태로 선언된 변수들은 각 타입의 zero value값으로 초기화된다.

| **타입** | **값** |
|:--|:--|
| 정수(int) | 0 |
| 실수(float) | 0.0 |
| boolean | false |
| 문자열(string) | "" |
| 실수(float) | 0.0 |
| interface, slice, channel, map, pointer, function | 0.0 |

배열(array)이나 구조체(Struct)의 요소(element)들은 각 필드들이 zero value를 가진채 초기화된다.

~~~go
import "fmt"

type hello struct {
	s string
	i []int
	p *hello
}
func main() {
	var h hello
	fmt.Print(h)
}
// { [] <nil>}
~~~
hello라는 구조체를 정의했을때, 특정값 없이 선언하면, 위와 같이 zero 값이 들어가는걸 알 수 있다.