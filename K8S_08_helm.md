# 8. helm 패키지 매니저

## 8.1 `helm`이란
### 🛳️ Helm은 Kubernetes 패키지 매니저입니다.
* 쉽게 말해, Kubernetes에서 애플리케이션을 설치·관리·업데이트·삭제할 수 있게 해주는 도구입니다.
* Linux의 apt, yum, brew 같은 역할을 Kubernetes에서 수행합니다.

### ✅ Helm이 필요한 이유
* Kubernetes에서 앱을 배포하려면 여러 YAML 파일을 작성해야 합니다:
* Deployment, Service, ConfigMap, Ingress, PVC 등등…
* 복잡한 앱일수록 관리가 어려움
* ➡️ Helm은 이 복잡함을 템플릿과 버전 관리로 해결합니다.

### 🎯 Helm 핵심 개념
| 용어             | 설명                       |
| -------------- | ------------------------ |
| **Chart**      | 애플리케이션 패키지 (템플릿 + 설정 포함) |
| **Release**    | Chart를 설치한 인스턴스          |
| **Repository** | Chart를 저장하고 배포하는 저장소     |

### 📦 예: Helm으로 nginx 설치
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-nginx bitnami/nginx
```
* bitnami/nginx: Chart 이름
* my-nginx: 설치될 Release 이름

### 🛠️ Helm 사용 주요 명령
| 명령                 | 설명             |
| ------------------ | -------------- |
| `helm repo add`    | Chart 저장소 추가   |
| `helm search repo` | 저장소에서 Chart 검색 |
| `helm install`     | Chart 설치       |
| `helm upgrade`     | 설치된 앱 업데이트     |
| `helm uninstall`   | 앱 삭제           |
| `helm list`        | 설치된 앱 목록       |

### 📚 요약
* Helm은 Kubernetes의 패키지 매니저
* 반복되는 리소스 정의를 템플릿화
* 애플리케이션의 버전 관리 및 배포 자동화 지원
* Helm Chart는 Kubernetes 앱의 설치 스크립트

### 8.1.1 `helm` 설치

```bash
# Helm 3 최신 버전을 자동으로 설치합니다.
# curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash -s -- --version v3.2.2
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash # 최신버전설치
# helm installed into /usr/local/bin/helm

helm version
# version.BuildInfo{Version:"v3.18.4",  GitCommit:"d80 ...
```
#### ✅ 명령 설명:
* curl ... | bash: Helm 설치 스크립트를 받아서 바로 실행합니다.
* -s --: bash에 추가 인자를 넘기기 위한 표준 방식입니다.
* --version v3.2.2: 설치할 Helm 버전을 명시합니다. 최신이 아닌 특정 버전을 원할 때 유용합니다.

### 8.1.2 `chart` 생성

```bash
helm create mychart
# Creating mychart

ls mychart
# Chart.yaml  charts  templates  values.yaml
```

```bash
ls mychart/templates
# NOTES.txt
# _helpers.tpl
# deployment.yaml
# ingress.yaml
# service.yaml
# serviceaccount.yaml
# tests/
```
```bash
cat mychart/templates/service.yaml
```
```yaml
# mychart/templates/service.yaml
# 이 템플릿은 values.yaml과 함께 조합되어 최종 리소스를 생성
apiVersion: v1  # 이 Service 리소스가 사용할 Kubernetes API 버전. Service는 항상 v1을 사용함.
kind: Service   # 생성할 리소스의 종류. 여기서는 클러스터 내에서 트래픽을 라우팅하는 'Service'.

metadata:
  name: {{ include "mychart.fullname" . }}  
  # Service의 이름을 정의. Helm 템플릿 함수 include를 사용하여 fullname 헬퍼 함수의 결과를 삽입.
  # 일반적으로 "<release-name>-<chart-name>" 형식으로 자동 생성됨.

  labels:
    {{- include "mychart.labels" . | nindent 4 }}  
    # 라벨은 리소스 식별용 메타데이터.
    # mychart.labels 템플릿의 결과를 들여쓰기(nindent 4)를 적용해 삽입함.
    # 이 라벨은 나중에 selector 또는 관리 목적에 사용됨.

spec:
  type: {{ .Values.service.type }}  
  # Service의 유형을 설정. values.yaml에 정의된 값 사용.
  # 일반적으로 'ClusterIP', 'NodePort', 'LoadBalancer' 중 하나.

  ports:
    - port: {{ .Values.service.port }}  
      # 클러스터 내에서 노출할 포트. values.yaml에서 정의된 포트 번호.

      targetPort: http  
      # 이 서비스가 포워딩할 실제 컨테이너 포트 이름 또는 번호.
      # 'http'는 Deployment에서 정의한 containerPort의 name이 'http'여야 연결됨.

      protocol: TCP  
      # 네트워크 프로토콜. 일반적으로 TCP나 UDP를 사용. 웹 요청이므로 TCP.

      name: http
  selector:
    {{- include "mychart.selectorLabels" . | nindent 4 }}  
    # 이 Service가 트래픽을 전달할 Pod을 선택할 때 사용할 라벨 조건.
    # 해당 라벨이 붙은 Pod들에게만 트래픽이 전달됨.
    # Deployment에서 동일한 selectorLabels를 사용해야 연결됨.
```
#### 🔍 관련 Helm 헬퍼 템플릿 (_helpers.tpl) 예시
```tpl
{{- define "mychart.fullname" -}}
{{ printf "%s-%s" .Release.Name .Chart.Name }}
{{- end }}

{{- define "mychart.labels" -}}
app.kubernetes.io/name: {{ .Chart.Name }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}

{{- define "mychart.selectorLabels" -}}
app.kubernetes.io/name: {{ .Chart.Name }}
{{- end }}
```

```bash
vi mychart/values.yaml
```
```yaml
# values.yaml
replicaCount: 1

image:
  repository: nginx
  pullPolicy: IfNotPresent

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

...
# 약 40줄
service:
  type: LoadBalancer  # 기존 ClusterIP
  port: 8888          # 기존 80

...
```

### 8.1.3 chart 설치
#### ✅ Helm Chart 설치 명령어
```bash
helm install <릴리스이름> <차트경로 or 차트이름>
```

#### 📌 예시 1: 로컬 디렉터리의 차트 설치
```bash
helm install myapp ./mychart
# myapp: 설치될 Helm release의 이름
# ./mychart: 차트가 있는 디렉터리 경로
```

#### 📌 예시 2: 공개 저장소에서 차트 설치 (예: NGINX Ingress)
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install my-nginx ingress-nginx/ingress-nginx
```

#### 📌 유용한 옵션
| 옵션                        | 설명                    |
| ------------------------- | --------------------- |
| `--dry-run --debug`       | 설치 전 미리보기 (실제 설치 안 됨) |
| `--values my-values.yaml` | 커스텀 값 파일 적용           |
| `--set key=value`         | 커맨드라인에서 변수 직접 지정      |
| `--namespace myns`        | 특정 네임스페이스에 설치         |
| `--create-namespace`      | 네임스페이스가 없으면 자동 생성     |

#### 🔍 설치 확인
```bash
helm list
```

#### 📌 실습
* service.yaml과 values.yaml이 합쳐저서 최종 service리소스가 생성

```bash
helm install foo ./mychart
# NAME: foo
# LAST DEPLOYED: Tue Mar 10 14:26:02 2020
# NAMESPACE: default
# STATUS: deployed
# REVISION: 1
# NOTES:
#    ....
```

```bash
# service 리소스를 조회합니다.
kubectl get svc
# NAME         TYPE          CLUSTER-IP      EXTERNAL-IP    PORT(S)   
# kubernetes   ClusterIP     10.43.0.1       <none>         443/TCP   
# foo-mychart  LoadBalancer  10.43.142.107   10.0.1.1       8888:32597/TCP 
```

### 8.1.4 `chart` 리스트 조회

```bash
# 설치된 chart 리스트 확인하기
helm list
# NAME    NAMESPACE  REVISION  UPDATED   STATUS    CHART          APP VER
# foo     default    1         2020-3-1  deployed  mychart-0.1.0  1.16.0

# 다른 네임스페이스에는 설치된 chart가 없습니다.
helm list -n kube-system
# NAME   NAMESPACE   REVISION    UPDATED STATUS  CHART   APP   VERSION
```

### 8.1.5 chart 랜더링
* **Chart 렌더링(rendering)**은 템플릿으로 작성된 Helm 차트를 실제 Kubernetes가 이해할 수 있는 YAML 매니페스트로 변환하는 과정
#### 🎯 Chart 렌더링이란?
>* Helm Chart에 포함된 Go 템플릿 문법({{ }}) 과 사용자 값(values.yaml)을 바탕으로 
>* 실제 Kubernetes 리소스 YAML을 생성하는 작업입니다.

##### 🛠️ 렌더링 방법
###### ✅ 렌더링만 하고 결과 확인 (클러스터에 적용 X):
```bash
helm template myrelease ./mychart
```
###### ✅ 결과를 파일로 저장:
```bash
helm template myrelease ./mychart > output.yaml
```
##### 🧠 언제 사용하나요?
| 상황                      | 이유                                |
| ----------------------- | --------------------------------- |
| ❓ 실제 배포 전 결과 미리 보기      | 템플릿 오류 확인, 구조 검증                  |
| 🧪 CI/CD 파이프라인에서 자동 테스트 | 클러스터 없이 YAML 생성 가능                |
| 🛠 YAML을 수동 수정해 사용할 경우  | Helm 없이 `kubectl apply -f`로 배포 가능 |

#### 🔍 렌더링 vs 배포
| 기능             | `helm template` | `helm install` |
| -------------- | --------------- | -------------- |
| Kubernetes에 배포 | ❌ 아니요           | ✅ 예            |
| YAML 생성        | ✅ 예             | ✅ 내부적으로 생성     |
| 디버깅용으로 적합      | ✅ 매우 적합         | ❌ 덜 적합         |

```bash
# 📦 Helm Template 명령이 하는 일
# helm install과 달리 Kubernetes 클러스터에 아무것도 배포하지 않고, 템플릿을 렌더링만 합니다.
# 즉, mychart 디렉토리에 있는 Helm 템플릿 파일들 (templates/*.yaml)을 values와 함께 실제 YAML 형식으로 변환해 foo-output.yaml 파일로 저장합니다.
helm template foo ./mychart > foo-output.yaml
```
##### ✅ 명령 구성 요소
| 부분                | 의미                                                        |
| ----------------- | --------------------------------------------------------- |
| `helm template`   | Helm 차트를 **Kubernetes YAML 매니페스트로 변환** (렌더링)              |
| `foo`             | 릴리스 이름. Helm 템플릿 렌더링 시 이름으로 사용됨 (`{{ .Release.Name }}` 등) |
| `./mychart`       | 차트 디렉토리 경로 (현재 디렉토리의 `mychart`)                           |
| `>`               | 출력 리디렉션                                                   |
| `foo-output.yaml` | 변환된 YAML 결과를 저장할 **출력 파일** 이름                             |
##### ✅ 사용 시기
| 상황          | 사용 이유                              |
| ----------- | ---------------------------------- |
| CI/CD 파이프라인 | 클러스터 배포 전, 매니페스트를 미리 검토            |
| 디버깅         | 템플릿과 values가 렌더링된 결과를 확인하고 검증      |
| GitOps      | Helm chart → YAML 변환 후 Git 저장소에 커밋 |

```bash
cat foo-output.yaml
# 전체 YAML 정의서 출력
```

### 8.1.6 chart 업그레이드
* 이미 설치한 Helm 릴리스를 새로 수정된 차트(또는 values)로 재배포하는 작업

#### 🛠️ Helm Chart 업그레이드란?
>이미 설치된 Helm 차트를 수정된 설정 파일(values.yaml)이나 템플릿 코드로 변경사항을 반영하는 과정입니다.

#### 📌 기본 명령어
```bash
helm upgrade <RELEASE_NAME> <CHART_PATH_OR_NAME> [flags]
```
#### 📌 예시:
```bash
helm upgrade foo ./mychart
# foo는 릴리스 이름, ./mychart는 로컬 디렉토리 또는 리포지토리 차트 이름입니다.
```
#### 🔄 자주 사용하는 옵션
| 옵션                | 설명                         |
| ----------------- | -------------------------- |
| `--install`       | 릴리스가 없을 경우 자동 설치           |
| `-f values.yaml`  | 사용자 정의 값을 적용하여 업그레이드       |
| `--set key=value` | CLI에서 직접 값 지정              |
| `--dry-run`       | 실제 배포 없이 시뮬레이션 (렌더링 결과 확인) |

```bash
# 예:
helm upgrade foo ./mychart -f custom-values.yaml
# 또는:

helm upgrade foo ./mychart --set service.type=NodePort
```

#### 🧪 업그레이드 전 시뮬레이션 (dry-run)
```bash
helm upgrade foo ./mychart --dry-run --debug
# 템플릿 렌더링 결과와 실제 적용될 내용을 미리 확인할 수 있어 안전합니다.
```

#### 📦 Chart 업그레이드가 필요한 경우
| 상황               | 이유                                                   |
| ---------------- | ---------------------------------------------------- |
| `values.yaml` 수정 | 리소스 개수, 포트 등 변경                                      |
| 템플릿 코드 수정        | 버그 수정, 구조 변경                                         |
| 이미지 태그 업데이트      | 새 버전 앱 배포                                            |
| 롤링 업데이트 필요       | 무중단 배포 적용 가능 (`Deployment`, `RollingUpdate` 전략 활용 시) |

#### 🔁 업그레이드 vs 설치
| 항목    | `helm install` | `helm upgrade` |
| ----- | -------------- | -------------- |
| 역할    | 새 릴리스 배포       | 기존 릴리스 갱신      |
| 사용 대상 | 처음 배포할 때       | 배포된 릴리스 수정할 때  |
| 덮어쓰기  | ❌ 없음           | ✅ 기존 리소스 갱신    |

#### ⏪ 실패 시 되돌리기 (Rollback)
```bash
helm rollback foo [REVISION]
# 업그레이드 실패 후 원하는 이전 버전으로 쉽게 복구할 수 있습니다.
```

#### 📦 실습
* 기 설치한 chart에 대해 values.yaml값을 수정하고 업데이트할 수 있다.
* Service 타입을 NodePort로 수정
```bash
vi mychart/values.yaml
```
```yaml
# values.yaml
...

service:
  type: NodePort    # 기존 LoadBalancer
  port: 8888        
...
```

```bash
helm upgrade foo ./mychart
# Release "foo" has been upgraded. Happy Helming!
# NAME: foo
# LAST DEPLOYED: Mon Jul  6 19:26:35 2020
# NAMESPACE: default
# STATUS: deployed
# REVISION: 2
# ...
# export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services foo-mychart)
# export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
# echo http://$NODE_IP:$NODE_PORT
```
##### 🛠️ 명령어 구성 설명
| 항목          | 설명                                          |
| ----------- | ------------------------------------------- |
| `upgrade`   | Helm 릴리스를 업그레이드하는 명령                        |
| `foo`       | 업그레이드할 **릴리스 이름** (이미 `install` 했던 이름이어야 함) |
| `./mychart` | 업그레이드에 사용할 **차트의 경로** (로컬 폴더 또는 리포지토리 가능)   |

```bash
kubectl get svc
# NAME        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          
# kubernetes  ClusterIP   10.43.0.1      <none>        443/TCP   
# foo         NodePort    10.43.155.85   <none>        8888:32160/TCP 

helm list
# NAME     NAMESPACE  REVISION   UPDATED    STATUS      CHART      
# foo      default    2          2020-3-2   deployed    mychart-0.1.0 
```

### 8.1.7 chart 배포상태 확인

```bash
helm status foo
# Release "foo" has been upgraded. Happy Helming!
# NAME: foo
# LAST DEPLOYED: Mon Jul  6 19:26:35 2020
# NAMESPACE: default
# STATUS: deployed
# REVISION: 2
# ...
```

### 8.1.8 `chart` 삭제

```bash
helm delete foo
# release "foo" uninstalled

helm list
# NAME   NAMESPACE   REVISION    UPDATED STATUS  CHART   APP   VERSION
```


## 8.2 원격 레포지토리
* Helm에서 **원격 레포지토리(Remote Repository)**란, 미리 패키징된 Helm 차트들을 인터넷 상에서 다운로드(install)하거나 검색(search)할 수 있도록 제공하는 저장소를 의미

### 🔧 1. 원격 레포지토리란?
* Helm 차트는 .tgz 형식의 패키지로 묶여 Helm 리포지토리에 저장됩니다.
* 이러한 리포지토리는 GitHub Pages, S3, GCS, Nexus, Harbor 등 다양한 위치에 존재할 수 있습니다.
* 사용자는 Helm CLI를 통해 해당 레포지토리에서 차트를 설치하거나 업데이트할 수 있습니다.

### 🌐 2. 대표적인 공식 레포지토리
| 이름                   | 주소                                                   |
| -------------------- | ---------------------------------------------------- |
| Bitnami              | `https://charts.bitnami.com/bitnami`                 |
| Prometheus Community | `https://prometheus-community.github.io/helm-charts` |
| Grafana              | `https://grafana.github.io/helm-charts`              |
| ingress-nginx        | `https://kubernetes.github.io/ingress-nginx`         |

### 📥 3. 레포지토리 추가
```bash
helm repo add <이름> <URL>
# 예:

helm repo add bitnami https://charts.bitnami.com/bitnami
```

### 🔄 4. 레포지토리 목록 갱신
```bash
helm repo update
```

### 🔍 5. 차트 검색
```bash
helm search repo <키워드>
# 예:
helm search repo nginx
```

### ⬇️ 6. 설치
```bash
helm install mynginx bitnami/nginx
```
### 🔧 7. 설치 가능한 모든 레포지토리 보기
```bash
helm repo list
```

### ✅ 요약
| 명령어                | 설명                |
| ------------------ | ----------------- |
| `helm repo add`    | 원격 레포지토리 등록       |
| `helm repo update` | 등록된 레포지토리 정보 최신화  |
| `helm search repo` | 등록된 레포지토리에서 차트 검색 |
| `helm install`     | 원격 차트 설치          |

### 8.2.1 레포지토리 추가

```bash
# stable repo 추가
helm repo add bitnami https://charts.bitnami.com/bitnami
```

### 8.2.2 레포지토리 업데이트

```bash
# repo update
helm repo update
# ...Successfully got an update from the "stable" chart repository
# Update Complete. ⎈ Happy Helming!⎈
```

### 8.2.3 레포지토리 조회

```bash
# 현재 등록된 repo 리스트
helm repo list
# NAME    URL
# bitnami https://charts.bitnami.com/bitnami
```

### 8.2.4 레포지토리내 chart 조회

```bash
# stable 레포 안의 chart 리스트
helm search repo bitnami
# NAME                    CHART VERSION  APP VERSION    DESCRIPTION
# stable/aerospike        0.3.2          v4.5.0.5       A Helm chart ..
# stable/airflow          7.1.4          1.10.10        Airflow is a ..
# stable/ambassador       5.3.2          0.86.1         DEPRECATED ...
# stable/anchore-engine   1.6.8          0.7.2          Anchore container
# stable/apm-server       2.1.5          7.0.0          The server ...
# ...

# Airflow 차트는 Apache Airflow를 Kubernetes 클러스터에 손쉽게 배포할 수 있도록 
# Helm 패키지 형식으로 구성된 템플릿 모음
helm search repo bitnami/airflow
# NAME            CHART VERSION   APP VERSION     DESCRIPTION
# stable/airflow  7.2.0           1.10.10         Airflow is a plat...
```
##### 🚀 Airflow Chart 구성요소
| 구성 요소           | 설명                                                 |
| --------------- | -------------------------------------------------- |
| **Scheduler**   | DAG를 스케줄링하고 실행할 작업을 결정합니다                          |
| **Web Server**  | Airflow 웹 UI 제공                                    |
| **Worker**      | 태스크를 실행하는 Pod (Celery, KubernetesExecutor 등 사용 가능) |
| **Database**    | 메타데이터 저장용 (기본적으로 PostgreSQL)                       |
| **Redis**       | Celery 메시지 브로커                                     |
| **DAGs Config** | DAG 코드 마운트 또는 Git 연동                               |
| **Logs Volume** | 로그 저장용 PVC 또는 외부 저장소 연동 가능                         |

## helm 허브: https://artifacthub.io

## 8.3 외부 chart 설치 (WordPress)

### 8.3.1 `chart install`
* 원격(repository)으로 설치

```bash
helm install wp bitnami/wordpress \
  --version 15.0.0 \
  --set service.port=8080 \
  --namespace default

# --version 은 Bitnami에서 원하는 버전으로 바꾸시면 됩니다.
# (예: 15.0.0은 예시이며, helm search repo bitnami/wordpress로 최신 버전 확인 가능)
# service.port=8080은 WordPress 서비스 노출 포트를 8080으로 변경하는 설정입니다.
# --namespace default는 기본 네임스페이스에서 설치한다는 의미입니다.

# NAME: wp
# LAST DEPLOYED: Mon Jul  6 20:44:55 2020
# NAMESPACE: default
# STATUS: deployed
# REVISION: 1
# NOTES:
# ...

kubectl get pod
# NAME                            READY   STATUS              RESTARTS   AGE
# wp-mariadb-0                    0/1     ContainerCreating   0          61s
# wp-wordpress-6c5c794b7c-mkdmc   0/1     ContainerCreating   0          61s

kubectl get svc
# NAME           TYPE           CLUSTER-IP      EXTERNAL-IP                       PORT(S)                      AGE
# kubernetes     ClusterIP      10.43.0.1       <none>                            443/TCP                      27h
# wp-mariadb     ClusterIP      10.43.205.138   <none>                            3306/TCP                     105s
# wp-wordpress   LoadBalancer   10.43.216.253   192.168.164.130,192.168.164.131   80:31252/TCP,443:32190/TCP   105s
```

```yaml
# values.yaml
...
service:
  port: 80  -->  8080
...
```

```bash
# curl로 접근해 봅니다.
curl localhost:80
# browser : http://192.168.164.130/
```

### 8.3.2 `chart fetch`
* local로 다운해서 설치

```bash
helm fetch bitnami/wordpress --version 15.0.0 --untar

ls wordpress/
# Chart.yaml  README.md  charts  requirements.lock  
# requirements.yaml  templates  values.schema.json  values.yaml

# 사용자 입맛에 따라 세부 설정값 변경
vi wordpress/values.yaml
# ...

helm install wp-fetch ./wordpress
# WARNING: This chart is deprecated
# NAME: wp-fetch
# LAST DEPLOYED: Mon Jul  6 20:44:55 2020
# NAMESPACE: default
# STATUS: deployed
# REVISION: 1
# NOTES:
# ...
```

### Clean up

```bash
helm list
helm delete wp
helm delete wp-fetch
kubectl get pvc
kubectl delete pvc data-wp-mariadb-0
```