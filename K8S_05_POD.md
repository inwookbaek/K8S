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


echo "# K8S" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/inwookbaek/K8S.git
git push -u origin main