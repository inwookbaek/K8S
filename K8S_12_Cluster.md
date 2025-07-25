# 12. 클러스터 관리
* 클러스터 관리는 클러스터의 안정성, 확장성, 보안, 성능을 유지하고 운영하는 일련의 활동
* 서비스운에서 리소스 사용량 관리는 중요하다. 쿠버네티스에서는 가상의 논리클러스터인 네임스페이스를 이용하여 리소스를 관리한다.
* 리스소관리를 담담하는 LimitRange, ResourceQuota리소스가 있다.

## 🍎 LimitRange와 ResourceQuota의 주요 비교와 차이점
| 항목          | LimitRange                                                         | ResourceQuota                                                        |
| ----------- | ------------------------------------------------------------------ | -------------------------------------------------------------------- |
| 목적      | 네임스페이스 내 개별 컨테이너/파드에 대한 리소스 최소/최대 제한 및 기본값 설정 | 네임스페이스 전체 자원 사용량 총량 제한 및 관리                                          |
| 적용 단위 | 컨테이너 또는 파드 단위                                                      | 네임스페이스 단위 (전체 파드, 서비스, PVC 등 포함)                                     |
| 주요 기능 | - CPU, 메모리 등의 최소/최대 요청(request) 및 제한(limit) 설정<br>- 기본 요청 및 제한값 지정 | - 네임스페이스 내 CPU, 메모리, 저장공간 등의 총 사용량 제한<br>- 객체 수 제한 (파드 개수, PVC 개수 등) || 리소스 관리 범위   | 개별 컨테이너/파드가 요청하거나 제한할 수 있는 리소스 범위 제어 | 네임스페이스 전체 리소스 사용량과 오브젝트 수 총량 관리                                      |
| 설정 예시       | 컨테이너별 CPU 최소 200m \~ 최대 600m 제한, 기본 요청 300m                        | 네임스페이스 내 CPU 총 사용량 10 CPU, 파드 총 20개 제한                               |
| 네임스페이스 내 영향 | 모든 컨테이너/파드에 기본값과 제한 적용                                             | 네임스페이스 내 생성 가능한 리소스 총량 제한                                            |
| 주요 사용 목적    | 개별 파드의 리소스 요청과 제한 강제, 자원 오버/언더 요청 방지   | 네임스페이스별 자원 과도한 사용 방지 및 공평한 자원 분배                                     |

## 🍿요약:
* LimitRange는 개별 컨테이너/파드가 요청할 수 있는 최소/최대 리소스를 제한해, 자원 요청의 범위를 강제합니다.
* ResourceQuota는 네임스페이스 전체에서 사용할 수 있는 리소스 총량과 개체 수를 제한하여, 네임스페이스 단위 자원 소비를 관리합니다.
* 두 리소스는 함께 사용해 네임스페이스 내 자원 사용을 세밀하고 효율적으로 제어할 수 있습니다.

## 🍓 1클러스터 관리 주요 영역
| 구분                     | 주요 내용                                                                                                                                       |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| ** 클러스터 설치 및 구성**    | - 클러스터 구성 및 초기화 (kubeadm, kops, eksctl, gcloud 등) <br> - 마스터와 워커 노드 셋업 <br> - 네트워크(CNI) 플러그인 설정 (Calico, Flannel, Weave 등)   |
| ** 클러스터 모니터링과 로깅**| - 리소스 사용량 모니터링: CPU, 메모리, 디스크, 네트워크 <br> - 애플리케이션 상태 모니터링 (Prometheus, Grafana, ELK Stack) <br> - 로그 수집 및 분석 (Fluentd, Loki, ElasticSearch) |
| ** 인증과 권한 관리** | - RBAC(Role-Based Access Control) 정책 설정 <br> - 사용자 및 서비스 계정 권한 관리 <br> - 네트워크 정책(Network Policies)으로 통신제한                                  |
| ** 클러스터 확장 및 업그레이드** | - 노드 추가/삭제를 통한 수평 확장 <br> - 쿠버네티스 버전 업그레이드 관리 <br> - 자동 스케일링 도구 설정 (Cluster Autoscaler, HPA 등) |
| ** 보안 관리**           | - 네임스페이스 격리 및 네트워크 정책 설정 <br> - 이미지 스캐닝 및 신뢰할 수 있는 이미지 사용 <br> - 비밀 관리(Secrets), TLS 인증서 관리    |
| ** 백업 및 복구**         | - etcd 데이터 백업 <br> - 애플리케이션 데이터 및 설정 백업 <br> - 장애 시 신속 복구 계획 수립   |
| ** 자원 관리**           | - 리소스 요청(Requests) 및 제한(Limits) 설정 <br> - QoS 정책 관리 <br> - 네임스페이스별 자원 쿼터 관리    |

---

## 2. 클러스터 관리 도구
| 도구                   | 설명                               |
|------------------------|----------------------------------|
| **kubectl**            | 기본 명령어 툴, 클러스터 자원 제어          |
| **kubeadm, kops, eksctl, gcloud, az aks** | 클러스터 설치 및 초기화               |
| **Prometheus & Grafana** | 모니터링 및 시각화                     |
| **Fluentd, Loki, ElasticSearch** | 로그 수집 및 분석                   |
| **Helm**                | 애플리케이션 배포 및 관리               |
| **Cluster Autoscaler**  | 클러스터 자동 확장                    |
| **Kustomize**           | 선언적 설정 관리                     |
| **Velero**              | 백업 및 복구                        |

---

## 3. 클러스터 관리 시 고려사항
- 고가용성(HA) 구성 (마스터 노드, etcd 클러스터)
- 네트워크 및 스토리지 안정성 확보
- 정책에 따른 보안 강화
- 비용 최적화 및 리소스 효율화
- 운영 자동화 (CI/CD 파이프라인 연계)

## 12.1 리소스 관리

| 관리 영역               | 설명                                               |
|------------------------|--------------------------------------------------|
| **리소스 요청(Requests)**   | 컨테이너가 안정적으로 실행되기 위해 보장받는 최소 자원 (CPU, 메모리) 지정         |
| **리소스 제한(Limits)**    | 컨테이너가 사용할 수 있는 최대 자원 한도 지정                                   |
| **QoS(Quality of Service)** | 요청과 제한에 따라 Pod에 부여되는 우선순위 클래스 (Guaranteed, Burstable, BestEffort) |
| **네임스페이스별 자원 쿼터(Resource Quotas)** | 네임스페이스 단위로 사용할 수 있는 CPU, 메모리, 객체 수량 등 자원 사용량 제한 관리       |
| **수평 및 수직 확장(Horizontal/Vertical Scaling)** | Pod 수 증감(HPA) 또는 Pod 자원 크기 조절을 통한 유연한 자원 관리                 |
| **노드 자원 모니터링**       | 노드별 CPU, 메모리, 디스크 등 자원 사용량 모니터링 및 최적화                     |
| **리소스 효율화**           | 불필요한 자원 사용 방지, 오버프로비저닝 최소화, 비용 절감                        |

### 12.1.1 LimitRange
* LimitRange는 Kubernetes에서 네임스페이스 단위로 Pod나 컨테이너가 사용할 수 있는 자원의 최소, 최대, 기본 요청 및 제한을 강제하는 정책 리소스입니다. 
* 즉, 개발자가 자원 요청(request)이나 제한(limit)을 명시하지 않을 때 기본값을 지정하거나, 너무 크거나 작은 값을 제한해서 안정적인 클러스터 운영을 돕습니다.

#### 📌 LimitRange 주요 기능
| 주요 기능                         | 설명                                                                     | 예시                    |
| ---------------------------       | -------------------------------------------                              | --------------------- |
| 최소/최대 리소스 제한             | 컨테이너가 요청하거나 제한할 수 있는 CPU, 메모리의 최소값과 최대값 지정  | CPU 최소 100m, 최대 2 CPU |
| 기본 요청값 (Default Requests)    | Pod 스펙에 요청(request)을 명시하지 않았을 때 자동 적용되는 기본값       | 자동 할당                 |
| 기본 제한값 (Default Limits)      | Pod 스펙에 제한(limit)을 명시하지 않았을 때 자동 적용되는 기본값         | 자동 할당                 |
| 파드 레벨 제한 (Pod-level Limits) | 파드 전체 CPU/메모리 합산 제한 설정 가능                                 | CPU/메모리 총합 제한 설정 가능   |

```bash
kubectl run mynginx --image nginx

kubectl get pod mynginx -oyaml | grep resources
# resources: {}
```
```bash
vi K8S/CH05/limit-range.yaml
```
```yaml
# limit-range.yaml
# Kubernetes LimitRange 리소스를 정의한 예시
apiVersion: v1                  # Kubernetes API 버전
kind: LimitRange               # 리소스 종류: LimitRange는 네임스페이스 내 컨테이너의 리소스 제한과 요청 기본값을 설정
metadata:
  name: limit-range            # LimitRange 리소스의 이름
spec:
  limits:
  - default:                   # 컨테이너가 리소스 제한(limit)을 명시하지 않을 때 적용되는 기본값
      cpu: 400m                # CPU 기본 제한값 400 milli CPU (0.4 CPU)
      memory: 512Mi            # 메모리 기본 제한값 512MiB
    defaultRequest:            # 컨테이너가 리소스 요청(request)을 명시하지 않을 때 적용되는 기본값
      cpu: 300m                # CPU 기본 요청값 300 milli CPU (0.3 CPU)
      memory: 256Mi            # 메모리 기본 요청값 256MiB
    max:                       # 컨테이너가 요청하거나 제한할 수 있는 최대값
      cpu: 600m                # CPU 최대 600 milli CPU (0.6 CPU)
      memory: 600Mi            # 메모리 최대 600MiB
    min:                       # 컨테이너가 요청하거나 제한할 수 있는 최소값
      cpu: 200m                # CPU 최소 200 milli CPU (0.2 CPU)
      memory: 200Mi            # 메모리 최소 200MiB
    type: Container            # 이 LimitRange 정책이 적용되는 대상 타입 (여기서는 컨테이너)
```

```bash
kubectl apply -f K8S/CH05/limit-range.yaml
# limitrange/limit-range created

kubectl run nginx-lr --image nginx
# pod/nginx-lr created

kubectl get pod nginx-lr -oyaml | grep -A 6 resources
#    resources:
#      limits:
#        cpu: 400m
#        memory: 512Mi
#      requests:
#        cpu: 300m
#        memory: 256Mi
```

```bash
vi K8S/CH05/pod-exceed.yaml
```
```yaml
# pod-exceed.yaml
# LimitRange 설정을 초과하는 리소스 요청 및 제한을 가진 Pod 정의 예제
apiVersion: v1
kind: Pod
metadata:
  name: pod-exceed
spec:
  containers:
  - image: nginx
    name: nginx
    resources:
      limits:
        cpu: "700m"       # LimitRange max(cpu=600m) 초과
        memory: "700Mi"   # LimitRange max(memory=600Mi) 초과
      requests:
        cpu: "300m"       # LimitRange defaultRequest 및 min 이상이므로 문제 없음
        memory: "256Mi"   # LimitRange defaultRequest 및 min 이상이므로 문제 없음
```

```bash
# 일반사용자가 LimitRange의 max property를 초과해서 Pod 생성에러가 발생
kubectl apply -f K8S/CH05/pod-exceed.yaml
# Error from server (Forbidden): error when creating "STDIN": pods 
# "pod-exceed" is forbidden: [maximum cpu usage per Container 
# is 600m, but limit is 700m, maximum memory usage per Container 
# is 600Mi, but limit is 700Mi]
```

### Clean up

```bash
kubectl delete limitrange limit-range
kubectl delete pod --all
```

### 12.1.2 ResourceQuota
* **ResourceQuota**는 네임스페이스(namespace) 단위로 리소스 사용량을 제한하기 위한 객체입니다. 
* 클러스터 관리자가 CPU, 메모리, 스토리지, 오브젝트 수(Pod, Service 등) 등을 과도하게 사용하는 것을 방지하기 위해 사용됩니다.
* LimitRange는 개별 Pod에 대한 제약에 관여했다면 ResourceQuota는 전체 네임스페이스에 대한 제약을 설정

#### ✅ 왜 ResourceQuota를 사용할까?
* 특정 팀이나 애플리케이션이 전체 리소스를 독점하지 않도록 방지
* 네임스페이스별로 공정한 리소스 분배 가능
* 클러스터 리소스를 더 효율적으로 관리

#### 📌 주의사항
* ResourceQuota는 네임스페이스 단위로만 적용됩니다.
* LimitRange와 함께 사용하면 Pod/Container의 자원 요청(request)/제한(limit)을 자동 설정할 수 있습니다.
* 설정된 ResourceQuota를 초과하면 새로운 리소스 생성이 실패하게 됩니다.

```bash
vi K8S/CH05/res-quota.yaml
```
```yaml
# res-quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: res-quota
spec:
  hard:
    limits.cpu: 700m         # 전체 Pod의 limit.cpu 합계가 최대 700m (0.7 vCPU)
    limits.memory: 800Mi     # 전체 Pod의 limit.memory 합계가 최대 800MiB
    requests.cpu: 500m       # 전체 Pod의 request.cpu 합계가 최대 500m (0.5 vCPU)
    requests.memory: 700Mi   # 전체 Pod의 request.memory 합계가 최대 700MiB
```

```bash
# ResourceQuota 생성
kubectl apply -f K8S/CH05/res-quota.yaml
# resourcequota/res-quota created 

# Pod 생성 limit CPU 600m : limits.cpu: 700m 이하인 Pod 생성
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: rq-1
spec:
  containers:
  - image: nginx
    name: nginx
    resources:
      limits:
        cpu: "600m"
        memory: "600Mi"
      requests:
        cpu: "300m"
        memory: "300Mi"
EOF
# pod/rq-1 created

# kubectl get resourcequota -n [네임스페이스]
kubectl get resourcequota
kubectl get resourcequota -n default

# 1개더 생성(생성에러) - limits.cpu 초과
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: rq-2
spec:
  containers:
  - image: nginx
    name: nginx
    resources:
      limits:
        cpu: "600m"
        memory: "600Mi"
      requests:
        cpu: "300m"
        memory: "300Mi"
EOF
# Error from server (Forbidden): error when creating "STDIN": 
# pods "rq-2" is forbidden: exceeded quota: res-quota, 
# requested: limits.cpu=600m,limits.memory=600Mi,requests.cpu=300m, 
# used: limits.cpu=600m,limits.memory=600Mi,requests.cpu=300m, 
# limited: limits.cpu=700m,limits
```

### Clean up

```bash
kubectl delete resourcequota res-quota
kubectl delete pod --all
```

## 12.2 노드 관리
* 노드(Node) 관리는 클러스터의 물리적 또는 가상 머신을 제어하고 상태를 모니터링하며, 
* 필요에 따라 스케줄링, 유지보수, 업그레이드 등을 수행하는 작업

### 🧠 cordon vs drain 차이
| 기능       | cordon         | drain                |
| -------- | -------------- | -------------------- |
| 새 파드 배치  | ❌ 차단           | ❌ 차단                 |
| 기존 파드 제거 | ❌ 그대로 유지됨      | ✅ 강제 제거 (다른 노드로 이동)  |
| 목적       | 일시적 차단 또는 유지보수 | 실제 노드 비우기 (재시작 등 작업) |

### 12.2.1 Cordon
* 특정 노드에 더 이상 새로운 파드가 스케줄되지 않도록 막는 명령어입니다.
* 이미 실행 중인 파드는 그대로 유지되지만, 추가 배치가 불가능
* 노드 스케줄링 금지 (Cordoning) / 활성화(Uncordon)
  >* kubectl cordon [노드이름]
  >* kubectl uncordon [노드이름]

```bash
# 먼저 worker의 상태를 확인합니다.
kubectl get node worker1 -oyaml | grep spec -A 5
# spec:
#   podCIDR: 10.42.0.0/24
#   podCIDRs:
#   - 10.42.0.0/24
#   providerID: k3s://worker
# status:

# worker를 cordon시킵니다.
kubectl cordon worker1
# node/worker cordoned

# 다시 worker의 상태를 확인합니다. taint가 설정된 것을 확인할 수 있고 unschedulable이 true로 설정되어 있습니다.
kubectl get node worker1 -oyaml | grep spec -A 10
# spec:
#   podCIDR: 10.42.0.0/24
#   podCIDRs:
#   - 10.42.0.0/24
#   providerID: k3s://worker
#   taints:
#   - effect: NoSchedule
#     key: node.kubernetes.io/unschedulable
#     timeAdded: "2020-04-04T11:04:48Z"
#   unschedulable: true
# status:

# worker의 상태를 확인합니다.
kubectl get node
# NAME     STATUS                    ROLES    AGE   VERSION
# master   Ready                     master   32d   v1.18.6+k3s1
# worker   Ready,SchedulingDisabled  worker   32d   v1.18.6+k3s1
```


```bash
# ReplicaSet을 이용해서 여러개의 Pod생성
cat << EOF | kubectl apply -f -
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs
spec:
  replicas: 5
  selector:
    matchLabels:
      run: rs
  template:
    metadata:
      labels:
        run: rs
    spec:
      containers:
      - name: nginx
        image: nginx
EOF

kubectl get pod -o wide
# NAME     READY   STATUS    RESTARTS   AGE    IP          NODE     ...
# rs-xxxx  1/1     Running   0          3s     10.42.1.6   master   ...
# rs-xxxx  1/1     Running   0          3s     10.42.1.7   master   ...
# rs-xxxx  1/1     Running   0          3s     10.42.1.8   master   ...
# rs-xxxx  1/1     Running   0          3s     10.42.1.9   master   ...
# rs-xxxx  1/1     Running   0          3s     10.42.1.10  master   ...
```

```bash
# 명시적으로 worker1노드에 실행될 수 있고 nodeSelector를 추가
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: pod-worker
spec:
  containers:
  - image: nginx
    name: nginx
  nodeSelector:
    kubernetes.io/hostname: worker1
EOF
# pod/pod-worker created

kubectl get pod -owide
# NAME         READY  STATUS    RESTARTS   AGE     IP       NODE    ... 
# ...
# pod-worker   0/1    Pending   0          70s     <none>   <none>  ...

kubectl get node
# NAME      STATUS                     ROLES                  AGE   VERSION
#...
# worker1   Ready,SchedulingDisabled   <none>                 11d   v1.32.6+k3s1
```

### 12.2.2 Uncordon
*  cordon된(스케줄 차단된) 노드를 다시 스케줄 가능 상태로 되돌리는 명령어

```bash
kubectl uncordon worker1
# node/worker uncordoned

 k get node
# NAME      STATUS     ROLES                  AGE   VERSION
#...
# worker1   Ready      <none>                 11d   v1.32.6+k3s1

# taint가 사라졌습니다.
kubectl get node worker1 -oyaml | grep spec -A 10
# spec:
#   podCIDR: 10.42.1.0/24
#   podCIDRs:
#   - 10.42.1.0/24
#   providerID: k3s://worker
# status:
#   addresses:
#   - address: 172.31.16.173
#     type: InternalIP
#   - address: worker
#     type: Hostname

kubectl get node
# NAME     STATUS   ROLES    AGE   VERSION
# master   Ready    master   32d   v1.18.6+k3s1
# worker   Ready    worker   32d   v1.18.6+k3s1

kubectl get pod -owide
# NAME         READY   STATUS    RESTARTS   AGE     IP           NODE      NOMINATED NODE   READINESS GATES
# ...
# pod-worker   1/1     Running   0          6m49s   10.42.4.83   worker1   <none>           <none>

kubectl delete pod pod-worker
# pod/pod-worker deleted
```

### 12.2.3 Drain
* kubectl drain은 Kubernetes에서 노드를 비우는 명령어입니다.
* 즉, 해당 노드에서 실행 중인 **파드들을 모두 다른 노드로 안전하게 이동(퇴거)**시키고, 스케줄도 차단
* 📦 노드에서 모든 파드 퇴거 (Drain)
  >* 해당 노드에서 실행 중인 파드들을 모든 다른 노드로 안전하게 옮김
  >* (단, DaemonSet 파드는 그대로 남고, LocalStorage 사용 파드는 강제 종료될 수 있음)
  >* kubectl drain [노드이름] --ignore-daemonsets --delete-emptydir-data

#### 🧠 주요 옵션 설명
| 옵션                       | 설명                         |
| ------------------------ | -------------------------- |
| `--ignore-daemonsets`    | DaemonSet 파드는 무시 (삭제하지 않음) |
| `--delete-emptydir-data` | emptyDir를 사용하는 파드도 강제 삭제   |
| `--force`                | (경고 무시) 파드 강제 퇴거 시 사용 가능   |
| `--grace-period=0`       | 파드 강제 종료 (즉시)              |

#### 📌 drain 사용 시 주의사항
* PodDisruptionBudget(PDB) 가 설정된 경우, 제한을 초과하면 drain이 실패할 수 있음
* emptyDir를 사용하는 파드는 삭제되면 데이터가 손실될 수 있음
* StatefulSet 파드는 다른 노드로 옮기지 않고 재생성되므로, 서비스 중단 없이 재시작하려면 주의 필요

```bash
# Deployment를 생성 - worker1에 일부 Pod가 할당
cat << EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
EOF
# deployment.apps/pod-drain created

# nginx Pod가 worker1 노드에 생성된 것을 확인할 수 있습니다.
kubectl get pod -o wide
# NAME                     READY   STATUS              RESTARTS   AGE     IP           NODE      NOMINATED NODE   READINESS GATES
# nginx-5869d7778c-czf7r   1/1     Running   0          65s     10.42.4.84   worker1   <none>           <none>
# nginx-5869d7778c-j7jzb   1/1     Running   0          65s     10.42.0.39   master    <none>           <none>
# nginx-5869d7778c-nccfq   1/1     Running   0          65s     10.42.0.40   master    <none>           <none>
```

```bash
# worker1노드를 drain시킨다.
# 모든 노드에 존재하는 DaemonSet은 무시합니다.
kubectl drain worker1  --ignore-daemonsets
# node/worker cordoned
# evicting pod "nginx-xxx"
# evicting pod "nginx-xxx"
# ...

# nginx Pod가 어떻게 동작하는지 확인합니다.
# nginx-5869d7778c-czf7r pod가 worker1에서 master로 할당
kubectl get pod -owide
# NAME                     READY   STATUS    RESTARTS   AGE     IP           NODE     NOMINATED NODE   READINESS GATES
# nginx-5869d7778c-9fktx   1/1     Running   0          36s     10.42.0.41   master   <none>           <none>
# nginx-5869d7778c-j7jzb   1/1     Running   0          3m13s   10.42.0.39   master   <none>           <none>
# nginx-5869d7778c-nccfq   1/1     Running   0          3m13s   10.42.0.40   master   <none>           <none>

 
kubectl get node worker1 -oyaml | grep spec -A 10
# spec:
#   podCIDR: 10.42.1.0/24
#   podCIDRs:
#   - 10.42.1.0/24
#   providerID: k3s://worker
#   taints:
#   - effect: NoSchedule
#     key: node.kubernetes.io/unschedulable
#     timeAdded: "2020-04-04T15:37:25Z"
#   unschedulable: true
# status:

kubectl get node
# 워커노드(worker1) status 확인 : Ready,SchedulingDisabled
# NAME      STATUS                     ROLES                  AGE   VERSION
# master    Ready                      control-plane,master   12d   v1.32.6+k3s1
# worker1   Ready,SchedulingDisabled   <none>                 11d   v1.32.6+k3s1
```

```bash
# drain된 노드도 uncordon명령으로 되돌릴 수 있다.
kubectl uncordon worker1
# node/worker uncordoned status 확인  : Ready
# 워커노드(worker1) status 확인
# NAME      STATUS     ROLES                  AGE   VERSION
# master    Ready      control-plane,master   12d   v1.32.6+k3s1
# worker1   Ready      <none>                 11d   v1.32.6+k3s1
```

## 12.3 Pod 개수 유지
* drain 명령시 Pod가 갑자기 종료되는 것을 확인할 수 있다.
* Deployment리소스가 곧바로 새로운 Pod를 생헝해 주었지만 일시적으로 Pod개수가 현저히 줄어 든다.
* 만약, 트래픽을 많이 받는 서비스를 운영 중이었다면 순간적으로 모든 부하가 한쪽 Pod에 쏠려서 응답지연이 발생할 수 있다.
* PodDisruptionBudget(pdb)은 이러한 문제를 해결하고자 만든 리소스이다.
* pdb는 운영 중인 Pod의 개수를 항상 일정수준으로 유지할 수 있도록 Pod의 drain을 막아주는 역할을 한다.

### 🚒 PodDisruptionBudget?
* PodDisruptionBudget(PDB)은 Kubernetes에서 예기치 않은 장애가 아닌 "계획된 장애(예: 노드 drain, 업그레이드)" 중에도 최소한의 파드를 유지하기 위한 정책입니다.
* 즉, 너무 많은 파드가 동시에 중단되는 것을 방지해서 서비스 가용성을 확보한다.
* ✅ PodDisruptionBudget 기본 개념
  - **voluntary disruption(자발적 중단)**에 대해서만 작동합니다. : 예: kubectl drain, 노드 업그레이드 등
  - **involuntary disruption(비자발적 중단)**에는 영향 없음. : 예: 노드 장애, 파드 충돌
* 📌 주의 사항
  - PDB는 단독으로 파드 수를 유지하지는 않습니다. 반드시 Deployment, StatefulSet 등과 함께 사용해야 의미가 있습니다.
  - HPA와 함께 쓸 때도 유효합니다. 다만, 너무 높은 minAvailable 설정은 오히려 drain을 막아 유지보수가 어려워질 수 있습니다.

```bash
# 이전에 생성한 nginx deployment에 replica를 10으로 up
kubectl scale deploy nginx --replicas 10
# deployment.apps/mydeploy scaled
```

```bash
kubectl get  deploy nginx
# NAME    READY   UP-TO-DATE   AVAILABLE   AGE
# nginx   3/3     3            3           26m

vi K8S/CH05/nginx-pdb.yaml
```
```yaml
# nginx-pdb.yaml
# 자진 중단상황에서 최소 9개의 pod가 항상 실행될 수 있게 설정
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: nginx-pdb
spec:
  minAvailable: 9
  selector:
    matchLabels:
      app: nginx
```

```bash
# pdb를 생성합니다.
kubectl apply -f K8S/CH05/nginx-pdb.yaml
# poddisruptionbudget/nginx-pdb created

kubectl get pdb
# NAME        MIN AVAILABLE   MAX UNAVAILABLE   ALLOWED DISRUPTIONS   AGE
# nginx-pdb   9               N/A               0                     104s

# nginx pod개수 확인
kubectl get pods -l app=nginx

# worker1을 drain합니다.
kubectl drain worker1 --ignore-daemonsets
# node/worker cordoned
# evicting pod "nginx-xxx"
# evicting pod "nginx-xxx"
# error when evicting pod "mynginx-xxx" 
# (will retry after 5s): Cannot evict pod as it would violate the 
#   pod's disruption budget.
# pod/mynginx-xxx evicted
# evicting pod "mynginx-xxx"
# error when evicting pod "mynginx-xxx" 
# (will retry after 5s): Cannot evict pod as it would violate the 
#   pod's disruption budget.
# evicting pod "mynginx-xxx"
# pod/mynginx-xxx evicted
# node/worker evicted
```

### Clean up

```bash
kubectl delete pdb nginx-pdb
kubectl delete deploy nginx
kubectl delete rs rs
kubectl uncordon worker1
```
