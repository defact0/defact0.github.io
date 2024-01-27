---
title: "쿠버네티스의 kube-system Component에 대해 알아보자"
date: 2023-10-06
last_modified_at: 2023-10-06
categories:
  - Kubernetes
tags:
  - Component
  - Static Pods
  - kube-system
tagline: ""
header:
 overlay_image: /images/overlay_image.jpg
 overlay_filter: 0.5 ## same as adding an opacity of 0.5 to a black
---

## Kubernetes 클러스터의 구성 요소

* Control Plane Components
  * kube-apiserver
  * etcd
  * kube-scheduler
  * kube-controller-manager
  * cloud-controller-manager
* Node Components
  * kubelet
  * kube-proxy
  * Container runtime
* Addons 
* DNS
* Web UI (Dashboard)
* Container Resource Monitoring
* Cluster-level Logging
* Network Plugins

### 참고
* Kubernetes Documentation: [Kubernetes Components](https://kubernetes.io/docs/concepts/overview/components/)  

## kube-apiserver

* kube-system Component들은 API 서버로 통신
* 다른 Component들 끼리 직접 통신을 하지 않음
* RESTful API를 통해 클러스터 상태를 쿼리, 수정기능 제공

### 역할

* 클라이언트 인증
  * 인증 플러그인
  * 권한 승인 플러그인
* 승인 제어 플러그인을 통해 리소스를 확인, 수정
* 리소스 검증 및 영구 저장


### 참고
* Kubernetes Documentation: [kube-apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)  


<br>

## kube-controller-manager

* apiserver를 통해 클러스터의 공유 상태를 감시하고 현재 상태를 원하는 상태로 이동하려고 시도하는 제어 루프
* Kubernetes와 함께 제공되는 컨트롤러의 예
  * 복제 컨트롤러
  * 엔드포인트 컨트롤러
  * 네임스페이스 컨트롤러
  * 서비스 계정 컨트롤러


### 참고
* Kubernetes Documentation: [kube-controller-manager](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/)  


<br>

## kube-scheduler

* 포드를 노드에 할당하는 제어 평면 프로세스
* 각 Pod에 대해 유효한 배치가 되는 노드를 결정
* 노드의 상태를 점검
* 다수의 Pod 배치하는 경우 라운드로빈을 사용하여 분산


### 참고
* Kubernetes Documentation: [kube-scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)  

<br>

## 주요 컴포넌트 확인

```bash
$ kubectl get pod -n kube-system
```

* 관리형 쿠버네티스는 조회가 되지 않음
* CSP에서 관리하는 영역이기 때문

```bash
# master 노드에서 실행하면 yaml 파일들을 확인 할 수 있다.
$ cd /etc/kubernetes/manifestes/
```


## Static Pods

* 각 노드에서 kubelet에 의해 직접 실행(노드에 의해 실행)
* Static Pods는 apiserver를 통해 삭제가 안된다.(삭제하면 다시 생성하기 때문)

### 생성 방법

1. 대상 노드에 ssh 접속
2. `/etc/kubernetes/manifests`경로로 이동
3. Pod yaml 파일을 정의함 `/etc/kubernetes/manifests/static-web.yaml`
4. kubelet을 다시 시작 `systemctl restart kubelet`

`/etc/kubernetes/manifests`경로 대신 다른 경로에서 관리하고 싶다면?
1. `service kubelet status`에서 `/etc/systemd/system/kubelet.service.d` 확인
2. `kubelet.service.d`에는 `10-kubeadm.conf`파일이 있다.
3. `10-kubeadm.conf`파일에는 다시 `/var/lib/kubelet/config.yaml` 내용을 확인
4. `/var/lib/kubelet/config.yaml` 내용에서 `staticPodPath`항목을 수정하면된다.

### 참고
* Kubernetes Documentation: [Create static Pods](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)  