---
layout: post
title:  "Kubernetes Deployment 삽질!"
date:   2019-09-18 16:22:22 +0900
categories: Kubernetes Deployment
---

# Kubernetes Deployment

Kubernetes로 작업할 때 사용하는 것은 Kubectl 이라는 kubernetes 용 cli가 있다. 클러스터를 관리할 때 사용하는데, 쿠버네티스 클러스터 위에 application 배포시 유용하게 사용된다.

[Kubectl Tutorial](https://kubectl.docs.kubernetes.io/pages/kubectl_book/getting_started.html) 위 링크에 친절하게 설명된 튜토리얼이 있어 따라해보면 좋다.

쿠버네티스에는 Object라는 개념이 있는데, 예를 들어 Application 배포 시에도 이런 배포용 Object를 만들어 사용하는 것이다. 이 Object는 Yaml file 형식으로 표현할 수 있다.

아래는 예시이다.
~~~yaml
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

Pod은 Kubernetes의 가장 작은 개념이자 핵심 개념이다. 한개의 팟은 Application의 한 인스턴스로 동등하다. 그리고 이 인스턴스는 보통 한 개의 컨테이너다. 팟 하나에 하나의 ip 주소를 가지고 있다. 
즉, 한 pod은 한 컨테이너가 동작하는 한개의 인스턴스이다. 물론 여러개 컨테이너가 올라갈 수도 있지만, 난 간단히 한 pod에 한 컨테이너라고 이해하고 진행했다. 팟은 보통 직접만들기 보다는, 싱글톤(Singleton) 이라도 대부분 위와 같이 Deployment와 같은 컨트롤러를 사용한다. 컨트롤러는 클러스터 범위에서 복제와 롤아웃(Rollout) 관리 뿐만 아니라 자가치료 기능도 제공한다. 

[참고](https://kubernetes.io/ko/docs/concepts/workloads/pods/pod/)

## Object 생성하기

~~~bash
kubectl apply -f config.yml
~~~
yaml file 에 정의한 바를 기반으로 deployment/service를 생성할 수 있다. 다른 방식으로도 생성할 수 있지만, 이렇게 생성하는 것을 권장하고 있다.

~~~bash
kubectl create -f config.yml
~~~
이런식으로 생성해도 되지만, apply를 해도 만약 해당 이름의 deployment가 존재하지 않을 경우 생성해주기 때문에 create를 굳이 쓸필요가없다.

환경변수 설정시 등에서 활용할 수 있는 Configmap의 예시를 한번 들어보면, 아래와 같은 식으로 special-config라는 이름의 Configmap을 정의한 후에 

~~~yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
~~~
apply를 통해 만들어준다.
~~~bash
kubectl apply -f config.yml
# configmap/special-config created
kubectl get configmaps
# NAME             DATA   AGE
# special-config   2      8s
~~~
만들어준 configmap은 pod 생성시에 아래와 같은 식으로 참조하여 사용할 수 있다.
~~~yaml
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      envFrom:
      - configMapRef:
	  		name: special-config
~~~

~~~bash
envFrom:
- configMapRef:
	name: special-config
~~~
이렇게 정의하면 special-config내에 data에 정의한 모든 변수들을 container의 환경변수로 사용할 수 있다.
~~~bash
env:
- name: SPECIAL_LEVEL_KEY
  valueFrom:
    configMapKeyRef:
      name: special-config
      key: SPECIAL_LEVEL
~~~
이렇게 envFrom와 configMapRef 키워드 대신 env와 configMapKeyRef로 정의하면 speical-config configmap에서 원하는 변수를 정의해서 쓸 수 있다. 아무래도 위에 있는 envFrom 이 좀 더 쉽게 사용할 수 있는 방법이다.

## Deploy Updated Application

그렇다면 이미 돌아가고 있는 Instance을 업데이트 해야할 땐 어떻게 해야할까. 예를 들어 application에 변동 사항이 생겼고, source code에 변동이 생겼으니, build artifact가 달라졌을것이다. release나 master 같은 경우엔 당연히 build-number/ release-number를 다르게 해서 image에 tag를 달아주고 해당 image를 참조하여 build 하도록 하면 config.yml에 매번 변동이 생기기 때문에 pod를 rolling update 한다. 하지만 develop같은 경우엔 어떻게 될까. develop같은 경우엔 변동사항이 많고 굳이 release-number / build-number를 관리하지 않는다. 

[stackoverflow 에 많은 질문들](https://stackoverflow.com/questions/40366192/kubernetes-how-to-make-deployment-to-update-image)이 올라와있다. [공식 Kubernetes 가이드](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/)등을 참조해봤을 때, 
~~~
kubernetes set image deployments/{my_deployment} {container_name}={image_repo_url}:{tag_name}
~~~
요 set 커맨드를 이용해서 뭔가를 update를 줘서 kubernetes가 rolling update를 수행할 수 있게 하라는 답변들이 가장 많았다. 일단 rolling update를 수행하게 되면,

> imagePullPolicy: Always

요 imagePullPolicy가 Always로 설정되어있을때, 항상 image를 repository에서 pull 땡겨오기 때문에 같은 tag를 가진 이미지라도 업데이트된 이미지가 존재할때 해당 업데이트를 받아올수 있다.

사실 Rolling Restart 라는 개념을 이때 사용하면 딱 괜찮은데, [이슈](https://github.com/kubernetes/kubernetes/issues/13488)가 이미 존재했고, 1.15 버전부터는
~~~
kubectl rollout restart {pod}
~~~
라는 커맨드가 생겨서 한번에 해결할 수 있다.

하지만 우리가 사용하고 있는 버전이 1.14여서 해당 커맨드를 사용할 수 없는데, 이땐 각종 야매 방법을 써야한다

[야매1](https://techoverflow.net/2019/04/02/how-to-force-restarting-all-pods-in-a-kubernetes-deployment/) 뭐 이런 방법이 있을 수 있고, tag가 latest일 경우에는,
~~~
kubectl set image deployments/exercise exercise=docker.io/{username}/exercise:latest
kubectl set image deployments/exercise exercise=docker.io/{username}/exercise
~~~
이런식으로 두 번 치는걸로 이미지를 강제 업데이트 시켜 rolling update를 유도할 수 있다. 단 이 방법은 tag 가 latest일때만 작동한다. develop같이 branch 이름으로 하는 경우에는 또 동작하지않는다.

이를 다 테스트 해보기 위해 로컬에 kubernetes를 설치하고 하나하나 해봤는데, 다행히 Mac에는 Docker for Mac을 이용하면 쉽게 설치할 수 있었다.

https://eksworkshop.com/logging/