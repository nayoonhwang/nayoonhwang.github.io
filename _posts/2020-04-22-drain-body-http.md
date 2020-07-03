---
layout: post
title:  "HTTP Request 를 보낼 때 주의해야할 사항"
date:   2020-04-22 14:22:22 +0900
categories: golang http
---


공식 [http/#Client.Do](https://golang.org/pkg/net/http/#Client.Do) 의 문서에 따르면, 

> Do는 client 에서 요청한 정책(redirect, cookies, auth)을 따라 HTTP 요청을 보내고 HTTP 응답을 받는다. 

> client 정책이나, HTTP 연결 문제로 에러는 발생한다. 200대 status code가 아니라고 에러가 발생하진않는다.

> Return 된 에러가 nil일때, Response 는 nil이 아닌 body를 가지고, 이는 user측에서 닫아야한다. Body가 EOF까지 읽어지거나 닫히지않았을때 client의 밑단 RoundTripper(보통 Transport)는 keep-alive connection을 사용하지 못할 것이다.

이것때문에 영문법(not both)가 이중부정인지 부분부정인지까지 알아봤는데, not both는 부분부정이다. 따라서 EOF까지 읽어지거나 닫히거나 둘중 하나를 하면 keep-alive connection을 사용할 수 있다는 말이다.

> Request Body가 non-nil이면 에러가 발생한 경우에라도, 밑단 Transport에 의해 닫힐것이다. 에러가 발생하면, 어떠한 Response도 무조건 무시된다. 

이는 에러가 발생하면 무시될것이기 때문에 response를 굳이 읽지않아도 된다는 말이다.

저 말에 따라서, 원래 짰던 코드를 수정했다.

~~~go
client := http.DefaultClient
_, err := client.Do(req)
if err != nil {
    return nil, err
}
~~~
이런식으로 request만 날리고 response는 확인하지 않은채 넘어가려했었다. 하지만 문서에 따르면, 정상적으로 request를 송신했을때(에러가 발생하지 않았을때) response는 nil이 아닌 body를 가지기 때문에 user측에서 무조건 닫아야한다. 닫지 않으면 connection이 닫히지않아서 재사용할 수 없다.

그래서 이런식으로 수정했다.
~~~go
client := http.DefaultClient
resp, err := client.Do(req)
if err != nil {
    return nil, err
}
defer resp.Body.Close()
~~~
그런데, 실제로 이런식으로 짠것보단,

~~~go
client := http.DefaultClient
resp, err := client.Do(req)
if err != nil {
    return nil, err
}
io.Copy(ioutil.Discard, res.Body)
defer resp.Body.Close()
~~~
이렇게 짠게 훨씬 빨랐다.

~~~bash
~/github.com/templogs
❯ go run main.go
0 2020-04-22 15:27:35.835305 +0900 KST m=+0.000665513
100 2020-04-22 15:27:40.331843 +0900 KST m=+4.496988035
200 2020-04-22 15:27:44.774382 +0900 KST m=+8.939338429

~/github.com/templogs
❯ go run main.go
0 2020-04-22 15:27:51.634151 +0900 KST m=+0.000721815
100 2020-04-22 15:27:52.90517 +0900 KST m=+1.271697137
200 2020-04-22 15:27:54.049897 +0900 KST m=+2.416385286
~~~
보다시피 5초 -> 1초 때로 확 줄었다. 아마도 Connection 재사용 때문에 latency가 줄었을 것이라 추정되는데,
공식 문서를 따라했는데도(!) 이런결과가 나와서 살짝 당황스러웠다.

결론적으로는, 에러가 발생하지 않았으면, response body도 닫고 EOF까지 읽기도해줘야 커넥션을 잘 재사용할 수 있다. 최적화에 일부 도움이 될수 있을 듯하다.