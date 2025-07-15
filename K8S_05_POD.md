# 5. POD 살펴보기

## A. Pod란 무엇인가?

>* Pod는 가장 기본적이고 중요한 개념 중 하나.
>* Pod는 쿠버네티스에서 애플리케이션을 실행하는 가장 작은 단위
>* Pod는 하나 이상의 컨테이너를 포함할 수 있다.

### 📌 1. Pod란 무엇인가?
* Pod는 쿠버네티스에서 생성되고 관리되는 최소 배포 단위(Minimal Deployable Unit) 입니다. 각 Pod는:
* 하나 이상의 컨테이너(대개 하나)를 포함하고
* 동일한 네트워크 네임스페이스(IP, 포트)를 공유하며
* 같은 스토리지 볼륨을 공유하고
* 함께 스케줄되어 노드에 배치됩니다.

#### ✅ 주요 특징
| 항목      | 설명                                   |
| ------- | ------------------------------------ |
| 컨테이너 그룹 | 하나 이상의 컨테이너를 포함할 수 있으며, 컨테이너는 함께 실행됨 |
| 공유 자원   | IP 주소, 포트 네임스페이스, localhost, 볼륨 등 공유 |
| 짧은 생명주기 | 필요 시 생성되고, 작업이 끝나면 종료되는 것이 일반적       |
| 논리적 호스트 | 컨테이너들이 같은 호스트에서 실행되는 것처럼 동작          |


### 📌 2. Pod 구조
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
    - name: app-container
      image: nginx
      ports:
        - containerPort: 80
```

#### 주요 구성 요소 설명
| 항목           | 설명                      |
| ------------ | ----------------------- |
| `apiVersion` | 리소스 버전, 예: `v1`         |
| `kind`       | 리소스 종류: `Pod`           |
| `metadata`   | 이름, 라벨 등 메타 정보          |
| `spec`       | Pod 내부의 스펙 (컨테이너, 볼륨 등) |
| `containers` | 하나 이상의 컨테이너 정의          |


### 📌 3. Pod의 생명주기 (Lifecycle)
1. Pending : 스케줄러가 Pod를 노드에 할당하기 전 상태
2. Running : 컨테이너가 실행 중
3. Succeeded : 모든 컨테이너가 정상 종료된 상태 (종료 코드 0)
4. Failed :하나 이상의 컨테이너가 비정상 종료됨
5. Unknown : 상태를 알 수 없음 (노드와 통신 불가 등)

### 📌 4. Pod의 용도
| 용도         | 예시                                             |
| ---------- | ---------------------------------------------- |
| 단일 컨테이너 실행 | 대부분의 경우, 하나의 컨테이너만 포함                          |
| 사이드카 패턴 사용 | 로깅, 프록시, 모니터링용 컨테이너를 함께 실행                     |
| 멀티 컨테이너 통합 | tightly coupled 서비스 (예: web + cache container) |


### 📌 5. Pod vs Container
| 항목    | Pod                 | Container               |
| ----- | ------------------- | ----------------------- |
| 정의    | 쿠버네티스에서 관리되는 단위     | 실행 단위                   |
| 포함 관계 | 하나 이상의 컨테이너 포함      | Pod 내부 구성 요소            |
| 네트워킹  | Pod 단위로 IP 할당       | Pod 내부에서는 localhost로 통신 |
| 관리    | 쿠버네티스가 Pod 단위로 스케줄링 | 개별 컨테이너는 직접 제어되지 않음     |

### 📌 6. Pod의 네트워크 모델
* 각 Pod는 클러스터 내부 고유 IP를 가짐
* Pod 내부 컨테이너들은 localhost를 통해 서로 통신
* Pod 간 통신은 Pod IP 또는 서비스(Service) 를 통해 이루어짐

### 📌 7. Pod 관리 방식
Pod는 일반적으로 직접 생성하기보다는 다음 리소스를 통해 관리됩니다:
* Deployment: 무중단 배포 및 업데이트
* ReplicaSet: 복제본 유지
* DaemonSet: 모든 노드에 하나씩 배포
* StatefulSet: 상태 유지 필요한 Pod 관리

### 📌 8. Pod 관련 명령어 (kubectl)
```bash
# Pod 목록 보기
kubectl get pods

# 특정 Pod 상세 정보
kubectl describe pod <pod-name>

# Pod 생성
kubectl apply -f pod.yaml

# Pod 삭제
kubectl delete pod <pod-name>

# Pod 로그 확인
kubectl logs <pod-name>

# Pod 내부 접속
kubectl exec -it <pod-name> -- /bin/sh
```

### 📌 9. 실전 예제: nginx 웹서버 Pod 생성
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

### 10. yaml파일 옵션

Pod YAML은 크게 3부분으로 구성됩니다:

1. apiVersion, kind, metadata
2. spec
3. spec.containers 내부 구성

아래는 각 섹션별로 주요 옵션과 설명, 예시까지 정리한 내용입니다.

#### ✅ 1. 상위 메타 정보
| 필드                     | 설명                     | 예시                    |
| ---------------------- | ---------------------- | --------------------- |
| `apiVersion`           | 사용할 Kubernetes API 버전  | `v1`                  |
| `kind`                 | 리소스 종류                 | `Pod`                 |
| `metadata.name`        | Pod 이름                 | `my-pod`              |
| `metadata.labels`      | Pod에 적용할 라벨(key-value) | `app: nginx`          |
| `metadata.annotations` | 추가 정보 (비식별 메타데이터)      | `annotation: "value"` |
| `metadata.namespace`   | 속할 네임스페이스              | `default`             |

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: nginx
  annotations:
    description: "Nginx web server"
 ```

#### ✅ 2. spec 레벨 (Pod 사양)
| 필드                        | 설명                                      | 예시                       |
| ------------------------- | --------------------------------------- | ------------------------ |
| `spec.containers`         | 실행할 컨테이너 목록                             | 필수                       |
| `spec.restartPolicy`      | 재시작 정책 (`Always`, `OnFailure`, `Never`) | 기본: `Always`             |
| `spec.nodeSelector`       | Pod가 배치될 노드 선택                          | `disktype: ssd`          |
| `spec.volumes`            | Pod 전체에서 공유할 볼륨 정의                      | `emptyDir`, `hostPath` 등 |
| `spec.hostNetwork`        | 호스트 네트워크 사용 여부                          | `true/false`             |
| `spec.dnsPolicy`          | DNS 정책 (`ClusterFirst`, `Default`)      | 기본: `ClusterFirst`       |
| `spec.serviceAccountName` | 사용할 서비스 어카운트                            | `default`                |
| `spec.affinity`           | 노드/Pod 간 배치 제어                          | 스케줄링 정책                  |
| `spec.tolerations`        | Taints 허용 설정                            | Tolerate 된 노드에서만 실행      |


```yaml
spec:
  restartPolicy: OnFailure
  nodeSelector:
    disktype: ssd
  hostNetwork: false
  serviceAccountName: my-service-account
```

#### ✅ 3. containers 배열 내부 옵션
💡 필수 필드
| 필드      | 설명      | 예시             |
| ------- | ------- | -------------- |
| `name`  | 컨테이너 이름 | `nginx`        |
| `image` | 사용할 이미지 | `nginx:latest` |

💡 주요 선택 필드
| 필드                | 설명                                  | 예시                        |
| ----------------- | ----------------------------------- | ------------------------- |
| `ports`           | 노출할 포트                              | `containerPort: 80`       |
| `env`             | 환경변수                                | `name: VAR, value: "123"` |
| `command`         | 실행할 커맨드 (엔트리포인트 override)           | `["/bin/sh", "-c"]`       |
| `args`            | 커맨드에 넘길 인자                          | `["echo hello"]`          |
| `volumeMounts`    | 볼륨 마운트 위치 설정                        | `mountPath: /data`        |
| `resources`       | CPU/메모리 자원 설정                       | `requests`, `limits`      |
| `livenessProbe`   | 살아있는지 확인 (헬스 체크)                    | HTTP, TCP, exec 방식        |
| `readinessProbe`  | 준비 완료 상태 확인                         | 서비스에 포함 여부 결정             |
| `imagePullPolicy` | 이미지 풀 정책 (`Always`, `IfNotPresent`) | 기본: `IfNotPresent`        |

```yaml
containers:
  - name: nginx
    image: nginx:1.25
    ports:
      - containerPort: 80
    env:
      - name: ENV_MODE
        value: production
    volumeMounts:
      - name: my-volume
        mountPath: /usr/share/nginx/html
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

#### ✅ 4. volumes 섹션
Pod 내에서 공유할 수 있는 디스크 공간 정의입니다.

| 유형                      | 설명                    |
| ----------------------- | --------------------- |
| `emptyDir`              | 임시 디렉토리 (Pod 종료 시 삭제) |
| `hostPath`              | 호스트 노드의 디렉토리 사용       |
| `configMap`             | ConfigMap 데이터를 마운트    |
| `secret`                | 시크릿 데이터를 마운트          |
| `persistentVolumeClaim` | PVC 기반 스토리지 마운트       |

```yaml

volumes:
  - name: my-volume
    emptyDir: {}
```

#### ✅ 전체 예시: 다양한 옵션 포함 Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: full-options-pod
  labels:
    app: demo
spec:
  restartPolicy: OnFailure
  nodeSelector:
    disktype: ssd
  volumes:
    - name: html-volume
      emptyDir: {}
  containers:
    - name: nginx
      image: nginx:1.25
      ports:
        - containerPort: 80
      volumeMounts:
        - name: html-volume
          mountPath: /usr/share/nginx/html
      env:
        - name: ENV
          value: production
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "200m"
          memory: "256Mi"
      livenessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 5
        periodSeconds: 10
```

#### ✅ 기타 고급 옵션
| 필드                              | 설명                     |
| ------------------------------- | ---------------------- |
| `initContainers`                | 메인 컨테이너 실행 전 초기 작업 수행  |
| `securityContext`               | 보안 컨텍스트 설정 (UID, 권한 등) |
| `terminationGracePeriodSeconds` | 종료 전 대기 시간             |
| `schedulerName`                 | 사용자 정의 스케줄러 이름         |

### 11. kind 필드
>kind는 Kubernetes YAML 파일에서 생성하려는 리소스(Resource)의 종류를 지정하는 필드
>예: kind: Pod, kind: Deployment, kind: Service 등

쿠버네티스에서는 다양한 리소스를 kind로 정의하고 있으며, 이들을 카테고리별로 정리

#### 📌 1. Workloads (애플리케이션 실행 관련)
| kind          | 설명                                  |
| ------------- | ----------------------------------- |
| `Pod`         | 하나 이상의 컨테이너를 포함하는 기본 실행 단위          |
| `ReplicaSet`  | 지정된 수의 Pod 복제본을 유지                  |
| `Deployment`  | 무중단 업데이트 가능한 Pod 및 ReplicaSet 관리    |
| `StatefulSet` | 상태가 필요한 애플리케이션용 리소스 (고유 네트워크, 볼륨 등) |
| `DaemonSet`   | 각 노드에 하나씩 Pod 배포                    |
| `Job`         | 한 번 실행 후 종료되는 작업                    |
| `CronJob`     | 정기적으로 실행되는 Job                      |


#### 📌 2. Service 및 네트워킹
| kind                     | 설명                                              |
| ------------------------ | ----------------------------------------------- |
| `Service`                | Pod에 대한 안정적인 네트워크 접근 제공 (ClusterIP, NodePort 등) |
| `Ingress`                | 외부에서 클러스터 내부로 HTTP/HTTPS 접근 라우팅                 |
| `IngressClass`           | Ingress 컨트롤러의 설정 정의                             |
| `Endpoint` / `Endpoints` | Service가 라우팅할 실제 Pod IP 정보                      |
| `NetworkPolicy`          | Pod 간 통신 정책 제어 (허용/차단)                          |

#### 📌 3. Configuration (설정 및 데이터)
| kind                          | 설명                                |
| ----------------------------- | --------------------------------- |
| `ConfigMap`                   | 환경설정 파일, 문자열 데이터를 Key-Value로 저장   |
| `Secret`                      | 비밀번호, 토큰 등 민감 정보 저장 (Base64 인코딩됨) |
| `PersistentVolume` (PV)       | 클러스터 수준에서의 스토리지 자원 정의             |
| `PersistentVolumeClaim` (PVC) | Pod에서 사용할 스토리지 요청                 |
| `StorageClass`                | 동적 볼륨 프로비저닝을 위한 스토리지 종류 정의        |
| `VolumeAttachment`            | 외부 스토리지를 노드에 연결하는 객체              |

#### 📌 4. Cluster 관리 관련
| kind                  | 설명                              |
| --------------------- | ------------------------------- |
| `Namespace`           | 리소스의 논리적 구분을 위한 격리 공간           |
| `Node`                | 쿠버네티스 클러스터의 작업 노드               |
| `Event`               | 클러스터 내 이벤트 (오류, 상태 변화 등)        |
| `LimitRange`          | Namespace 내 리소스 제한 설정           |
| `ResourceQuota`       | 전체 리소스 사용량 제한                   |
| `PodDisruptionBudget` | 유지 가능한 최소 Pod 수를 지정 (업데이트 시 사용) |

#### 📌 5. RBAC (권한 관리)
| kind                 | 설명                      |
| -------------------- | ----------------------- |
| `ServiceAccount`     | Pod가 사용할 수 있는 서비스 계정    |
| `Role`               | 네임스페이스 단위의 권한 정의        |
| `ClusterRole`        | 클러스터 전체 권한 정의           |
| `RoleBinding`        | Role과 사용자, 그룹 매핑        |
| `ClusterRoleBinding` | ClusterRole과 사용자, 그룹 매핑 |

#### 📌 6. Custom Resource 관련
| kind                             | 설명                   |
| -------------------------------- | -------------------- |
| `CustomResourceDefinition` (CRD) | 사용자 정의 리소스 생성        |
| `CustomResource` (사용자 정의)        | CRD를 기반으로 생성된 개별 리소스 |

예:
```yaml
kind: MyCustomApp
apiVersion: mygroup.io/v1
```

#### 📌 7. 기타 자주 사용되는 리소스
| kind                   | 설명                                   |
| ---------------------- | ------------------------------------ |
| `Binding`              | 리소스를 특정 객체에 바인딩 (거의 자동으로 사용됨)        |
| `Lease`                | 리더 선출 등에 사용되는 경량 리소스                 |
| `TokenReview`          | 토큰 인증 검증을 위한 리소스                     |
| `CSIDriver`, `CSINode` | CSI (Container Storage Interface) 설정 |


#### ✅ 전체 요약 표
| 카테고리          | 예시 kind                                              |
| ------------- | ---------------------------------------------------- |
| Workloads     | `Pod`, `Deployment`, `StatefulSet`, `Job`, `CronJob` |
| Networking    | `Service`, `Ingress`, `NetworkPolicy`                |
| Configuration | `ConfigMap`, `Secret`, `PVC`, `StorageClass`         |
| Cluster 관리    | `Namespace`, `Node`, `LimitRange`, `Event`           |
| RBAC          | `Role`, `ClusterRole`, `ServiceAccount`              |
| 사용자 정의        | `CustomResourceDefinition`, `MyCustomKind`           |

## 12. pod vs node

### ✅ 1. 기본 개념
| 항목    | **Pod**                        | **Node**                          |
| ----- | ------------------------------ | --------------------------------- |
| 정의    | 쿠버네티스에서 실행되는 **애플리케이션의 최소 단위** | 쿠버네티스 클러스터를 구성하는 **물리적 또는 가상 서버** |
| 역할    | 애플리케이션 컨테이너를 담고 실행             | Pod를 실제로 실행하는 인프라 자원              |
| 구성 요소 | 하나 이상의 컨테이너 + 공유 네트워크 + 볼륨     | Kubelet, 컨테이너 런타임, kube-proxy 등   |
| 생성 위치 | Node 위에 스케줄링되어 실행됨             | 클러스터의 일원으로 Pod 실행을 담당             |


### ✅ 2. 아키텍처 상의 위치 관계
```plaintext
[Cluster]
   |
   |-- [Node 1]
   |     |-- [Pod A] -> Container(s)
   |     |-- [Pod B]
   |
   |-- [Node 2]
         |-- [Pod C]
         |-- [Pod D]
```
* Pod는 항상 Node 위에서 실행됩니다.
* 하나의 Node에는 여러 개의 Pod가 올라갈 수 있습니다.

### ✅ 3. 기능적 차이
| 구분           | Pod                                             | Node                           |
| ------------ | ----------------------------------------------- | ------------------------------ |
| 생성 방법        | `kubectl apply -f pod.yaml` 또는 `Deployment`로 생성 | 클러스터 초기 구성 또는 자동 확장            |
| 생명 주기        | 애플리케이션 실행 단위로 자주 생성/삭제됨                         | 일반적으로 고정되어 있고 장기간 유지됨          |
| 네트워크         | Pod 단위로 **IP 주소 부여**                            | Node에도 자체 IP가 있고, 클러스터 네트워크 구성 |
| 자원           | Pod 단위로 CPU/Memory 요청/제한 가능                     | Node는 전체 리소스를 보유한 실행 환경        |
| 컨트롤러에 의해 관리됨 | 예: `Deployment`, `ReplicaSet`                   | `Kubelet` 데몬에 의해 제어됨           |


### ✅ 4. 실생활에 비유하자면…
* Node = 아파트 건물
* Pod = 아파트 건물 안의 하나의 집(가구)
* node > pod > container 관계

### ✅ 5. 기술 요소 비교
| 항목        | Pod                        | Node                       |
| --------- | -------------------------- | -------------------------- |
| 실행 환경     | Node에서 실행됨                 | 자체적인 물리적/가상 머신             |
| 구성 요소     | 컨테이너, 볼륨, IP               | OS, kubelet, 컨테이너 런타임      |
| 상태 확인 명령어 | `kubectl get pods`         | `kubectl get nodes`        |
| 삭제 시 동작   | 애플리케이션 종료                  | 모든 Pod가 재스케줄되거나 장애 발생 가능   |
| IP        | 고유한 Pod IP를 갖고, 클러스터 내부 통신 | 고유한 Node IP, 외부 통신 시 관문 역할 |

### ✅ 6. 예시 명령어 비교
```bash
# 전체 노드 보기
kubectl get nodes

# 전체 파드 보기
kubectl get pods -A

# 특정 노드에 스케줄된 파드 보기
kubectl get pods --all-namespaces --field-selector spec.nodeName=<노드이름>
```

### ✅ 요약
| 항목    | Pod                | Node         |
| ----- | ------------------ | ------------ |
| 무엇인가? | 컨테이너 묶음 (최소 실행 단위) | Pod가 실행되는 서버 |
| 포함 관계 | Node 위에 배포됨        | 여러 Pod를 호스팅  |
| 수명    | 짧거나 유동적            | 상대적으로 오래 유지  |
| 용도    | 애플리케이션 실행          | 인프라 구성 단위    |


## B. 실습

### 5.1 Pod 소개

#### 5.1.1 Pod의 특징
1. 1개이상의 컨테이너 실행 : 보톤은 1개의 pod에 1개의 컨테이너를 실행, 3개이상 가능하지만 실질적으로 3개이상 넘어가는 경우는 거의 없다.
1. 동일 노드에 할당 : pod내에 실행되는 컨테이너들은 반드시 동일한 노드에 할당하며 동일한 생명주기를 갖는다.
1. 고유의 Pod IP : 
   * Pod IP는 노드IP와는 별개로 클러스터 내에서 접근가능한 고유의 IP를 할당 받는다.
   * 일반적으로 포트포워딩을 이용, 노드IP와 포워딩포트를 이용하여 접근
   * 쿠버네티스에서는 다른 노드에 위치한 Pod라 할지라도 NAT통신없이도 POD의 고유IP를 이용하여 접근가능
1. IP 공유 : Pod내의 컨테이너끼리는 localhost를 통해 서로 네트워크접근이 가능, 포트를 이용 구분
1. volume 공유 
   * Pod안의 컨테이너들은 동일한 볼륨과 연결가능
   * 파일시스템기반 파일을 공우

##### --dry-run과 -o yaml을 조합하면 pod를 실제로 생성하지 않고 템플릿파일을 생성할 수 있다.
* 리소스를 실제로 만들지 않고 YAML만 생성할 때 자주 사용

##### ✅ 개념 정리
###### 🧪 --dry-run
* 리소스를 실제로 생성하지 않고 시뮬레이션만 합니다.
* --dry-run=client: 클라이언트에서만 시뮬레이션 (로컬 검증)
* --dry-run=server: 서버에서 유효성 검사까지 수행 (쿠버네티스 API 서버에 접속)

######🧾 -o yaml
* 출력을 YAML 형식으로 보여줍니다.
* JSON도 가능: -o json

##### ✅ 조합 예시
######▶️ 목적: 리소스 YAML을 만들되, 실제로는 생성하지 않기
```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml
```

###### 설명:
* kubectl run nginx: 이름이 nginx인 Pod 생성 명령
* --image=nginx: nginx 이미지를 사용
* --restart=Never: Pod로 생성 (Deployment가 아님)
* --dry-run=client: 실제로는 생성하지 않음
* -o yaml: 결과를 YAML로 출력

#####📤 출력 결과:
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

##### ✅ 파일로 저장하려면?
```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml > nginx-pod.yaml
```
* 이렇게 하면 nginx-pod.yaml 파일이 생성되고, 나중에 다음과 같이 적용할 수 있습니다:

```bash
kubectl apply -f nginx-pod.yaml
```

##### ✅ 다른 리소스 예시들
* Deployment 생성 YAML만 출력
```bash
kubectl create deployment webapp --image=nginx --dry-run=client -o yaml
```

* Service YAML 생성 (클러스터IP 서비스)
```bash
kubectl expose pod nginx --port=80 --target-port=80 --name=nginx-service --dry-run=client -o yaml
```

##### ✅ 요약
| 옵션                 | 역할                         |
| ------------------ | -------------------------- |
| `--dry-run=client` | 리소스를 생성하지 않고 명령어만 테스트 (로컬) |
| `--dry-run=server` | 쿠버네티스 API 서버에서 유효성 검사까지 수행 |
| `-o yaml`          | 결과를 YAML 형태로 출력            |
| `> file.yaml`      | 출력 결과를 파일로 저장              |


##### 실습

```bash
# mynginx.yaml 이라는 YAML 정의서 생성
kubectl run mynginx --image=nginx --dry-run=client -o yaml > mynginx.yaml
vi mynginx.yaml # yaml파일 자동생성
```

```yaml
# mynginx.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: mynginx
  name: mynginx
spec:
  containers:
  - image: nginx
    name: mynginx
  restartPolicy: Never
```

```bash
kubectl apply -f mynginx.yaml
# pod/myn

kgp # kubectl get pods
kubectl exec -it mynginx -- /bin/sh
cat /usr/share/nginx/html/index.html
exit
```
### 5.2 labeling 시스템
>라벨은 쿠버네티스 리소스를 논리적으로 그룹화하고, **선택(Selector)**하거나 조직화하거나 자동화할 수 있게 해주는 핵심 도구

#### ✅ 1. 라벨(Label)이란?
* Key-Value 형식의 메타데이터
* 리소스(예: Pod, Deployment, Node, Service 등)에 임의로 부착 가능
* 쿠버네티스에서 리소스를 필터링, 선택, 그룹화할 때 사용됨
* 라벨은 오직 조회 및 제어 목적으로만 사용 (리소스 동작에는 직접 영향 없음)

#### ✅ 2. 라벨 형식
```yaml
labels:
  app: nginx
  tier: frontend
  environment: production
```

#### ▶️ 규칙
| 항목     | 설명                                     |
| ------ | -------------------------------------- |
| Key    | 문자열이며 선택적으로 prefix 포함 가능 (`team/app`)  |
| Value  | 문자열 (공백, 특수문자 없이)                      |
| Prefix | DNS 형식으로, 선택적 (예: `mycompany.com/app`) |

🎯 예시 키/값:
* app: nginx
* tier: backend
* env: staging
* team: devops

#### ✅ 3. 라벨 추가 방법
🎯 리소스 생성 시
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: nginx
    env: dev
```

🎯 리소스 생성 후 추가
```bash
kubectl label pod my-pod version=1.0
```

🎯 라벨 수정
```bash
kubectl label pod my-pod env=prod --overwrite
```

🎯 라벨 삭제
```bash
kubectl label pod my-pod version-
```

#### ✅ 4. 라벨 셀렉터(Label Selector)
라벨을 이용해 리소스를 선택할 수 있습니다.

🔹 Equality-based Selector (기본)
```bash
kubectl get pods -l app=nginx
kubectl get pods -l env=prod,tier=frontend
```

🔹 Set-based Selector
```bash
kubectl get pods -l 'env in (prod,staging)'
kubectl get pods -l 'app notin (redis,mysql)'
kubectl get pods -l '!deprecated'
```

#### ✅ 5. 활용 예시
🎯 서비스(Service)에서 라벨로 대상 선택
```yaml
# Service
spec:
  selector:
    app: nginx
    tier: frontend
```
→ 이 라벨을 가진 Pod들만 로드밸런싱 대상이 됨

🎯 ReplicaSet에서 Pod 선택
```yaml
# ReplicaSet
spec:
  selector:
    matchLabels:
      app: nginx
```
→ 해당 라벨의 Pod만 관리 대상으로 간주

🎯 Helm, ArgoCD, GitOps에서도 라벨 사용
* 어떤 앱이 어느 방식으로 배포됐는지 추적 가능
* 예: helm.sh/chart: mychart-1.2.3, argocd.argoproj.io/instance: myapp

#### ✅ 6. 라벨 vs 어노테이션 비교
| 항목 | **Label**     | **Annotation**           |
| -- | ------------- | ------------------------ |
| 목적 | 선택, 필터, 그룹화   | 정보 기록, 툴이 읽는 메타데이터       |
| 포맷 | Key-Value     | Key-Value (값이 더 길 수 있음)  |
| 용도 | 셀렉터/컨트롤러에서 사용 | 설명, 빌드번호, 툴 설정 등         |
| 예시 | `app=nginx`   | `checksum/config=abc123` |

#### ✅ 7. 실전 예: kubectl get로 라벨 보기
```bash
kubectl get pods --show-labels

# 라벨 컬럼 지정 출력
kubectl get pods -L app,env
```

#### ✅ 8. 고급 활용: 라벨 기반 자동화
* CI/CD 파이프라인: environment=dev 라벨 가진 리소스만 배포
* 모니터링/로깅 툴: team=devops인 리소스만 수집
* Taint/Toleration과 결합해 노드 배치 제어
* 네트워크 정책에서 라벨 기반으로 통신 허용

#### ✅ 결론: 라벨은 쿠버네티스의 핵심 기능
| 장점     | 설명                               |
| ------ | -------------------------------- |
| 유연성    | 어떤 리소스에도 부착 가능, 자유롭게 조합 가능       |
| 관리 편의성 | 셀렉터로 그룹화, 검색, 자동화 가능             |
| 확장성    | Helm, GitOps, CI/CD 파이프라인과 연동 가능 |

#### 8. 실습
 
###### 1. 라벨정보부여
```bash
# `label` 명령을 이용하는 방법
kubectl label pod mynginx hello=world
# pod/mynginx labeled

kubectl label pod mynginx hello=world --overwrite
kubectl get pod mynginx -oyaml
```

###### 2. 선언형 명령을 이용하는 방법
```bash
# pod 삭제
kubectl delete pod mynginx

# pod 생성
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  # hello=world 라벨 지정
  labels:
    hello: world  
    run: mynginx
  name: mynginx
spec:
  containers:
  - image: nginx
    name: mynginx
EOF

# 
kubectl get pod mynginx -L run
# NAME      READY   STATUS    RESTARTS   AGE    RUN
# mynginx   1/1     Running   0          7m4s   mynginx

kubectl get pod mynginx -L xxx
# NAME      READY   STATUS    RESTARTS   AGE    XXX
# mynginx   1/1     Running   0          7m4s         <- XXX값은 없음

# 모든 라벨 정보 표시
kubectl get pod mynginx --show-labels
# NAME      READY   STATUS    RESTARTS   AGE     LABELS
# mynginx   1/1     Running   0          9m29s   hello=world,run=mynginx
```


###### 3 라벨을 이용한 조건 필터링

```bash
# 새로운 yournginx Pod 생성
kubectl run yournginx --image nginx
# pod/yournginx created

# key가 run인 Pod들 출력
kubectl get pod -l run
# NAME       READY   STATUS    RESTARTS   AGE
# mynginx    1/1     Running   0          19m
# yournginx  1/1     Running   0          20m

# key가 run이고 value가 mynginx인 Pod 출력
kubectl get pod -l run=mynginx
# NAME      READY   STATUS    RESTARTS   AGE
# mynginx   1/1     Running   0          19m

# key가 run이고 value가 yournginx Pod 출력
kubectl get pod -l run=yournginx
# NAME       READY   STATUS    RESTARTS   AGE
# yournginx  1/1     Running   0          20m
```

###### 4 `nodeSelector`를 이용한 노드 선택
* Pod에 라벨이 붙은 노드 중 특정 노드에만 스케줄되도록 지정하는 기능
* 즉, Pod가 특정 조건의 노드에만 배포되게 하려면 노드에 라벨을 붙이고, Pod에 해당 라벨을 nodeSelector로 명시

###### ✅ 1. 사용 예시
▶️ Step 1: 노드에 라벨 추가
```bash
kubectl label node node01 disktype=ssd
```

▶️ Step 2: Pod YAML에 nodeSelector 추가
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  nodeSelector:
    disktype: ssd
  containers:
    - name: nginx
      image: nginx
```
* 이렇게 하면 이 Pod는 disktype=ssd 라벨이 붙은 노드에만 스케줄됩니다.

###### ✅ 2. 라벨 확인 및 삭제
* 현재 노드 라벨 확인
```bash
kubectl get nodes --show-labels

# 또는 특정 노드의 라벨만 보기:
kubectl describe node master | grep Labels
kubectl describe node worker1 | grep Labels
kubectl describe node worker2 | grep Labels
kubectl describe node worker3 | grep Labels

# 라벨 삭제
kubectl label node master disktype-
```

###### ✅ 3. 실전 활용 예
예: GPU 노드 전용 작업 스케줄링
```bash
kubectl label node gpu-node accelerator=nvidia
```

```yaml
spec:
  nodeSelector:
    accelerator: nvidia
```

###### ✅ 4. 주의할 점
| 항목            | 설명                                                        |
| ------------- | --------------------------------------------------------- |
| ❌ 강제는 아님      | 조건에 맞는 노드가 없으면 Pod는 **Pending 상태**가 됨                     |
| 🎯 정확한 일치만 허용 | `nodeSelector`는 단순 **Key=Value** 조건만 가능 (Set/NotIn 조건 없음) |
| ✅ 단일 조건만 지원   | 다중 조건은 여러 key-value 쌍으로 나열 가능                             |

```yaml
nodeSelector:
  disktype: ssd
  zone: us-central1-a
```

###### ✅ 5. 한계점과 대안
| 방식                   | 설명                             | 복잡한 조건 지원 |
| -------------------- | ------------------------------ | --------- |
| `nodeSelector`       | 단순 Key=Value 매칭                | ❌ 제한적     |
| `nodeAffinity`       | 더 정교한 조건 (In, NotIn, Exists 등) | ✅ 가능      |
| `taints/tolerations` | 노드에서 특정 Pod만 허용                | ✅ 제어 가능   |


###### ✅ 6. nodeSelector vs nodeAffinity
| 항목     | nodeSelector | nodeAffinity                                           |
| ------ | ------------ | ------------------------------------------------------ |
| 복잡한 조건 | 불가           | 가능 (In, NotIn, Exists)                                 |
| 연산자 사용 | 없음           | 지원 (`matchExpressions`)                                |
| 표현 방식  | 단순 key=value | `requiredDuringSchedulingIgnoredDuringExecution` 구조 사용 |

###### ✅ 예시 정리: 다양한 nodeSelector 구성
* 여러 조건을 동시에
```yaml
nodeSelector:
  disktype: ssd
  zone: us-east1

# 조건이 맞는 노드가 없으면?
# Pod는 Pending 상태가 되고 스케줄되지 않음
# 해결: 라벨 맞게 수정하거나 nodeSelector를 제거해야 함
```

###### ✅ 요약
| 항목    | 내용                                    |
| ----- | ------------------------------------- |
| 기능    | Pod가 특정 라벨을 가진 노드에만 배치되도록 제한          |
| 사용 방법 | Pod spec에 `nodeSelector` 추가           |
| 요구사항  | 노드에 해당 라벨이 존재해야 함                     |
| 한계    | 단순한 조건만 가능 → 고급 조건은 `nodeAffinity` 사용 |

###### ✅ 실습

```bash
kubectl get node
kubectl get node  --show-labels
# NAME     STATUS   ROLES    AGE   VERSION        LABELS
# master   Ready    master   14d   v1.18.6+k3s1   beta.kubernetes.io/...
# worker   Ready    <none>   14d   v1.18.6+k3s1   beta.kubernetes.io/...
```

```bash
kubectl label node master disktype=ssd
# node/master labeled

kubectl label node worker1 disktype=hdd
kubectl label node worker2 disktype=hdd
kubectl label node worker3 disktype=hdd
# node/worker labeled
```

```bash
# disktype 라벨 확인
kubectl get node --show-labels | grep disktype
# NAME     STATUS   ROLES    AGE   VERSION        LABELS
# master   Ready    master   14d   v1.18.6+k3s1   ....disktype=ssd,....
# worker   Ready    <none>   14d   v1.18.6+k3s1   ....disktype=hdd,....

kubectl get node --show-labels | grep disktype=ssd
kubectl get node --show-labels | grep disktype=hdd

mkdir ~/K8S/CH05
vi ~/K8S/CH05/node-selector.yaml
```

```yaml
# node-selector.yaml
apiVersion: v1
kind: Pod
metadata:
  name: node-selector
spec:
  containers: 
  - name: nginx
    image: nginx
  # 특정 노드 라벨 선택
  nodeSelector:
    disktype: ssd
```

```bash
kubectl apply -f ~/K8S/CH05/node-selector.yaml
# pod/node-selector created
```

```bash
# master node에서 실행중이 pod(node-selector)
kubectl get pod node-selector -o wide
# NAME           READY   STATUS    RESTARTS   AGE   IP          NODE   
# node-selector  1/1     Running   0          19s   10.42.0.8   master

# node-selector.yaml : 기존 ssd에서 hdd로 변경
vi ~/K8S/CH05/node-selector.yaml
```

```yaml
# node-selector.yaml
apiVersion: v1
kind: Pod
metadata:
  name: node-selector
spec:
  containers: 
  - name: nginx
    image: nginx
  # 특정 노드 라벨 선택
  nodeSelector:
    disktype: hdd     # 기존 ssd에서 hdd로 변경
```

```bash
# 기존의 Pod 삭제
# Pod는 수정이 불가능한 리소스이므로, 변경 시 삭제 후 다시 생성해야 한다.
kubectl delete pod node-selector
# pod/node-selector deleted

# 새로 라벨링한 Pod 생성
kubectl apply -f ~/K8S/CH05/node-selector.yamll
# pod/node-selector created

kubectl get pod node-selector -o wide
# NAME            READY   STATUS    RESTARTS   AGE   IP          NODE      NOMINATED NODE   READINESS GATES
# node-selector   1/1     Running   0          15s   10.42.4.3   worker1   <none>           <none>
```

## 5.3 실행명령 및 파라미터 지정
### 📘 command vs args
| 항목        | 역할                 | 컨테이너에서 대체하는 것            |
| --------- | ------------------ | ------------------------ |
| `command` | 컨테이너가 시작할 때 실행할 명령 | Dockerfile의 `ENTRYPOINT` |
| `args`    | 명령에 전달되는 인자        | Dockerfile의 `CMD`        |

### ✅ 확인 방법
1. kubectl apply -f cmd.yaml로 실행
2. kubectl logs cmd로 출력 확인 (hello가 출력됨)
3. kubectl get pod에서 STATUS가 Completed로 표시됨

### ✅ restartPolicy 옵션 종류
* restartPolicy는 Kubernetes Pod가 종료되었을 때 다시 시작할지 여부를 정의하는 옵션

| 옵션 값        | 설명                                                         |
| ----------- | ---------------------------------------------------------- |
| `Always`    | **항상 재시작** (기본값)<br>보통 `Deployment`, `ReplicaSet`에서 사용됩니다. |
| `OnFailure` | **비정상 종료(Exit code ≠ 0)** 시만 재시작<br>예: 배치 작업에서 유용          |
| `Never`     | **절대 재시작하지 않음**<br>한 번만 실행하고 종료되는 작업에 사용                   |

```bash
vi K8S/CH05/cmd.yaml
```

```yaml
# cmd.yaml
apiVersion: v1
kind: Pod
metadata:
  name: cmd
spec:
  restartPolicy: OnFailure
  containers: 
  - name: nginx
    image: nginx
    # 실행명령
    command: ["/bin/echo"]
    # 파라미터
    args: ["hello"]
```

```bash
kubectl apply -f K8S/CH05/cmd.yaml
# pod/cmd created

kubectl logs -f cmd
# hello
```

## 4 환경변수 설정
* env 옵션은 Pod 내 컨테이너에 환경변수(environment variables) 를 설정

```bash
vi K8S/CH05/env.yaml
```

```yaml
# env.yaml
apiVersion: v1
kind: Pod
metadata:
  name: env
spec:
  containers: 
  - name: nginx
    image: nginx
    env:
    - name: hello
      value: "world!"
```

```bash
# env.yaml 파일을 이용하여 Pod 생성
kubectl apply -f K8S/CH05/env.yaml
#pod/env created

# 환경변수 hello 확인
kubectl exec env -- printenv
kubectl exec env -- printenv | grep hello
# hello=world!
```

## 5.5 `Volume` 연결
* volume 은 컨테이너가 사용하는 데이터 저장소를 정의하는 기능
* 기본적으로 컨테이너는 비휘발성 저장소가 없고, 컨테이너가 재시작되면 내부 데이터는 모두 사라진다.
* 이를 보완하기 위해 volumes를 사용하여 데이터를 유지하거나 공유할 수 있다.

### 🔍 주요 구성 요소
1. volumes (Pod 레벨)
   - Pod에서 사용할 볼륨의 정의
   - 다양한 볼륨 타입 가능 (예: emptyDir, hostPath, configMap, secret, persistentVolumeClaim 등)
2. volumeMounts (컨테이너 레벨)
   - 볼륨을 컨테이너 내 파일시스템에 어디에 마운트할지 지정
   - 하나의 볼륨을 여러 컨테이너에서 공유 가능

### 📚 주요 Volume 타입
| 타입                            | 설명                                          |
| ----------------------------- | ------------------------------------------- |
| `emptyDir`                    | Pod가 생성될 때 빈 디렉토리 생성. Pod가 삭제되면 데이터도 사라짐.   |
| `hostPath`                    | 노드의 실제 경로를 마운트 (주의해서 사용해야 함).               |
| `configMap`                   | ConfigMap의 키/값을 파일로 제공.                     |
| `secret`                      | Secret 데이터를 볼륨으로 마운트.                       |
| `persistentVolumeClaim` (PVC) | 영속적 스토리지(Persistent Volume)를 연결. 데이터 유지 가능. |
| `nfs`                         | NFS(Network File System)를 통해 외부 공유 저장소 마운트. |

### ✅ 예시: emptyDir 볼륨 공유
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shared-volume
spec:
  containers:
  - name: writer
    image: busybox
    command: ["sh", "-c", "echo hello > /data/file.txt && sleep 3600"]
    volumeMounts:
    - name: shared-data
      mountPath: /data
  - name: reader
    image: busybox
    command: ["sh", "-c", "cat /data/file.txt; sleep 3600"]
    volumeMounts:
    - name: shared-data
      mountPath: /data
  volumes:
  - name: shared-data
    emptyDir: {}
```

### ✅ 실습:
```bash
vi K8S/CH05/volume.yaml
```

```yaml
# volume.yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume
spec:
  containers: 
  - name: nginx
    image: nginx

    # 컨테이너 내부의 연결 위치 지정
    volumeMounts:
    - mountPath: /container-volume
      name: my-volume
  
  # host 서버의 연결 위치 지정
  volumes:
  - name: my-volume
    hostPath:
      path: /home
```

```bash
kubectl apply -f K8S/CH05/volume.yaml
# pod/volume created

# STATUS running 확인하고 실행
kubectl exec volume -- ls /container-volume
# ubuntu

ls /home
# ubuntu
```

#### emptyDir
* emptyDir 는 가장 기본적인 임시 볼륨
* Pod가 생성될 때 빈 디렉터리를 제공하며, Pod 안의 컨테이너들 간에 데이터를 공유하거나 임시 파일 저장에 사용
* Pod가 노드에 스케줄되면, Kubernetes가 해당 노드에 빈 디렉터리(Volume) 를 생성합니다.
* Pod 내 여러 컨테이너가 같은 디렉터리를 공유할 수 있습니다.
* Pod가 삭제되면 데이터도 사라짐 (비휘발성 아님)

##### ⚠️ 주의
* Pod가 재시작되거나 삭제되면 emptyDir에 있던 데이터는 모두 사라짐.
* 영속적인 저장이 필요할 경우 persistentVolumeClaim을 사용 할 것

```bash
vi K8S/CH05/volume-empty.yaml
```

```yaml
# volume-empty.yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-empty                 # Pod 이름
  labels:
    name: volume-empty              # Pod에 부여된 라벨
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:                  # nginx 컨테이너에 볼륨 마운트
    - name: shared-storage
      mountPath: /container-volume
  - name: redis
    image: redis
    volumeMounts:                  # redis 컨테이너에도 동일한 볼륨 마운트
    - name: shared-storage
      mountPath: /container-volume
  volumes: 
  - name: shared-storage
    emptyDir: {}                   # 비어 있는 임시 디렉터리 (emptyDir 볼륨)

```

```bash
# pod 내부의 컨테이너간 공유 디렉토리 pod 생성
kubectl apply -f K8S/CH05/volume-empty.yaml
# pod/volume-empty created

# pod 내부의 각각의 컨테이너에 접근하기
$ kubectl exec -it volume-empty --container nginx -- bash
# root@volume-empty:/$ cd /container-volume/
# root@volume-empty:/container-volume$ touch volume.txt
# root@volume-empty:/container-volume# ls -al
# total 8
# drwxrwxrwx 2 root root 4096 Oct 22 09:59 .
# drwxr-xr-x 1 root root 4096 Oct 22 09:55 ..
# -rw-r--r-- 1 root root    0 Oct 22 09:59 volume.txt

# 다른 컨테이너에서 내용 확인
$ kubectl exec -it volume-empty --container redis -- bash
# root@volume-empty:/data# ls -al /container-volume/
# total 8
# drwxrwxrwx 2 root root 4096 Oct 22 09:59 .
# drwxr-xr-x 1 root root 4096 Oct 22 09:55 ..
# -rw-r--r-- 1 root root    0 Oct 22 09:59 volume.txt
```

## 5.6 리소스 관리

### 5.6.1 requests
* requests는 컨테이너가 실행되기 위해 필요한 최소한의 리소스 양을 정의

#### ✅ 기본 개념: requests와 limits
```yaml
resources:
  requests:
    cpu: "100m"         # 최소 요구: 0.1 CPU
    memory: "128Mi"     # 최소 요구: 128 MiB 메모리
  limits:
    cpu: "500m"         # 최대 허용: 0.5 CPU
    memory: "256Mi"     # 최대 허용: 256 MiB 메모리
```
| 항목         | 설명                         |
| ---------- | -------------------------- |
| `requests` | Pod이 배치될 때 최소한 보장받아야 할 리소스 |
| `limits`   | 컨테이너가 사용할 수 있는 최대 리소스      |

#### 💡 단위 정리
##### CPU
* 1 = 1 vCPU (가상 CPU 코어)
* 500m = 0.5 vCPU
* 100m = 0.1 vCPU

##### Memory
* Mi = Mebibytes (2^20 bytes)
* Gi = Gibibytes (2^30 bytes)

#### ⚠️ 중요 포인트
| 상황                 | 결과                                |
| ------------------ | --------------------------------- |
| 리소스 부족한 노드에 Pod 요청 | 스케줄되지 않음 (Pending 상태)             |
| `limits` 미설정 시     | 컨테이너가 제한 없이 리소스를 사용할 수 있음 (주의 필요) |
| `requests`를 설정하면   | 해당 리소스는 **노드에서 예약됨**              |

#### ✅ 요약
| 항목         | 역할                               |
| ---------- | -------------------------------- |
| `requests` | **스케줄링 기준**이 되는 최소 리소스 요구량       |
| `limits`   | **리소스 사용 상한선**                   |
| 설정 위치      | `spec.containers[].resources` 아래 |

```bash
vi K8S/CH05/requests.yaml
```

```yaml
# requests.yaml
apiVersion: v1
kind: Pod
metadata:
  name: requests
spec:
  containers: 
  - name: nginx
    image: nginx
    resources:
      requests:
        cpu: "250m"
        memory: "500Mi"
```

### 5.6.2 limits
* memory: "1Gi"를 초과하면 컨테이너가 메모리 부족(OOM: Out Of Memory)으로 강제 종료
* status = OOMKilled (Out Of Memory Killed)

```bash
vi K8S/CH05/limits.yaml
```

```yaml
# limits.yaml
apiVersion: v1
kind: Pod
metadata:
  name: limits
spec:
  restartPolicy: OnFailure
  containers: 
  - name: mynginx
    image: python:3.7
    command: [ "python" ]
    args: [ "-c", "arr = []\nwhile True: arr.append(range(1000))" ]
    resources:
      limits:
        cpu: "500m"
        memory: "1Gi"
```

```bash
kubectl apply -f K8S/CH05/limits.yaml
# pod/limits created

watch kubectl get pod limits
# NAME    READY   STATUS     RESTARTS   AGE
# limits  1/1     Running    0          1m
# ...
# limits  0/1     OOMKilled  1          1m
```

```bash
vi K8S/CH05/resources.yaml
```

```yaml
# resources.yaml
apiVersion: v1
kind: Pod
metadata:
  name: resources
spec:
  containers: 
  - name: nginx
    image: nginx
    resources:
      requests:
        cpu: "250m"
        memory: "500Mi"
      limits:
        cpu: "500m"
        memory: "1Gi"
```

```bash
kubectl apply -f K8S/CH05/resources.yaml

watch kubectl get pod
kubectl describe pod limits
kubectl logs limits             # 단일 컨테이너 로그 확인
kubectl logs limits -c mynginx   # 다중 컨테이너 중 특정 컨테이너 로그
```

#### ✅ 주요 STATUS 값 및 설명
| STATUS                | 의미                                                                                          |
| --------------------- | ------------------------------------------------------------------------------------------- |
| **Pending**           | Pod가 스케줄되었지만 아직 컨테이너가 실행되지 않음<br>→ 노드에 자원이 부족하거나 이미지 다운로드 중일 수 있음                           |
| **Running**           | 하나 이상의 컨테이너가 실행 중이며, 정상 작동 중                                                                |
| **Succeeded**         | 모든 컨테이너가 정상 종료(exit code 0) 되었고, Pod는 완료됨<br>(보통 `restartPolicy: Never` 또는 `OnFailure`인 경우) |
| **Failed**            | 하나 이상의 컨테이너가 비정상 종료(exit code ≠ 0)하여 실패함                                                    |
| **CrashLoopBackOff**  | 컨테이너가 반복적으로 충돌(Crash)하고 있음<br>→ 일정 시간 간격으로 재시도 중 (Back-off)                                 |
| **ImagePullBackOff**  | 컨테이너 이미지를 가져오지 못해 재시도 중<br>→ 이미지 이름 오류, 인증 실패, 존재하지 않는 이미지 등                                |
| **ErrImagePull**      | 이미지 다운로드에 실패했음 (즉시 실패 상태)                                                                   |
| **Terminating**       | Pod가 삭제 중이며, 종료 절차 진행 중                                                                     |
| **ContainerCreating** | 컨테이너가 아직 생성 중 (볼륨 마운트, 이미지 다운로드 등 준비 중)                                                     |
| **Init:**\<n>/<m>     | Init 컨테이너 실행 중 (n번째 Init 컨테이너가 m개 중 몇 개 완료됐는지 보여줌)                                          |

## 5.7 상태 확인

### 5.7.1 `livenessProbe`
* 컨테이너가 정상적으로 동작 중인지 주기적으로 확인하는 헬스 체크 기능
* 컨테이너가 **살아있지 않다(Liveness probe 실패)**고 판단되면, Kubernetes는 그 컨테이너를 자동으로 재시작
* 즉, 죽은 프로세스를 감지하고 복구하는 데 사용

#### 🔍 주요 파라미터 설명
| 항목                               | 설명                           |
| -------------------------------- | ---------------------------- |
| `httpGet` / `tcpSocket` / `exec` | 헬스 체크 방식 선택 (아래 참고)          |
| `initialDelaySeconds`            | 컨테이너 시작 후 처음 체크까지 대기 시간 (초)  |
| `periodSeconds`                  | 프로브 실행 간격 (기본 10초)           |
| `timeoutSeconds`                 | 응답 대기 시간 초과 시 실패로 간주 (기본 1초) |
| `failureThreshold`               | 몇 번 연속 실패하면 컨테이너를 재시작할지      |
| `successThreshold`               | 몇 번 연속 성공해야 "성공" 상태로 인정할지    |

#### 🧪 예시 1: HTTP 기반 Liveness Probe
* localhost:8080/healthz로 HTTP 요청을 보내 응답 코드 200~399면 OK
```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
```
* 해당 포트로 TCP 연결이 성공하면 살아있는 것으로 간주

#### 🧪 예시 2: TCP Socket Liveness Probe
```yaml
livenessProbe:
  tcpSocket:
    port: 3306
  initialDelaySeconds: 10
  periodSeconds: 15

```

#### 🧪 예시 3: 명령 실행 (exec) 기반
```yaml
livenessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 10
```
#### 📌 Liveness vs Readiness
* 둘 다 함께 쓰는 것이 일반적.
| 항목               | 설명                                          |
| ---------------- | ------------------------------------------- |
| `livenessProbe`  | **죽었는지** 판단 → 실패 시 컨테이너 **재시작**             |
| `readinessProbe` | **준비됐는지** 판단 → 실패 시 **Service에서 제외**, 재시작 X |

#### 📌 실습

```bash
vi K8S/CH05/liveness.yaml
```

```yaml
# liveness.yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness
spec:
  containers: 
  - name: nginx
    image: nginx
    livenessProbe:
      httpGet:
        path: /live
        port: 80
```

```bash
kubectl apply -f K8S/CH05/liveness.yaml
# pod/liveness created

# <CTRL> + <C>로 watch를 종료할 수 있습니다.
# RESTARTS 값이 계속 증가하는 것 확인
watch kubectl get pod liveness
# NAME       READY   STATUS    RESTARTS   AGE
# liveness   1/1     Running   2          71s

# 기본적으로 nginx에는 /live 라는 API가 없습니다.
kubectl logs -f liveness
# ...
# 10.42.0.1 - - [13/Aug/2020:12:31:24 +0000] "GET /live HTTP/1.1" 404 153 "-" "kube-probe/1.18" "-"
# 10.42.0.1 - - [13/Aug/2020:12:31:34 +0000] "GET /live HTTP/1.1" 404 153 "-" "kube-probe/1.18" "-"

kubectl logs liveness --previous

```

#### live파일을 생성해서 /live가 정상적으로 호출되도록 설정
```bash
kubectl exec liveness -- touch /usr/share/nginx/html/live

kubectl logs liveness
# 10.42.0.1 - - [14/Aug/2020:08:50:08 +0000] "GET /live HTTP/1.1" 404 153 "-" "kube-probe/1.18" "-"
# 10.42.0.1 - - [14/Aug/2020:08:50:18 +0000] "GET /live HTTP/1.1" 200 0 "-" "kube-probe/1.18" "-"

kubectl get pod liveness
# NAME       READY   STATUS      RESTARTS   AGE
# liveness   1/1     Running     2          12m
``` 

### 5.7.2 `readinessProbe`
* 컨테이너가 서비스 요청을 받을 준비가 되었는지 판단하는 헬스 체크
* livenessProbe가 살아있는지 확인하는 데 반해, readinessProbe는 준비되었는지 확인하는 데 사용

#### ✅ 작동 방식
* readinessProbe가 성공하면 → 해당 Pod는 Service 엔드포인트에 포함됩니다.
* readinessProbe가 실패하면 → 트래픽 전달 대상에서 제외됩니다.
* 컨테이너는 재시작되지 않습니다.

#### 📄 예시: readinessProbe 설정

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
```
→ 컨테이너가 시작된 후 5초 뒤부터 /ready에 10초마다 HTTP 요청을 보내 성공 여부 확인

#### 🔍 주요 파라미터
| 필드                               | 설명                           |
| -------------------------------- | ---------------------------- |
| `httpGet` / `tcpSocket` / `exec` | 검사 방식 (HTTP, TCP, 또는 명령어 실행) |
| `initialDelaySeconds`            | 최초 검사까지 기다리는 시간              |
| `periodSeconds`                  | 검사 주기                        |
| `timeoutSeconds`                 | 검사 타임아웃 시간                   |
| `failureThreshold`               | 실패 허용 횟수 (이후 "준비되지 않음"으로 간주) |
| `successThreshold`               | 연속 성공 횟수 필요 수치 (기본 1)        |

#### 🔁 readiness vs liveness
| 항목     | `readinessProbe`   | `livenessProbe` |
| ------ | ------------------ | --------------- |
| 목적     | **준비됐는지 확인**       | **살아있는지 확인**    |
| 실패 시   | 서비스 트래픽에서 **제외됨**  | 컨테이너가 **재시작됨**  |
| 회복 가능성 | 다시 준비 상태로 변경될 수 있음 | 실패 시 무조건 재시작    |

#### 🔁 readinessProbe

```bash
vi K8S/CH05/readiness.yaml
```

```yaml
# readiness.yaml
apiVersion: v1
kind: Pod
metadata:
  name: readiness
spec:
  containers: 
  - name: nginx
    image: nginx
    readinessProbe:
      httpGet:
        path: /ready
        port: 80
```

```bash
kubectl apply -f K8S/CH05/readiness.yaml
# pod/readiness created

kubectl logs -f readiness
# 10.42.0.1 - - [28/Jun/2020:13:31:28 +0000] "GET /ready HTTP/1.1" 
# 404 153 "-" "kube-probe/1.18" "-"

# 기본적으로 nginx에는 /ready 라는 API가 없습니다.
# /ready 호출에 404 에러가 반환되어 준비 상태가 완료되지 않습니다.
kubectl get pod
# NAME        READY   STATUS      RESTARTS   AGE
# readiness   0/1     Running     0          2m

# /ready URL 생성
kubectl exec readiness -- touch /usr/share/nginx/html/ready

# READY 1/1로 준비 완료 상태로 변경되었습니다.
kubectl get pod
# NAME        READY   STATUS      RESTARTS   AGE
# readiness   1/1     Running     0          2m
```

#### 🔁 명령실행
```bash
vi K8S/CH05/readiness-cmd.yaml
```

```yaml
# readiness-cmd.yaml
# exec 방식의 readinessProbe를 사용하여, 지정한 파일(/tmp/ready)이 존재할 때만 컨테이너를 "준비됨(Ready)" 상태로 인식
apiVersion: v1                  # Kubernetes core API 그룹 버전
kind: Pod                       # 리소스 종류: Pod
metadata:
  name: readiness-cmd           # Pod 이름

spec:
  containers:                   # Pod 안에 포함된 컨테이너 목록
  - name: nginx                 # 컨테이너 이름
    image: nginx                # nginx 공식 이미지 사용

    readinessProbe:             # 컨테이너의 준비 상태를 체크하는 프로브
      exec:                     # 명령어 실행 방식의 프로브 사용
        command:
        - cat                   # 실행할 명령어: cat
        - /tmp/ready            # 대상 파일: /tmp/ready
```
#### ✅ 작동 방식

```bash
# 컨테이너가 시작되면 Kubernetes는 readinessProbe로 아래 명령어를 주기적으로 실행
cat /tmp/ready

# 해당 파일이 존재하고, cat 명령이 성공(exit code 0) 하면:
# → Pod는 "Ready" 상태로 간주되어 서비스 트래픽을 받을 수 있음

# 해당 파일이 없으면 실패(exit code ≠ 0):
# → Kubernetes는 Pod를 Ready 상태로 만들지 않음, Service 트래픽에서 제외됨
```

```bash
kubectl apply -f K8S/CH05/readiness-cmd.yaml
# pod/readiness-cmd created

# /tmp/ready 라는 파일이 존재하지 않기 때문에
# READY 0/1로 준비가 완료 되지 않음
kubectl get pod
# NAME            READY   STATUS      RESTARTS   AGE
# readiness-cmd   0/1     Running     0          2m

# /tmp/ready 파일 생성
kubectl exec readiness-cmd -- touch /tmp/ready

# READY 1/1로 준비 완료 상태로 변경
kubectl get pod
# NAME            READY   STATUS      RESTARTS   AGE
# readiness-cmd   1/1     Running     0          2m
```

## 5.8 두 개 컨테이너 실행
* 한 개의 pod에 2개의 컨테이너 실행
* second(pod)에 nginx, curl 2개의 컨테이너 실행

```bash
vi K8S/CH05/second.yaml
```

```yaml
# second.yaml
# 두 개의 컨테이너(Nginx, Curl)를 동시에 실행하여 한 컨테이너가 다른 컨테이너의 서비스에 주기적으로 접근하는 구조
# second.yaml
apiVersion: v1                  # Kubernetes core API 그룹의 버전
kind: Pod                       # 생성할 리소스 종류는 Pod
metadata:
  name: second                  # Pod의 이름은 'second'

spec:
  containers:                   # 이 Pod에 포함될 컨테이너 목록

  - name: nginx                 # 첫 번째 컨테이너: nginx 웹 서버
    image: nginx                # nginx 공식 이미지 사용

  - name: curl                  # 두 번째 컨테이너: curl 유틸리티 사용 컨테이너
    image: curlimages/curl      # 경량 curl 이미지 사용
    command: ["/bin/sh"]        # 쉘을 실행
    # args: ["-c", "while true; do sleep 5; curl -s localhost; done"]
    args:                       # 쉘에서 실행할 명령
      - "-c"
      - "while true; do sleep 5; curl -s localhost; done"  # 무한 루프 실행: 5초마다 localhost에 curl 요청
```
### ✅ 동작 방식
* ginx 컨테이너: 기본 포트(80번)에서 HTTP 웹 서버 실행
* curl 컨테이너:
* 5초마다 curl localhost 실행
* localhost는 같은 Pod 내에 있는 nginx 컨테이너를 의미함 (Pod 내 컨테이너는 네트워크를 공유함)

### 📌 중요한 특징
| 항목                   | 설명                                                |
| -------------------- | ------------------------------------------------- |
| **네트워크 공유**          | Pod 내부 컨테이너는 **같은 localhost** (127.0.0.1)와 포트를 공유 |
| **curl** → **nginx** | curl 컨테이너가 nginx 서버의 루트 페이지(`/`)에 계속 요청           |
| **무한 루프**            | curl 컨테이너는 `while true` 루프로 계속 동작 (종료되지 않음)       |

### 🔍 Pod 상태 및 동작 확인
```bash
kubectl apply -f second.yaml
kubectl get pods
kubectl logs second -c curl    # curl 컨테이너의 요청 결과 출력
```

```bash
kubectl apply -f K8S/CH05/second.yaml
# pod/second created

kubectl get pod second
# NAME     READY   STATUS    RESTARTS   AGE
# second   2/2     Running   0          3m44s
# READY 2/2 -> 2개의 컨테이너가 정상적으로 실행중

kubectl logs second
# error: a container name must be specified for pod second, choose one of: [nginx curl]

# nginx 컨테이너 지정
kubectl logs second -c nginx
# 127.0.0.1 - - [22/Jun/2020:13:37:00 +0000] "GET / HTTP/1.1" 200 
# 612 "-" "curl/7.70.0-DEV" "-"
# 127.0.0.1 - - [22/Jun/2020:13:37:09 +0000] "GET / HTTP/1.1" 200 
# 612 "-" "curl/7.70.0-DEV" "-"

# curl 컨테이너 지정
kubectl logs second -c curl
# ...
```

## 5.9 초기화 컨테이너
* 컨테이너끼리는 실행순서를 보장하지 않느다.
* 명시적으로 메인컨테이너 실행전에 초기화작업을 할 경우 `initContainers` property를 이용
* Kubernetes Pod 안에서 메인 컨테이너가 실행되기 전에 순차적으로 실행되는 컨테이너
* 이들은 초기화 작업 전용으로 사용되며, 일반 컨테이너와는 다른 목적을 가진다.

### ✅ 주요 특징
| 항목     | 설명                                                        |
| ------ | --------------------------------------------------------- |
| 실행 시점  | 메인 컨테이너 **실행 전에** 순서대로 실행됨                                |
| 실행 순서  | 정의된 순서대로 **직렬 실행** (하나씩 차례로)                              |
| 완료 조건  | 모든 `initContainer`가 **정상 종료(exit code 0)** 되어야 메인 컨테이너 실행 |
| 실패 시   | 재시도 반복, Pod는 Pending 상태 유지                                |
| 재시작 정책 | 항상 `restartPolicy: Always`처럼 작동함 (성공할 때까지 반복)             |

### 🔧 사용 용도
* Config 파일 생성 또는 다운로드
* DB 연결 확인 또는 마이그레이션
* 공유 볼륨에 초기 데이터 준비
* 외부 시스템과의 통신 준비

### 🔍 상태 확인
```bash
# Init Containers: 항목에서 상태와 로그를 확인 가능
kubectl logs <pod-name> -c <init-container-name>
```

### 📌 로그 보기
```bash
kubectl logs <pod-name> -c <init-container-name>
```

### 🆚 비교: initContainer vs container
| 항목    | `initContainer` | `container` (메인) |
| ----- | --------------- | ---------------- |
| 실행 시점 | 먼저 실행됨          | 이후 실행됨           |
| 실행 순서 | 순차 실행           | 병렬 가능            |
| 용도    | 준비 작업           | 실제 앱 실행          |
| 실패 시  | 재시도             | 정책에 따라 재시작       |

```bash
vi K8S/CH05/init-container.yaml
```

* 메인컨테이너 실행전에 git clone 
* init container엣 미리 git pull후 컨테이너의 공유공간인 emptyDir volume(/tmp)를 통해 git repository 전달
* 메인컨테이너에서 git repository에 데이터가 이미 있는 것을 가정하고 로직을 수행
```yaml
# init-container.yaml
apiVersion: v1                   # 이 리소스는 Kubernetes core v1 API 그룹을 사용
kind: Pod                        # 리소스 타입은 Pod
metadata:
  name: init-container           # Pod의 이름

spec:
  restartPolicy: OnFailure       # 컨테이너가 실패(exit code 0이 아님)했을 때만 재시작

  containers:                    # 메인 컨테이너 정의
  - name: busybox                # 메인 컨테이너 이름
    image: k8s.gcr.io/busybox    # busybox 이미지 (간단한 유틸리티 제공용)
    command: [ "ls" ]            # 실행할 명령어: ls
    args: [ "/tmp/moby" ]        # 인자: /tmp/moby 디렉토리 목록을 출력
    volumeMounts:                # 볼륨을 컨테이너에 마운트
    - name: workdir              # 마운트할 볼륨 이름
      mountPath: /tmp            # 컨테이너 내에 /tmp 경로로 마운트

  initContainers:                # 메인 컨테이너 실행 전 먼저 실행되는 초기화 컨테이너들
  - name: git                    # init 컨테이너 이름
    image: alpine/git            # Git 명령어가 있는 경량 alpine 이미지
    command: ["sh"]              # 실행할 셸 명령어 인터프리터
    args:
    - "-c"
    - "git clone https://github.com/moby/moby.git /tmp/moby"  # moby 저장소를 /tmp/moby에 클론
    volumeMounts:
    - name: workdir              # 같은 볼륨을 /tmp로 마운트하여 메인 컨테이너와 공유
      mountPath: "/tmp"

  volumes:                       # Pod 전체에서 사용할 볼륨 정의
  - name: workdir                # 볼륨 이름
    emptyDir: {}                 # Pod가 살아 있는 동안 임시로 존재하는 빈 디렉토리 볼륨

```
#### 🔍 작동 순서 요약
* initContainer (git)가 실행되어 moby 저장소를 /tmp/moby에 clone
* 성공하면 busybox 메인 컨테이너가 실행되어 /tmp/moby 디렉토리를 ls로 출력
* 두 컨테이너는 emptyDir 볼륨을 통해 /tmp 디렉토리를 공유함
* restartPolicy: OnFailure 덕분에 실패할 경우만 재시작함


```bash
kubectl apply -f K8S/CH05/init-container.yaml
# pod/init-container created

watch kubectl get pod
# NAME            READY   STATUS      RESTARTS   AGE
# init-container  0/1     Init:0/1    0          2s

# initContainer log 확인
# git clone 작업으로 시간이 좀 소요
kubectl logs init-container -c git -f
# Cloning into '/tmp/moby'...
# Updating files: 100% (10521/10521), done.

# 메인 컨테이너 (busybox) log 확인
kubectl logs init-container
# AUTHORS
# CHANGELOG.md
# CONTRIBUTING.md
# Dockerfile
# Dockerfile.buildx
# ...
```

## 5.10 Config 설정
* Config 설정은 애플리케이션의 설정 값, 환경 변수, 설정 파일 등을 외부에서 관리하고 주입하기 위한 기능
* 주요 객체는 **ConfigMap**과 **Secret**이며, 이들을 통해 컨테이너 이미지를 수정하지 않고도 설정을 바꿀 수 있다.

### ✅ 1. ConfigMap이란?
* 일반 텍스트 형태의 설정 데이터를 저장하는 리소스
* 설정 파일, 명령줄 인자, 환경 변수 등으로 주입 가능

#### 📄 예시: ConfigMap 생성 (YAML)
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_MODE: "production"
  LOG_LEVEL: "debug"
```

### ✅ 2. Pod에서 ConfigMap 사용하는 방법
#### 1️⃣ 환경 변수로 사용
```yaml
containers:
- name: myapp
  image: myapp:latest
  env:
  - name: APP_MODE
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_MODE
# APP_MODE 환경 변수에 "production"이 주입됨
```
#### 2️⃣ 설정 파일로 마운트
```yaml
volumes:
- name: config-volume
  configMap:
    name: app-config

containers:
- name: myapp
  ...
  volumeMounts:
  - name: config-volume
    mountPath: /etc/config
# /etc/config/APP_MODE와 /etc/config/LOG_LEVEL 파일이 생성됨
```

### ✅ 3. ConfigMap 생성 명령어
#### 🔹 직접 생성
```bash
kubectl create configmap app-config --from-literal=APP_MODE=production --from-literal=LOG_LEVEL=debug
```

#### 🔹 파일로 생성
```bash
kubectl create configmap app-config --from-file=config.txt
```

### ✅ 4. Secret이란?
* ConfigMap과 유사하지만, 민감한 데이터(비밀번호, API 키 등)를 저장하는 리소스
* Base64 인코딩되어 저장되며, 접근 권한이 제한됨

```bash
kubectl create secret generic db-secret \
  --from-literal=DB_USER=admin \
  --from-literal=DB_PASS=pass123
```

#### ✅ ConfigMap vs Secret
| 항목    | ConfigMap         | Secret        |
| ----- | ----------------- | ------------- |
| 용도    | 일반 설정 데이터         | 민감한 정보 (암호 등) |
| 인코딩   | 일반 텍스트            | Base64 인코딩    |
| 접근 제어 | 제한 없음 (기본)        | RBAC으로 제한 가능  |
| 사용 방법 | env, volume, file | 동일            |

#### 📌 요약
| 기능        | 설명                     |
| --------- | ---------------------- |
| ConfigMap | 일반 설정 데이터 저장 및 주입      |
| Secret    | 민감 정보 저장               |
| 사용처       | 환경 변수, 볼륨 마운트, 명령 인자 등 |
| 장점        | 앱 이미지 수정 없이 설정 변경 가능   |


### 5.10.1 ConfigMap 리소스 생성

```bash
kubectl create configMap <configMap> <data-source>
```

```bash
vi K8S/CH05/game.properties
```

```properties
# game.properties
weapon=gun
health=3
potion=5
```

```bash
kubectl create configmap game-config --from-file=K8S/CH05/game.properties
# configmap/game-config created
```

```bash
kubectl get configmap game-config -o yaml  # 축약하여 cm
# apiVersion: v1
# data:
#   game.properties: |
#     weapon=gun
#     health=3
#     potion=5
# kind: ConfigMap
# metadata:
#   name: game-config
#   namespace: default

kubectl get cm game-config
kubectl get cm game-config -o yaml
```

####  Kubernetes에서 ConfigMap을 생성
```bash
kubectl create configmap special-config \
            --from-literal=special.power=10 \
            --from-literal=special.strength=20
# configmap/special-config created
```

##### 🧾 각 항목 설명
| 부분                         | 설명                                         |
| -------------------------- | ------------------------------------------ |
| `kubectl create configmap` | ConfigMap을 생성하는 `kubectl` 명령어              |
| `special-config`           | 생성될 ConfigMap의 이름                          |
| `--from-literal=키=값`       | `ConfigMap`에 저장할 **단일 key-value 쌍**을 직접 지정 |
| `special.power=10`         | key: `special.power`, value: `"10"`        |
| `special.strength=20`      | key: `special.strength`, value: `"20"`     |

##### 🔍 생성된 ConfigMap 확인
```bash
kubectl get configmap special-config -o yaml
```

##### 🧪 사용 예: Pod에서 환경 변수로 사용
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-demo
spec:
  containers:
  - name: demo
    image: busybox
    command: ["sh", "-c", "env; sleep 3600"]
    env:
    - name: POWER
      valueFrom:
        configMapKeyRef:
          name: special-config
          key: special.power
    - name: STRENGTH
      valueFrom:
        configMapKeyRef:
          name: special-config
          key: special.strength
# → 컨테이너 내에서 POWER=10, STRENGTH=20 환경 변수가 설정됨        
```


```bash
kubectl get cm special-config -o yaml
# apiVersion: v1
# kind: ConfigMap
# metadata:
#   name: special-config
#   namespace: default
# data:
#   special.power: 10
#   special.strength: 20
```

#### 사용자가 직접 생성
```bash
vi K8S/CH05/monster-config.yaml
```
```yaml
# monster-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: monster-config
  namespace: default
data:
  monsterType: fire
  monsterNum: "5"
  monsterLife: "3"
```

```bash
kubectl apply -f K8S/CH05/monster-config.yaml
# configmap/monster-config created

kubectl get cm monster-config -o yaml
# apiVersion: v1
# kind: ConfigMap
# metadata:
#   name: monster-config
#   namespace: default
# data:
#   monsterType: fire
#   monsterNum: "5"
#   monsterLife: "3"
```

### 5.10.2 ConfigMap 활용

#### 볼륨 연결
* ConfigMap을 Pod 내부에서 설정 파일처럼 마운트하여 사용하는 구조
```bash
vi K8S/CH05/game-volume.yaml
```
```yaml
# game-volume.yaml
apiVersion: v1                   # Kubernetes core API 버전
kind: Pod                        # 리소스 종류: Pod
metadata:
  name: game-volume              # Pod 이름

spec:
  restartPolicy: OnFailure       # 컨테이너가 실패했을 때만 재시작

  containers:
  - name: game-volume            # 컨테이너 이름
    # 에러 image: k8s.gcr.io/busybox    # busybox 이미지 사용 (경량 Linux shell)
    image: busybox    # 도커허브에서 가져옴. busybox 이미지 사용 (경량 Linux shell)
    command: [ "/bin/sh", "-c", "cat /etc/config/game.properties" ]
                                 # 컨테이너 실행 시 해당 파일 내용을 출력하고 종료

    volumeMounts:
    - name: game-volume          # 마운트할 볼륨 이름
      mountPath: /etc/config     # /etc/config 경로에 ConfigMap 내용이 파일로 마운트됨

  volumes:
  - name: game-volume            # 사용할 볼륨 정의
    configMap:
      name: game-config          # 마운트할 ConfigMap 이름 (사전에 생성되어 있어야 함)

```
##### ✅ 작동 방식
* game-config라는 이름의 ConfigMap에 저장된 key-value가
* /etc/config/ 경로에 파일 형태로 마운트됩니다.
* cat /etc/config/game.properties 명령으로 game.properties 파일 내용을 출력함

```bash
kubectl apply -f K8S/CH05/game-volume.yaml
# pod/game-volume created

# 로그를 확인합니다.
kubectl logs game-volume
# weapon=gun
# health=3
# potion=5
```

```bash
# 필요시 pod 일괄삭제
kubectl delete pods --all -n default
```


#### 환경변수 - valueFrom
* configMap을 Pod의 환경변수로 사용
* Pod를 생성하며, 그 안에 환경변수를 ConfigMap에서 가져와 설정하는 예제

```bash
vi K8S/CH05/special-env.yaml
```

```yaml
# special-env.yaml
apiVersion: v1                   # Kubernetes API 버전 (core/v1)
kind: Pod                        # 리소스 종류는 Pod
metadata:
  name: special-env             # Pod 이름 지정
spec:
  restartPolicy: OnFailure      # 컨테이너가 실패할 때만 재시작
  containers:
  - name: special-env           # 컨테이너 이름
    # image: k8s.gcr.io/busybox # busybox 이미지를 사용 (경량 쉘 환경)
    image: busybox              # 도커허브 busybox 이미지를 사용 (경량 쉘 환경)
    command: [ "printenv" ]     # 컨테이너가 시작되면 printenv 명령어 실행
    args: [ "special_env" ]     # printenv에 인자로 환경변수 이름을 줘서 출력하도록 함
    env:                        # 환경변수 설정
    - name: special_env         # 환경변수 이름
      valueFrom:                # 환경변수 값을 ConfigMap에서 참조
        configMapKeyRef:
          name: special-config  # 참조할 ConfigMap 이름
          key: special.power    # ConfigMap 내 키 이름

```
* pod이 오류(CreateContainerConfigError)가 발생했을 경우
* kubectl get configmap special-config 명령우로 컨피그파일 확인후 없을 경우 생성(상기예제실행
)
```bash
kubectl apply -f K8S/CH05/special-env.yaml
# pod/special-env created

kubectl logs special-env
# 10
```

#### 환경변수 - envFrom
* Kubernetes Pod(또는 컨테이너) 설정에서 ConfigMap이나 Secret 전체 데이터를 환경변수로 한 번에 불러올 때 사용하는 옵션
* 차이점: env vs envFrom
  - env: 개별 환경변수를 하나씩 정의할 때 사용
  - envFrom: ConfigMap 또는 Secret 전체를 환경변수로 한꺼번에 가져올 때 사용

```bash
vi K8S/CH05/monster-env.yaml
```

```yaml
# monster-env.yaml
apiVersion: v1
kind: Pod
metadata:
  name: monster-env         # Pod 이름
spec:
  restartPolicy: OnFailure  # 실패 시 재시작 정책
  containers:
  - name: monster-env       # 컨테이너 이름
    image: busybox          # busybox 이미지 사용
    command: [ "printenv" ] # 환경변수 전체 출력 명령 실행
    envFrom:
    - configMapRef:
        name: monster-config  # monster-config ConfigMap 전체를 환경변수로 로드

```
##### 동작 방식
* monster-config ConfigMap 내 모든 key-value 쌍이 컨테이너의 환경변수로 설정됩니다.
* 컨테이너가 시작되면서 printenv 명령어가 실행되어 환경변수들이 출력됩니다.
* ConfigMap에 예를 들어 MONSTER_TYPE=dragon, MONSTER_LEVEL=10 등이 있으면 해당 값들이 컨테이너 내 환경변수로 자동 적용됩니다.

##### 주의사항
* monster-config ConfigMap이 반드시 사전에 생성되어 있어야 합니다.
* ConfigMap 키는 환경변수 이름으로 유효해야 합니다 (대문자, 숫자, 언더스코어 권장).

```bash
kubectl apply -f K8S/CH05/monster-env.yaml
# pod/monster-env created

kubectl logs monster-env | grep monster
# HOSTNAME=monster-env
# monsterLife=3
# monsterNum=5
# monsterType=fire
```

## 5.11 민감 데이터 관리

### 5.11.1 Secret 리소스 생성
* Kubernetes에서 **민감한 정보(비밀번호, 토큰, 인증키 등)**를 안전하게 저장하고 Pod에 전달할 수 있게 해주는 리소스
* Secret리소스는 각 노드에서 사용될 떄 디스크에 저장되지 않고 `tmpfs라는 메모리 기반 파일시스템을 사용`해서 보안에 더울 강함
* Secret리소스를 조회할 때 평문으로 바로 조회되지 않고 base64로 한번 인코딩되어 표시된다.

#### ✅ Secret이 필요한 이유
* 비밀번호, API 키, TLS 인증서 같은 민감 정보를 명확하게 분리해서 관리 가능
* 일반 ConfigMap처럼 평문으로 저장되지만, base64 인코딩되어 있으며 접근이 제한됨
* Secret은 etcd에 암호화 저장할 수 있고, RBAC으로 접근을 통제할 수 있음

#### 🧪 Secret 생성 방법
##### 1. CLI로 생성 (--from-literal)
```bash
kubectl create secret generic my-secret \
  --from-literal=username=admin \
  --from-literal=password=1234
```

##### 2. YAML로 생성
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: YWRtaW4=          # base64로 인코딩된 값 ("admin")
  password: MTIzNA==          # base64로 인코딩된 값 ("1234")
```
>echo -n 'admin' | base64 같은 명령어로 인코딩 가능

#### 📦 Pod에 Secret을 사용하는 방법
##### ✅ 방법 1: 환경변수로 주입
```yaml
env:
- name: SECRET_USERNAME
  valueFrom:
    secretKeyRef:
      name: my-secret
      key: username
```

#### ✅ 방법 2: 파일로 마운트
```yaml
volumes:
- name: secret-vol
  secret:
    secretName: my-secret

volumeMounts:
- name: secret-vol
  mountPath: "/etc/secret"
  readOnly: true
```
>/etc/secret/username, /etc/secret/password 같은 파일로 주입됨

#### 🔐 타입 종류 (선택사항)
* Opaque – 기본 타입 (직접 만든 키-값)
* kubernetes.io/dockerconfigjson – 프라이빗 레지스트리 로그인용
* kubernetes.io/tls – TLS 인증서 저장용

#### 💡 참고사항
* Secret의 base64 인코딩은 암호화가 아닙니다, 다만 노출을 완화합니다.
* etcd를 암호화하지 않으면 클러스터 관리자에겐 평문처럼 노출될 수 있습니다.
* RBAC을 통해 Secret 접근 권한을 제한하는 것이 매우 중요합니다.


###### (1) secret 리소스 생성 - base64 인코딩
```bash
echo -ne admin | base64
# admin 문자열을 개행 없이 (-n) 이스케이프 문자 해석 허용 (-e) 하여 출력합니다. (결과) YWRtaW4=

echo -ne password123 | base64
# cGFzc3dvcmQxMjM=
```

###### (2) secret 리소스 생성 - Yaml 파일생성
```bash
vi K8S/CH05/user-info.yaml
```
```yaml
# user-info.yaml
apiVersion: v1                 # Kubernetes core API 버전
kind: Secret                   # 리소스 종류: Secret (민감한 데이터 저장용)
metadata:
  name: user-info              # Secret 리소스 이름
type: Opaque                   # 일반적인 Key-Value 타입 (기본값)
data:                          # 인코딩된 데이터 영역 (base64로 인코딩되어야 함)
  username: YWRtaW4=           # "admin" → base64 인코딩된 값
  password: cGFzc3dvcmQxMjM=   # "password123" → base64 인코딩된 값
```
###### 🔐 type: Opaque의 의미
* Opaque는 Secret의 기본 타입으로, 일반적인 key-value 쌍을 저장할 때 사용.
* TLS 인증서나 docker 레지스트리 인증용 Secret은 다른 타입(kubernetes.io/tls, kubernetes.io/dockerconfigjson)을 사용.

```bash
kubectl apply -f K8S/CH05/user-info.yaml
# secret/user-info created

kubectl get secret user-info -o yaml
# apiVersion: v1
# data:
#   password: cGFzc3dvcmQxMjM=
#   username: YWRtaW4=
# kind: Secret
# metadata:
#   ...
#   name: user-info
#   namespace: default
#   resourceVersion: "6386"
#   selfLink: /api/v1/namespaces/default/secrets/user-info
#   uid: bd6ca3a6-17f1-4690-aefe-6200e402aa35
# type: Opaque
```

###### data의 필드값을 직접 base64로 인코딩하지 않고 쿠버네티스가 대신처리하려면 `stringData`를 사용
* 사람이 읽기 쉬운 형식으로 비밀 데이터를 작성할 수 있는 편리한 방법
```bash
vi K8S/CH05/user-info-stringdata.yaml
```
```yaml
# user-info-stringdata.yaml
apiVersion: v1                   # API 버전
kind: Secret                     # Secret 리소스
metadata:
  name: user-info-stringdata     # Secret 이름
type: Opaque                     # 일반 Key-Value 타입
stringData:                      # 평문으로 키-값 작성 (자동 인코딩)
  username: admin                # 평문으로 작성됨 → base64: YWRtaW4=
  password: password123          # 평문으로 작성됨 → base64: cGFzc3dvcmQxMjM=
```
###### 🔍 설명: stringData vs data
| 항목           | 설명                                                                |
| ------------ | ----------------------------------------------------------------- |
| `data`       | 값이 반드시 **Base64 인코딩**되어 있어야 함                                     |
| `stringData` | 값을 **평문(Plaintext)** 으로 작성 가능. `kubectl apply` 시 자동으로 Base64 인코딩됨 |


```bash
kubectl apply -f K8S/CH05/user-info-stringdata.yaml
# secret/user-info created

kubectl get secret user-info-stringdata -o yaml
# apiVersion: v1
# data:
#   password: cGFzc3dvcmQxMjM=
#   username: YWRtaW4=
# kind: Secret
# metadata:
#   ...
#   name: user-info-stringdata
#   namespace: default
#   resourceVersion: "6386"
#   selfLink: /api/v1/namespaces/default/secrets/user-info
#   uid: bd61b459-a0d4-4158-aefe-6200f202aa35
# type: Opaque
```

###### (3) secret 리소스 생성 - 명령어로 생성
```bash
vi K8S/CH05/user-info.properties
# user-info.properties
username=admin
password=password123
```

```bash
kubectl create secret generic user-info-from-file \
                    --from-env-file=K8S/CH05/user-info.properties
# secret/user-info-from-file created

kubectl get secret user-info-from-file -oyaml
# apiVersion: v1
# data:
#   password: cGFzc3dvcmQxMjM=
#   username: YWRtaW4=
# kind: Secret
# metadata:
#   creationTimestamp: "2020-07-07T12:03:14Z"
#   name: user-info-from-file
#   namespace: default
#   resourceVersion: "3647019"
#   selfLink: /api/v1/namespaces/default/secrets/user-info-from-file2
#   uid: 57bc52e1-4158-4f13-a0d4-0581b45950db
# type: Opaque
```

### 5.11.2 Secret 활용

#### 볼륨 연결
* Kubernetes의 Secret을 파일 형태로 Pod에 마운트하는 예제

```bash
vi K8S/CH05/secret-volume.yaml
```
```yaml
# secret-volume.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume              # Pod 이름
spec:
  restartPolicy: OnFailure         # 실패 시 재시작
  containers:
  - name: secret-volume            # 컨테이너 이름
    image: busybox                 # busybox 이미지 사용
    command: [ "sh" ]
    args: ["-c", "ls /secret; cat /secret/username"]  # /secret 디렉토리 목록 출력 후, username 파일 내용 출력
    volumeMounts:
    - name: secret                 # 마운트할 볼륨 이름
      mountPath: "/secret"        # Secret 데이터를 /secret 경로에 파일로 마운트

  volumes:
  - name: secret
    secret:
      secretName: user-info        # 미리 생성된 Secret 리소스 이름
```
##### 📦 동작 설명
###### ✅ 마운트 동작
* user-info라는 이름의 Secret에서 데이터를 읽어와 /secret 디렉토리에 파일로 마운트합니다.

```bash
kubectl apply -f K8S/CH05/secret-volume.yaml
# pod/secret-volume created

kubectl logs secret-volume
# password
# username
# admin
```

#### 환경변수 - env
* 환경변수로 secret리소스정보를 추출할 수 있다.

```bash
vi K8S/CH05/secret-env.yaml
```
```yaml
# secret-env.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env                  # Pod 이름
spec:
  restartPolicy: OnFailure         # 실패 시에만 재시작
  containers:
  - name: secret-env               # 컨테이너 이름
    image: busybox                 # busybox 이미지 사용
    command: [ "printenv" ]        # 환경변수를 출력하는 명령 실행
    env:                           # 환경변수 목록
    - name: USERNAME               # 환경변수 이름
      valueFrom:
        secretKeyRef:
          name: user-info          # 참조할 Secret 이름
          key: username            # Secret의 key 이름 (Base64 디코딩되어 적용됨)
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: user-info
          key: password
```
##### 🔐 동작 방식
* user-info라는 Secret에 저장된 username과 password 값을 각각 USERNAME, PASSWORD 환경변수로 주입합니다.
* 컨테이너가 시작되면 printenv 명령이 실행되어 전체 환경변수가 출력되고,
* 그 중 USERNAME, PASSWORD 값을 확인할 수 있습니다.
* user-info Secret이 먼저 존재해야 합니다.


```bash
kubectl apply -f K8S/CH05/secret-env.yaml
# pod/secret-env created

kubectl logs secret-env
# PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
# HOSTNAME=secret-envfrom
# USERNAME=admin
# PASSWORD=1f2d1e2e67df
# ...
```

#### 환경변수 - envFrom
* Kubernetes Secret 데이터를 여러 환경변수로 한 번에 주입하는 방식
* 전체 환경변수를 부르고자할 때 `secretRef`를 사용

```bash
vi K8S/CH05/secret-envfrom.yaml
```
```yaml
# secret-envfrom.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-envfrom             # Pod 이름
spec:
  restartPolicy: OnFailure         # 실패 시에만 재시작
  containers:
  - name: secret-envfrom           # 컨테이너 이름
    image: busybox                 # busybox 이미지 사용 (작고 빠른 셸 환경)
    command: [ "printenv" ]        # 환경변수들을 출력하는 명령어 실행
    envFrom:
    - secretRef:
        name: user-info            # 참조할 Secret 이름
```
###### 🔍 동작 방식
* envFrom.secretRef.name을 통해 user-info Secret의 모든 key가 자동으로 환경변수로 설정됩니다.

```bash
kubectl apply -f K8S/CH05/secret-envfrom.yaml
# pod/secret-envfrom created

kubectl logs secret-envfrom
# PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
# HOSTNAME=secret-envfrom
# username=admin
# password=1f2d1e2e67df
# ...
```

## 5.12 메타데이터 전달
* Pod, Deployment, Service 등 리소스의 정보를 컨테이너 내부로 전달하는 것을 의미
* 이를 통해 애플리케이션이 자신이 속한 환경(이름, 네임스페이스, 라벨 등)을 알 수 있게 된다.

### 5.12.1 Volume 연결

```bash
vi K8S/CH05/downward-volume.yaml
```
```yaml
# downward-volume.yaml
# Downward API를 활용하여 Pod의 라벨 정보를 파일로 컨테이너 내부에 전달하는 예제
apiVersion: v1
kind: Pod
metadata:
  name: downward-volume                 # Pod 이름
  labels:
    zone: ap-north-east                 # 라벨: zone
    cluster: cluster1                   # 라벨: cluster
spec:
  restartPolicy: OnFailure             # 실패할 때만 재시작

  containers:
  - name: downward                     # 컨테이너 이름
    image:  busybox                    # busybox 이미지 사용
    command: ["sh", "-c"]
    args: ["cat /etc/podinfo/labels"]  # 마운트된 파일을 출력
    volumeMounts:
    - name: podinfo                    # podinfo 볼륨을
      mountPath: /etc/podinfo          # /etc/podinfo 에 마운트

  volumes:
  - name: podinfo
    downwardAPI:
      items:
      - path: "labels"                 # 파일 이름: /etc/podinfo/labels
        fieldRef:
          fieldPath: metadata.labels  # 이 Pod의 전체 라벨 값을 전달
```

```bash
kubectl apply -f K8S/CH05/downward-volume.yaml
# pod/downward-volume created

# Pod의 라벨 정보와 비교해 보시기 바랍니다.
kubectl logs downward-volume
# cluster="cluster1"
# zone="ap-north-east"
```

### 5.12.2 환경변수 - env

```bash
vi K8S/CH05/downward-env.yaml
```
```yaml
# downward-env.yaml
# Downward API를 사용해 Pod의 메타데이터 및 상태 정보를 환경변수로 전달하는 예제
apiVersion: v1
kind: Pod
metadata:
  name: downward-env                         # Pod 이름
spec:
  restartPolicy: OnFailure                   # 실패 시에만 재시작
  containers:
  - name: downward                           # 컨테이너 이름
    image: busybox                           # busybox 이미지 (경량 셸)
    command: [ "printenv" ]                  # 환경변수 출력 명령 실행
    env:
    - name: NODE_NAME                        # 실행 중인 노드 이름
      valueFrom:
        fieldRef:
          fieldPath: spec.nodeName

    - name: POD_NAME                         # 현재 Pod 이름
      valueFrom:
        fieldRef:
          fieldPath: metadata.name

    - name: POD_NAMESPACE                    # 네임스페이스
      valueFrom:
        fieldRef:
          fieldPath: metadata.namespace

    - name: POD_IP                           # Pod의 IP 주소
      valueFrom:
        fieldRef:
          fieldPath: status.podIP

```

```bash
kubectl apply -f K8S/CH05/downward-env.yaml
# pod/downward-env created

kubectl logs downward-env
# POD_NAME=downward-env
# POD_NAMESPACE=default
# POD_IP=10.42.0.219
# NODE_NAME=master
# ...
```

### Clean up

```bash
kubectl delete pod --all
```






