---
layout: post
title:  "HTTP Request 를 보낼 때 주의해야할 사항 - 2"
date:   2020-04-22 20:22:22 +0900
categories: golang http
---

몇십만건의 HTTP 요청을 날려야하는 요구사항이 있다고하자.

대부분의 요청은 정상적으로 수행될 것이다. 하지만 Network issue가 발생하여 갑자기 연결이 끊어진다던지등의 이슈는 충분히 발생가능하다. 따라서 Retry logic을 미리 세워두는 것이 좋다.

Python 의 경우엔 [잘 설명된 블로그](https://www.peterbe.com/plog/best-practice-with-retries-with-requests)와 [공식문서](https://urllib3.readthedocs.io/en/latest/reference/urllib3.util.html) 에 따르면,
~~~python
from requests.packages.urllib3.util.retry import Retry

retry = Retry(
    total=retries,
    read=retries,
    connect=retries,
    backoff_factor=backoff_factor,
    status_forcelist=status_forcelist,
)
~~~

urllib 내에 Retry라는 객체가 존재해서, Total(총 Retry 횟수), Read, Connect(특정 에러에 몇번 Retry하도록 할것인지), method_whitelist, status_forcelist (Retry하도록 강제하는 케이스들) 등 여러가지 parameter를 전달할 수 있게 하였다.

반면, Go는 어떠할까?

~~~go
package main

import (
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
)

func main() {
	var (
		err      error
		response *http.Response
		retries  int = 3
	)
	for retries > 0 {
		response, err = http.Get("https://non-existent")
		if err != nil {
			log.Println(err)
			retries -= 1
		} else {
			break
		}
	}
	if response != nil {
		defer response.Body.Close()
		data, err := ioutil.ReadAll(response.Body)
		if err != nil {
			log.Fatal(err)
		}
		fmt.Printf("data = %s\n", data)
	}
}
~~~

GO는 조금 불편하지만, 이런식으로 Retry 횟수와 Retry logic을 명확히 명세하여 프로그램을 돌려주어야 한다.
