---
layout: post
title:  "Kubernetes Local Setup"
categories: Kubernetes
---

# Kubernetes 에서 Deployment

Kubernetes로 작업할 때 사용하는 것은 Kubectl 이라는 kubernetes 용 cli가 있다. 클러스터를 관리할 때 사용하는데, 쿠버네티스 클러스터 위에 application 배포시 유용하게 사용된다.

[Kubectl Tutorial](https://kubectl.docs.kubernetes.io/pages/kubectl_book/getting_started.html) 위 링크에 친절하게 설명된 튜토리얼이 있어 따라해보면 좋다.

쿠버네티스에는 Object라는 개념이 있는데, 예를 들어 Application 배포 시에도 이런 배포용 Object를 만들어 사용하는 것이다. 이 Object는 Yaml file 형식으로 표현할 수 있다.

아래는 예시이다.
~~~
apiVersion: apps/v1 # apps/v1beta2를 사용하는 1.9.0보다 더 이전의 버전용
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # 템플릿에 매칭되는 파드 2개를 구동하는 디플로이먼트임
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
~~~
모든 쿠버네티스 오브젝트는 오브젝트의 구성을 결정하는 두 개의 필드 spec status 를 가진다. spec은 우리가 만들 오브젝트가 가져야하는 조건을 의미한다. Kubernetes는 제공된 spec을 기준으로 Object를 생성한다. 
status 는 오브젝트의 실제 상태를 나타내고, Kubernetes 시스템에 의해 업데이트 된다. Kubernetes는 오브젝트의 실제 상태(status)를 spec에 일치시키기 위해 능동적으로 관리한다. 
예를 들어 위 yaml을 기준으로 object를 생성 했을때 spec에 정의된 replicas: 2 에 의해 2개의 인스턴스를 생성하여 컨테이너를 띄운다. 
만약 그 중 1개에서 Fail 이 발생한다면, Kubernetes 시스템에서 자체적으로 대체 인스턴스를 생성하여 spec과 status 간의 차이를 제거하려할 것이다.

Pod은 Kubernetes의 가장 작은 개념이자 핵심 개념이다. 한개의 팟은 Application의 한 인스턴스로 동등하다. 그리고 이 인스턴스는 보통 한 개의 컨테이너다. 
즉, 한 pod은 한 컨테이너가 동작하는 한개의 인스턴스이다. 물론 여러개 컨테이너가 올라갈 수도 있지만, 난 간단히 한 pod에 한 컨테이너라고 이해하고 진행했다. 

## Object 생성하기

