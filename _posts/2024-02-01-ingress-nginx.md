---
title: "Kubernetes 클러스터 외부로 들어오는 트래픽을 관리하는 Nginx Ingress Controller"
date: 2024-02-01
last_modified_at: 2024-02-01
categories:
  - kubernetes
tags:
  - Ingress
  - Nginx
tagline: "클러스터 내 여러 서비스에 대한 단일 진입점 역할을 한다."
header:
  overlay_image: /images/overlay_image.jpg
  overlay_filter: 0.5 ## same as adding an opacity of 0.5 to a black
---

<div class="notice">
스터디 전용 클러스터에 Nginx Ingress Controller를 설치하고 Ingress를 통해서 테스트 웹어플리케이션에 접속하고자 한다.<br>
CSP에서 제공하는 서비스 환경이 아니라 ALB 같은 서비스는 사용할 수 없는 상황이다.
</div>

### 구성도

![](/images/2024-02-01-ingress-nginx/architecture.png)


## 01. Nginx Ingress Conntroller 설치

```bash
# Nginx Ingress Conntroller 설치
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/baremetal/deploy.yaml

# 배포된 오브젝트 확인
kubectl get all -n ingress-nginx
```

* k8s cluster 환경에 맞는 yaml파일을 다운로드 받아야 한다. ([ingress-nginx Github 링크](https://github.com/kubernetes/ingress-nginx/tree/main/deploy/static/provider) )

## 02. Nginx Ingress 구성

### 02-1. 서비스 app 배포

첨부된 파일 내용을 클러스터에 적용한다. 

```bash
apiVersion: v1
kind: Namespace
metadata:
  name: jeongdo
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: jeongdo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
        - name: nginx-container
          image: nginx:latest
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: jeongdo
spec:
  selector:
    app: nginx-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins-deployment
  namespace: jeongdo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins-app
  template:
    metadata:
      labels:
        app: jenkins-app
    spec:
      containers:
        - name: jenkins-container
          image: jenkins/jenkins:jdk17
          ports:
            - containerPort: 8080
            - containerPort: 50000
---
apiVersion: v1
kind: Service
metadata:
  name: jenkins-service
  namespace: jeongdo
spec:
  selector:
    app: jenkins-app
  ports:
    - name: http
      protocol: TCP
      port: 8080
      targetPort: 8080
    - name: agent
      protocol: TCP
      port: 50000
      targetPort: 50000
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gitlab-deployment
  namespace: jeongdo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gitlab-app
  template:
    metadata:
      labels:
        app: gitlab-app
    spec:
      containers:
        - name: gitlab-container
          image: gitlab/gitlab-ce:latest
          ports:
            - containerPort: 80
            - containerPort: 443
---
apiVersion: v1
kind: Service
metadata:
  name: gitlab-service
  namespace: jeongdo
spec:
  selector:
    app: gitlab-app
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 80
  - name: https
    protocol: TCP
    port: 443
    targetPort: 443
```

### 02-2. Ingress 설정

첨부된 파일 내용을 클러스터에 적용한다.

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-nginx
  namespace: jeongdo
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: www.test.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-service
                port:
                  number: 80
    - host: jenkins.test.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: jenkins-service
                port:
                  number: 8080
    - host: gitlab.test.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: gitlab-service
                port:
                  number: 80
```

ingress-nginx의 상세 정보를 보았을 때 아래와 같이 백엔드 정보가 잘 설정되어 있어야 한다.  
만일 오류가 발생하면 `nginx-service:80 (<error: endpoints "nginx-service" not found>)`이런 형태로 출력된다.

```bash
irteam@neca-platkuber-wa801:~/jeongdo$ kubectl describe ingress ingress-nginx -n jeongdo
Name:             ingress-nginx
Labels:           <none>
Namespace:        jeongdo
Address:          10.162.0.13
Ingress Class:    nginx
Default backend:  <default>
Rules:
  Host              Path  Backends
  ----              ----  --------
  www.test.com
                    /   nginx-service:80 (10.10.18.69:80,10.10.18.70:80,10.10.251.5:80)
  jenkins.test.com
                    /   jenkins-service:8080 (10.10.18.73:8080)
  gitlab.test.com
                    /   gitlab-service:80 (10.10.18.72:80)
Annotations:        nginx.ingress.kubernetes.io/rewrite-target: /
Events:             <none>
```

### 02-3. hosts 파일 수정

ingress.test.com 접속을 위해 자신의 PC의 hosts파일을 수정해야 한다.

```bash
# hosts 수정
sudo vi /etc/hosts
## Windows OS인 경우는 Windows\System32\drivers\etc\hosts 경로의 파일을 수정한다.

# hosts 추가할 내용
10.162.0.140 www.test.com
10.162.0.140 jenkins.test.com
10.162.0.140 gitlab.test.com

# DNS cache 를 갱신
dscacheutil -flushcache
```

ingress-nginx-controller의 포트정보를 확인해야 한다.

```bash
kubectl -n ingress-nginx get svc
NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.103.49.70    <none>        80:31092/TCP,443:31304/TCP   17m
ingress-nginx-controller-admission   ClusterIP   10.98.228.105   <none>        443/TCP                      17m
```

hosts에 추가한 도메인과 NodePort를 조합하여 아래와 같은 URL이 된다.

```
http://www.test.com:31092/
```

## 03. 배포된 테스트 웹 어플리케이션 초기 패스워드확인

### 03-1. Jenkins 초기패스워드 확인방법

* jenkins Pod를 생성하고나서 바로 확인한다.

```bash
# pod 이름 확인
kubectl -n jeongdo get pod

# jenkins-864c8d545f-st55z의 로그확인
kubectl -n jeongdo logs jenkins-864c8d545f-st55z

# 아래와 같이 로그에서 패스워드정보를 확인할 수 있다.
## Jenkins initial setup is required. An admin user has been created and a password generated.
## Please use the following password to proceed to installation:
## 
## 51292ca4adf34296a40a6aee4b8e307f
## 
## This may also be found at: /var/jenkins_home/secrets/initialAdminPassword
```

### 03-2. Gitlab 초기패스워드 확인방법
* Username: root 입력
* Password: 서버 터미널 프로그램에서 cat /etc/gitlab/initial_root_password 입력해 확인된 비밀번호 입력

```bash
# pod 이름 확인
kubectl -n jeongdo get pod

# gitlab-deployment-b876bd5bb-k9mf9 접속
kubectl -n jeongdo exec gitlab-deployment-b876bd5bb-k9mf9 -it -- bash

# gitlab-deployment-b876bd5bb-k9mf9 컨테이너 bash에서 아래 명령어 입력
cat /etc/gitlab/initial_root_password  | grep Password:

# 아래와 같이 패스워드정보를 확인할 수 있다.
## Password: QYxjy1BmeGPuwJKk/wxkStpDhFpOkH+OzwlQCnRkKl0=
```


## 04. 주의할점

* Nginx Ingress Conntroller 오브젝트 라벨명이 기본적으로 ingress-nginx으로 되어 있어 사용자가 ingress 설정 시 ingress-nginx으로 맞춰야한다.

## 05. 참고

* [Github > kubernetes/ingress-nginx](https://github.com/kubernetes/ingress-nginx)