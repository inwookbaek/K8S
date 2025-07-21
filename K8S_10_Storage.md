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

### 10.1.1 hostPath PV

```yaml
# hostpath-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-volume
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /tmp
```

```bash
kubectl apply -f hostpath-pv.yaml
# persistentvolume/my-volume created

kubectl get pv
# NAME        CAPACITY  ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
# my-volume   1Gi       RWO            Retain           Available           manual                  12s
```

### 10.1.2 NFS PV

```yaml
# my-nfs.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-nfs
spec:
  storageClassName: nfs
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteMany
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: <NFS_SERVER_IP>
```

### 10.1.3 awsElasticBlockStore PV

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
apiVersion: v1
kind: PersistentVolume
metadata:
  name: aws-ebs
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
```

### 10.1.4 그외 다른 `PersistentVolume`

더 다양한 종류들을 확인하고 싶으시다면 다음 페이지를 참고 바랍니다. [https://kubernetes.io/docs/concepts/storage/volumes](https://kubernetes.io/docs/concepts/storage/volumes)
