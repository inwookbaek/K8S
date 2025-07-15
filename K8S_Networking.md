# 6. 쿠버네티스 네트워킹
>* 쿠버네티스 네트워킹은 Service리소스가 담당
>* Service리소스도 Yaml형식으로 정의
>* Service리소스는 Pop IP와는 다른 독자적인 IP를 부여 받아 서비스의 EndPoint를 제공
>* 라벨링시스템을 통해 Pod로 트레픽을 전달
>* Service리소스는 Pop앞단에 위치하여 마치 로드발렌서처럼 동작

## 쿠버네티스 네트워킹 기본 개념
1. Pod 간 통신 (Pod-to-Pod communication)
* 클러스터 내 모든 Pod는 서로 직접 통신 가능해야 한다.
* 기본적으로 모든 Pod는 고유한 IP 주소를 가진다.
* Pod IP는 클러스터 내에서 유일하며, 서비스 검색 없이도 IP로 통신 가능.
* 네트워크 정책(NetworkPolicy)으로 통신 허용/차단 가능.

2. 서비스(Service) 네트워킹
* Pod IP는 변할 수 있으므로, 서비스가 추상화 역할을 한다.
* 서비스는 고정된 IP와 DNS 이름을 제공해, Pod 집합에 대한 접근을 가능케 함.
* ClusterIP (클러스터 내부), NodePort, LoadBalancer 등 서비스 타입별 접근 경로 제공.

3. 노드 간 네트워킹 (Node-to-Node communication)
* 노드들 간에는 네트워크가 연결되어야 하고, Pod IP가 어디서든 라우팅 가능해야 한다.
* 네트워크 플러그인(CNI 플러그인)이 이 역할 담당.

## 쿠버네티스 네트워킹 기본 원칙 (3대 원칙)
* 모든 Pod는 서로 직접 통신 가능해야 한다.
* 모든 노드는 모든 Pod IP와 통신 가능해야 한다.
* 호스트 네트워크는 별도 관리되고, Pod 네트워크와 격리되어야 한다.

## 주요 네트워크 구성 요소
| 구성 요소                                 | 설명                                       |
| ------------------------------------- | ---------------------------------------- |
| **Pod IP**                            | 각 Pod가 부여받는 유니크 IP                       |
| **Service IP (ClusterIP)**            | 클러스터 내에서 고정된 서비스 접근용 IP                  |
| **CNI (Container Network Interface)** | 네트워크 플러그인 (Calico, Flannel, Weave, etc.) |
| **NetworkPolicy**                     | 네트워크 트래픽의 허용/차단 정책                       |
| **Ingress**                           | 클러스터 외부에서 내부 서비스로의 HTTP/HTTPS 접근         |


## 네트워크 플러그인 (CNI) 예시
* Calico: 네트워크 정책 및 보안에 강점
* Flannel: 간단하고 빠른 Overlay 네트워크 제공
* Weave Net: 손쉬운 설치와 자동 메쉬 네트워킹
* Cilium: eBPF 기반 고성능 보안과 네트워킹

## Pod와 서비스 간 통신 흐름
* Pod → Pod 직접 IP 접근 가능
* Pod → Service (ClusterIP) → Service의 Endpoint(실제 Pod들)로 트래픽 전달
* 외부 → NodePort 또는 LoadBalancer → 내부 Service → Pod

### 요약
* 쿠버네티스 네트워킹은 Pod IP 기반 통신 보장
* 서비스는 안정적인 접속을 위한 추상화
* CNI 플러그인이 실제 네트워크 연결을 담당
* 네트워크 정책으로 보안 및 접근 제어 가능

## 6.1 Service 소개
* 쿠버네티스의 **Service(서비스)**는 Pod 집합에 대한 네트워크 접근을 안정적이고 일관되게 제공하는 리소스.
* 서비스는 Pod가 죽거나 재시작되어도 변하지 않는 IP와 DNS 이름을 제공하여, 클라이언트가 항상 동일한 방법으로 접근할 수 있게 한다.

### 🔧 왜 Service가 필요한가?
* Pod는 재시작 시 IP가 바뀜 → 직접 접근 불가능
* Service는 Pod 앞에 고정된 접점(IP, DNS)을 제공
* 여러 Pod(레플리카)에 대해 로드밸런싱 역할 수행

### ✅ Service의 종류
| 종류                  | 설명                                                      |
| ------------------- | ------------------------------------------------------- |
| **ClusterIP (기본값)** | 클러스터 내부에서만 접근 가능 (내부 서비스용)                              |
| **NodePort**        | 외부에서 노드의 포트를 통해 접근 가능                                   |
| **LoadBalancer**    | 클라우드 환경에서 외부 로드밸런서를 자동 생성                               |
| **ExternalName**    | 서비스 이름을 외부 도메인으로 매핑 (DNS alias)                         |
| **Headless**        | 클러스터 DNS를 통해 Pod IP 목록 직접 반환 (LoadBalancer 없이 직접 접근 가능) |

### 📄 ClusterIP 예제 (기본 타입)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app              # 이 라벨을 가진 Pod들을 대상으로
  ports:
  - protocol: TCP
    port: 80                 # 클라이언트가 접근할 서비스 포트
    targetPort: 8080         # Pod 내부에서 실제로 사용하는 포트
# my-service라는 이름으로 클러스터 내 DNS에 등록됨
# (예: http://my-service.default.svc.cluster.local)
```

### 🌐 NodePort 예제
```yaml
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080   # (30000~32767 범위)
# 클러스터 외부에서: http://<노드IP>:30080 으로 접근 가능
```

### LoadBalancer 예제 (클라우드 환경에서만)
```yaml
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
```
* 클라우드에서 외부 IP가 자동으로 부여됨
* 외부 사용자 → LoadBalancer IP → 서비스 → Pod

### 🔍 서비스 검색 (Service Discovery)
* 모든 Service는 클러스터 내 DNS 이름을 갖습니다.
  - 예: my-service.default.svc.cluster.local
* 애플리케이션은 이 이름을 통해 서비스에 접근

### 🎯 요약
| 기능     | 설명                                 |
| ------ | ---------------------------------- |
| 로드밸런싱  | 동일 레이블을 가진 여러 Pod에 트래픽 분산          |
| IP 안정성 | Pod가 죽거나 재시작돼도 서비스 IP는 유지          |
| DNS 제공 | 내부 DNS 이름으로 서비스 접근 가능              |
| 외부 노출  | NodePort, LoadBalancer로 외부에서 접근 가능 |

### 🔍 Pod vs Service 비교표
* Pod는 언제든지 종료/삭제될 수 있는 리소스인 특징으로 불안정한 자원이다.
* 이러한 문제를 해결하기 위해 Pod의 생명주기와는 상관없이 안정적인 서비스를 제공하는 Service리소스를 사용한다.

| 항목        | **Pod**                              | **Service**                               |
| --------- | ------------------------------------ | ----------------------------------------- |
| **정의**    | 하나 이상의 컨테이너를 실행하는 가장 작은 배포 단위        | Pod 집합에 대한 네트워크 접근을 제공하는 추상 리소스           |
| **역할**    | 실제 애플리케이션이 동작하는 컨테이너 환경              | 외부나 내부에서 Pod에 안정적으로 접근하기 위한 엔드포인트 제공      |
| **IP**    | Pod마다 고유한 IP 부여됨 (비영구적, 재시작 시 변경됨)   | 고정된 ClusterIP 할당됨 (서비스 생명주기 동안 유지)        |
| **접근 방법** | 직접 접근 가능하지만 IP가 변경될 수 있어 불안정함        | DNS 또는 IP를 통해 안정적인 접근 가능                  |
| **내부 통신** | Pod 간 직접 통신 가능 (같은 네임스페이스 또는 클러스터 내) | 다른 Pod나 외부 트래픽이 안정적으로 Pod에 도달하도록 중간 역할 수행 |
| **로드밸런싱** | 개별 Pod로 직접 접근 시 로드밸런싱 불가능            | 동일 라벨을 가진 여러 Pod에 대해 자동 로드밸런싱 지원          |
| **수명**    | 일시적일 수 있고, 재시작 시 삭제/재생성됨             | 상대적으로 안정적이고 Pod 변경과 관계없이 계속 유지됨           |
| **구성 대상** | 컨테이너 (애플리케이션 자체)                     | 특정 레이블을 가진 Pod (애플리케이션을 감싸는 접점)           |

### 🎯 예시로 이해하기
#### ✅ Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-app
    image: nginx
# → 실제로 컨테이너(Nginx)가 동작하는 실행 단위
```

#### ✅ Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 8080
# → 레이블이 app: my-app인 Pod들을 찾아 하나의 IP로 묶어 접근 가능하게 함
# (클러스터 외부 또는 내부에서)
```
### 🔑 정리
* Pod는 "무엇을 실행할지" 정의한 것 (실행 단위)
* Service는 "어떻게 접근할지" 정의한 것 (네트워크 접근 단위)
* 둘은 함께 사용되어, 쿠버네티스 애플리케이션의 확장성, 안정성, 가용성을 보장합니다.

## 🎯 실습

```bash
kubectl run mynginx --image nginx
# pod/mynginx created

# Pod IP는 사용자마다 다릅니다.
kubectl get pod -owide
# NAME     READY   STATUS     RESTARTS   AGE   IP           NODE    ...
# mynginx  1/1     Running    0          12d   10.42.0.26   master  ...

kubectl exec mynginx -- curl -s 10.42.0.26
# <html>
# <head>
# <title>Welcome to nginx!</title>
# <style>
#     body {
# ...
```















