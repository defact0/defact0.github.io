---
title: "쿠버네티스의 디플로이먼트(Deployment)에 대해 알아보자"
date: 2023-09-13
last_modified_at: 2023-09-13
categories:
  - Kubernetes
tags:
  - Deployments
  - workloads
tagline: ""
header:
 overlay_image: /images/overlay_image.jpg
 overlay_filter: 0.5 ## same as adding an opacity of 0.5 to a black
---

> 디플로이먼트(Deployment)는 파드와 레플리카셋(ReplicaSet)에 대한 선언적 업데이트를 제공한다.
> * Kubernetes Documentation: [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)  

* 디플로이먼트에서 의도하는 상태 를 설명하고, 
* 디플로이먼트 컨트롤러(Controller)는 현재 상태에서 의도하는 상태로 비율을 조정하며 변경한다.

<br>

# 디플로이먼트(Deployment)

* 애플리케이션을 다운타임 없이 업데이트를 가능하도록 도와주는 리소스
* 레플리카셋과 레플리케이션컨트롤러 상위에 배포

## 모든 포드를 업데이트하는 방법

* Recreate: 파드를 모두 종료하고 바로 다시 생성, 다운타임 발생
* RollingUpdate: 다운타임 없이 애플리케이션의 파드들을 새버전으로 하나씩 배포

## 작성요령

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3 # 몇 개의 pod를 배포할지
  selector: # 생성된 ReplicaSet가 관리할 포드를 찾는 방법을 정의
    matchLabels:
      app: nginx
  template: # 아래부터는 pod를 정의
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

위 yaml 내용을 배포하면 아래와 같은 결과를 얻을 수 있다.

```bash
$ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-86dcfdf4c6-96lwz   1/1     Running   0          11s
pod/nginx-deployment-86dcfdf4c6-c66vq   1/1     Running   0          11s
pod/nginx-deployment-86dcfdf4c6-rks5q   1/1     Running   0          11s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   9d

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           11s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-86dcfdf4c6   3         3         3       11s
```

deployment리소스 1개, replicaset리소스 1개, pod리소스 3개가 만들어 진 것을 확인 할 수 있다.

## 스케일링

```bash
# 직접 수정을 하여 replicas 수를 조정
kubectl edit deploy nginx

# replicas 옵션을 사용
kubectl scale deploy nginx --replicas=10
```

<br>

# 애플리케이션 롤링 업데이트와 롤백

RollingUpdate는 다운타임 없이 애플리케이션의 파드들을 새버전으로 하나씩 배포하는 방식이다.

* 새 버전을 실행하는 동안 구 버전 포드와 연결
* 서비스의 레이블셀렉터를 수정하여 진행
* 애플리케이션 간의 하위호환이 있어야 한다.

 ```bash
# history 10개 기록지원
$ kubectl create -f nginx.yaml --record=ture

# history 확인
$ kubectl rollout history deployment nginx

# deployment의 pod 이미지 변경
$ kubectl set image deployment nginx nginx=banana/nginx:v2

# 특정 버전으로 되돌아 가기
$ kubectl rollout undo deployment nginx --to-revision=1

# 업데이트 일시정지
$ kubectl rollout pause deployment nginx

# 업데이트 일시정지 취소
$ kubectl rollout undo deployment nginx

# 업데이트 재시작
$ kubectl rollout resume deployment nginx
 ```

## 세부 전략 설정

### maxUnavailable

* 최소 몇개의 포드를 유지해야 하는지(기본값은 25%)
* 예를 들어 이 값을 30%로 설정하면 롤링 업데이트가 시작되는 즉시 기존 ReplicaSet를 원하는 포드의 70%로 축소할 수 있습니다

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
 name: nginx-deployment
 labels:
   app: nginx
spec:
 replicas: 3
 selector:
   matchLabels:
     app: nginx
 template:
   metadata:
     labels:
       app: nginx
   spec:
     containers:
     - name: nginx
       image: nginx:1.14.2
       ports:
       - containerPort: 80
 strategy:
   type: RollingUpdate
   rollingUpdate:
     maxUnavailable: 1 # maxUnavailable 설정
```

### maxSurge

* 최대 몇개까지 포드를 생성할 것인가!(기본값은 25%)
* 예를 들어 이 값을 30%로 설정하면 롤링 업데이트가 시작될 때 새 ReplicaSet를 즉시 확장할 수 있으므로 이전 및 새 포드의 총 개수가 원하는 포드의 130%를 초과하지 않습니다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
 name: nginx-deployment
 labels:
   app: nginx
spec:
 replicas: 3
 selector:
   matchLabels:
     app: nginx
 template:
   metadata:
     labels:
       app: nginx
   spec:
     containers:
     - name: nginx
       image: nginx:1.14.2
       ports:
       - containerPort: 80
 strategy:
   type: RollingUpdate
   rollingUpdate:
     maxSurge: 1 # maxSurge 설정
     # maxUnavailable: 1 # maxUnavailable를 추가해서 사용 가능
```

## 업데이트를 실패하는 케이스

* Kubernetes Documentation: [Failed Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#failed-deployment)  

* 부족한 할당량(Insufficient quota)
* 레디네스 프로브 실패(Readiness probe failures)
* 이미지 가져오기 오류(Image pull errors)
* 권한 부족(Insufficient permissions)
* 제한 범위(Limit ranges)
* 응용 프로그램 런타임 구성 오류(Application runtime misconfiguration)

```bash
$ kubectl patch deployment/nginx-deployment -p '{"spec":{"progressDeadlineSeconds":600}}'
```

* 업데이트를 실패하는 경우에는 `기본적`으로 600초(10분) 후에 업데이트를 중지한다.