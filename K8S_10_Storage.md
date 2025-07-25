# 10. 스토리지

## 📘 Kubernetes Storage란?
### 📌 1. 정의
* Kubernetes Storage는 Pod가 컨테이너 외부에 데이터를 지속적으로 저장할 수 있도록 해주는 시스템입니다.
* 컨테이너(Pod)는 기본적으로 **비휘발성 스토리지(NON-persistent)**를 제공하지 않기 때문에, 
* 데이터 저장을 위해 Volume, PersistentVolume, PVC, StorageClass와 같은 구성 요소들이 사용됩니다.

### 📐 2. 스토리지 필요성
| 상황             | 문제점              | 스토리지 역할              |
| -------------- | ---------------- | -------------------- |
| 웹 앱 로그 저장      | Pod 재시작 시 로그 유실  | 외부 저장소로 보존           |
| MySQL 같은 DB 운영 | 데이터가 Pod와 함께 삭제됨 | Persistent Volume 필요 |
| 업로드 파일 보존      | emptyDir는 휘발성    | PVC로 외부 디스크 연결       |

### 🧱 3. 핵심 구성 요소
| 구성 요소                         | 역할                                    |
| ----------------------------- | ------------------------------------- |
| `Volume`                      | Pod 내에서 마운트되는 디스크 영역 (일시적도 가능)        |
| `PersistentVolume` (PV)       | 클러스터 관리자가 만든 실제 저장소 객체 (물리 디스크 등과 연결) |
| `PersistentVolumeClaim` (PVC) | 개발자가 요청하는 스토리지 사용 청구서 (PV와 연결됨)       |
| `StorageClass`                | 동적 프로비저닝 시 사용되는 정책 세트 (속도/복제 등 포함)    |

### 🔁 4. 흐름도
```plaintext
[ Pod ]
   |
[ PVC (요청) ]
   |
[ PV (할당 or 동적 생성) ]
   |
[ 물리적 저장소 (NFS, EBS, Ceph, local 등) ]

✅ PVC는 사용자가 만들고, PV는 직접 만들거나 StorageClass로 자동 생성할 수 있습니다.
```

### 🧪 5. 실습 예제 요약
#### 1) PVC 정의
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

#### 2) Pod에서 PVC 사용
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: www
  volumes:
  - name: www
    persistentVolumeClaim:
      claimName: my-pvc
```
>✅ 이 구조에서:
>* Nginx는 /usr/share/nginx/html 경로에 1Gi 스토리지를 마운트
>* PVC는 PV를 자동으로 바인딩 (동적 또는 수동)

### ⚙️ 6. StorageClass 예시
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
# PVC에 storageClassName: fast를 지정하면, 해당 정책으로 PV가 자동 생성됩니다.
```

### 📊 7. accessModes 설명
| accessModes     | 의미            | 예시                       |
| --------------- | ------------- | ------------------------ |
| `ReadWriteOnce` | 단일 노드에서 읽기/쓰기 | EBS                      |
| `ReadOnlyMany`  | 여러 노드에서 읽기 전용 | NFS                      |
| `ReadWriteMany` | 여러 노드에서 읽기/쓰기 | NFS, GlusterFS, CephFS 등 |

### 🧩 8. 주요 스토리지 백엔드
| 스토리지                | 유형    | 동적 프로비저닝 지원 |
| ------------------- | ----- | ----------- |
| AWS EBS             | 블록    | ✅           |
| GCP Persistent Disk | 블록    | ✅           |
| NFS                 | 파일    | ⚠️ 직접 설정 필요 |
| Ceph / Rook         | 파일/블록 | ✅           |
| local (hostPath)    | 테스트용  | ❌           |


### 🔍 9. 실무 시 고려사항
| 항목    | 설명                                       |
| ----- | ---------------------------------------- |
| 영속성   | 로그, DB, 업로드 등 유지돼야 할 데이터에 필수             |
| 고가용성  | `ReadWriteMany` 스토리지는 고가용성 구성 가능         |
| 백업/복구 | PV 기반이면 볼륨 스냅샷 등으로 가능                    |
| 용량 관리 | PVC → LimitRange or ResourceQuota로 관리 가능 |


### 🧠 10. 정리
* Kubernetes 스토리지는 휘발성 컨테이너에 영속성을 부여합니다.
* 실무에서는 PVC + StorageClass 기반으로 동적 프로비저닝을 많이 사용합니다.
* DB, 미디어 저장소, 로그 수집 등에 필수 요소입니다.


## 10.1 `PersistentVolume`
>* **PersistentVolume(PV)**는 Kubernetes에서 지속적인 저장소를 제공하기 위해 사용하는 리소스 객체
>* 일반적인 emptyDir 같은 볼륨은 파드(Pod)가 삭제되면 같이 사라지지만, PV는 파드의 수명과 무관하게 데이터를 유지할 수 있도록 설계

### 🔹 주요 개념
1. PersistentVolume (PV)
   - 클러스터 내의 실제 저장소(로컬 디스크, NFS, AWS EBS, GCE PD 등)를 추상화한 객체입니다.
   - 클러스터 관리자가 사전에 할당하거나, 사용자가 PVC(PersistentVolumeClaim)를 통해 동적으로 생성할 수 있습니다.
2. PersistentVolumeClaim (PVC)
   - 사용자가 필요로 하는 저장소의 **요구사항(크기, 접근 방식 등)**을 정의한 객체입니다.
   - PVC가 생성되면 Kubernetes는 조건에 맞는 PV를 찾아 바인딩합니다.
3. StorageClass
   - PV를 동적으로 생성할 때 어떤 방식으로 생성할지를 정의한 템플릿입니다.
   - 예: fast-ssd, slow-hdd, aws-gp2 등

### 🔸 PV 사용 흐름
1. 관리자 또는 스토리지 클래스에 의해 PV 생성
2. 사용자가 PVC로 저장소 요청
3. PVC와 조건이 일치하는 PV가 자동으로 바인딩됨
4. 파드에서 PVC를 마운트해 사용
5. 파드가 삭제되어도 PV의 데이터는 유지됨

### 10.1.1 hostPath PV
* hostPath는 Kubernetes에서 로컬 노드의 디렉토리를 PersistentVolume으로 사용하는 방식
* hostPath는 노드의 파일 시스템 상의 특정 경로를 파드에 마운트하는 볼륨 타입입니다.
* 즉, Kubernetes 클러스터 노드(서버)의 실제 디렉토리를 파드에서 사용할 수 있도록 해줍니다.

#### ⚠️ hostPath 주의사항
* ✅ 테스트용, 단일 노드 클러스터에서만 권장됩니다.
* ❌ 프로덕션 환경에서는 사용 비추천: 노드가 변경되면 데이터가 손실되거나 접근할 수 없습니다.
* 📌 hostPath는 스토리지 계층이 아닌 노드 자체 디스크를 사용하기 때문에 고가용성, 이식성 부족 등의 문제가 있습니다.

#### 🔄 사용 예시 흐름
1. 위와 같은 hostPath PV 생성
2. PVC(PersistentVolumeClaim)를 통해 요청
3. 파드에서 PVC를 마운트해 사용

```bash
vi K8S/CH05/hostpath-pv.yaml
```
```yaml
# hostpath-pv.yaml
apiVersion: v1                    # Kubernetes API 버전 (v1은 core 리소스에 해당)
kind: PersistentVolume           # 이 리소스는 PersistentVolume(PV) 객체임을 명시
metadata:
  name: my-volume                # PV의 이름을 설정 (PVC에서 이 이름으로 직접 참조 가능)
spec:
  storageClassName: manual       # 수동으로 관리되는 스토리지 클래스 이름 (동적 프로비저닝 없음)
  capacity:
    storage: 1Gi                 # PV의 전체 용량 (1GiB)
  accessModes:
    - ReadWriteOnce              # 하나의 노드에서 읽기/쓰기 가능한 접근 모드
  persistentVolumeReclaimPolicy: Retain  # PVC가 삭제돼도 PV의 데이터를 유지 (Retain, Recycle, Delete 중 선택 가능)
  hostPath:
    path: /tmp                   # 클러스터 노드의 /tmp 디렉토리를 PV로 사용
    type: DirectoryOrCreate      # 디렉토리가 없을 경우 자동으로 생성 (Directory, File, Socket 등 가능)

```

```bash
kubectl apply -f K8S/CH05/hostpath-pv.yaml
# persistentvolume/my-volume created

kubectl get pv
# NAME        CAPACITY  ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
# my-volume   1Gi       RWO            Retain           Available           manual                  12s
kubectl describe pv my-volume
```

### 10.1.2 NFS PV
#### ✅ NFS PersistentVolume(PV)란?
* NFS PV는 Kubernetes에서 외부 NFS 서버에 위치한 디렉토리를 파드에 마운트할 수 있도록 해주는 PersistentVolume입니다.
* 이 방식은 여러 파드 또는 여러 노드에서 동시에 접근 가능한 공유 저장소를 제공하기 때문에, 다중 노드에서 데이터 공유가 필요할 때 매우 유용
* 현재, NTS 서버가 없기 떄문에 테스트는 skip

#### ✅ 이 PV를 사용하기 위한 조건
1. NFS 서버 (192.168.164.130)에서 /tmp 디렉토리가 export 되어 있어야 함
  * /etc/exports 예시:
    ```bash
    /tmp  *(rw,sync,no_subtree_check)
    ```
2. 모든 워커 노드에 NFS 클라이언트 패키지 설치
    ```bash
    # Ubuntu:
    sudo apt install nfs-common -y

    # CentOS/RHEL:
    sudo yum install nfs-utils -y
    ```

```bash
vi K8S/CH05/my-nfs.yaml
```
```yaml
# my-nfs.yaml
apiVersion: v1                     # Kubernetes core API 버전
kind: PersistentVolume            # 리소스 종류: PersistentVolume (PV)
metadata:
  name: my-nfs                    # PV의 이름
spec:
  storageClassName: nfs          # 사용할 스토리지 클래스 이름 (PVC와 매칭 시 필요)
  capacity:
    storage: 5Gi                 # 스토리지 용량 (논리적 표시, 실제 NFS 서버 용량과 무관)
  accessModes:
    - ReadWriteMany              # 여러 Pod/노드에서 동시에 읽기 및 쓰기 가능
  mountOptions:                  # NFS 마운트 시 사용할 옵션
    - hard                       # 연결이 끊겨도 재시도하도록 설정 (soft는 에러 발생)
    - nfsvers=4.1                # NFS 버전 4.1 사용 (보통 성능 및 호환성 좋음)
  nfs:
    path: /tmp                   # NFS 서버에서 공유한 디렉토리 경로 (서버에서 export된 경로)
    server: 192.168.164.130      # NFS 서버의 IP 주소 또는 도메인 이름
```

### 10.1.3 awsElasticBlockStore PV
#### ✅ awsElasticBlockStore PV란?
* awsElasticBlockStore는 AWS의 EBS 볼륨을 Kubernetes 파드에서 사용할 수 있도록 해주는 PersistentVolume 타입입니다.
& Amazon EC2 인스턴스에 연결된 EBS 볼륨을 파드에 마운트해서 상태 저장 데이터를 유지할 수 있습니다.

#### 🔸 주요 주의사항
| 항목              | 설명                                                    |
| --------------- | ----------------------------------------------------- |
| `volumeID`      | **이미 AWS에서 생성된 EBS 볼륨의 ID**여야 합니다.                    |
| `fsType`        | 해당 EBS 볼륨에 포맷된 파일 시스템 (보통 `ext4`)                     |
| `ReadWriteOnce` | AWS EBS는 **하나의 노드에서만 마운트 가능**합니다. 다중 파드에서 동시에 마운트 불가. |
| 위치 제약           | \*\*PV를 생성하는 노드와 EBS가 같은 가용 영역(AZ)\*\*에 있어야 함.        |

#### ✅ EBS 볼륨 사전 준비✅ EBS 볼륨 사전 준비
* AWS CLI 또는 콘솔로 EBS 볼륨을 먼저 만들어야 합니다:
```bash
aws ec2 create-volume \
  --availability-zone us-west-2a \
  --size 10 \
  --volume-type gp2
# 생성 후 출력되는 volumeId를 위 YAML의 volumeID에 사용하면 됩니다. 
```

---

**![주의](warning.png) 주의** 

다음 예제는 AWS 플랫폼 위에서 적절한 권한이 부여된 환경에서만 동작합니다. 볼륨을 생성하여 `<volume-id>`를 입력해 주시기 바랍니다. (예제에서는 `vol-1234567890abcdef0`)

```bash
aws ec2 create-volume --availability-zone=eu-east-1a \
  --size=80 --volume-type=gp2
# {
#     "AvailabilityZone": "us-east-1a",
#     "Tags": [],
#     "Encrypted": false,
#     "VolumeType": "gp2",
#     "VolumeId": "vol-1234567890abcdef0",
#     "State": "creating",
#     "Iops": 240,
#     "SnapshotId": "",
#     "CreateTime": "YYYY-MM-DDTHH:MM:SS.000Z",
#     "Size": 80
# }
```
---

```yaml
# aws-ebs.yaml
# PersistentVolume 리소스를 정의하는 YAML 파일입니다.
apiVersion: v1                         # API 버전 (v1은 여전히 최신이며 유효합니다)
kind: PersistentVolume                 # 리소스의 종류는 PersistentVolume입니다.
metadata:
  name: aws-ebs                        # PV의 이름을 'aws-ebs'로 지정합니다.
spec:
  capacity:
    storage: 1Gi                       # 요청하는 저장 용량은 1GiB입니다.
  accessModes:
    - ReadWriteOnce                   # 하나의 노드에서만 읽기/쓰기 가능 (RWO)
  persistentVolumeReclaimPolicy: Retain # PV가 사용 종료되었을 때 삭제되지 않고 보존(Retain)됩니다.
  storageClassName: manual            # 이 PV에 사용할 StorageClass 이름을 명시 ('manual'은 일반적인 예시)
  volumeMode: Filesystem              # 볼륨을 파일시스템 형태로 마운트합니다. (기본값이지만 명시적으로 적어줌)
  awsElasticBlockStore:               # AWS EBS를 백엔드로 사용하는 설정입니다.
    volumeID: <volume-id>             # 실제 EBS Volume ID를 입력해야 합니다. (예: vol-0123456789abcdef0)
    fsType: ext4                      # EBS 볼륨의 파일 시스템 유형은 ext4입니다.

```

### 10.1.4 그외 다른 `PersistentVolume`
#### 📌 다른 `PersistentVolume` 비교표
| 타입            | 주요 목적                    | 라이프사이클   | 공유 가능성   | 사용 예시    |
| ------------- | ------------------------ | -------- | -------- | -------- |
| `azureDisk`   | Azure Persistent Storage | 파드와 무관   | ❌ (RWO만) | DB 볼륨    |
| `emptyDir`    | 임시 저장공간 (캐시, 임시파일)       | 파드 수명 동안 | ❌        | 캐시, 버퍼   |
| `downwardAPI` | 파드 메타데이터 주입              | 파드 수명 동안 | 읽기 전용    | 모니터링, 추적 |
| `configMap`   | 환경설정/구성 데이터 주입           | 파드 수명 동안 | 읽기 전용    | 설정파일 분리  |


더 다양한 종류들을 확인하고 싶으시다면 다음 페이지를 참고 바랍니다. [https://kubernetes.io/docs/concepts/storage/volumes](https://kubernetes.io/docs/concepts/storage/volumes)


## 10.2 PersistentVolumeClaim
* PersistentVolumeClaim(PVC)은 사용자가 원하는 스토리지의 요청(Request) 을 나타냅니다. 쉽게 말해:
  - PersistentVolume(PV): 실제 스토리지 (공급자 제공 — EBS, AzureDisk, NFS 등)
  - PersistentVolumeClaim(PVC): 사용자의 요청 (파드가 사용하고 싶다고 "요청"하는 볼륨)

### ✅ PVC가 필요한 이유
>Pod는 직접 PersistentVolume을 사용하지 않습니다. 대신 PersistentVolumeClaim을 통해 스토리지를 요청하고, Kubernetes는 해당 PVC에 맞는 PV를 자동으로 연결(Binding)합니다.

### ✅ PVC 기본 구조 예시
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc                            # PVC 이름
spec:
  accessModes:
    - ReadWriteOnce                       # 접근 모드 (PV와 호환되어야 함)
  resources:
    requests:
      storage: 1Gi                        # 요청하는 스토리지 용량
  storageClassName: manual               # PV의 storageClass와 일치해야 함
```

### ✅ 요약 흐름
```text
Pod → PVC → PV
      ↑       ↑
    요청      실제 볼륨
```

### 📝 추가 팁
| 속성                           | 설명                                               |
| ---------------------------- | ------------------------------------------------ |
| `accessModes`                | `ReadWriteOnce`, `ReadOnlyMany`, `ReadWriteMany` |
| `storageClassName`           | PV와 PVC를 연결하는 키 역할 (동적/수동 프로비저닝 제어)              |
| `resources.requests.storage` | 최소한 요청하는 스토리지 크기                                 |

### ❓ PVC가 없으면?
* 파드는 PV에 직접 접근할 수 없기 때문에, 스토리지를 사용하지 못합니다.
* PVC는 Kubernetes의 스토리지 요청 인터페이스입니다.

```bash
vi K8S/CH05/my-pvc.yaml
```
```yaml
# my-pvc.yaml
apiVersion: v1                              # Kubernetes API 버전 (v1은 PVC에 대해 현재도 최신입니다)
kind: PersistentVolumeClaim                 # PVC 리소스: 파드가 사용할 스토리지를 요청함
metadata:
  name: my-pvc                              # PVC의 이름
spec:
  accessModes:
    - ReadWriteOnce                         # 하나의 노드에서 읽기/쓰기 가능 (RWO)
  resources:
    requests:
      storage: 1Gi                          # 요청하는 스토리지 용량
  volumeMode: Filesystem                    # (최신 관행) 볼륨을 파일시스템으로 사용할 것인지 명시 (기본값이지만 명확성 향상)
  storageClassName: manual                  # 수동 바인딩을 위해 storageClass 지정

```

```bash
kubectl apply -f K8S/CH05/my-pvc.yaml
# persistentvolumeclaim/my-pvc created

# 앞에서 생성한 my-volume을 선점하였습니다.
kubectl get pvc
# NAME          STATUS   VOLUME      CAPACITY  ACCESS MODES    ... 
# my-pvc        Bound    my-volume   1Gi       RWO             ...

kubectl get pv
# NAME        CAPACITY  ACCESS MODES   RECLAIM POLICY   STATUS   
# my-volume   1Gi       RWO            Retain           Bound    
#
#                 CLAIM              STORAGECLASS    REASON   AGE
#                 default/my-pvc     manual                   11s
```

```bash
vi K8S/CH05/use-pvc.yaml
```
```yaml
# use-pvc.yaml
apiVersion: v1                            # Pod 리소스는 여전히 v1 API를 사용합니다
kind: Pod                                 # 이 리소스는 Pod를 정의합니다
metadata:
  name: use-pvc                           # 생성할 Pod의 이름
spec:
  containers:
  - name: nginx                           # 컨테이너 이름
    image: nginx                          # 사용할 컨테이너 이미지 (nginx 웹 서버)
    volumeMounts:
    - mountPath: /test-volume             # 컨테이너 내부에 마운트할 경로
      name: vol                           # 마운트할 볼륨 이름 (아래 volumes의 name과 일치해야 함)
  volumes:
  - name: vol                             # 위의 volumeMount에서 참조한 이름
    persistentVolumeClaim:
      claimName: my-pvc                   # 사용할 PVC(PersistentVolumeClaim) 이름
```
#### ✅ 주요 설명
* 이 Pod는 my-pvc라는 PVC를 사용해서 /test-volume 경로에 볼륨을 마운트합니다.
* my-pvc는 앞서 정의한 PVC로, 이미 바인딩된 PV가 있어야 합니다.
* 컨테이너는 해당 경로에서 읽기/쓰기가 가능하며, PV가 ReadWriteOnce이면 하나의 노드에만 동시에 마운트됩니다.

#### ✅ 최신 관행 적용 여부
| 항목                      | 설명                                |
| ----------------------- | --------------------------------- |
| `volumeMounts`          | 정상, 최신 YAML 구조와 동일                |
| `persistentVolumeClaim` | 정확히 명시되어 있으며, 최신 Kubernetes에서도 동일 |
| 추가 개선 포인트 없음 ✅          | 불필요한 항목 없이 깔끔하게 작성됨               |

#### ✅ 팁: PVC를 여러 Pod에서 쓰고 싶다면?
* PVC가 사용하는 PV가 accessModes: ReadWriteMany를 지원하는 경우 (예: NFS, CSI 기반 RWX 스토리지), 여러 Pod에서 동시에 사용할 수 있습니다. 
* 그렇지 않다면 하나의 노드/Pod만 동시에 사용 가능합니다.

```bash
kubectl apply -f K8S/CH05/use-pvc.yaml
# pod/use-pvc created

# 데이터 저장
kubectl get pod use-pvc -owide
kubectl exec use-pvc -- sh -c "echo 'hello' > /test-volume/hello.txt"
kubectl exec use-pvc -- cat /test-volume/hello.txt
```

```bash
kubectl delete pod use-pvc
# pod/use-pvc deleted

kubectl apply -f K8S/CH05/use-pvc.yaml
# pod/use-pvc created

kubectl exec use-pvc -- cat /test-volume/hello.txt
# hello
```

### Clean up

```bash
kubectl delete pod use-pvc
kubectl delete pvc my-pvc
kubectl delete pv my-volume
```


## 10.3 `StorageClass`

* NFS를 Kubernetes에서 사용할 때는 StorageClass를 통해 동적 프로비저닝도 가능합니다. 
* 단, 이를 위해서는 NFS CSI 드라이버 또는 NFS 프로비저너가 클러스터에 설치되어 있어야 합니다.

### 🧱 StorageClass란?
* StorageClass는 PVC를 통해 스토리지를 요청할 때,
* 어떤 스토리지 타입 (EBS, NFS, Ceph 등) 을,
* 어떤 속성(속도, 크기, 접근 방식) 으로,
* 어떤 Provisioner(CSI) 를 통해 만들지 지정하는 설정 템플릿입니다.

# 📦 예시: AWS EBS용 StorageClass
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  fsType: ext4
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

### 📁 예시: NFS용 StorageClass
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-client
provisioner: nfs-subdir-external-provisioner
parameters:
  archiveOnDelete: "true"   # PVC 삭제 시 데이터를 백업 디렉터리로 이동
reclaimPolicy: Retain
```

### 📌 StorageClass를 사용하는 PVC 예시
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: ebs-sc
```

### 🧠 기본 StorageClass
```bash
kubectl get storageclass
```

### 🏁 정리 요약
| 항목              | 설명                                                                |
| --------------- | ----------------------------------------------------------------- |
| 역할              | PVC 요청에 따라 스토리지를 어떻게 만들지 결정                                       |
| 주요 구성 요소        | `provisioner`, `parameters`, `reclaimPolicy`, `volumeBindingMode` |
| 연결 방식           | PVC → StorageClass → PV (자동 생성됨)                                  |
| 기본 StorageClass | `kubectl get storageclass`로 확인 가능                                 |


### 10.3.1 `StorageClass` 소개
* local-path: https://github.com/rancher/local-path-provisioner

```bash
# local-path라는 이름의 StorageClass, sc = StorageClass의 약어
kubectl get sc
# NAME                  PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
# local-path (default)  rancher.io/local-path   Delete          WaitForFirstConsumer   false                  20d

kubectl get sc local-path -oyaml
# apiVersion: storage.k8s.io/v1
# kind: StorageClass
# metadata:
#   annotations:
#     objectset.rio.cattle.io/id: ""
#   ...
#   name: local-path
#   resourceVersion: "172"
#   selfLink: /apis/storage.k8s.io/v1/storageclasses/local-path
#   uid: 3aede349-0b94-40c8-b10a-784d38f7c120
# provisioner: rancher.io/local-path
# reclaimPolicy: Delete
# volumeBindingMode: WaitForFirstConsumer
```

```bash
vi K8S/CH05/my-pvc-sc.yaml
```
```yaml
# my-pvc-sc.yaml
apiVersion: v1                                 # PersistentVolumeClaim 리소스는 v1 API를 사용함 (최신 버전)
kind: PersistentVolumeClaim                    # 이 리소스는 PVC(PersistentVolumeClaim)를 정의함
metadata:
  name: my-pvc-sc                              # PVC의 이름. Pod에서 이 이름으로 참조 가능
spec:
  storageClassName: local-path                 # 사용할 StorageClass 이름 (예: local-path, standard 등)
  accessModes:
    - ReadWriteOnce                            # 하나의 노드에서만 읽기/쓰기가 가능한 모드 (RWO)
  resources:
    requests:
      storage: 1Gi                             # 요청할 스토리지 용량 (1Gi = 1 Gibibyte)
```
#### ✅ 설명 요약
| 항목                           | 내용                                                              |
| ---------------------------- | --------------------------------------------------------------- |
| `storageClassName`           | 이 PVC가 어떤 StorageClass로부터 스토리지를 동적으로 할당받을지 지정                   |
| `accessModes`                | `ReadWriteOnce`는 단일 노드에서 읽기/쓰기 가능 (일반적인 EBS, local-path의 기본 모드) |
| `resources.requests.storage` | 최소 요구하는 스토리지 용량. 스토리지 프로비저너는 이 크기를 기준으로 PV를 생성함                 |


```bash
kubectl apply -f K8S/CH05/my-pvc-sc.yaml
# persistentvolumeclaim/my-pvc-sc created

kubectl get pvc my-pvc-sc
# NAME         STATUS    VOLUME     CAPACITY   ACCESS MODES  STORAGECLASS   AGE 
# my-pvc-sc    Pending                                       local-path     11s 

```bash
vi K8S/CH05/use-pvc-sc.yaml
```

```yaml
# use-pvc-sc.yaml
apiVersion: v1                             # Pod 리소스는 v1 API를 사용 (기본이며 최신 버전)
kind: Pod                                  # 이 리소스는 Pod를 정의함
metadata:
  name: use-pvc-sc                         # 생성할 Pod의 이름

spec:
  volumes:
  - name: vol                              # Pod에서 사용할 볼륨 이름 정의
    persistentVolumeClaim:
      claimName: my-pvc-sc                 # 연결할 PVC 이름 (my-pvc-sc.yaml에서 정의한 이름과 일치해야 함)

  containers:
  - name: nginx                            # 컨테이너 이름
    image: nginx                           # 사용할 컨테이너 이미지 (공식 nginx)
    volumeMounts:
    - mountPath: "/usr/share/nginx/html"  # 컨테이너 내부에서 볼륨이 마운트될 경로
      name: vol                            # 마운트할 볼륨 이름 (위에 정의한 `vol`과 동일해야 함)
```

```bash
# pod 생성
kubectl apply -f K8S/CH05/use-pvc-sc.yaml
# pod/use-pvc-sc created

# STATUS가 pending 에서 Bound로 변경
# Pod가 PVC를 사용하는 경우 동적으로 볼륨이 생성
kubectl get pvc my-pvc-sc
# NAME         STATUS   VOLUME            CAPACITY   ACCESS MODES   STORAGECLASS   AGE
# my-pvc-sc    Bound    pvc-479cff32-xx   1Gi        RWO            local-path     92s         

# 기존에 생성하지 않은 신규 volume이 생성된 것을 확인
kubectl get pv
# NAME              CAPACITY  ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS   REASON   AGE
# pvc-479cff32-xx   1Gi       RWO            Delete           Bound    default/my-pvc-sc   local-path              3m

# pv 상세 정보 확인 (hostPath 등)
kubectl get pv pvc-479cff32-xx -oyaml
# apiVersion: v1
# kind: PersistentVolume
# metadata:
#     ...
#   name: pvc-b1727544-f4be-4cd6-acb7-29eb8f68e84a
#   ...
# spec:
#   ...
#   hostPath:
#     path: /var/lib/rancher/k3s/storage/pvc-b1727544-f4be-4cd6-acb7-29eb8f68e84a
#     type: DirectoryOrCreate
#   nodeAffinity:
#     required:
#       nodeSelectorTerms:
#       - matchExpressions:
#         - key: kubernetes.io/hostname
#           operator: In
#           values:
#           - worker
#    ...
```

### 10.3.2 NFS StorageClass 설정
* NFS StorageClass는 외부 NFS 서버를 통해 동적으로 PersistentVolume(PV)를 생성하고 PVC와 자동으로 연결되도록 도와주는 리소스

#### 🧭 NFS StorageClass 개념 정리
| 항목       | 설명                                                                                                                   |
| -------- | -------------------------------------------------------------------------------------------------------------------- |
| 📦 목적    | NFS 기반 스토리지를 PVC와 연결할 수 있도록 동적 Provisioning 지원                                                                       |
| 🔧 구성요소  | NFS 서버, Helm 기반 NFS 프로비저너, StorageClass, PVC                                                                         |
| 🧰 사용 도구 | [`nfs-subdir-external-provisioner`](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner) (공식 Helm 차트) |

#### ✅ 1. 외부 NFS 서버 기반 NFS StorageClass 설정
##### 🔹 1단계: NFS 서버 준비 (기존 NFS 서버)
* 서버 IP: 192.168.164.139
* 공유 경로: /exports/k8s-data
* 클러스터 노드가 해당 공유 디렉토리에 접근 가능해야 함 (mount -t nfs로 테스트 가능)

##### 🔹 2단계: Helm 차트 설치
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add nfs-subdir https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm repo update

helm search repo nfs
# NAME                                            CHART VERSION   APP VERSION     DESCRIPTION
# nfs-subdir/nfs-subdir-external-provisioner      4.0.18          4.0.2           nfs-subdir-external-provisioner is an automatic...

helm install nfs-client nfs-subdir/nfs-subdir-external-provisioner \
  --set nfs.server=192.168.1.100 \
  --set nfs.path=/exports/k8s-data \
  --set storageClass.name=nfs-client \
  --namespace nfs \
  --create-namespace
```

#### ✅ 2. StorageClass 직접 정의 (참고용)
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-client
provisioner: nfs-subdir-external-provisioner
parameters:
  archiveOnDelete: "true"        # PVC 삭제 시 데이터를 백업 디렉토리로 보관
reclaimPolicy: Retain            # PVC 삭제 후에도 PV는 유지
volumeBindingMode: Immediate     # 즉시 PV 바인딩
```

#### ✅ 3. PVC에서 사용 예시
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-nfs-pvc
spec:
  accessModes:
    - ReadWriteMany                # NFS는 RWX(ReadWriteMany)를 지원함
  storageClassName: nfs-client    # 위에서 생성된 StorageClass 참조
  resources:
    requests:
      storage: 1Gi
```

#### ✅ 4. Pod에서 PVC 사용 예시
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: use-nfs-pvc
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: my-vol
      mountPath: /usr/share/nginx/html
  volumes:
  - name: my-vol
    persistentVolumeClaim:
      claimName: my-nfs-pvc
```     

#### ✅ 요약 정리
| 구성 요소        | 설명                                     |
| ------------ | -------------------------------------- |
| NFS 서버       | `/exports/k8s-data` 공유 폴더 제공           |
| Helm 차트      | NFS 프로비저너 + StorageClass 자동 설치         |
| StorageClass | `nfs-client` 이름으로 동적 볼륨 생성 가능          |
| PVC          | `storageClassName: nfs-client`으로 자동 연동 |
| Pod          | PVC를 마운트하여 NFS 스토리지 사용 가능              |

###### 🔧 내부에 NFS 서버 포함할 경우
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add nfs-subdir https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm repo update

# nfs가 이미 존재여부 확인
helm list -n ctrl

# ✅ 릴리스 삭제 방법
helm uninstall nfs -n ctrl

helm search repo nfs
# NAME                                            CHART VERSION   APP VERSION     DESCRIPTION
# nfs-subdir/nfs-subdir-external-provisioner      4.0.18          4.0.2           nfs-subdir-external-provisioner is an automatic...

# local 내부에 NTFS 서버 생성
helm install nfs stable/nfs-server-provisioner \
    --set persistence.enabled=true \
    --set persistence.size=10Gi \
    --version 1.1.1 \
    --namespace ctrl

# NAME: nfs
# LAST DEPLOYED: Wed Jul  8 13:19:46 2020
# NAMESPACE: ctrl
# STATUS: deployed
# REVISION: 1
# TEST SUITE: None
# NOTES:
# ...
```
###### 🔧 각 옵션 설명
| 옵션                               | 설명                                                            |
| -------------------------------- | ------------------------------------------------------------- |
| `nfs`                            | Helm 릴리스 이름 (이 이름으로 Helm 리소스를 추적)                             |
| `stable/nfs-server-provisioner`  | 설치할 Helm 차트 위치 (`stable` 저장소에 있는 `nfs-server-provisioner` 차트) |
| `--set persistence.enabled=true` | NFS 서버의 데이터를 PVC에 저장하도록 설정 (내부 볼륨 사용)                         |
| `--set persistence.size=10Gi`    | 위 PVC의 크기를 10Gi로 설정                                           |
| `--version 1.1.1`                | 특정 버전의 차트를 설치 (안정성과 호환성 확보용)                                  |
| `--namespace ctrl`               | Helm 리소스를 배치할 네임스페이스                                          |
| `--create-namespace`             | 네임스페이스가 없으면 자동으로 생성                                           |


```bash
# ✅ 1. 설치된 리소스 확인 명령
# 🔹 Helm 릴리스 확인 (특정 네임스페이스)
helm list -n ctrl

# 🔹 Helm 릴리스가 생성한 리소스 전체 보기
kubectl get all -n ctrl

# ✅ 2. 삭제 후 남아있는 PVC/PV 정리 방법
# 🔹 Helm 릴리스 삭제
helm uninstall nfs -n ctrl

# 🔹 PVC 삭제
kubectl delete pvc -n ctrl <PVC이름>
# kubectl delete pvc -n ctrl --all


🔹 PV 삭제
kubectl get pv
kubectl delete pv <PV이름>

# ✅ 3. StorageClass 확인 및 테스트 방법
# 🔹 현재 존재하는 StorageClass 확인
kubectl get storageclass

# 🔹 StorageClass 상세 확인
kubectl describe storageclass nfs-client

# nfs-server-provisioner라는 Pod가 생성되어 있습니다.
# STATUS : ContainerCreating -> Running 확인
kubectl get pod -n ctrl
# NAME                                                   READY   STATUS              RESTARTS   AGE
# ...
# nfs-nfs-subdir-external-provisioner-55d7d88595-tv95b   0/1     ContainerCreating   0          22s
```

--- -------------------------------------------------------------------------------------------------

###### ✅ 외부에 NTFS 서버 생성할 경우
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add nfs-subdir https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm repo update

# 외부에 NTFS 서버 생성
helm install nfs nfs-subdir/nfs-subdir-external-provisioner \
  --set nfs.server=192.168.164.130 \
  --set nfs.path=/exports/k8s \
  --set storageClass.name=nfs-client \
  --namespace ctrl \
  --create-namespace
 
#  --create-namespace 옵션을 추가하면 ctrl 네임스페이스가 없다면 자동으로 생성됩니다. 

# STATUS가 Running이 아닐경우 (계속 ContainerCreating) 
# ❗ 1. NFS 서버가 실제로 동작 중인지 확인 필요
# NFS 서비스 상태 확인
sudo systemctl status nfs-server

# NFS export 확인
sudo exportfs -v

# ✅ 없으면 아래와 같이 설정합니다:
# 🔹 NFS 서버 설치

sudo apt update
sudo apt install -y nfs-kernel-server

# 🔹 공유 디렉토리 생성 및 권한 설정
sudo mkdir -p /exports/k8s
sudo chmod 777 /exports/k8s

# 🔹 /etc/exports 설정
sudo vi /etc/exports
# 아래 내용추가
# /exports/k8s *(rw,sync,no_subtree_check,no_root_squash)

# 🔹 설정 적용 및 서비스 시작
sudo exportfs -ra
sudo systemctl enable --now nfs-server

# 🔹 방화벽 확인 (필요시 비활성화)
# 테스트용 전체 해제 (주의)
sudo ufw disable

# 또는 NFS 관련 포트만 열기 (2049, 111)
sudo ufw allow 2049/tcp
sudo ufw allow 111/tcp

# 🔄 3. 다시 Helm 설치 (이미 설정되었으면 이 단계는 생략)
helm uninstall nfs -n ctrl
helm install nfs nfs-subdir/nfs-subdir-external-provisioner \
  --set nfs.server=192.168.164.130 \
  --set nfs.path=/exports/k8s \
  --set storageClass.name=nfs-client \
  --namespace ctrl \
  --create-namespace
  
```
📌 옵션별 설명
| 옵션                                           | 설명                                                                                                |
| -------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `nfs`                                        | Helm 릴리스 이름입니다. 해당 이름으로 Kubernetes 리소스들이 생성됩니다. (예: Pod, Deployment 이름 앞에 붙음)                     |
| `nfs-subdir/nfs-subdir-external-provisioner` | 사용할 Helm 차트입니다. <br> `nfs-subdir`는 Helm 저장소 이름, <br> `nfs-subdir-external-provisioner`는 차트 이름입니다. |
| `--set nfs.server=192.168.164.130`           | 사용할 **NFS 서버의 IP 주소**입니다. <br> PVC 요청 시 이 서버에 마운트하여 디렉터리를 생성합니다.                                  |
| `--set nfs.path=/exports/k8s`                | NFS 서버 내 공유 디렉터리 경로입니다. <br> 이 경로 안에 PVC별 하위 디렉터리(`pvc-xxxx`)가 생성됩니다.                             |
| `--set storageClass.name=nfs-client`         | 생성할 StorageClass의 이름입니다. <br> 이후 PVC에서 `storageClassName: nfs-client` 로 지정해 사용할 수 있습니다.           |
| `--namespace ctrl`                           | 차트를 설치할 Kubernetes 네임스페이스입니다. <br> (`ctrl`이라는 네임스페이스가 존재해야 함)                                     |
| `--create-namespace`                         | 위에서 지정한 네임스페이스(`ctrl`)가 없다면 자동으로 생성합니다.                                                           |

--- -------------------------------------------------------------------------------------------------

###### ✅  `stable/nfs-server-provisioner`  vs `nfs-subdir/nfs-subdir-external-provisioner` 차이표
| 항목              | `stable/nfs-server-provisioner`  | `nfs-subdir/nfs-subdir-external-provisioner` |
| --------------- | -------------------------------- | -------------------------------------------- |
| 📦 차트 출처        | Helm `stable` 저장소 (Deprecated)   | CNCF SIG 공식 GitHub Helm 저장소                  |
| 🖥 NFS 서버 포함 여부 | ✅ **내부에 NFS 서버를 포함**             | ❌ **외부 NFS 서버 필요 (별도 구성)**                   |
| 📁 볼륨 경로        | `/export` 등 내부에서 관리              | 외부 NFS 서버의 경로 (`nfs.path`)                   |
| 🧠 사용 목적        | **로컬 테스트용**에 적합 (모두-in-one)      | **실제 운영 환경**의 외부 NFS 활용                      |
| 🛠 Helm 차트 상태   | 더 이상 유지보수 안됨 (stable deprecated) | 현재 유지보수 중                                    |
| 🔐 보안           | 외부 접근 제한 불가 (모든 노드 공유)           | 외부 NFS 서버에서 보안 제어 가능                         |
| 📁 PVC 동작 방식    | `/export/pvc-xxxx` 생성            | `/exports/k8s/pvc-xxxx`에 서브디렉터리 자동 생성        |

###### 🎯 어떤 걸 써야 할까?
| 목적             | 추천 차트                                            |
| -------------- | ------------------------------------------------ |
| 로컬 테스트 / 실습용   | `stable/nfs-server-provisioner` (빠르고 간단)         |
| 실제 클러스터, 운영 환경 | `nfs-subdir-external-provisioner` (보안/성능/확장성 우수) |

```bash
# nfs-server-provisioner라는 Pod가 생성되어 있습니다.
kubectl get pod -n ctrl
# NAME                                                   READY   STATUS    RESTARTS   AGE
# ...
# nfs-nfs-subdir-external-provisioner-55d7d88595-8bvms   1/1     Running   0          8m25s


# 이것은 StatefulSet로 구성되어 있습니다.
kubectl get statefulset  -n ctrl
# NAME                         READY   AGE
# nfs-nfs-server-provisioner   1/1     57s

# nfs-server-provisioner Service도 있습니다.
kubectl get svc  -n ctrl
# NAME                                  TYPE           CLUSTER-IP     ..
# nginx-nginx-ingress-default-backend   ClusterIP      10.43.79.133   .. 
# nginx-nginx-ingress-controller        LoadBalancer   10.43.182.174  ..  
# nfs-nfs-server-provisioner            ClusterIP      10.43.248.122  ..  

# 새로운 nfs StorageClass 생성
kubectl get sc
# NAME                 PROVISIONER                                RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
# local-path (default) rancher.io/local-path                      Delete          WaitForFirstConsumer   false                  20d
# nfs                  cluster.local/nfs-nfs-server-provisioner   Delete          Immediate              true                   10s
```

```bash
vi K8S/CH05/nfs-sc.yaml
```
```yaml
# nfs-sc.yaml
# NFS StorageClass를 이용하여 PVC를 생성하면 자동으로 PV가 샹성되고 
# 사용자가 직접 NFS 정보를 몰라도 자동으로 연결된다.
apiVersion: v1                        # 리소스의 API 버전 (PersistentVolumeClaim은 v1)
kind: PersistentVolumeClaim           # 생성할 리소스의 종류: PVC
metadata:
  name: nfs-sc                        # PVC 이름 (Pod에서 참조할 이름)

spec:
  # 기존 local-path에서 nfs로 변경
  storageClassName: nfs              # 사용할 StorageClass 이름 (nfs-subdir-external-provisioner 설치 시 설정한 값과 동일해야 함)

  # accessModes를 ReadWriteMany로 변경
  accessModes:                       # 볼륨 접근 모드 설정
    - ReadWriteMany                  # 여러 Pod에서 동시에 읽기/쓰기가 가능한 모드 (NFS에서 주로 사용)

  resources:                         # 요청하는 스토리지 자원 설정
    requests:
      storage: 1Gi                   # 최소 1Gi 용량을 요청 (프로비저너가 이 크기의 디렉터리를 생성함)

```

```bash
kubectl apply -f K8S/CH05/nfs-sc.yaml
# persistentvolumeclaim/nfs-sc created

# pvc 리소스 확인
kubectl get pvc
# NAME        STATUS   VOLUME             CAPACITY   ACCESS MODES  ...
# my-pvc-sc   Bound    pvc-b1727544-xxx   1Gi        RWO           ...
# nfs-sc      Bound    pvc-49fea9cf-xxx   1Gi        RWO           ...

# pv 리소스 확인
kubectl get pv pvc-49fea9cf-xxx
# NAME                CAPACITY   ACCESS MODES   RECLAIM  POLICY  STATUS    CLAIM            STORAGECLASS   REASON  AGE
# pvc-49fea9cf-xxx    1Gi        RWX            Delete           Bound     default/nfs-sc   nfs                    5m

# pv 상세 정보 확인 (nfs 마운트 정보)
kubectl get pv pvc-49fea9cf-xxx -oyaml
# apiVersion: v1
# kind: PersistentVolume
# metadata:
#   ...
# spec:
#   accessModes:
#   - ReadWriteMany
#   capacity:
#     storage: 1Gi
#   claimRef:
#     apiVersion: v1
#     kind: PersistentVolumeClaim
#     name: nfs-sc
#     namespace: default
#     resourceVersion: "10084380"
#     uid: 2e95f6c4-2b43-4375-808f-0c93e44a1003
#   mountOptions:
#   - vers=3
#   nfs:
#     path: /export/pvc-2e95f6c4-2b43-4375-808f-0c93e44a1003
#     server: 10.43.248.122
#   persistentVolumeReclaimPolicy: Delete
#   storageClassName: nfs
#   volumeMode: Filesystem
# status:
#   phase: Bound
```

##### PVC를 사용하는  Pod 생성하기
```bash
vi K8S/CH05/use-nfs-sc.yaml
```
```yaml
# use-nfs-sc.yaml

# 첫 번째 Pod: master 노드에 생성
apiVersion: v1                            # API 버전 (v1은 core 리소스)
kind: Pod                                 # 리소스 종류: Pod
metadata:
  name: use-nfs-sc-master                 # Pod 이름
spec:
  volumes:
  - name: vol                             # 볼륨 이름 정의
    persistentVolumeClaim:
      claimName: nfs-sc                   # 앞서 생성한 PVC(nfs-sc)를 참조

  containers:
  - name: nginx                           # 컨테이너 이름
    image: nginx                          # 사용할 이미지 (nginx 웹 서버)
    volumeMounts:
    - mountPath: "/usr/share/nginx/html" # 컨테이너 내 마운트 위치
      name: vol                           # 마운트할 볼륨 이름 (위 volumes에서 정의한 이름)

  nodeSelector:
    kubernetes.io/hostname: master        # 해당 Pod를 master 노드에 고정되도록 설정

---

# 두 번째 Pod: worker 노드에 생성
apiVersion: v1
kind: Pod
metadata:
  name: use-nfs-sc-worker                 # Pod 이름
spec:
  volumes:
  - name: vol
    persistentVolumeClaim:
      claimName: nfs-sc                   # 동일한 PVC를 사용 (ReadWriteMany이므로 공유 가능)

  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: "/usr/share/nginx/html"
      name: vol

  nodeSelector:
    kubernetes.io/hostname: worker1        # 해당 Pod를 worker 노드에 고정되도록 설정
```

```bash
kubectl apply -f K8S/CH05/use-nfs-sc.yaml
# pod/use-nfs-sc-master created
# pod/use-nfs-sc-worker created

kubectl get pod -o wide
# NAME               READY  STATUS    RESTARTS  AGE   IP          NODE
# ...
# use-nfs-sc-master  1/1    Running   0         19s   10.42.0.8   master
# use-nfs-sc-worker  1/1    Running   0         19s   10.42.0.52  worker
```

```bash
# master Pod에 index.html 파일을 생성합니다.
kubectl exec use-nfs-sc-master -- sh -c \
      "echo 'hello world' >> /usr/share/nginx/html/index.html"
kubectl exec use-nfs-sc-master -- cat /usr/share/nginx/html/index.html

# worker Pod에서 호출을 합니다.
kubectl exec use-nfs-sc-worker -- curl -s localhost
kubectl exec use-nfs-sc-worker -- cat /usr/share/nginx/html/index.html
# hello world
```

### Clean up

```bash
kubectl delete pod --all
kubectl delete pvc nfs-sc my-pvc-sc
```

## 10.4 쿠버네티스 스토리지 활용

### 10.4.1 MinIO 스토리지 설치
* S3-compatible 오브젝트 스토리지인 MinIO(https://)
* MinIO는 쿠버네티스에서 자주 사용하는 고성능 오브젝트 스토리지입니다.
* Amazon S3와 호환되는 API를 제공하여, 클라우드 네이티브 환경에서 경량 오브젝트 스토리지로 많이 사용

#### ☁️ MinIO란?
| 항목    | 설명                                        |
| ----- | ----------------------------------------- |
| 유형    | 오브젝트 스토리지 (S3 호환)                         |
| 특징    | 고성능, 경량, 자체 호스팅 가능                        |
| 사용 사례 | 백업 저장소, ML 데이터 저장, 로그 보관 등                |
| 배포 형태 | Standalone 또는 Distributed (4개 이상의 디스크/노드) |


```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# MinIO 차트 다운로드 및 압축 해제
# helm pull bitnami/minio --version 13.3.2 --untar
helm pull bitnami/minio --untar 
ll -al minio/

# vi 편집기를 실행합니다.
vi minio/values.yaml
```
##### 🧾 항목별 비교: stable/minio → bitnami/minio
| 항목              | `stable/minio` (`values.yaml`)                                       | `bitnami/minio` (`values.yaml`)                                      | 비고       |
| --------------- | -------------------------------------------------------------------- | -------------------------------------------------------------------- | -------- |
| accessKey       | `accessKey: minioadmin`                                              | `auth.rootUser: minioadmin`                                          | ✅ 이름 변경됨 |
| secretKey       | `secretKey: minioadmin`                                              | `auth.rootPassword: minioadmin`                                      | ✅ 이름 변경됨 |
| storageClass    | `persistence.storageClass: nfs-client`                               | `persistence.storageClass: nfs-client`                               | 동일       |
| ingress.enabled | `ingress.enabled: true`                                              | `ingress.enabled: true`                                              | 동일       |
| ingress.hosts   | `ingress.hosts[0]: minio.example.com`                                | `ingress.hostname: minio.example.com`                                | 약간 다름    |
| resources       | `resources.requests.cpu: 100m`<br>`resources.requests.memory: 128Mi` | `resources.requests.cpu: 100m`<br>`resources.requests.memory: 128Mi` | 동일       |



```yaml
...
# 약 112번째 줄에서 accessKey와 secretKey 변경
## 인증 정보
auth:
  rootUser: "myaccess"
  rootPassword: "mysecret"

## 스토리지 설정
persistence:
  enabled: true
  storageClass: "nfs"
  accessModes:
    - ReadWriteMany
  size: 1Gi

## Ingress 설정
ingress:
  enabled: true
  labels: {}

  annotations:
    kubernetes.io/ingress.class: nginx

  hostname: minio.192.168.164.130.sslip.io
  path: /
  pathType: ImplementationSpecific

  tls: []  # TLS 안 쓰면 비워두기

## 리소스 요청 (리소스 줄이기)
resources:
  requests:
    memory: 1Gi  # 기존 4Gi → 1Gi로 감소
    cpu: 250m    # 기본값 또는 적당히 설정 (필요 시 추가)
```

```bash
# minio 삭제/설치
helm uninstall minio
helm install minio ./minio

# 접속정보 : user/pass
export ROOT_USER=$(kubectl get secret --namespace default minio -o jsonpath="{.data.root-user}" | base64 -d)
export ROOT_PASSWORD=$(kubectl get secret --namespace default minio -o jsonpath="{.data.root-password}" | base64 -d)
echo ${ROOT_USER}
echo ${ROOT_PASSWORD}
echo "ID: $ROOT_USER, PW: $ROOT_PASSWORD"

kubectl get pod
# NAME                      READY   STATUS             RESTARTS   AGE
# minio-7f58448457-vctrp    1/1     Running            0          2m

kubectl get pvc
# NAME     STATUS   VOLUME             CAPACITY   ACCESS MODES   STORAGECLASS   AGE
# minio    Bound    pvc-cff81820-xxx   10Gi       RWO            nfs            2m40s

kubectl get pv
# NAME               CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM           STORAGECLASS   REASON   AGE   
# pvc-cff81820-xxx   10Gi       RWO            Delete           Bound    default/minio   nfs                     3m


kubectl port-forward svc/minio-console 9090:9090
# Forwarding from 127.0.0.1:9090 -> 9090
# Forwarding from [::1]:9090 -> 9090

```

### Clean up

```bash
helm delete minio
```