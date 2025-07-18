# 7. 쿠버네티스 컨트롤러

## 7.1 🎯 컨트롤러란?
>컨트롤러 = 자동화된 상태 관리자
>* Kubernetes의 선언적 모델에 따라, 사용자가 정의한 원하는 상태와 실제 클러스터의 현재 상태를 비교해서 필요한 조치를 자동으로 수행합니다.
>* 참고사이트 : https://kubernetes.io/docs/concepts/architecture/controller
>* 컨트롤러는  control-loop하는 작업을 지속절으로 돌면서 특정 리소스에대해 모니터링을 한다.

>📌 예시:
>* 사용자가 replicas: 3으로 설정한 Deployment가 있는데, 실제 Pod가 2개라면?
>  - ReplicaSet 컨트롤러가 새 Pod을 1개 추가합니다.

### 🧠 핵심 개념
| 개념         | 설명                               |
| ---------- | -------------------------------- |
| **원하는 상태** | 사용자가 만든 리소스(YAML 등)에 정의된 이상적인 상태 |
| **현재 상태**  | 실제 클러스터에서 실행 중인 리소스 상태           |
| **컨트롤러**   | 두 상태를 비교하고 일치하도록 조정하는 자동화 로직     |

### 🔧 대표적인 컨트롤러 종류
#### 1. ReplicaSet 컨트롤러
* 역할: 지정된 수의 Pod가 항상 실행되도록 보장
* Deployment 뒤에 숨겨져 있음

#### 2. Deployment 컨트롤러
* 역할: ReplicaSet을 관리하며 롤링 업데이트, 롤백 등 지원
* 새 버전의 애플리케이션 배포 시 중요

#### 3. Job 컨트롤러
* 역할: 한 번만 실행되어 완료되는 작업 관리
  - 예: 데이터 마이그레이션, 배치 작업

#### 4. CronJob 컨트롤러
* 역할: Job을 정해진 시간(schedule) 에 실행
  - 예: 매일 백업, 주기적 리포트

#### 5. DaemonSet 컨트롤러
* 역할: 모든 노드에 1개씩 Pod 실행
  - 예: 로그 수집기, 모니터링 에이전트

#### 6. StatefulSet 컨트롤러
* 역할: 고유 ID, 안정적 저장소, 순차적 배포 보장
  - 예: 데이터베이스(MySQL, Cassandra)

#### 7. ReplicaController (구버전)
* Deployment 등장 전 사용되던 기본 컨트롤러. 지금은 거의 사용 안 함.

### 📊 동작 방식 요약 (컨트롤 루프)
* 클러스터 상태 감시 (Watch)
* 사용자 선언 상태와 비교
* 차이 발견 시, API 서버에 수정 요청 (patch)
* 지속적으로 반복 (Loop)

### 🔁 예: Deployment 컨트롤러 동작 흐름
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 3
  ...
```
➡ 실제 Pod가 2개만 실행 중이면
➡ 컨트롤러가 자동으로 1개 더 생성하여 총 3개 유지.

### 📌 요약
| 컨트롤러            | 역할                 |
| --------------- | ------------------ |
| **ReplicaSet**  | 원하는 개수의 Pod 유지     |
| **Deployment**  | 애플리케이션 업데이트/롤백 자동화 |
| **Job**         | 단일 실행 작업           |
| **CronJob**     | 정해진 시간에 Job 반복 실행  |
| **DaemonSet**   | 노드마다 Pod 실행        |
| **StatefulSet** | 상태 있는 애플리케이션 관리    |

## 7.2 🔁 ReplicaSet이란?
>* ReplicaSet(레플리카셋) 은 지정한 수(Replicas)의 동일한 Pod 복제본을 항상 유지하도록 보장하는 Kubernetes 리소스입니다.
>* 즉, 시스템 장애나 사용자 삭제 등으로 일부 Pod가 사라지더라도, ReplicaSet은 자동으로 새로운 Pod을 생성해서 원래 개수를 맞춰줍니다.

### 🧠 주요 역할
| 기능                   | 설명                                    |
| -------------------- | ------------------------------------- |
| ✅ Pod 개수 유지          | 사용자가 지정한 `replicas` 수를 항상 유지          |
| 🔄 자가 복구             | Pod가 죽거나 삭제되면 새로 자동 생성                |
| 🔍 Label selector 사용 | 특정 label을 가진 Pod만 관리                  |
| 📦 Deployment에서 사용   | Deployment가 내부적으로 ReplicaSet을 생성하고 관리 |

### 📄 예제: ReplicaSet YAML
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
spec:
  replicas: 3               # 유지하고 싶은 Pod 개수
  selector:
    matchLabels:
      app: nginx            # 이 label을 가진 Pod만 관리
  template:                 # 생성할 Pod의 템플릿
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
```

### 🧩 구성 요소 설명
| 항목         | 설명                         |
| ---------- | -------------------------- |
| `replicas` | 유지할 Pod 수                  |
| `selector` | 어떤 Pod를 관리할지 결정 (label 기준) |
| `template` | 생성할 Pod의 템플릿 (PodSpec과 동일) |

### 🛠 작동 방식
* 사용자: replicas: 3 설정
* ReplicaSet: 해당 label(app: nginx)을 가진 Pod가 3개인지 확인
* 부족하면 생성, 많으면 삭제
* Pod가 죽거나 삭제되면 자동 재생성

### 🧾 ReplicaSet vs Deployment
| 항목      | ReplicaSet                        | Deployment             |
| ------- | --------------------------------- | ---------------------- |
| 목적      | 특정 수의 Pod 유지                      | **버전 관리 및 롤링 업데이트** 지원 |
| 직접 사용   | 가능 (잘 안 씀)                        | 예, 일반적으로 사용            |
| 업데이트 지원 | ❌ 수동으로 해야 함                       | ✅ 자동 롤링 업데이트, 롤백 가능    |
| 관계      | Deployment가 내부적으로 ReplicaSet을 사용함 |                        |


* ➡️ 실제로는 ReplicaSet을 직접 쓰기보다는 Deployment를 사용하는 게 일반적입니다.

### 🔍 상태 확인 명령어
```bash
kubectl get rs
kubectl describe rs nginx-replicaset
kubectl get pods --show-labels
```
#####❗ 주의사항
* selector와 template.metadata.labels가 반드시 일치해야 Pod 관리 가능
* replicas 수만 맞추지, Pod의 스펙 변경은 감지하지 않음
  - 업데이트하려면 Deployment를 사용해야 함

### ✅ 요약
| 특징      | 내용                        |
| ------- | ------------------------- |
| 목적      | Pod 개수 자동 유지              |
| 관리 방식   | Label selector로 특정 Pod 관리 |
| 실전 사용   | Deployment가 내부적으로 사용      |
| 자동 복구   | Pod 장애 시 자동 재생성           |
| 수동 업데이트 | 스펙 변경 시 직접 수정 필요          |

### 실습

```bash
vi K8S/CH05/myreplicaset.yaml
```
```yaml
# myreplicaset.yaml
apiVersion: apps/v1               # ReplicaSet 리소스의 API 버전
kind: ReplicaSet                 # 생성할 리소스 종류: ReplicaSet
metadata:
  name: myreplicaset             # ReplicaSet의 이름 지정

spec:
  replicas: 2                    # 유지할 파드(Pod) 개수: 2개
  selector:                      # 어떤 파드를 이 ReplicaSet이 관리할지 결정하는 조건
    matchLabels:
      run: nginx-rs              # run=nginx-rs 라벨을 가진 파드만 관리

  template:                      # 파드를 생성할 때 사용할 템플릿 (Pod 정의)
    metadata:
      labels:
        run: nginx-rs            # 생성되는 파드에 적용될 라벨 (selector와 일치해야 함)
    spec:
      containers:
      - name: nginx              # 컨테이너 이름
        image: nginx             # 사용할 Docker 이미지 (기본 nginx 이미지)
```
#### 🔍 이 YAML의 의미
* myreplicaset이라는 이름의 ReplicaSet을 생성
* 항상 2개의 nginx Pod를 유지
* Pod에는 run=nginx-rs 라벨이 붙음
* 해당 라벨을 기준으로 이 ReplicaSet이 파드를 관리

#### ✅ 적용 명령어
```bash
kubectl apply -f K8S/CH05/myreplicaset.yaml

# 📋 상태 확인
kubectl get rs
kubectl get pods
kubectl describe rs myreplicaset

# 💡 팁: Pod를 하나 삭제해보세요
kubectl delete pod <pod-name>
# ReplicaSet이 자동으로 다시 새 Pod 1개를 생성하여 2개로 복원됩니다.

kubectl get replicaset  # 축약 시, rs
# NAME            DESIRED   CURRENT   READY   AGE
# myreplicaset    2         2         2       1m

kubectl get pod
# NAME                READY   STATUS      RESTARTS   AGE
# myreplicaset-jc496  1/1     Running     0          6s
# myreplicaset-xr216  1/1     Running     0          6s
```

```bash
# 복제본 개수 확장
# kubectl scale rs --replicas <NUMBER> <NAME>
# replica scaling
kubectl scale rs --replicas 4 myreplicaset
# replicaset.apps/rs scaled

kubectl get rs
# NAME            DESIRED   CURRENT   READY   AGE
# myreplicaset    4         4         4       1m

kubectl get pod
# NAME                READY   STATUS      RESTARTS   AGE
# myreplicaset-jc496  1/1     Running     0          2m
# myreplicaset-xr216  1/1     Running     0          2m
# myreplicaset-dc20x  1/1     Running     0          9s
# myreplicaset-3pq2t  1/1     Running     0          9s

kubectl delete pod myreplicaset-jc496
# pod "myreplicaset-jc496" deleted

kubectl get pod
# NAME                READY   STATUS      RESTARTS   AGE
# myreplicaset-xr216  1/1     Running     0          3m
# myreplicaset-dc20x  1/1     Running     0          1m
# myreplicaset-3pq2t  1/1     Running     0          1m
# myreplicaset-0y18b  1/1     Running     0          11s
```

```bash
# ReplicaSet 정리
kubectl delete rs --all
```

## 7.3 📦 Deployment란?
>Deployment는 ReplicaSet을 관리하고, Pod의 배포 및 롤링 업데이트(무중단 배포)를 수행하는 Kubernetes 리소스입니다.

### ✅ 요약 정의:
>사용자가 정의한 상태에 따라 Pod를 자동 생성, 업데이트, 롤백하며 안정적으로 배포해주는 고급 컨트롤러입니다.

### 🔁 Deployment의 핵심 기능
| 기능                      | 설명                          |
| ----------------------- | --------------------------- |
| 🔄 **롤링 업데이트**          | 무중단으로 새 버전 배포               |
| ↩️ **롤백**               | 이전 상태로 쉽게 되돌리기 가능           |
| 🔧 **ReplicaSet 자동 관리** | 직접 관리하지 않아도 됨               |
| 🛠 **스케일링**             | `replicas` 수 변경으로 Pod 개수 조절 |
| ✅ **자가 복구**             | Pod가 죽으면 자동으로 다시 생성         |


### 📄 Deployment 예제 YAML
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
```

### ✅ 설명
* replicas: 3 → nginx 파드 3개 유지
* selector 와 template.metadata.labels는 일치해야 함
* Deployment가 내부적으로 ReplicaSet을 생성하고 관리함

### 🔍 Deployment vs ReplicaSet 비교
| 항목         | Deployment               | ReplicaSet             |
| ---------- | ------------------------ | ---------------------- |
| 자동 롤링 업데이트 | ✅ 있음                     | ❌ 없음                   |
| 롤백 기능      | ✅ 있음                     | ❌ 없음                   |
| 실무 사용 빈도   | 매우 높음                    | 낮음 (보통 Deployment가 대체) |
| 직접 관리 여부   | ReplicaSet을 자동으로 생성하고 관리 | 수동으로 Pod 정의 필요         |

### ⚙️ 실전에서 자주 사용하는 명령어
```bash
# 배포
kubectl apply -f deployment.yaml

# 배포 상태 확인
kubectl get deployments
kubectl describe deployment nginx-deploy

# 스케일링
kubectl scale deployment nginx-deploy --replicas=5

# 롤아웃 상태 보기
kubectl rollout status deployment nginx-deploy

# 롤백
kubectl rollout undo deployment nginx-deploy
```

### 📚 요약
| 항목     | 설명                                   |
| ------ | ------------------------------------ |
| 역할     | 애플리케이션을 선언적으로 배포하고 업데이트함             |
| 내부 구성  | ReplicaSet → Pod                     |
| 실무 용도  | **CI/CD**, 무중단 배포, 자동 롤백 등           |
| 대체 가능성 | ReplicaSet 또는 Pod 단독 사용보다 훨씬 고수준 리소스 |

### ⚙️ 실습

```bash
vi K8S/CH05/mydeploy.yaml
```
```yaml
# mydeploy.yaml
apiVersion: apps/v1                   # Deployment 리소스가 속한 API 그룹 및 버전
kind: Deployment                      # 리소스 종류: Deployment
metadata:
  name: mydeploy                      # 이 Deployment의 이름

spec:
  replicas: 10                        # 유지하고 싶은 파드 수 (10개)

  selector:                           # 어떤 Pod를 이 Deployment가 관리할지 정의
    matchLabels:
      run: nginx                      # 'run=nginx' 라벨을 가진 Pod만 관리 대상으로 삼음

  strategy:                           # 배포 전략 설정
    type: RollingUpdate               # 롤링 업데이트 방식 사용 (기본값)
    rollingUpdate:
      maxUnavailable: 25%            # 업데이트 중 동시에 사용할 수 없는 Pod 최대 비율
      maxSurge: 25%                  # 업데이트 중 동시에 생성할 수 있는 Pod 최대 비율

  template:                           # 관리 대상 Pod의 템플릿 정의
    metadata:
      labels:
        run: nginx                    # selector와 일치해야 함

    spec:
      containers:
      - name: nginx                   # 컨테이너 이름
        image: nginx                  # 사용할 nginx 이미지

```

#### 🔁 롤링 업데이트(rollingUpdate)란?
* Deployment는 새 버전의 이미지를 배포할 때, 전체를 한번에 교체하지 않고 점진적으로 교체합니다.

#### 🔧 설정된 전략 의미
| 항목                    | 값                                  | 의미 |
| --------------------- | ---------------------------------- | -- |
| `maxUnavailable: 25%` | 전체 파드 중 최대 25%는 업데이트 도중 사용 불가능해도 됨 |    |
| `maxSurge: 25%`       | 업데이트 도중 최대 25%의 파드를 추가로 생성 가능      |    |

#### 📊 예시: 10개의 파드를 업데이트하는 경우
| 설정 값                  | 계산 결과 (10개 기준)                     |
| --------------------- | ---------------------------------- |
| `maxUnavailable: 25%` | 최대 2개까지 삭제 가능                      |
| `maxSurge: 25%`       | 최대 2개까지 더 생성 가능 (즉 12개까지 잠시 존재 가능) |

#### 🪜 동작 순서 (예시 흐름)
* 기존 10개 중 2개를 종료
* 새 버전의 파드 2개를 생성 → 10개 유지
* 다음 2개 종료 → 2개 생성 반복
* 최종적으로 10개가 모두 새 이미지로 전환됨

#### ✅ 상태 확인
```bash
kubectl apply -f mydeploy.yaml      # 배포 실행
kubectl rollout status deployment mydeploy
kubectl get pods -l run=nginx       # 배포된 파드 목록 확인
```
* 🔄 이미지 변경 예시 (롤링 업데이트 유도)

```bash
kubectl set image deployment mydeploy nginx=nginx:1.25
```
* ➡️ 위 명령은 새 nginx 이미지를 설정하며 롤링 업데이트가 자동 시작됩니다.

#### 🔁 롤백
```bash
kubectl rollout undo deployment mydeploy
```
* ➡️ 이전 버전으로 되돌립니다.

#### 📌 요약
| 항목         | 내용                         |
| ---------- | -------------------------- |
| 이름         | `mydeploy`                 |
| Pod 수      | 10개                        |
| 이미지 버전     | nginx:1.7.9                |
| 배포 전략      | `RollingUpdate` (점진적 업데이트) |
| 최대 비가용 수   | 25% (2개)                   |
| 최대 추가 생성 수 | 25% (2개)                   |

```bash
kubectl apply --record -f K8S/CH05/mydeploy.yaml
# deployment.apps/mydeploy created

kubectl get deployment  # 축약 시, deploy
# NAME       READY   UP-TO-DATE   AVAILABLE   AGE
# mydeploy   10/10   10           10          10m

kubectl get rs
# NAME              DESIRED   CURRENT   READY   AGE
# mydeploy-649xxx   10        10        10       1m

kubectl get pod
# NAME                   READY   STATUS       RESTARTS   AGE
# mydeploy-649xxx-bbxx   1/1     Running      0          9s
# mydeploy-649xxx-dtxx   1/1     Running      0          2m9s
# ...
```

```bash
# 이미지 주소 변경
kubectl set image deployment <NAME> <CONTAINER_NAME>=<IMAGE>

# 기존 nginx 버전 1.7.9에서 1.9.1로 업데이트
# kubectl set image deployment mydeploy nginx=nginx:1.9.1 --record 
# Flag --record has been deprecated,
kubectl set image deployment mydeploy nginx=nginx:1.21-alpine
# deployment.apps/mydeploy image updated

# 업데이트 진행 상황 확인합니다.
kubectl get pod
# NAME                   READY   STATUS             RESTARTS   AGE
# mydeploy-649xxx-bbxx   1/1     ContainerCreating  0          9s
# mydeploy-649xxx-dtxx   1/1     Running            0          2m9s
# ...

# 배포 상태확인, Deployment가 성공적으로 업데이트되고 있는지 확인
# 시간이 조금 소요된다.
kubectl rollout status deployment mydeploy
# Waiting for deployment "mydeploy" rollout to finish: 
# 7 out of 10 new replicas have been updated...
# Waiting for deployment "mydeploy" rollout to finish: 
# 7 out of 10 new replicas have been updated...
# Waiting for deployment "mydeploy" rollout to finish: 
# 7 out of 10 new replicas have been updated...
# Waiting for deployment "mydeploy" rollout to finish: 
# 8 out of 10 new replicas have been updated...
# ...
# deployment "mydeploy" successfully rolled out

# 특정 Pod의 이미지 tag 정보를 확인합니다.
kubectl get pod mydeploy-6674d7b969-9trgl  -o yaml | grep "image: nginx"
#   - image: nginx:1.21-alpine
```

```bash
# 1.9.1 버전에서 (존재하지 않는) 1.9.21 버전으로 업데이트 (에러 발생)
kubectl set image deployment mydeploy nginx=nginx:1.9.21 --record
# deployment.apps/mydeploy image updated

# Pod의 상태확인
kubectl get pod
# NAME                  READY   STATUS            RESTARTS  AGE
# mydeploy-6498-bbk9v   1/1     Running           0         9m38s
# mydeploy-6498-dt5d7   1/1     Running           0         9m28s
# mydeploy-6498-wrpgt   1/1     Running           0         9m38s
# mydeploy-6498-sbkzz   1/1     Running           0         9m27s
# mydeploy-6498-hclwx   1/1     Running           0         9m26s
# mydeploy-6498-98hd5   1/1     Running           0         9m25s
# mydeploy-6498-5gjrg   1/1     Running           0         9m24s
# mydeploy-6498-4lz4p   1/1     Running           0         9m38s
# mydeploy-6fbf-7kzpf   0/1     ErrImagePull      0         48s
# mydeploy-6fbf-rfgbd   0/1     ErrImagePull      0         48s
# mydeploy-6fbf-v5ms5   0/1     ErrImagePull      0         48s
# mydeploy-6fbf-rccw4   0/1     ErrImagePull      0         48s
# mydeploy-6fbf-ncqd2   0/1     ImagePullBackOff  0         48s
```


```bash
# 지금까지의 배포 히스토리를 확인합니다.
kubectl rollout history deployment mydeploy
# deployment.apps/mydeploy
# REVISION  CHANGE-CAUSE
# 1         kubectl apply --record=true --filename=mydeploy.yaml
# 2         kubectl set image deployment mydeploy nginx=nginx:1.9.1 
#                   --record=true
# 3         kubectl set image deployment mydeploy nginx=nginx:1.9.21 
#                   --record=true

# 잘못 설정된 1.9.21에서 --> 1.9.1로 롤백
kubectl rollout undo deployment mydeploy
# deployment.apps/mydeploy rolled back

kubectl rollout history deployment mydeploy
# deployment.apps/mydeploy
# REVISION  CHANGE-CAUSE
# 1         kubectl apply --record=true --filename=mydeploy.yaml
# 3         kubectl set image deployment mydeploy nginx=nginx:1.9.21 
#                    --record=true
# 4         kubectl set image deployment mydeploy nginx=nginx:1.9.1 
#                    --record=true

kubectl get deployment mydeploy -oyaml | grep image
# image: nginx:1.9.1

# 1.9.1 --> 1.7.9 (revision 1)로 롤백 (처음으로 롤백)
kubectl rollout undo deployment mydeploy --to-revision=1
# deployment.apps/mydeploy rolled back

# 복제본 개수 조절
# kubectl scale deployment --replicas <NUMBER> <NAME>
kubectl scale deployment mydeploy --replicas 5
# deployment.apps/mydeploy scaled

# 10개에서 5개로 줄어가는 것을 확인할 수 있습니다.
kubectl get pod
# NAME                  READY   STATUS           RESTARTS  AGE
# mydeploy-6498-bbk9v   1/1     Running          0         9m38s
# mydeploy-6498-dt5d7   1/1     Running          0         9m28s
# mydeploy-6498-wrpgt   1/1     Running          0         9m38s
# mydeploy-6498-sbkzz   1/1     Running          0         9m27s
# mydeploy-6498-98hd5   1/1     Running          0         9m27s
# mydeploy-6498-3srxd   0/1     Terminating      0         9m25s
# mydeploy-6498-5gjrg   0/1     Terminating      0         9m24s
# mydeploy-6498-4lz4p   0/1     Terminating      0         9m38s
# mydeploy-6fbf-7kzpf   0/1     Terminating      0         9m38s
# mydeploy-6fbf-d245c   0/1     Terminating      0         9m38s

# 다시 Pod의 개수를 10개로 되돌립니다.
kubectl scale deployment mydeploy --replicas=10
# deployment.apps/mydeploy scaled

# 5개가 새롭게 추가되어 다시 10개가 됩니다.
kubectl get pod
# NAME                  READY   STATUS              RESTARTS  AGE
# mydeploy-6498-bbk9v   1/1     Running             0         9m38s
# mydeploy-6498-dt5d7   1/1     Running             0         9m28s
# mydeploy-6498-wrpgt   1/1     Running             0         9m38s
# mydeploy-6498-sbkzz   1/1     Running             0         9m27s
# mydeploy-6498-98hd5   1/1     Running             0         9m25s
# mydeploy-6498-30cs2   0/1     ContainerCreating   0         5s
# mydeploy-6fbf-sdjc8   0/1     ContainerCreating   0         5s
# mydeploy-6498-w8fkx   0/1     ContainerCreating   0         5s
# mydeploy-6498-qw89f   0/1     ContainerCreating   0         5s
# mydeploy-6fbf-19glc   0/1     ContainerCreating   0         5s

# 직접 수정 " replicas: 10  --> 3으로 수정
kubectl edit deploy mydeploy
# apiVersion: apps/v1
# kind: Deployment
# metadata:
# ...
# spec:
#   progressDeadlineSeconds: 600
#   replicas: 10                  # --> 3으로 수정
#   revisionHistoryLimit: 10
#   selector:
#     matchLabels:
#       run: nginx
# 
# <ESC> + :wq
```

```bash
kubectl get pod
# NAME                  READY   STATUS     RESTARTS  AGE
# mydeploy-6498-bbk9v   1/1     Running    0         12m8s
# mydeploy-6498-dt5d7   1/1     Running    0         12m8s
# mydeploy-6498-wrpgt   1/1     Running    0         12m8s
```

```bash
# deployment 쿠버네티스 컨셉](07-02.png)
kubectl delete deploy --all
```

## 7.4 🏷️ StatefulSet이란?
>* Stateful 한 pod를 생성하는 경우에 사용
>* Deloyment, ReplicaSet과는 다르게 복제된 Pod 가 완벽히 동일하지 않고 순서에 따라 고유의 역할을 가진다.
>* 상태정보를 저장하는 리소스
>* Deloyment와 유사하게 Pod의 배포와 replica개수를 관리하지만, 다른 점은 Deployment 에서는 모든 Pod이 완벽히 동일, Pod끼리 순서가 없지만
>* StatefulSet은 각 Pod에 고유한 이름, 네트워크 ID, 저장소(Persistent Volume) 를 부여하여
>* 상태(순서와 고유성) 를 유지할 수 있게 해주는 리소스입니다.

### ✅ 주요 특징
| 기능                   | 설명                                               |
| -------------------- | ------------------------------------------------ |
| 📛 고유한 이름 유지         | 각 Pod에 **순번이 붙은 고정 이름** 부여 (예: `web-0`, `web-1`) |
| 🧠 고정 네트워크 ID        | 각각의 Pod는 고유한 DNS 주소를 가짐 (예: `web-0.service`)     |
| 💾 고정 스토리지 유지        | Pod가 재시작되거나 재배포되어도 **동일한 볼륨을 재사용**               |
| 🔁 순차적 생성·삭제         | Pod는 **순서대로 생성/삭제**됨 (0번부터 차례로)                  |
| 🛠 데이터베이스, 메시지큐 등 사용 | Kafka, Redis, MySQL 등 **상태 있는 서비스**에 적합          |

### 🏗️ 구조 요약
* Pod 이름: nginx-0, nginx-1, nginx-2 …
* Pod DNS: nginx-0.nginx.default.svc.cluster.local
* PersistentVolumeClaim(PVC): Pod당 하나씩 고정 할당


### 📊 차이 요약표: ReplicaSet vs Deployment vs StatefulSet
>ReplicaSet, Deployment, StatefulSet은 모두 Kubernetes에서 Pod를 여러 개 생성하고 관리하는 컨트롤러이지만, 용도와 특성은 꽤 다릅니다.

| 항목              | **ReplicaSet** 🛠 | **Deployment** 🚀    | **StatefulSet** 🧱        |
| --------------- | ----------------- | -------------------- | ------------------------- |
| 📌 목적           | 파드 복제만 관리         | 파드 배포 + 업데이트 + 복제    | 상태 유지가 필요한 파드 관리          |
| 🎯 사용 사례        | 간단한 테스트용          | 일반적인 앱 배포 (웹, API 등) | DB, Kafka, Redis 등        |
| 📦 Pod 이름       | 무작위 (랜덤)          | 무작위 (랜덤)             | 고정 (ex: `app-0`, `app-1`) |
| 📡 네트워크 정체성     | 없음                | 없음                   | 고정 DNS 할당됨                |
| 💾 스토리지         | 공유 or 임시          | 공유 or 임시             | Pod마다 고정 PVC 연결           |
| 🔁 업데이트 전략      | 수동                | 롤링 업데이트 자동           | 순차 업데이트 (기본)              |
| ↩️ 롤백 기능        | ❌ 없음              | ✅ 있음                 | ❌ 없음 (직접 관리)              |
| 🔄 Pod 생성/삭제 순서 | 순서 없음             | 순서 없음                | 순차적으로 처리                  |
| 🧠 실무 사용 추천도    | 낮음                | 매우 높음                | 특정 경우에만 사용                |

#### 📘 각 리소스 설명
##### 🛠 ReplicaSet
* Pod 복제 수만 유지
* 데이트/롤백 같은 기능 없음
* 실무에서는 Deployment가 대신 사용됨

```bash
kubectl scale rs myrs --replicas=3
```

##### 🚀 Deployment
* ReplicaSet을 자동으로 생성/관리
* 롤링 업데이트, 롤백, 버전 관리 가능
* 일반적인 애플리케이션 배포에 가장 많이 사용됨

```bash
kubectl rollout status deployment myapp
kubectl rollout undo deployment myapp
```

#####🧱 StatefulSet
* 각 Pod의 고유 정체성, 네트워크, 스토리지 보장
* 데이터베이스, Zookeeper, Kafka 등 상태가 중요한 앱에 사용
* Headless Service 필요 (clusterIP: None)
* PVC는 Pod 삭제 후에도 유지됨

```bash
kubectl get pvc
kubectl delete pod app-0  # → app-0만 다시 생성됨, 이름 유지
```

##### 🧠 그림으로 이해
```scss
Deployment → ReplicaSet → Pod(s)

StatefulSet ─┬─ Pod-0 (고정 DNS, 고정 스토리지)
             ├─ Pod-1
             └─ Pod-2
```

##### ✅ 선택 기준 요약
* 필요한 기능	선택
* Stateless한 웹 서비스	✅ Deployment
* Pod 이름, DNS, 저장소 고정	✅ StatefulSet
* 수동 테스트용 컨트롤러	🆗 ReplicaSet

### 📄 예제: StatefulSet YAML
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "web"            # Headless 서비스 필요
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

### 🧩 구성 요소 설명
| 항목                            | 설명                              |
| ----------------------------- | ------------------------------- |
| `serviceName`                 | Headless Service 이름. DNS를 위해 필요 |
| `replicas`                    | 파드 개수 (web-0, web-1, web-2)     |
| `volumeClaimTemplates`        | Pod당 하나씩 PVC가 자동 생성됨            |
| `selector`와 `template.labels` | 반드시 일치해야 함                      |

### 🧪 실행 결과
* od 이름: web-0, web-1, web-2
* DNS: 각 Pod는 자신의 고정 주소를 가짐
* web-0.web.default.svc.cluster.local
* PVC: www-web-0, www-web-1, www-web-2 생성됨

### ✅ StatefulSet vs Deployment 차이
| 항목       | Deployment    | StatefulSet            |
| -------- | ------------- | ---------------------- |
| Pod 이름   | 무작위 (랜덤)      | 고정 (`이름-0`, `이름-1`…)   |
| 네트워크     | 변경됨           | 고정 DNS                 |
| 스토리지     | 공유 가능, 임시     | Pod당 고정 PVC            |
| 생성/삭제 순서 | 병렬, 순서 없음     | 순차적 (0 → 1 → 2…)       |
| 사용 사례    | 웹 서버, 백엔드 API | 데이터베이스, Kafka, Redis 등 |

### ⚠️ StatefulSet 사용할 때 주의사항
* Headless Service 반드시 필요 (clusterIP: None)
* Pod 삭제 시에도 PVC는 유지됨
* 순차 생성/삭제로 인해 속도는 느림
* Pod 이름은 절대 바뀌지 않음 (중요한 특성!)

### 📚 요약
| 항목    | 내용                                   |
| ----- | ------------------------------------ |
| 목적    | 상태가 있는 앱을 안정적으로 배포                   |
| 특징    | 고정 이름, DNS, 스토리지, 순차 생성              |
| 사용 예  | DB, Redis, Kafka, Zookeeper          |
| 구성 요소 | StatefulSet + Headless Service + PVC |


### ✅ 실습

```bash
vi K8S/CH05/mysts.yaml
```
```yaml
# mysts.yaml
# Kubernetes에서 StatefulSet을 정의한 예제로, 각 Pod에 고유한 스토리지, 고정 DNS, 순차 생성을 적용
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysts                       # StatefulSet 이름
spec:
  serviceName: mysts               # Headless 서비스 이름 (필수)
  replicas: 3                      # 3개의 고유한 Pod 생성 (mysts-0, mysts-1, mysts-2)
  selector:
    matchLabels:
      run: nginx                   # Pod 선택 조건 (라벨이 run=nginx인 Pod만 관리)

  template:
    metadata:
      labels:
        run: nginx                 # Pod 템플릿의 라벨 (selector와 일치해야 함)
    spec:
      containers:
      - name: nginx
        image: nginx              # NGINX 컨테이너 실행
        volumeMounts:
        - name: vol               # 아래 PVC에 연결
          mountPath: /usr/share/nginx/html  # HTML 경로에 마운트

  volumeClaimTemplates:           # Pod마다 독립적인 PVC 생성
  - metadata:
      name: vol                   # 이 이름이 volumeMounts에서 참조됨
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi            # 각 Pod에 1Gi 크기의 고정 볼륨 요청
```

```bash
kubectl apply -f K8S/CH05/mysts.yaml
# statefulset.apps/mysts created
# service/mysts created

kubectl get statefulset   # 축약 시, sts
# NAME    READY   AGE
# mysts   2/3     20s

kubectl get pod
# NAME      READY   STATUS              RESTARTS   AGE
# mysts-0   1/1     Running             0          29s
# mysts-1   0/1     ContainerCreating   0          20s
# mysts-2   0/1     Pending             0          10s
```

```bash
kubectl exec mysts-0 -- hostname
# mysts-0

kubectl exec mysts-1 -- hostname
# mysts-1
```

```bash
kubectl exec mysts-0 -- ls /usr/share/nginx/html/index.html

# Pod의 호스트 이름을 웹 페이지로 저장하는 작업을 수행
kubectl exec mysts-0 -- sh -c \
  'echo "$(hostname)" > /usr/share/nginx/html/index.html'
  
kubectl exec mysts-1 -- sh -c \
  'echo "$(hostname)" > /usr/share/nginx/html/index.html'

kubectl exec mysts-0 -- cat /usr/share/nginx/html/index.html
kubectl exec mysts-0 -- cat /usr/share/nginx/html/index.html

# -s 옵션은 "silent mode" (조용한 모드) 를 의미
kubectl exec mysts-0 -- curl -s http://localhost
# mysts-0
kubectl exec mysts-1 -- curl -s http://localhost
# mysts-1
```
##### 💡 의미별 설명:
| 구성 요소                                | 설명                          |
| ------------------------------------ | --------------------------- |
| `kubectl exec mysts-0`               | Pod `mysts-0` 안에서 명령 실행     |
| `--`                                 | exec 명령어의 끝. 이후는 실제 실행할 명령  |
| `sh -c '...'`                        | 셸 명령 실행. 문자열 내부 명령을 처리      |
| `hostname`                           | 현재 Pod의 호스트 이름(=Pod 이름)을 출력 |
| `> /usr/share/nginx/html/index.html` | nginx 기본 웹 디렉터리에 저장         |


```bash
# 현재 클러스터에 존재하는 모든 PVC(PersistentVolumeClaim) 를 조회
kubectl get persistentvolumeclaim
# NAME          STATUS   VOLUME        CAP  MODE   STORAGECLASS   AGE
# vol-mysts-0   Bound    pvc-09d-xxx   1Gi  RWO    local-path     118s
# vol-mysts-1   Bound    pvc-421-xxx   1Gi  RWO    local-path     109s
# vol-mysts-2   Bound    pvc-x42-xxx   1Gi  RWO    local-path     60s
```
##### 🧠 항목 설명
| 항목             | 설명                                            |
| -------------- | --------------------------------------------- |
| `NAME`         | PVC 이름 (StatefulSet이 자동 생성함)                  |
| `STATUS`       | 현재 상태<br>➡️ `Bound`: PV와 정상 연결됨               |
| `VOLUME`       | 이 PVC가 바인딩된 실제 PV(Persistent Volume)의 ID      |
| `CAPACITY`     | 요청된 저장 용량 (여기선 1Gi)                           |
| `ACCESS MODES` | 접근 방식 (RWO = ReadWriteOnce, 단일 노드에서 읽기/쓰기 가능) |
| `STORAGECLASS` | 사용할 스토리지 클래스 (예: `local-path`)                |
| `AGE`          | PVC가 생성된 이후 경과 시간                             |


```bash
# --replicas=0	파드를 0개로 설정 → 모든 파드 삭제됨 (mysts-0, mysts-1, mysts-2 등)
# 생성될 때와 반대로 식별자의 역순으로 삭제되는 것 확인
kubectl scale sts mysts --replicas=0
# statefulset.apps/mysts scaled

kubectl get pod
# NAME      READY   STATUS       RESTARTS   AGE
# mysts-0   1/1     Running      0          29s
# mysts-1   0/1     Terminating  0          20s
```

### StatefulSet 예제
* `StatefulSet` 리소스의 좋은 사용 예시로 MySQL DB 클러스터를 구축하는 예제가 쿠버네티스 공식 사이트에 존재하지만 내용이 비교적 장황하여 본래 리소스의 특징을 파악하기 어려워 간단한 NGINX 예제로 대체하였습니다. 다음 페이지에서 직접 MySQL 클러스터 구축 예제를 확인해 보시기 바랍니다.
* 참고 : https://kubernetes.io/docs/tasks/run-application/run-replicated-stateful-application

```bash
kubectl delete sts mysts
kubectl delete svc mysts
kubectl delete pvc --all
```

## 7.4 StatefulSet

```yaml
# mysts.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysts
spec:
  serviceName: mysts
  replicas: 3
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        volumeMounts:
        - name: vol
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: vol
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
---
apiVersion: v1
kind: Service
metadata:
  name: mysts
spec:
  clusterIP: None
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 80
  selector:
    run: nginx
```

```bash
kubectl apply -f mysts.yaml
# statefulset.apps/mysts created
# service/mysts created

kubectl get statefulset   # 축약 시, sts
# NAME    READY   AGE
# mysts   2/3     20s

kubectl get pod
# NAME      READY   STATUS              RESTARTS   AGE
# mysts-0   1/1     Running             0          29s
# mysts-1   0/1     ContainerCreating   0          20s
# mysts-2   0/1     Pending             0          10s
```

```bash
kubectl exec mysts-0 -- hostname
# mysts-0

kubectl exec mysts-1 -- hostname
# mysts-1
```

```bash
kubectl exec mysts-0 -- sh -c \
  'echo "$(hostname)" > /usr/share/nginx/html/index.html'
kubectl exec mysts-1 -- sh -c \
  'echo "$(hostname)" > /usr/share/nginx/html/index.html'

kubectl exec mysts-0 -- curl -s http://localhost
# mysts-0
kubectl exec mysts-1 -- curl -s http://localhost
# mysts-1
```

```bash
kubectl get persistentvolumeclaim
# NAME          STATUS   VOLUME        CAP  MODE   STORAGECLASS   AGE
# vol-mysts-0   Bound    pvc-09d-xxx   1Gi  RWO    local-path     118s
# vol-mysts-1   Bound    pvc-421-xxx   1Gi  RWO    local-path     109s
# vol-mysts-2   Bound    pvc-x42-xxx   1Gi  RWO    local-path     60s
```

```bash
kubectl scale sts mysts --replicas=0
# statefulset.apps/mysts scaled

kubectl get pod
# NAME      READY   STATUS       RESTARTS   AGE
# mysts-0   1/1     Running      0          29s
# mysts-1   0/1     Terminating  0          20s
```

### StatefulSet 예제

`StatefulSet` 리소스의 좋은 사용 예시로 MySQL DB 클러스터를 구축하는 예제가 쿠버네티스 공식 사이트에 존재하지만 내용이 비교적 장황하여 본래 리소스의 특징을 파악하기 어려워 간단한 NGINX 예제로 대체하였습니다. 다음 페이지에서 직접 MySQL 클러스터 구축 예제를 확인해 보시기 바랍니다.

[https://kubernetes.io/docs/tasks/run-application/run-replicated-stateful-application](https://kubernetes.io/docs/tasks/run-application/run-replicated-stateful-application)


```bash
kubectl delete sts mysts
kubectl delete svc mysts
kubectl delete pvc --all
```


## 7.5 📘 DaemonSet이란?
>* "모든 노드에서 반드시 실행되어야 하는 파드를 자동으로 관리하는 컨트롤러"
>* 즉, 클러스터 내의 모든 노드에 1개씩 Pod를 배포하고,
>* 노드가 추가되면 자동으로 해당 노드에도 배포되며,
>* 노드가 제거되면 자동으로 해당 Pod도 제거됩니다.

### 🔧 DaemonSet은 언제 쓰나요?
| 사용 사례        | 설명                                      |
| ------------ | --------------------------------------- |
| 🪵 로그 수집기    | Fluentd, Filebeat, etc.                 |
| 📊 모니터링 에이전트 | Prometheus Node Exporter, Datadog Agent |
| 🔒 보안 데몬     | Falco, sysdig                           |
| 🌐 네트워크 관리   | CNI 플러그인 (Calico, Flannel 등)            |
| 🧪 테스트용 Pod  | 모든 노드에서 네트워크 테스트 수행 등                   |

### 🧬 예시 YAML
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-logger
spec:
  selector:
    matchLabels:
      name: node-logger
  template:
    metadata:
      labels:
        name: node-logger
    spec:
      containers:
      - name: logger
        image: busybox
        command: ["sh", "-c", "while true; do echo Logging from $(hostname); sleep 10; done"]
# 📌 이 예제는 모든 노드에서 로깅하는 busybox 컨테이너를 실행합니다.
```

### ✅ 특징 요약
| 항목             | 설명                      |
| -------------- | ----------------------- |
| 🎯 배포 위치       | 모든 노드 or 특정 노드(레이블 기준)  |
| ➕ 노드 추가 시      | 자동으로 Pod 생성             |
| ➖ 노드 삭제 시      | 자동으로 Pod 제거             |
| 🔄 수동 조정 필요 없음 | replicas 설정 없이 자동 관리    |
| 🔎 확인          | `kubectl get daemonset` |

### ✅ 실습

```bash
vi K8S/CH05/fluentd.yaml
```
```yaml
# fluentd.yaml
#  Fluentd를 Kubernetes의 DaemonSet으로 설정하여, 모든 노드에서 컨테이너 로그를 수집하도록 구성한 것
apiVersion: apps/v1                       # 사용하는 API 버전 (앱스 그룹의 v1 버전)
kind: DaemonSet                           # 이 리소스는 DaemonSet (모든 노드에 1개씩 파드 배포)
metadata:
  name: fluentd                           # DaemonSet 이름

spec:
  selector:
    matchLabels:
      name: fluentd                       # 이 라벨을 가진 Pod만 이 DaemonSet이 관리함

  template:                               # DaemonSet이 생성할 Pod 템플릿 정의
    metadata:
      labels:
        name: fluentd                     # Pod에 적용할 라벨 (selector와 일치해야 함)

    spec:
      containers:
      - name: fluentd                     # 컨테이너 이름
        image: quay.io/fluentd_elasticsearch/fluentd:v3.0.0  # 사용 이미지 (Elasticsearch output 포함된 Fluentd)
        
        volumeMounts:
        - name: varlibdockercontainers    # 볼륨 이름 (아래 volumes와 연결됨)
          mountPath: /var/lib/docker/containers  # 호스트의 컨테이너 로그 경로를 컨테이너 내부로 마운트
          readOnly: true                  # 로그만 읽으므로 읽기 전용 마운트 설정

      volumes:
      - name: varlibdockercontainers      # 위에서 사용한 볼륨 이름
        hostPath:
          path: /var/lib/docker/containers  # 호스트 시스템의 실제 디렉터리 경로 (Docker 컨테이너 로그 위치)
```

#### 📌 주요 기능 설명
| 항목               | 설명                                                   |
| ---------------- | ---------------------------------------------------- |
| `DaemonSet`      | 클러스터 모든 노드에 Fluentd 파드를 배포                           |
| `hostPath`       | 노드의 실제 경로 `/var/lib/docker/containers` 를 Pod 내부로 마운트 |
| `volumeMounts`   | 컨테이너에서 로그 파일을 읽을 수 있도록 설정                            |
| `readOnly: true` | 로그 수집만 하기 때문에 쓰기 불필요                                 |

#### 📦 Fluentd 컨테이너는 어떤 일을 하나요?
* 노드에 생성되는 Docker 컨테이너 로그 파일을 읽고
* 로그를 파싱하여
* Elasticsearch, Logstash, 혹은 다른 로그 백엔드로 전달합니다
* (기본적으로는 Elastic Stack에 연동되도록 설정된 이미지입니다)

#### 💡 실행 후 확인 방법
```bash
kubectl apply -f K8S/CH05/fluentd.yaml           # DaemonSet 배포
kubectl get pods -l name=fluentd -o wide  # 모든 노드에 배포됐는지 확인
kubectl logs <fluentd-pod-name>         # 로그 확인
```

#### ⚠️ 주의사항
| 주의 항목                        | 설명                                                                                                               |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `/var/lib/docker/containers` | **컨테이너 런타임이 Docker인 경우**에만 해당 경로 유효<br>→ containerd 환경이라면 다른 경로(`/var/log/pods`, `/var/log/containers`)를 마운트해야 함 |
| 로그 출력 대상                     | 이 Fluentd 이미지는 Elasticsearch 연동용이므로, 추가 설정 파일이 필요할 수 있습니다 (`fluent.conf`)                                        |


```bash
# pod가 서버마다 하나씩 생성
kubectl apply -f K8S/CH05/fluentd.yaml
# daemonset.apps/fluentd created

kubectl get daemonset   # 축약 시, ds
# NAME      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE ..
# fluentd   2         2         2       2            2                ..

kubectl get pod -owide
# NAME           READY  STATUS    RESTARTS  AGE   IP          NODE    ..
# fluentd-q9vcc  1/1    Running   0         92s   10.42.0.8   master  ..
# fluentd-f3gt3  1/1    Running   0         92s   10.42.0.10  worker  ..

kubectl logs fluentd-q9vcc
# 2020-07-05 04:12:05 +0000 [info]: parsing config file is succeeded ..
# 2020-07-05 04:12:05 +0000 [info]: using configuration file: <ROOT>
#   <match fluent.**>
#     @type null
#   </match>
# </ROOT>
# 2020-07-05 04:12:05 +0000 [info]: starting fluentd-1.4.2 pid=1 ..
# 2020-07-05 04:12:05 +0000 [info]: spawn command to main: cmdline=..
# ...
```

```bash
kubectl delete ds --all
```

## 7.6 Job & CronJob
### ✨ Job & CronJob 핵심 차이점
| 구분    | Job             | CronJob              |
| ----- | --------------- | -------------------- |
| 역할    | 단발성 작업 실행       | 반복 주기 작업 실행          |
| 실행 방식 | 한 번 실행, 완료되면 종료 | 스케줄에 따라 반복적으로 Job 생성 |
| 사용처   | 일회성 배치 작업       | 정기적 자동화 작업           |

### 🏃‍♀️ Kubernetes Job
#### 1. 개념
* Job은 Kubernetes에서 단발성 작업(일회성 배치 작업)을 실행하고 완료를 보장하기 위한 리소스입니다.
* 지정한 작업이 성공적으로 완료될 때까지 하나 이상의 Pod를 생성하고 관리합니다.
* 예를 들어, 데이터 처리, 백업, 마이그레이션, 일회성 스크립트 실행 등에 사용됩니다.

#### 2. 작동 방식
* Job은 Pod 템플릿을 사용하여 작업용 Pod를 생성합니다.
* 생성된 Pod가 성공(종료 코드 0)하면 Job은 완료된 것으로 간주합니다.
* Pod가 실패하면 backoffLimit만큼 재시도합니다.
* completions 필드를 통해 수행할 작업의 총 횟수를 지정할 수 있습니다.
* parallelism 필드를 통해 동시에 실행할 Pod 수를 지정할 수 있습니다.

#### 3. 주요 필드
| 필드             | 설명                   |
| -------------- | -------------------- |
| `completions`  | 작업 완료 횟수 (기본 1)      |
| `parallelism`  | 동시에 실행할 Pod 수 (기본 1) |
| `backoffLimit` | 실패 시 재시도 횟수 (기본 6)   |
| `template`     | 실행할 Pod의 템플릿         |

### ⏰ Kubernetes CronJob
#### 1. 개념
* CronJob은 Kubernetes에서 정해진 스케줄에 따라 주기적으로 Job을 실행하는 리소스입니다.
* 리눅스의 cron 스케줄러처럼 동작해서, 특정 시간에 작업을 자동으로 반복 수행합니다.
* 예를 들어, 매일 백업, 정기 로그 정리, 주기적 보고서 생성 등에 사용됩니다.

#### 2. 작동 방식
* CronJob은 스케줄에 맞춰 새로운 Job을 생성합니다.
* 생성된 Job은 단발성 작업처럼 실행되고 완료됩니다.
* 실패 시 Job의 backoffLimit만큼 재시도합니다.
* 스케줄은 cron 표현식을 사용해 지정합니다.

#### 3. 주요 필드
| 필드                           | 설명                                |
| ---------------------------- | --------------------------------- |
| `schedule`                   | cron 형식의 스케줄 (ex: `"0 0 * * *"`)  |
| `jobTemplate`                | 실행할 Job의 템플릿                      |
| `startingDeadlineSeconds`    | 스케줄 놓친 Job을 실행할 최대 시간(초)          |
| `concurrencyPolicy`          | 동시 실행 정책 (Allow, Forbid, Replace) |
| `successfulJobsHistoryLimit` | 성공한 Job 보관 개수                     |
| `failedJobsHistoryLimit`     | 실패한 Job 보관 개수                     |

### 7.6.1 Job

#### 🏃‍♂️ Job
* 한 번 실행해서 작업을 완료하는 Kubernetes 워크로드입니다.
* Pod를 생성해 작업을 수행하고, 작업 완료 시 Pod가 종료됩니다.
* 작업이 성공적으로 완료될 때까지 필요한 만큼 Pod를 재시작하거나 재생성합니다.
  - 예) 데이터베이스 백업, 일회성 스크립트 실행 등

```bash
vi K8S/CH05/train.py
```
```python
# train.py
import os, sys, json
from tensorflow import keras
from tensorflow.keras.datasets import mnist
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense, Dropout
from tensorflow.keras.optimizers import RMSprop

#####################
# parameters
#####################
epochs = int(sys.argv[1])              # 학습 epoch 수
activate = sys.argv[2]                 # 출력층 활성화 함수
dropout = float(sys.argv[3])           # dropout 비율
print(sys.argv)
#####################

batch_size, num_classes, hidden = (128, 10, 512)
loss_func = "categorical_crossentropy"
opt = RMSprop()

# 데이터 전처리
(x_train, y_train), (x_test, y_test) = mnist.load_data()
x_train = x_train.reshape(60000, 784).astype('float32') / 255
x_test = x_test.reshape(10000, 784).astype('float32') / 255

y_train = keras.utils.to_categorical(y_train, num_classes)
y_test = keras.utils.to_categorical(y_test, num_classes)

# 모델 구성
model = Sequential()
model.add(Dense(hidden, activation='relu', input_shape=(784,)))
model.add(Dropout(dropout))
model.add(Dense(num_classes, activation=activate))

model.summary()

# 컴파일 및 학습
model.compile(loss=loss_func, optimizer=opt, metrics=['accuracy'])

history = model.fit(
    x_train, y_train,
    batch_size=batch_size,
    epochs=epochs,
    validation_data=(x_test, y_test)
)

# 평가
score = model.evaluate(x_test, y_test, verbose=0)
print('Test loss:', score[0])
print('Test accuracy:', score[1])
```
```
vi K8S/CH05/Dockerfile
```
>* 시스템자원이 적을 경우 시간이 너무 많이 걸린다.
>* 간단한 도커이미지를 만들어서 테스트할 것 또는
>* 도커 이미지 빌드 : 가상머신에서 실행힐 경우 소요시간이 많이 걸림 - WSL에서 테스트해 볼 것
```Dockerfile
# Dockerfile
FROM python:3.9

# pip 업그레이드 + 패키지 설치
RUN pip install --upgrade pip && \
    pip install tensorflow==2.5 
    # pip install keras==2.4.3 # tf2.5에서 keras는 통합되어 설치필요 없음
    
COPY K8S/CH05/train.py .
ENTRYPOINT ["python", "train.py"]
```

```bash
whoami
sudo systemctl status docker
sudo groupadd docker
sudo usermod -aG docker $(whoami)
logout

export USERNAME=iwbaek # docker hub login id

# Docker는 예전 "legacy builder" 대신, 더 빠르고 강력한 BuildKit을 쓰라고 안내하고 있습니다.
# export DOCKER_BUILDKIT=1
# 해결방법
sudo mkdir -p /usr/lib/docker/cli-plugins
sudo curl -SL https://github.com/docker/buildx/releases/latest/download/docker-buildx-linux-amd64 \
  -o /usr/lib/docker/cli-plugins/docker-buildx
sudo chmod +x /usr/lib/docker/cli-plugins/docker-buildx

# 도커 이미지 빌드 : 가상머신에서 실행힐 경우 소요시간이 많이 걸림 - WSL에서 테스트해 볼 것
# Watchdog: BUG: soft lockup - CPU#10 stuck for 21s! [runc:4688]  오류메시지 발생 
# 오류원인: CPU 코어 #10에서 21초 동안 응답 없음(soft lockup)을 감지
# 해결방법: 
# 2. 커널 파라미터 조정
# /etc/sysctl.conf에 다음 설정 추가 또는 수정:
# kernel.watchdog_thresh=30  # watchdog 임계값 증가 (기본값 10초)
# kernel.softlockup_panic=0  # soft lockup 시 패닉 방지
# 변경 후 적용:
# sudo sysctl -p

DOCKER_BUILDKIT=1 sudo docker build -f K8S/CH05/Dockerfile -t $USERNAME/train .
# Dockerfile 에서는 변수($USERNAME) 사용할 수 없음
# DOCKER_BUILDKIT=0 docker build -f K8S/CH05/Dockerfile -t iwbaek/train .
# Sending build context to Docker daemon  3.249MB
# Step 1/4 : FROM python:3.6.8-stretch
# 3.6.8-stretch: Pulling from library/python
# 6f2f362378c5: Pull complete
# ...
ps -ef | grep -E "^$USER|UID"

docker ps -a
docker ps -a --format "table {{.ID}}\t{{.Image}}\t{{.Status}}\t{{.Names}}"
docker images
docker run --rm iwbaek/train 3 softmax 0.5  

# 도커 이미지 업로드를 위해 도커허브에 로그인합니다.
 docker login -u $USERNAME
# Login with your Docker ID to push and pull images from Docker Hub. ..
# Username: $USERNAME
# Password:
# WARNING! Your password will be stored unencrypted in /home/..
# Configure a credential helper to remove this warning. See
# https://docs.docker.com/engine/reference/commandline/..
# 
# Login Succeeded

# 도커 이미지 업로드
docker push $USERNAME/train
# The push refers to repository [docker.io/$USERNAME/train]
```

```
vi K8S/CH05/job.yaml
```
```yaml
# job.yaml
apiVersion: batch/v1       # 배치 작업(Job)을 위한 API 버전
kind: Job                  # 리소스 종류: Job
metadata:
  name: myjob              # Job 이름
spec:
  template:                # Job이 실행할 Pod 템플릿
    spec:
      containers:
      - name: ml           # 컨테이너 이름
        image: iwbaek/train  # 사용할 Docker 이미지 
        args: ['3', 'softmax', '0.5']  # train.py에 전달할 인자
      restartPolicy: Never       # 실패 시 재시작하지 않음
  backoffLimit: 2          # 실패 시 최대 2번까지 재시도
```
```bash
# Job 생성
kubectl apply -f K8S/CH05/job.yaml
# job.batch/myjob created

# Job 리소스 확인
kubectl get job
# NAME    COMPLETIONS   DURATION   AGE
# myjob   0/1           9s         9s

# Pod 리소스 확인
kubectl get pod
# NAME          READY   STATUS      RESTARTS   AGE
# myjob-l5thh   1/1     Running     0          9s

# docker run --rm iwbaek/train 3 softmax 0.5
# docker run iwbaek/train 3 softmax 0.5

# 로그 확인
kubectl logs -f myjob-l5thh 
# ...
# Layer (type)                 Output Shape              Param #
# =================================================================
# dense_1 (Dense)              (None, 512)               401920
# ...

# Pod 완료 확인
kubectl get pod
# NAME          READY   STATUS      RESTARTS   AGE
# myjob-l5thh   0/1     Completed   0          3m27s

# Job 완료 확인
kubectl get job
# NAME    COMPLETIONS   DURATION   AGE
# myjob   1/1           51s         4m
```

```bash
vi K8S/CH05/job-bug.yaml
```
```yaml
# job-bug.yaml
apiVersion: batch/v1          # Kubernetes Job API 버전
kind: Job                     # Job 리소스 생성
metadata:
  name: myjob-bug             # Job 이름
spec:
  template:                   # 실행될 Pod 정의
    spec:
      containers:
      - name: ml              # 컨테이너 이름
        image: iwbaek/train   # 도커 이미지 (사용자 정의)
        # 👇 아래 인자 전달 오류 주의,  Python 내부에서 int("bug-string")을 시도할 때 오류 발생
        args: ['bug-string', 'softmax', '0.5']
        # ⬆️ 'bug-string'이 숫자여야 하는 자리에 문자열이 들어감(고의적으로 에러발생)
      restartPolicy: Never    # 실패 시 재시작하지 않음
  backoffLimit: 2             # 최대 2번 재시도 후 중단

```

```bash
kubectl apply -f K8S/CH05/job-bug.yaml

# 2번 재시도 후(총 3번 실행) failed
kubectl get pod
# NAME               READY   STATUS              RESTARTS   AGE
# myjob-bug-8f867      0/1     Error               0          6s
# myjob-bug-s23xs      0/1     Error               0          4s
# myjob-bug-jz2ss      0/1     ContainerCreating   0          1s

kubectl get job myjob-bug -oyaml | grep type
# type: Failed

# 에러 원인 확인
kubectl logs -f myjob-bug-jz2ss
# /usr/local/lib/python3.6/site-packages/tensorflow/python/framework/
#   dtypes.py:502: FutureWarning: Passing (type, 1) or '1type' 
#   as a synonym of type is deprecated; in a future version of numpy, 
#   it will be understood as (type, (1,)) / '(1,)type'.
#   np_resource = np.dtype([("resource", np.ubyte, 1)])
# Traceback (most recent call last):
#   File "train.py", line 11, in <module>
#    epochs = int(sys.argv[1])
# ValueError: invalid literal for int() with base 10: 'bug-string'
```

```bash
kubectl delete job --all
```

### 7.6.2 CronJob
#### ⏰ CronJob
* Job을 주기적으로 실행하는 스케줄러입니다.
* 리눅스 cron처럼 정해진 시간마다 Job을 생성해 실행합니다.
* 정기적인 배치 작업에 사용됩니다.
 - 예) 매일 데이터베이스 백업, 로그 청소, 이메일 발송 등

```bash
vi K8S/CH05/cronjob.yaml
```
```yaml
# cronjob.yaml
apiVersion: batch/v1             # 최신 API 버전
kind: CronJob                    # 리소스 종류
metadata:
  name: hello                    # CronJob 이름
spec:
  schedule: "*/1 * * * *"        # 매 1분마다 실행
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure   # 실패한 경우에만 재시작

```

```bash
kubectl apply -f K8S/CH05/cronjob.yaml
# cronjob.batch/hello created

kubectl get cronjob
# NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
# hello   */1 * * * *   False     0        <none>          4s

kubectl get job
# NAME               COMPLETIONS   DURATION   AGE
# hello-1584873060   0/1           3s         3s
# hello-1584873060   0/1           3s         62s
```

```bash
kubectl delete cronjob --all
```