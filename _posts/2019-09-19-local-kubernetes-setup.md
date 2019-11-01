---
layout: post
title:  "mac local kubernetes setup"
date:   2019-09-19 16:00:22 +0900
categories: kubernetes
---

# Docker for Mac
Kubernetes를 이용해서 배포해야할때, 
항상 Cloud위에서 수행하는 것보다는 로컬에 테스트 환경을 구축하고 테스트해보는 것이 생산성에 도움이 될 때가 많다. 그리고 그렇게 어렵지 않다.

![docker desktop](http://collabnix.com/wp-content/uploads/2019/05/Screen-Shot-2019-05-11-at-6.17.51-PM.png)

docker desktop for mac을 설치하게 되면, Kubernetes가 활성화된 Docker를 사용할 수 있다. 활성화 되어있지 않다면 preference에 들어가서
enable만 해주면 된다.

~~~bash
kubectl get nodes
# NAME             STATUS   ROLES    AGE    VERSION
# docker-desktop   Ready    master   114m   v1.14.6
~~~
잘 출력된다.

# 간단한 서버로 k8s test해보기

~~~go
package main

import (
	"net/http"
)

func main() {
	http.HandleFunc("/hello", func(w http.ResponseWriter, req *http.Request) {
		w.Write([]byte("Hello World"))
	})

	http.ListenAndServe(":5000", nil)
}
~~~

hello 엔드포인트로 get 을 날리면 response로 Hello World를 출력하는 간단한 서버이다.
이 서버를 컨테이너화 시켜 배포하기 위해 dockerfile을 간단히 작성해본다.
~~~bash
FROM golang:latest

WORKDIR /app

COPY . .

RUN go build -o main .

EXPOSE 5000
CMD ["./main"]
~~~
이 도커파일을 image화 시켜 docker hub에 publish 하면 된다.
~~~bash
docker login # docker hub 인증
docker build .
docker tag {image_id} {username}/exercise:v1
docker push {username}/exercise:v1
~~~

이제 push 된 이미지를 땡겨와서 kubernetes로 배포하는 배포파일을 만들어본다.
~~~
apiVersion: apps/v1
kind: Deployment
metadata:
  name: exercise
  labels:
    app: exercise
spec:
  replicas: 1
  selector:
    matchLabels:
      app: exercise
  template:
    metadata:
      labels:
        app: exercise
    spec:
      terminationGracePeriodSeconds: 30
      containers:
      - name: exercise
        image: docker.io/{username}/exercise:v1
        imagePullPolicy: Always
        ports:
        - containerPort: 5000
~~~
replica는 1개로 해서 1개의 팟을 띄워 컨테이너를 올리는 Deployment 오브젝트를 만들어본다. 5000번 포트를 사용하고 있는 서버이기 때문에 container 의 포트를 5000으로 지정해준다.

> * terminationGracePeriodSeconds: 30 는 pod 종료 시에 graceful shutdown을 위해 30초의 grace period를 두도록 하는 옵션이다.
* imagePullPolicy는 로컬에 동일한 이름의 이미지가 존재해도, 항상 이미지를 pull하도록 하는 옵션이다.
이 외에도 여러가지 옵션을 테스트 해볼 수 있다.

```bash
kubernetes apply -f sample.yml
```
이제 pod이 의도한 대로 잘 생겼는지 확인해본다.

```bash
kubectl get pod
# NAME                       READY   STATUS    RESTARTS   AGE
# exercise-b97678cfd-mpppx   1/1     Running   0          9s
```

```bash
kubectl describe pod/exercise-b97678cfd-mpppx # 아까 조회한 pod의 name을 이용해서 pod의 자세한 리소스를 조회할 수 있다.
kubectl logs pod/exercise-b97678cfd-mpppx # 해당 팟에서 발생한 로그도 조회 할 수 있다.
```
describe를 통해 아래와 같은 이벤트 정보도 조회할 수 있다. 
```
Events:
  Type    Reason     Age   From                     Message
  ----    ------     ----  ----                     -------
  Normal  Scheduled  47s   default-scheduler        Successfully assigned default/exercise-b97678cfd-mpppx to docker-desktop
  Normal  Pulling    45s   kubelet, docker-desktop  Pulling image "docker.io/nayoonhwang/exercise:v1"
  Normal  Pulled     42s   kubelet, docker-desktop  Successfully pulled image "docker.io/nayoonhwang/exercise:v1"
  Normal  Created    42s   kubelet, docker-desktop  Created container exercise
  Normal  Started    41s   kubelet, docker-desktop  Started container exercise
```  

정상적으로 동작하는지 테스트해보기 위해서 Hello World 출력을 확인해본다. 컨테이너의 포트와 호스트의 포트를 포워딩해주기 위해(docker run 시에 -p 옵션을 통해 port binding 해주는 것과 같이) 아래와 같이 입력한다.

```bash
kubectl port-forward deployments/exercise 5000
```

exercise라는 이름의 deployment의 5000번 포트를 포워딩 해주는 커맨드이다.

```bash
curl localhost:5000/hello
# Hello World
```
컴퓨터에서 돌고있는 쿠버네티스 pod이 정상 동작임을 확인했다. 이제 다른 여러가지 테스트를 수행해볼수 있다~!
