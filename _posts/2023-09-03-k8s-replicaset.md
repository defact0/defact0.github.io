---
title: "쿠버네티스의 레플리카셋(ReplicaSet)에 대해 알아보자"
date: 2023-09-03
last_modified_at: 2023-09-03
categories:
  - Kubernetes
tags:
  - ReplicaSet
  - workloads
tagline: ""
header:
 overlay_image: /images/overlay_image.jpg
 overlay_filter: 0.5 ## same as adding an opacity of 0.5 to a black
---

> 레플리카셋의 목적은 레플리카 파드 집합의 실행을 항상 안정적으로 유지하는 것이다.
> * Kubernetes Documentation: [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)  

<br>

## 레플리카셋

* 레플리카셋은 레플리케이션컨트롤러를 대체하는 기능(업그레이드 버전)
* 기존대비 레이블을 더 유연하게 선택가능하다.
  * 레플리케이션컨트롤러: 특정 레이블 일치여부 확인
  * 레플리카셋: 특정 레이블 포함여부 확인

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # 케이스에 따라 레플리카를 수정한다.
  replicas: 3
  selector: # 레이블 셀렉터: 관리하는 포드 범위 결정
    matchLabels:
      tier: frontend
  template: # 포드 템플릿: 새로운 포드를 설명
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3

```

레플리카셋 관련된 명령어는 아래와 같다.

```bash
# 레플리카셋 조회
$ kubectl get rs

# 레플리카셋 상세조회
$ kubectl describe rs nginx

# 레플리카셋 삭제
$ kubectl delete rs nginx
```