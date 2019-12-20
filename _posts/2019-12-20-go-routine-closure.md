---
layout: post
title:  "클로저를 고루틴으로 실행할때 주의할점."
date:   2019-11-30 10:45:22 +0900
categories: goroutine closure
---

go routine 만들 떄 익명함수로 할때 마지막 변수갑스로 다 써서 개망할뻔

> Closure(클로저)는 함수 바깥에 있는 변수를 참조하는 함수값을 일컫는다. 함수가 선언될 때의 환경을 계속 유지하여 프로그램의 흐름을 변수에 저장하기 위해 사용한다. 클로저는 함수형 언어의 큰 특징이며, Go 는 클로저를 통해 함수형 언어의 기능을 구현하고 있다. 

참고: http://pyrasis.com/book/GoForTheReallyImpatient/Unit25

# 클로저를 고루틴으로 실행하기

반복문 내에 go routine를 삽입하여 concurrent한 Request를 날리는 로직을 작성하고 싶다고 하자.

처음엔 이런식으로 하면 될줄 알았다.
~~~go
package main
 
import (
    "fmt"
)
 
func main() {
    user := "helloworld"
 
    for i := 0; i < 100; i++ {
        go func() {
            http.Get("http://post.naver.com/%v/%v", user, i) // 반복문의 변수를 클로저에서 바로 사용
        }()
    }
}
~~~

*예상한 결과*
~~~go
http.Get("http://post.naver.com/helloworld/1")
http.Get("http://post.naver.com/helloworld/2")
http.Get("http://post.naver.com/helloworld/3")
...
http.Get("http://post.naver.com/helloworld/99")
~~~

*실제 결과*
~~~go
http.Get("http://post.naver.com/helloworld/100")
http.Get("http://post.naver.com/helloworld/100")
http.Get("http://post.naver.com/helloworld/100")
...
http.Get("http://post.naver.com/helloworld/100")
~~~

그렇게 되는 이유는,

클로저를 고루틴으로 사용할때는 반복문이 끝난 뒤에 고루틴이 실행되기 때문이다. 따라서, 고루틴이 생성된 시점의 변수 i의 값은 100이다. 따라서 모두 
~~~go
http.Get("http://post.naver.com/helloworld/100")
~~~
값으로 요청이 날라가게 되는것이다. 따라서 클로저를 고루틴으로 실행할때 반복문에 의해 바뀌는 변수는 반드시 파라미터로 넘겨줘야한다. 파라미터로 넘겨주는 시점에 해당 변수의 값이 복사되므로, 고루틴이 생성될때 그대로 사용할 수 있다. 

~~~go
package main
 
import (
    "fmt"
)
 
func main() {
    user := "helloworld"
 
    for i := 0; i < 100; i++ {
        go func(n int) {          // 익명 함수를 고루틴으로 실행(클로저)
            http.Get("http://post.naver.com/%v/%v", user, n)
        }(i)                      // 반복문의 변수는 매개변수로 넘겨줌
    }
}
~~~