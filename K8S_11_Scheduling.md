# 11. 고급 스케줄링

## 11.1 고가용성 확보 - Pod 레벨
* Pod의 리소스 사용량에 따라 자동으로 확장하는 HPAA(수평 Pod 자동 확장, Horizontal Pod Autoscaler)는 
* 쿠버네티스에서 Pod의 CPU/메모리 사용량 또는 사용자 정의 메트릭을 기준으로 자동으로 Replica 수를 조절해주는 기능
* 즉, 트래픽 증가 시 Pod 수를 자동으로 늘리고, 부하가 줄어들면 다시 줄이는 오토스케일링 기능

### ✅ HPA 작동 개념
* 대상: Deployment, ReplicaSet, StatefulSet
* 기준: 기본은 CPU 사용률, 필요 시 메모리나 사용자 정의 메트릭도 사용 가능
* 동작: 메트릭 서버(Metrics Server)가 주기적으로 자원 사용량을 수집 → 조건 충족 시 스케일 조정
* 📦 Metrics Server 설치 필요, HPA는 metrics-server가 있어야 작동합니다.

### 🧠 HPA 구성 요소 요약
| 항목                  | 역할                         |
| ------------------- | -------------------------- |
| `metrics-server`    | 리소스 사용량 수집 (CPU, Memory 등) |
| `kubectl autoscale` | HPA 객체 생성                  |
| `HPA 객체`            | 실제로 스케일 조건과 목표를 관리하는 리소스   |

### 11.1.1 metrics server 설치
* HPA (Horizontal Pod Autoscaler), kubectl top pod/node 등의 리소스 사용량 정보를 사용하려면 반드시 Metrics Server가 설치되어 있어야 한다.

#### ✅ Metrics Server란?
* metrics-server는 쿠버네티스 클러스터의 각 노드와 Pod의 CPU 및 메모리 사용량을 수집하고 이를 API로 제공합니다.
  - kubectl top 명령이나 HPA에 필요함.
* 이 서버를 통해 Pod의 작업량을 모니터링하다가 사용자가 지정한 일정수준의 임계값을 넘으면 replica개수를 동적으로 조절하여 Pod개수를 늘인다.
* 일정시간이 지난후 Pod의 작업량이 적어지게 되면 다시 Pod개수를 줄여주는 역할도 수행한다.

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

helm install metrics-server bitnami/metrics-server \
    --namespace ctrl --create-namespace

# ※ 필요 시 --set args={--kubelet-insecure-tls} 옵션도 추가 가능
# NAME: metrics-server
# LAST DEPLOYED: Wed Jul  8 17:50:32 2020
# NAMESPACE: ctrl
# STATUS: deployed
# REVISION: 1
# NOTES:
# The metric server has been deployed.
# 
# In a few minutes you should be able to list metrics using the following
# command:
# 
#   kubectl get --raw "/apis/metrics.k8s.io/v1beta1/nodes"

# metrics-server가 정상적으로 올라오기까지 시간이 조금 걸립니다.
kubectl get pod -nctrl
# NAME                              READY   STATUS    RESTARTS   AGE
# metrics-server-8555869558-k7gb6   0/1     Running   0          34s
```

```bash
# 리소스 사용량을 모니터링할 Pod를 하나 생성합니다.
kubectl run mynginx --image nginx

# Pod별 리소스 사용량을 확인합니다.
kubectl top pod
# NAME        CPU(cores)   MEMORY(bytes)
# mynginx     0m           2Mi

# Node별 리소스 사용량을 확인합니다.
kubectl top node
# NAME      CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
# master    57m          2%     1846Mi          46%
# worker    43m          2%      970Mi          24%

kubectl delete pod mynginx
# pod/mynginx deleted
```

### 11.1.2 자동 확장할 Pod 생성

```php
# image: k8s.gcr.io/hpa-example
<?php
  $x = 0.0001;
  for ($i = 0; $i <= 1000000; $i++) {
    $x += sqrt($x);
  }
  echo "OK!";
?>
```
```bash
vi K8S/CH05/heavy-cal.yaml
```
```yaml
# heavy-cal.yaml
# Deployment: CPU 부하 테스트용 애플리케이션 배포
apiVersion: apps/v1                      # Deployment 리소스의 API 버전
kind: Deployment                         # 리소스 종류: Deployment
metadata:
  name: heavy-cal                        # Deployment 이름
spec:
  selector:
    matchLabels:
      run: heavy-cal                     # 이 레이블을 가진 Pod과 연결
  replicas: 1                            # 초기 Pod 수
  template:
    metadata:
      labels:
        run: heavy-cal                   # Pod에 부여될 레이블 (Service와 연결됨)
    spec:
      containers:
      - name: heavy-cal                  # 컨테이너 이름
        image: registry.k8s.io/hpa-example  # ✅ 최신 공식 이미지 주소로 변경됨
        ports:
        - containerPort: 80              # 컨테이너가 열어줄 포트
        resources:                       # HPA 작동을 위한 리소스 설정
          limits:
            cpu: 500m                    # 최대 CPU 사용량 0.5 vCPU
          requests:                      # hpa가 정상적으로 동작하기 위해서 반드시 requests가 정의되어야 한다.
            cpu: 300m                    # 최소 보장 CPU 0.3 vCPU

---
# Service: 내부 접근을 위한 ClusterIP 서비스 정의
apiVersion: v1                           # Service는 core/v1 API 사용
kind: Service                            # 리소스 종류: Service
metadata:
  name: heavy-cal                        # Service 이름
spec:
  ports:
  - port: 80                             # 서비스가 노출할 포트 (클러스터 내 접근용)
    targetPort: 80                       # 연결될 컨테이너 포트
  selector:
    run: heavy-cal                       # 이 레이블을 가진 Pod에 연결
```

```bash
kubectl apply -f K8S/CH05/heavy-cal.yaml
# deployment.apps/heavy-cal created
# service/heavy-cal created
```

### 11.1.3 `hpa` 생성 - 선언형 명령
* 웹서버 자동확장시킬 hpa생성

```bash
vi K8S/CH05/hpa.yaml  
```
```yaml
# Horizontal Pod Autoscaler (HPA): CPU 부하에 따라 Pod 수 자동 조절
apiVersion: autoscaling/v1                     # HPA 리소스의 API 버전 (v1: CPU 기준 자동 확장 지원)
kind: HorizontalPodAutoscaler                  # 리소스 종류: HPA
metadata:
  name: heavy-cal                              # HPA의 이름 (관리할 대상 리소스와 동일하게 설정함)
spec:
  maxReplicas: 50                              # 최대 생성할 Pod 수
  minReplicas: 1                               # 최소 유지할 Pod 수
  scaleTargetRef:                              # 확장 대상 리소스 정의
    apiVersion: apps/v1                        # 대상 리소스의 API 버전 (Deployment는 apps/v1)
    kind: Deployment                           # 확장 대상 리소스 종류
    name: heavy-cal                            # 확장할 Deployment 이름
  targetCPUUtilizationPercentage: 50           # 목표 CPU 사용률 (Pod 평균 사용률이 50% 넘으면 스케일 아웃)
```
#### ✅ 요약
| 항목                               | 설명                             |
| -------------------------------- | ------------------------------ |
| `minReplicas`                    | Pod 수가 줄어들 수 있는 하한선            |
| `maxReplicas`                    | 스케일 아웃할 수 있는 상한선               |
| `targetCPUUtilizationPercentage` | 평균 CPU 사용률 기준으로 스케일 조정 트리거     |
| `scaleTargetRef`                 | 어떤 Deployment를 대상으로 자동 확장할지 지정 |


```bash
# hpa 리소스 생성
kubectl apply -f K8S/CH05/hpa.yaml  
# horizontalpodautoscaler.autoscaling/heavy-cal autoscaled
```

### 11.1.4 `hpa` 생성 - 명령형 명령
*  HPA 리소스를 자동 생성

```bash
kubectl delete hpa heavy-cal

# 대상 Deployment: heavy-cal / 확장 기준: 평균 CPU 사용률이 50% 초과 시 / 최소 Pod 수: 1 /최대 Pod 수: 50
kubectl autoscale deployment heavy-cal --cpu-percent=50 --min=1 --max=50
# horizontalpodautoscaler.autoscaling/heavy-cal autoscaled

# 상태확인
kubectl get hpa
# NAME        REFERENCE                   TARGET    MINPODS  MAXPODS   REPLICAS   AGE
# heavy-cal   Deployment/heavy-cal/scale  0% / 50%  1        50        1          18s

kubectl describe hpa heavy-cal
```

### 11.1.5 자동확장 테스트

```bash
vi K8S/CH05/heavy-load.yaml
```
```yaml
# heavy-load.yaml
apiVersion: v1                      # core/v1 API 그룹: Pod 리소스 정의에 사용
kind: Pod                           # 리소스 종류: Pod (단일 컨테이너)
metadata:
  name: heavy-load                 # Pod 이름
spec:
  containers:
  - name: busybox                 # 컨테이너 이름
    image: busybox                # 가벼운 Linux 유틸리티용 이미지 사용
    command: ["/bin/sh"]         # 기본 실행 셸
    args:                        # 무한 루프 실행
      - "-c"
      - "while true; do wget -q -O- http://heavy-cal; done" # heavy-cal 서비스에 반복 요청하여 CPU 부하 생성
```

```bash
kubectl apply -f K8S/CH05/heavy-load.yaml
# pod/heavy-load created

# watch문으로 heavy-cal를 계속 지켜보고 있으면 Pod의 개수가 증가하는 것을 확인할 수 있습니다.
watch kubectl top pod
# NAME                         CPU(cores)   MEMORY(bytes)
# heavy-load                   7m           1Mi
# heavy-cal-548855cf99-9s44l   140m         12Mi
# heavy-cal-548855cf99-lnbvm   122m         13Mi
# heavy-cal-548855cf99-lptbq   128m         13Mi
# heavy-cal-548855cf99-qpdng   89m          12Mi
# heavy-cal-548855cf99-tvgfn   137m         13Mi
# heavy-cal-548855cf99-x64mg   110m         12Mi
```

```bash
kubectl delete pod heavy-load
```

## 고가용성 확보 - Node 레벨 vs Pod 레벨
### 🔁 1. 고가용성 확보 – Pod 레벨
| 항목           | 설명                                                     |
| ------------ | ------------------------------------------------------ |
| **목적**       | 애플리케이션(Pod) 장애 발생 시 자동 복구                              |
| **핵심 전략**    | 동일한 앱의 여러 복제본(Replica)을 여러 노드에 분산 실행                   |
| **대표 기술**    | `Deployment`, `ReplicaSet`, `StatefulSet`, `HPA` 등     |
| **보장 범위**    | Pod 또는 컨테이너 단위의 장애 (CrashLoop, OOM 등)                  |
| **필수 구성 요소** | 2개 이상의 Pod (replicas ≥ 2), 다중 노드 권장                    |
| **보완**       | `PodAntiAffinity`, `topologySpreadConstraints`로 Pod 분산 |

### 🖥️ 2. 고가용성 확보 – Node 레벨
| 항목           | 설명                                                                 |
| ------------ | ------------------------------------------------------------------ |
| **목적**       | 워커 노드 장애 시에도 애플리케이션 가용성 유지                                         |
| **핵심 전략**    | 여러 워커 노드 운영 + Pod의 자동 재스케줄링                                        |
| **대표 기술**    | 다중 워커 노드, `tolerations`, `affinity`, `DaemonSet`                   |
| **보장 범위**    | 노드 장애 또는 네트워크 단절                                                   |
| **필수 구성 요소** | 2개 이상의 워커 노드                                                       |
| **보완**       | `nodeSelector`, `node affinity`, `taints & tolerations` 사용 시 주의 필요 |

### 📊 3. 비교 요약
| 비교 항목 | Pod 레벨 HA                       | Node 레벨 HA                          |
| ----- | ------------------------------- | ----------------------------------- |
| 단위    | Pod (컨테이너)                      | 노드 (물리/가상 머신)                       |
| 주 대상  | Pod 자체의 실패 (crash, error 등)     | 노드 장애, 네트워크 장애                      |
| 구성 요소 | ReplicaSet, HPA, AntiAffinity 등 | 다중 노드, Toleration, Affinity 등       |
| 작동 방식 | 실패 시 다른 Pod가 자동 생성됨             | 노드 장애 시, 다른 노드로 Pod 재스케줄            |
| 필수 조건 | 다수 Pod 복제본                      | 다수 워커 노드                            |
| 예시 구성 | `replicas: 3`, HPA              | 워커 노드 2+, `tolerations`, `affinity` |
| 주요 한계 | 노드가 하나면 아무리 복제해도 무용지물           | Pod가 1개뿐이면 노드가 살아도 장애 발생            |


## 11.2 고가용성 확보 - Node 레벨
* Cluster AutoScale의 경우 클라우드 서비스를 지원해야 하기 때문에 K3S클러스터로 테스트하기가 어렵다.
* 따라서, AWS EKS와 GCP GKE기준으로 Cluster AutoScaling하는 방법을 알아보기로 한다.

### 11.2.1 AWS EKS Cluster AutoScaler 설정
* AWS EKS에서 Cluster Autoscaler를 설정하면, 워크로드에 따라 자동으로 EC2 인스턴스를 추가하거나 제거해 클러스터의 노드 수를 자동 조절할 수 있다. 

#### ✅ 1. 전제 조건 확인
* EKS 클러스터가 이미 생성되어 있어야 합니다.
* Managed Node Group 또는 Self-managed Node Group 중 하나는 반드시 구성되어 있어야 합니다.
* kubectl, eksctl, awscli, helm 설치되어 있어야 합니다.

#### ✅ 2. IAM 정책 설정 (노드 그룹용)
* Cluster Autoscaler가 노드를 조절할 수 있도록 IAM 정책을 추가해야 합니다.


```bash
NAME=k8s
REGION=ap-northeast-2

helm repo add autoscaler https://kubernetes.github.io/autoscaler
helm repo update

helm upgrade --install cluster-autoscaler autoscaler/cluster-autoscaler \
  --namespace kube-system \
  --set autoDiscovery.clusterName=$NAME \
  --set awsRegion=$REGION \
  --set cloudProvider=aws \
  --set rbac.create=true \
  --set extraArgs.balance-similar-node-groups=true \
  --set extraArgs.skip-nodes-with-system-pods=false \
  --set extraArgs.skip-nodes-with-local-storage=false
# sslCertPath는 AWS EKS에서는 일반적으로 필요하지 않으며, 대부분 기본 CA기본 루트 인증서(Certificate Authority)로 동작합니다.

# 🧪 설치 확인
kubectl get pods -n kube-system | grep cluster-autoscaler

kubectl logs -n kube-system deployment/cluster-autoscaler-aws-cluster-autoscaler | grep scale-up
kubectl logs -n kube-system deployment/cluster-autoscaler-aws-cluster-autoscaler | grep scale-down
# 로그에 scale-up 또는 scale-down 메시지가 보이면 정상 작동 중입니다.
```

### 11.2.2 GCP GKE Cluster AutoScaler 설정

```bash
# gcloud CLI 설치
curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-$(uname | tr '[:upper:]' '[:lower:]')-x86_64.tar.gz
tar -xf google-cloud-cli-*.tar.gz
./google-cloud-sdk/install.sh
./google-cloud-sdk/bin/gcloud init

CLUSTER_NAME=k8s
REGION=us-central1-a

# Google Kubernetes Engine (GKE) 클러스터를 생성하면서 자동 확장(Autoscaling) 기능을 활성화하고, 노드 수의 범위를 설정
gcloud container clusters create $CLUSTER_NAME \
    --enable-autoscaling \
    --min-nodes=1 \
    --num-nodes=2 \
    --max-nodes=4 \
    --node-locations=$REGION \
    --machine-type=n1-highcpu-8

# 클러스터 목록 확인
gcloud container clusters list    

# 노드풀 설정 확인
gcloud container node-pools list --cluster $CLUSTER_NAME
```
###### ✅ 각 옵션 설명
| 옵션                                 | 설명                                          |
| ---------------------------------- | ------------------------------------------- |
| `gcloud container clusters create` | GKE 클러스터 생성 명령어                             |
| `$CLUSTER_NAME`                    | 생성할 클러스터 이름 (예: `my-gke-cluster`)           |
| `--enable-autoscaling`             | **노드 자동 확장** 활성화                            |
| `--min-nodes=1`                    | 최소 노드 수                                     |
| `--num-nodes=2`                    | 초기 노드 수 (기본값)                               |
| `--max-nodes=4`                    | 최대 노드 수                                     |
| `--node-locations=$REGION`         | 노드를 생성할 리전 또는 영역 (예: `us-central1-a`)       |
| `--machine-type=n1-highcpu-8`      | 사용하는 머신 유형 (vCPU 8개, 메모리 낮음 – CPU 집약 워크로드용) |


### 11.2.3 Cluster AutoScaling 활용

```bash
# 인위적으로 Pod 증가
kubectl scale deployment heavy-cal --replicas=50
# deployment.apps/heavy-cal scaled

# Pod 리스트
kubectl get pod
# NAME                        READY   STATUS    RESTARTS   AGE
# heavy-cal-548855cf99-x64m   1/1     Running   0          2s
# heavy-cal-548855cf99-dfx2   1/1     Running   0          2s
# heavy-cal-548855cf99-sf3x   1/1     Running   0          2s
# ....
# heavy-cal-548855cf99-a21t   0/1     Pending   0          2s
# heavy-cal-548855cf99-g8ib   0/1     Pending   0          2s
# heavy-cal-548855cf99-b754   0/1     Pending   0          2s
# ...

watch kubectl get node
# NAME             STATUS   ROLES    AGE   VERSION
# ip-172-31-42-5   Ready             13h   v1.18.6
# ip-172-31-44-9   Ready             1m    v1.18.6
# ....             Ready             1m    v1.18.6
# ....
```

### Clean up

```bash
kubectl delete hpa heavy-cal
kubectl delete deploy heavy-cal
kubectl delete svc heavy-cal
helm delete metrics-server -nctrl
```

## 11.3 `Taint & Toleration`

### 11.3.1 Taint
* Taint(테인트) 는 특정 노드에 "이 조건을 만족하지 않으면 Pod을 배치하지 마라"는 제약을 거는 메커니즘

#### ✅ Taint란?
* Taint는 노드에 적용되는 규칙
* 해당 노드에 특정 Pod만 배치되도록 제한
* Pod이 해당 조건을 "tolerate(참음)"하지 않으면 스케줄되지 않음
* 즉, Taint를 설정하면: "일반적인 Pod은 이 노드에 오지 마. 조건을 만족하는 Pod만 와!"


```bash
# taint 방법
kubectl taint nodes $NODE_NAME <KEY>=<VALUE>:<EFFECT>
```
##### ✅ 명령어 형식
| 항목            | 설명                                                            |
| ------------- | ------------------------------------------------------------- |
| `<NODE_NAME>` | Taint를 적용할 노드 이름                                              |
| `<KEY>`       | Taint의 식별 키                                                   |
| `<VALUE>`     | (선택 사항) 키에 해당하는 값                                             |
| `<EFFECT>`    | Taint의 효과: `NoSchedule`, `PreferNoSchedule`, `NoExecute` 중 하나 |

##### ✅ Taint 효과 비교 표
| 항목                   | `NoSchedule`   | `PreferNoSchedule`   | `NoExecute`               |
| -------------------- | -------------- | -------------------- | ------------------------- |
| **스케줄링 차단**          | ✅ 강제 차단        | ⚠️ 가능한 한 피함 (강제 아님)  | ✅ 강제 차단                   |
| **기존 Pod 퇴출**        | ❌ 퇴출하지 않음      | ❌ 퇴출하지 않음            | ✅ 퇴출함 (Toleration 없으면 제거) |
| **주 용도**             | 지정된 Pod만 배치 제한 | 특정 노드에 부드럽게 우선 순위 부여 | 비정상 노드 격리, 장애 복구          |
| **엄격성**              | 강함 (하드 제한)     | 약함 (소프트 제한)          | 가장 강함 (스케줄 + 유지 둘 다 제한)   |
| **Toleration 필요 조건** | 없으면 배치 안 됨     | 없어도 배치될 수 있음         | 없으면 배치도 안 되고 퇴출도 됨        |

### 11.3.2 Toleration
* Toleration(톨러레이션) 은 노드에 설정된 Taint(테인트)를 "참을 수 있게" 해주는 Pod의 설정입니다.
* 즉, 노드에 Taint가 있어도, 해당 조건을 toleration으로 참으면 Pod이 그 노드에 배치될 수 있습니다.

#### ✅ Taint와 Toleration 매칭 원리
| Node(Taint)                     | Pod(Toleration)     | 결과                      |
| ------------------------------- | ------------------- | ----------------------- |
| `dedicated=frontend:NoSchedule` | 없음                  | ❌ 배치 안 됨                |
| `dedicated=frontend:NoSchedule` | 위와 같은 toleration    | ✅ 배치 가능                 |
| `gpu=true:NoExecute`            | 없음                  | ❌ 스케줄도 안 되고, 있던 Pod 퇴출됨 |
| `gpu=true:NoExecute`            | toleration 있음 (60초) | ✅ 배치되고 60초 유지됨          |

```bash
# project=A 라는 taint를 NoSchedule로 설정
kubectl taint node worker1 project=A:NoSchedule
# node/worker tainted
```
##### 🔍 의미:
| 항목            | 값            |
| ------------- | ------------ |
| 노드 이름         | `worker1`     |
| Taint 키       | `project`    |
| Taint 값       | `A`          |
| 효과 (`effect`) | `NoSchedule` |

```bash
# worker1 노드의 전체 YAML 출력 중에서 taints 필드와 그 아래 4줄을 출력
# 즉, 노드에 설정된 Taint 정보를 빠르게 확인할 수 있다.
kubectl get node worker1 -oyaml | grep -A 4 taints
#  taints:
#  - effect: NoSchedule
#    key: project
#    value: A

vi K8S/CH05/no-tolerate.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: no-tolerate           # Pod 이름
spec:
  containers:
  - name: nginx               # 컨테이너 이름
    image: nginx              # 사용 이미지
```

```bash
kubectl apply -f K8S/CH05/no-tolerate.yaml
# pod/no-tolerate created

kubectl get pod -o wide
# NAME         READY   STATUS    RESTARTS  AGE   IP        NODE    ...
# no-tolerate  1/1     Running   0         3s    <none>    master  ... 

vi K8S/CH05/tolerate.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tolerate                          # Pod 이름
spec:
  containers:
  - name: nginx                           # 컨테이너 이름
    image: nginx                          # nginx 이미지 사용

  tolerations:                            # Taint를 참을 수 있게 하는 설정
  - key: "project"                        # 노드에 설정된 taint의 key
    value: "A"                            # 노드의 taint value
    operator: "Equal"                     # key/value 일치 조건
    effect: "NoSchedule"                 # NoSchedule taint 효과에 대한 toleration
```

```bash
kubectl apply -f K8S/CH05/tolerate.yaml
# pod/tolerate created

kubectl get pod -o wide
# NAME         READY   STATUS    RESTARTS   AGE    IP        NODE
# no-tolerate  1/1     Running   0          1m     <none>    master
# tolerate     1/1     Running   0          15s    <none>    worker
```

```bash
# worker에 taint를 추가합니다.
# 이번에는 key만 존재하는 taint를 적용해 봅니다.
kubectl taint node worker1 badsector=:NoSchedule
# node/worker tainted

kubectl get node worker1 -oyaml | grep -A 7 taints
#  taints:
#  - effect: NoSchedule
#    key: project
#    value: A
#  - effect: NoSchedule
#    key: badsector
```
##### ✅ 의미:
| 항목      | 값            |
| ------- | ------------ |
| 노드 이름   | `worker1`    |
| Taint 키 | `badsector`  |
| Taint 값 | 없음 (`""`)    |
| 효과      | `NoSchedule` |

```bash
vi K8S/CH05/badsector.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: badsector
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: "project"
    value: "A"
    operator: "Equal"
    effect: "NoSchedule"
  - key: "badsector"
    operator: "Exists"          # badsector key가 있고 값이 없어도 허용
    effect: "NoSchedule"        # NoSchedule 효과에 대해서만 허용
```
 
```bash
kubectl apply -f K8S/CH05/badsector.yaml
# pod/badsector created

kubectl get pod -o wide
# NAME         READY   STATUS    RESTARTS   AGE    IP         NODE
# no-tolerate  1/1     Running   0          5m     <none>     master
# tolerate     1/1     Running   0          2m     <none>     worker
# badsector    1/1     Running   0          16s    <none>     worker
```

```bash
kubectl taint node worker1 project-
kubectl taint node worker1 badsector-
kubectl delete pod --all
```


## 11.4 `Affinity & AntiAffinity`
### 🔁 비교 요약표 : Taint/Toleration과 Affinity/AntiAffinity
| 항목              | **Taint & Toleration**                     | **Affinity & AntiAffinity**                   |
| --------------- | ------------------------------------------ | --------------------------------------------- |
| 🔧 **목적**       | 노드에 Pod가 **못 올라오게 차단**                     | Pod가 선호하는 노드 또는 함께/떨어져 있어야 하는 Pod 지정          |
| 📍 **기준**       | **노드 기준** – 노드에 Taint 설정                   | **Pod 또는 노드 기준** – 라벨 기반 규칙 설정                |
| ✅ **Pod 허용 방식** | Taint가 있는 노드에는 **Toleration이 있어야만** 스케줄 가능 | 조건이 맞으면 스케줄링 **선호**, 조건이 안 맞아도 가능 (soft)      |
| ❗ **강제성 여부**    | 기본적으로 **강제 (NoSchedule, NoExecute)**       | 기본적으로 **권고 (preferred)** 또는 **강제 (required)** |
| 📌 **사용 예시**    | 특정 노드에 특정 작업만 배치하고 싶을 때                    | Pod 간에 같이/떨어져 배치하거나, 특정 노드에만 배치하고 싶을 때        |

### 11.4.1 `NodeAffinity`
* NodeAffinity는 쿠버네티스에서 Pod이 어떤 노드에 배치될지를 제어하는 데 사용하는 스케줄링 제약 조건. 
* 노드에 설정된 라벨을 기준으로 Pod이 어느 노드에 올라갈지 선택하거나 제한할 수 있다.

#### ✅ Node Affinity 종류
| 유형                                                | 설명                                     | 강제 여부       |
| ------------------------------------------------- | -------------------------------------- | ----------- |
| `requiredDuringSchedulingIgnoredDuringExecution`  | **반드시 충족해야** 배치됨                       | 강제 (`hard`) |
| `preferredDuringSchedulingIgnoredDuringExecution` | 조건을 **충족하는 노드를 선호**, 없으면 다른 노드에도 배치 가능 | 선택 (`soft`) |

#### 🧠 matchExpressions 상세
* key: 노드 라벨 키
* operator: 비교 연산자 (In, NotIn, Exists, DoesNotExist, Gt, Lt)
* values: 키에 대해 허용할 값 목록

```bash
vi K8S/CH05/node-affinity.yaml
```
```yaml
# node-affinity.yaml
apiVersion: v1                   # 리소스의 API 버전
kind: Pod                        # 생성할 리소스의 종류: Pod
metadata:
  name: node-affinity            # Pod 이름 지정
spec:
  containers:
  - name: nginx                  # 컨테이너 이름
    image: nginx                 # 사용할 이미지 (nginx 웹 서버)
  affinity:                      # affinity 설정 시작
    nodeAffinity:                # 노드 친화성 설정
      requiredDuringSchedulingIgnoredDuringExecution:  # 반드시 만족해야 하는 하드 제약 조건
        nodeSelectorTerms:       # 하나 이상의 조건 그룹
        - matchExpressions:      # 조건 표현식 목록
          - key: disktype        # 노드의 라벨 키
            operator: In         # disktype 값이 아래 목록 중 하나와 일치해야 함
            values:
            - ssd                # 허용되는 값: ssd
```
##### 💡 설명 요약:
* 이 Pod는 disktype=ssd 라벨이 붙은 노드에서만 실행될 수 있습니다.
* 해당 조건을 만족하는 노드가 없으면 Pod은 Pending 상태가 됩니다.
* requiredDuringSchedulingIgnoredDuringExecution은:
  - 스케줄링 시점에는 반드시 조건을 만족해야 하며,
  - 스케줄 이후에는 조건이 바뀌어도 Pod은 계속 실행됨.

```bash
kubectl apply -f K8S/CH05/node-affinity.yaml
# pod/node-affinity created

kubectl get pods node-affinity -o wide
# NAME           READY   STATUS    RESTARTS  AGE   IP          NODE   ..
# node-affinity  1/1     Running   0         19s   10.42.0.8   master ..
```

### 11.4.2 `PodAffinity`
* PodAffinity는 다른 Pod과의 위치 관계를 기준으로, 새로운 Pod의 스케줄링 위치를 제어하는 쿠버네티스의 기능입니다.
* "어떤 조건을 만족하는 다른 Pod과 같은 노드 또는 같은 topology 영역에 배치되기를 원함."
* 예: 웹 서버 Pod은 반드시 같은 노드/zone의 Redis Pod 옆에 배치되게.

```bash
vi K8S/CH05/pod-affinity.yaml
```
```yaml
# pod-affinity.yaml
# 같은 label(app=affinity)을 가진 Pod끼리 동일한 노드에 스케줄링되도록 유도하는 PodAffinity 설정
apiVersion: apps/v1             # 앱 리소스를 위한 API 버전
kind: Deployment                # Deployment로 여러 Pod 생성
metadata:
  name: pod-affinity            # Deployment 이름
spec:
  selector:
    matchLabels:
      app: affinity             # Pod 셀렉터: 레이블이 app=affinity인 것 선택
  replicas: 2                   # 2개의 복제본(Pod) 생성
  template:
    metadata:
      labels:
        app: affinity           # Pod 템플릿의 레이블 설정
    spec:
      containers:
      - name: nginx             # 컨테이너 이름
        image: nginx            # nginx 이미지 사용
      affinity:                 # affinity 규칙 정의
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - affinity      # app=affinity인 다른 Pod 옆에 붙게
            topologyKey: "kubernetes.io/hostname" # 같은 노드에 붙게 지정
```
##### 🔍 주요 필드 설명
| 필드                                                | 설명                                                                                          |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| `requiredDuringSchedulingIgnoredDuringExecution`  | **강제 조건** (만족해야 스케줄됨)                                                                       |
| `preferredDuringSchedulingIgnoredDuringExecution` | **선호 조건** (가능하면 만족, 아니어도 OK)                                                                |
| `labelSelector`                                   | 대상 Pod을 찾기 위한 라벨 조건                                                                         |
| `topologyKey`                                     | affinity가 적용될 **범위** (노드 단위: `kubernetes.io/hostname`, zone: `topology.kubernetes.io/zone`) |


```bash
kubectl apply -f K8S/CH05/pod-affinity.yaml
# deployment.apps/pod-affinity created

kubectl get pod -o wide
# NAME              READY  STATUS    RESTARTS  AGE   IP           NODE
# pod-affinity-xxx  1/1    Running   0         11m   10.42.0.165  worker
# pod-affinity-xxx  1/1    Running   0         11m   10.42.0.166  worker
```

### 11.4.3 `PodAntiAffinity`
* PodAntiAffinity는 Kubernetes에서 다른 특정 Pod와 같은 노드(또는 토폴로지 영역)에 배치되지 않도록 하는 제약 조건입니다. 즉, 서로 떨어져 배치되도록 합니다.

```bash
vi K8S/CH05/pod-antiaffinity.yaml
```
```yaml
# pod-antiaffinity.yaml
# 같은 레이블(app=antiaffinity)을 가진 Pod들끼리는 같은 노드에 배치되지 않도록 설정하는 PodAntiAffinity 예제
apiVersion: apps/v1               # API 버전
kind: Deployment                  # Deployment 리소스: 여러 Pod을 관리
metadata:
  name: pod-antiaffinity          # Deployment 이름
spec:
  selector:
    matchLabels:
      app: antiaffinity           # Pod 셀렉터 정의
  replicas: 2                     # 2개의 Pod 복제본 생성
  template:
    metadata:
      labels:
        app: antiaffinity         # Pod 템플릿에 부여할 라벨
    spec:
      containers:
      - name: nginx               # 컨테이너 이름
        image: nginx              # Nginx 이미지 사용
      affinity:                   # affinity 설정
        podAntiAffinity:         # pod 간 반감성(같이 배치되지 않도록)
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - antiaffinity    # 같은 app=antiaffinity인 Pod들과
            topologyKey: "kubernetes.io/hostname"  # 같은 노드(hostname)에 있지 않게
```
##### 🔍 주요 필드 설명
| 필드                                                | 설명                                                 |
| ------------------------------------------------- | -------------------------------------------------- |
| `requiredDuringSchedulingIgnoredDuringExecution`  | 반드시 해당 조건을 만족해야 함                                  |
| `preferredDuringSchedulingIgnoredDuringExecution` | 되도록 만족하지만 불가능하면 무시                                 |
| `labelSelector`                                   | 회피할 Pod의 라벨 조건 (예: app=web)                        |
| `topologyKey`                                     | 회피 범위 (`kubernetes.io/hostname` → 노드 단위, zone도 가능) |

```bash
kubectl apply -f K8S/CH05/pod-antiaffinity.yaml
# deployment.apps/pod-antiaffinity created

kubectl get pod -o wide
# NAME                 READY  STATUS    RESTARTS AGE  IP           NODE
# pod-antiaffinity-xxx 1/1    Running   0        10s  10.42.0.168  master
# pod-antiaffinity-xxx 1/1    Running   0        11s  10.42.0.167  worker
```

### 11.4.4 `PodAffinity`와 `PodAntiAffinity` 활용법
#### 🔄 PodAffinity vs PodAntiAffinity 비교
| 항목             | PodAffinity                                             | PodAntiAffinity |
| -------------- | ------------------------------------------------------- | --------------- |
| 목적             | 같은 노드/zone에 배치                                          | 다른 노드/zone에 분산  |
| 활용             | Pod 간 밀접한 통신이 필요할 때                                     | 장애 격리, 리소스 분산 등 |
| topologyKey 예시 | `kubernetes.io/hostname`, `topology.kubernetes.io/zone` |                 |

#### cache 서버 설정
* Cache 서버 설정은 일반적으로 Redis나 Memcached 같은 인메모리 캐시 시스템을 **Pod + Service + PVC(Optional)**로 배포해 구성
##### ✅ 대표적인 Cache 서버 옵션
| Cache 서버      | 특징                             |
| ------------- | ------------------------------ |
| **Redis**     | 가장 널리 사용됨, pub/sub, persist 지원 |
| **Memcached** | 매우 빠름, 단순한 key-value 용도에 적합    |

```bash
vi K8S/CH05/redis-cache.yaml
```
```yaml
# redis-cache.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-cache  # Deployment 리소스 이름
spec:
  selector:
    matchLabels:
      app: store  # 이 라벨을 가진 Pod을 선택
  replicas: 2       # 2개의 Redis Pod 복제본 생성
  template:
    metadata:
      labels:
        app: store  # 각 Pod에 라벨 부여
    spec:
      affinity:
        # 같은 라벨(app=store)을 가진 Pod끼리는 서로 다른 노드에 스케줄링
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - store
            topologyKey: "kubernetes.io/hostname"  # hostname이 다르면 다른 노드
      containers:
      - name: redis-server
        image: redis   # Redis 이미지 사용
        ports:
        - containerPort: 6379  # Redis 기본 포트
```
###### 📌 설명 요약
* replicas: 2: Redis 서버를 2개 실행
* podAntiAffinity: 같은 노드에 Redis Pod이 동시에 스케줄되지 않도록 설정 → 고가용성 확보
* topologyKey: "kubernetes.io/hostname": 노드 단위 분산
* containerPort: 6379: Redis 기본 포트 노출 (필수는 아니지만 명시 추천)


#### web 서버 설정
* web-server.yaml은 하기의 스케줄링 정책이 잘 적용되어 있다.
  - web-store 앱끼리는 서로 다른 노드에 (PodAntiAffinity로 분산 배치)
  - store 라벨 가진 캐시 서버와는 같은 노드에 (PodAffinity로 밀집 배치)

```bash
vi K8S/CH05/web-server.yaml
```
```yaml
# web-server.yaml
apiVersion: apps/v1                     # API 버전: Deployment는 apps/v1 사용
kind: Deployment                        # 리소스 종류: Deployment
metadata:
  name: web-server                      # Deployment 이름: web-server
spec:
  selector:
    matchLabels:
      app: web-store                    # Pod 선택기: app=web-store 라벨을 가진 Pod 선택
  replicas: 2                          # 복제본 수: 2개 Pod 생성
  template:
    metadata:
      labels:
        app: web-store                  # Pod에 붙일 라벨: app=web-store
    spec:
      affinity:
        # web 서버끼리 멀리 스케줄링
        # app=web-store 라벨을 가진 Pod끼리 멀리 스케줄링
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:   # 필수 조건 (스케줄링 시 강제 적용, 실행 중 무시 가능)
          - labelSelector:                                  # 라벨 선택자
              matchExpressions:                            # 조건
              - key: app                                   # app 키가
                operator: In                              # 지정된 값들 중 하나여야 함
                values:
                - web-store                                # 값: web-store
            topologyKey: "kubernetes.io/hostname"         # 같은 노드(hostname)에는 배치하지 말 것
        # web-cache 서버끼리 가까이 스케줄링 
        # app=store 라벨을 가진 Pod끼리 가까이 스케줄링
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:   # 필수 조건, 실행 중 무시 가능
          - labelSelector:                                  # 라벨 선택자
              matchExpressions:                            # 조건
              - key: app                                   # app 키가
                operator: In                              # 지정된 값 중 하나여야 함
                values:
                - store                                   # 값: store (캐시 서버 라벨)
            topologyKey: "kubernetes.io/hostname"         # 같은 노드(hostname)에 배치할 것
      containers:
      - name: web-app                                   # 컨테이너 이름: web-app
        image: nginx                                    # 컨테이너 이미지: nginx 공식 이미지
```

```bash
kubectl apply -f K8S/CH05/redis-cache.yaml
# deployment.app/redis-cache created

kubectl apply -f K8S/CH05/web-server.yaml
# deployment.app/web-server created

kubectl get pod -owide
# NAME             READY  STATUS    RESTARTS  AGE     IP            NODE
# redis-cache-xxx  1/1    Running   0         10s     10.42.0.151   master
# redis-cache-xxx  1/1    Running   0         10s     10.42.0.152   worker
# web-server-xxxx  1/1    Running   0         11s     10.42.0.153   master
# web-server-xxxx  1/1    Running   0         11s     10.42.0.154   worker
```

### Clean up

```bash
kubectl delete deploy --all
kubectl delete pod --all
```