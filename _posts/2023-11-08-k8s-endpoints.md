---
title: "쿠버네티스의 Endpoints에 대해 알아보자"
date: 2023-11-08
last_modified_at: 2023-11-08
categories:
  - Kubernetes
tags:
  - Endpoints
  - Network
tagline: ""
header:
 overlay_image: /images/overlay_image.jpg
 overlay_filter: 0.5 ## same as adding an opacity of 0.5 to a black
---

Kubernetes API에서 엔드포인트 (리소스 종류는 복수형)는 네트워크 엔드포인트 목록을 정의합니다.  
일반적으로 서비스에서 참조하여 트래픽을 전송할 수 있는 포드를 정의합니다.

## Endpoints

* 쿠버네티스의 서비스 기능을 사용해서 외부 서비스와 연결
* 외부 서비스와 연결 수행 시 외부 IP를 직접 endpoint라는 별도의 자원에서 설정(레이블 사용x)
*  Kubernetes 서비스가 Pod로의 트래픽 분배를 보장할 수 있도록 하기 위해 엔드포인트는 추상화 계층 역할을 하는 데 필요
* Endpoint의 가장 큰 문제는 스케일링
   * 서비스에서 포드를 추가하거나 제거할 때마다 전체 Endpoints 객체가 업데이트되고 네트워크를 통해 모든 노드로 전송
   * 이 문제를 완화하기 위해 EndpointSlices 객체가 새롭게 추가(Kubernetes 1.21에서 table 상태)

## EndpointSlices

* Endpoint의 스케일링 문제 대안으로 나옴
* 엔드포인트에 보다 확장 가능한 대안을 제공할 수 있는 API 리소스
* 여러 리소스에 네트워크 Endpoint를 분산시킬 수 있다
* 기본적으로, EndpointSlices는 100개의 Endpoint에 도달하면 "가득찬 것"로 간주되며, 추가 Endpoint를 저장하기 위해서는 추가 EndpointSlices가 생성


장점

* 네트워크 트래픽을 줄임
* 성능저하 방지(컨트롤 플레인 및 노드)
* 더 나은 성능과 안전성

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
---
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: my-service-1 # 관례적으로 서비스 이름을 사용합니다.
                     # EndpointSlice 이름의 접두사
  labels:
    # "kubernetes.io/service-name" 라벨을 설정해야 합니다.
    # 서비스 이름과 일치하도록 값을 설정합니다.
    kubernetes.io/service-name: my-service
addressType: IPv4
ports:
  - name: '' # 포트 9376이 잘 알려진 포트로 할당되지 않았기 때문에 비어 있습니다.
             # port (by IANA)
    appProtocol: http
    protocol: TCP
    port: 9376
endpoints:
  - addresses:
      - "10.4.5.6"
  - addresses:
      - "10.1.2.3"
```


## Endpoint와 EndpointSlices 차이점

* Endpoint는 전통적인 방식으로 모든 서비스 엔드포인트를 한 번에 가져오는 데 사용되며, EndpointSlice는 대규모 클러스터에서 더 효율적으로 동작하도록 개선된 방식입니다. 
* EndpointSlice는 서비스 엔드포인트를 부분적으로 가져와서 자원을 더 효과적으로 활용할 수 있도록 합니다.

1. **Endpoint:**
   - **개념:** Endpoint는 쿠버네티스 서비스에 속하는 개별 Pod의 IP 주소 및 포트를 나타냅니다.
   - **사용:** 서비스의 엔드포인트는 Pod의 주소를 추적하고 클라이언트가 서비스에 연결할 때 사용됩니다.
   - **활용:** 주로 작은 규모의 클러스터에서 사용되며, 서비스의 모든 엔드포인트가 한 번에 가져와집니다.

2. **EndpointSlice:**
   - **개념:** EndpointSlice는 여러 개의 서브셋으로 나뉘어진 서비스 엔드포인트의 더 효율적인 표현입니다.
   - **사용:** EndpointSlice는 대규모 클러스터에서 성능 및 확장성을 향상시키기 위해 도입되었습니다. 서비스 엔드포인트를 여러 Slice로 나누어 각 Slice를 필요한 경우에만 가져올 수 있습니다.
   - **활용:** 대규모 클러스터에서 효과적으로 사용되며, 필요한 시점에만 필요한 엔드포인트 슬라이스를 가져와서 자원을 절약합니다.

## 참고
* Kubernetes Documentation: [Endpoints](https://kubernetes.io/docs/concepts/services-networking/service/#endpoints)
* [Kubernetes EndpointSlices](https://kmaster.tistory.com/112)