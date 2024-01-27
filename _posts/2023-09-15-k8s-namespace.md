---
title: "쿠버네티스의 네임스페이스(Namespaces)에 대해 알아보자"
date: 2023-09-15
last_modified_at: 2023-09-15
categories:
  - Kubernetes
tags:
  - Namespaces
  - workloads
tagline: ""
header:
 overlay_image: /images/overlay_image.jpg
 overlay_filter: 0.5 ## same as adding an opacity of 0.5 to a black
---

> 네임스페이스 는 단일 클러스터 내에서의 리소스 그룹 격리 메커니즘을 제공한다.
> * Kubernetes Documentation: [Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)  

* 리소스를 분리된 영역으로 나눈다.
* Multi-tenant 환경을 분리하여 리소스를 생산, 개발, QA 환경 등으로 사용
* 리소스 이름은 네임스페이스 내에서만 고유 명칭으로 사용

```bash
# 네임스페이스 확인
$ kubeclt get ns
$ kubectl get namespace
```

* 기본 질의 결과는 default 네임스페이스에서 한다.
* 다른 사용자의 접근 제한 가능, 리소스 양도 제어 가능

```bash
# banana 이라는 네임스페이스의 pods를 조회
kubectl get pods --namespace=banana

# 모든 네임스페이스 내 모든 파드의 목록 조회
kubectl get pods --all-namespace

# 네임스페이스 생성 명령
$ kubectl create namespace tomato

# tomato 네임스페이스에 대한 spec을 생성하고, tomato-ns.yaml이라는 파일에 해당 내용을 기록한다.
$ kubectl create namespace tomato --dry-run=client -o yaml > tomato-ns.yaml
```

* 네임스페이스 생성관련 문서[링크](https://kubernetes.io/docs/tasks/administer-cluster/namespaces/#creating-a-new-namespace)