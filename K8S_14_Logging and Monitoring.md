# 14. 로깅과 모니터링
* 쿠버네티스는 기본적으로 내장하고 있는 로깅 및 모니터링 시스템이 없다.
* EFK stack을 이용하여 모니터링에는 Prometheus Grafana를 이용하여 시스템을 구축

## 📘 1. 로깅 (Logging)
* 로깅은 애플리케이션 및 시스템 로그를 수집하고 분석하는 작업입니다.

### 🔹 기본 구조
* 쿠버네티스에서 로그는 보통 3단계로 구성됩니다:
  1. 로그 생성: 컨테이너나 애플리케이션에서 표준 출력 (stdout) 및 표준 에러 (stderr) 로 출력
  2. 로그 수집 (Log Collector): 로그를 수집하는 에이전트 (예: fluentd, logstash)
  3. 로그 저장 및 분석: 로그 저장소 (예: Elasticsearch, Loki) 및 분석 UI (예: Kibana, Grafana)

### 🔹 대표적인 로깅 스택
| 구성요소            | 역할          |
| --------------- | ----------- |
| `Fluentd`       | 로그 수집/전송    |
| `Elasticsearch` | 로그 저장소      |
| `Kibana`        | 로그 시각화 및 검색 |

## 📊 2. 모니터링 (Monitoring)
* 모니터링은 **클러스터, 노드, 파드, 애플리케이션 등의 성능 지표(metric)**를 수집하고 **경고(alert)**를 설정하는 기능입니다.

### 🔹 주요 요소
| 도구                     | 설명              |
| ---------------------- | --------------- |
| **Prometheus**         | 시계열 데이터 수집 및 저장 |
| **Alertmanager**       | 경고 관리           |
| **Grafana**            | 시각화 대시보드        |
| **Node Exporter**      | 노드 메트릭 수집       |
| **Kube-state-metrics** | 쿠버네티스 객체 상태 수집  |

### 🔸 대표 아키텍처
```text
[Node Exporter]   [Kube-state-metrics]
       │                     │
       └────▶ [Prometheus] ◀────┘
                    │
             [Alertmanager]
                    │
               [Grafana]
```

#### 🎯 로깅 vs 모니터링 차이
| 항목     | 로깅 (Logging)           | 모니터링 (Monitoring)   |
| ------ | ---------------------- | ------------------- |
| 데이터 유형 | 로그 (텍스트 기반 이벤트)        | 메트릭 (숫자 기반 성능 지표)   |
| 목적     | 문제 원인 분석, 감사 기록        | 상태 확인, 성능 추적, 경고 발생 |
| 대표 도구  | Fluentd, Elasticsearch | Prometheus, Grafana |

## 14.1 로깅 시스템 구축
* Pod가 실행중일 떄는 kubectl logs로 확인 가능하지만 Pod가 죽게 되는등의 경우에는 그 로그기록도 사라지게 된다.
* 클러스터 레벨의 로깅시스템을 별도로 구축하여 로그기록을 저장하면  언제든지 그 로그기록을 확인할 수 있다.
* 쿠버네티스에는 컨테이너뿐만 아니라, kubelet등과 같은 쿠버네티스 시스템 컴퍼넌트가 존재한다.
* 보통, systemd를 통하여 쿠버네티스 컴퍼넌트를 실행하는데 이런 컴퍼넌트의 로그도 로깅시스템을 이용하여 확인할 수 있다.

### 14.1.2 클러스터 레벨 로깅 원리
* 쿠버네티스에서 **클러스터 레벨 로깅(Cluster-level Logging)**은 각 노드와 Pod에서 발생하는 로그를 중앙화된 시스템으로 모아, 
* Pod나 노드가 사라지더라도 로그를 유지·분석할 수 있도록 하는 구조

```bash
# 로그컨테이너 로그 조회
docker logs <CONTAINER_ID>
```

```bash
# 호스트서버의 로그 저장위치
/var/lib/docker/containers/<CONTAINER_ID>/<CONTAINER_ID>-json.log
```

```bash
# nginx라는 컨테이너를 하나 실행하고 CONTAINER_ID 값을 복사합니다.
docker run -d nginx
# 4373b7e095215c23057b1dc4423527239e56a33dbd

# docker 명령을 통한 로그 확인
docker logs 4373b7e095215c23057b1dc4423527239e56a33dbd
# /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will ...
# /docker-entrypoint.sh: Looking for shell scripts in /docker-...
# /docker-entrypoint.sh: Launching /docker-entrypoint.d/...
# ...

# 호스트 서버의 로그 파일 확인
sudo tail /var/lib/docker/containers/4373b7e095215c23057b1dc4423527239e56a33dbd/4373b7e095215c23057b1dc4423527239e56a33dbd-json.log
# {"log":"/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, \
# will attempt to perform configuration\n","stream":"stdout",\
# "time":"2020-07-11T03:22:11.817939191Z"}
# ...

# 컨테이너 정리
docker stop 4373b7e095215c23057b1dc4423527239e56a33dbd
docker rm 4373b7e095215c23057b1dc4423527239e56a33dbd
```

### 14.1.3 ElasticSearch
* 엘라스틱서치는 텍스트 검색에 최적화된 오픈 소스 검색엔지이다.
* 구조적인 테이블대신 json형식으로 데이터를 관리하고 Json문서를 파싱하여 인덱싱하여 저장하기 때문에 검색속도가 빠르다.
* EFK 스텍에서 엘라스틱서치를 로그장소로 이용한다.

#### 🧭 장단점
| 항목            | 장점                                        | 단점                           |
| ------------- | ----------------------------------------- | ---------------------------- |
| **검색 기능**     | 역색인 + 형태소 분석 → Full-text, 자동완성 등 지원       | —                            |
| **성능 & 확장성**  | 분산 구조, 빠른 색인/검색, 수평 확장 가능                 | 운영 복잡성 증가, 리소스 소비 큼          |
| **실시간성**      | Near real-time (<1초) 지원                   | 즉시 일관성 보장되지 않음               |
| **API & 생태계** | RESTful API, 다양한 언어 및 Elastic Stack 연동 지원 | —                            |
| **데이터베이스 대체** | —                                         | 트랜잭션, 일관성 부족으로 주 DB로 적합하지 않음 |
| **라이선스**      | —                                         | 오픈소스 정책 변화로 혼선 발생 가능         |

```scss
// 개념
Cluster → Node → Shard → (Document → Field)

// Cluster: 여러 Node의 집합.
// Node: 하나 이상의 Shard를 호스팅. 클러스터 상태 정보를 공유.
// Index: 하나 이상의 Shard 논리적 그룹.
// Shard: 실제 저장되는 Lucene 인덱스(물리 단위).
// Document: Shard 내 개별 데이터 단위(JSON).
// Field: Document 내 속성–값의 키-값 쌍.
```

#### 🧭 핵심 포인트 정리
| 개념       | 역할 및 특성                                                          |
| -------- | ----------------------------------------------------------------------- |
| Index    | 문서 그룹, 여러 Shard 포함. primary shard 수 고정, replica는 변경 가능  |
| Shard    | 데이터 분산 저장 단위. primary/replica 존재                             |
| Document | 저장되는 JSON 객체, `_source`, `_id`, `_index` 등의 메타 포함           |
| Field    | document 속성. 매핑(mapping)으로 타입 지정                              |
(참고) https://velog.io/@koo8624/Database-Elastic-Search-2편-아키텍처Architecture?utm_source=chatgpt.com

### 14.1.4 fluent-bit
* 일반적으로 EFK에서 F는 Fluentd를 가리킨다. fluent-bit는 Fluentd의 경량버전이다.
* fluent-bit는 데이터수집, 처리, 라우팅에 뛰어난 경량 버전 수집기이다.
* Fluentd는 로그수집, 집계처리기능이 있지만 fluent-bit는 데이터를 집계하는 기능은 없다.

#### 📋 요약
| 항목            | 설명                                           |
| ------------- | -------------------------------------------- |
| **사용 위치**     | Kubernetes 노드의 로그 파일 (`/var/log/containers`) |
| **배포 방식**     | DaemonSet, 또는 sidecar                        |
| **필터링/메타데이터** | Kubernetes 메타 삽입, 구조화, 라우팅                   |
| **출력 대상**     | Elasticsearch, Loki, HTTP, Kafka, OTLP 등 다양  |
| **장점**        | 경량, 확장성 높음, 설치 쉬움, 플러그인 풍부                   |
| **단점**        | 중앙집중형 로직 부족, 설정 복잡, sidecar 배포 번거로움          |

### 14.1.5 Kibana
* 웹을 통해 dashboard를 제공하는 데이터 시각화 플랫폼이다.
* 엘라스틱서치에 보관되어 있는 데이터를 조회하여 다양한 visual 컴포넌트로 표현한다.
* kibina는 KQL(Kibana Query Language) 질의언어를 사용하여 elastic-search로 쿼리할 수 있다.

### 14.1.6 EFK Stack
#### 🔧 EFK 스택 구성요소 및 역할
| 구성요소                      | 역할                                                                                                                          |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **Elasticsearch**         | 로그 저장소 및 검색·분석 엔진. JSON 문서 색인 기반으로 검색·조회 최적화                                        |
| **Fluentd 또는 Fluent Bit** | 각 Kubernetes 노드의 로그 파일(`/var/log/containers/*.log`)을 수집하고, 필터링 후 Elasticsearch로 전달  |
| **Kibana**                | Elasticsearch에 저장된 로그 데이터를 시각화하고 대시보드 제공     |
(참고) https://velog.io/%40borab/EFK-Stack-구축-docker-compose?utm_source=chatgpt.com 

#### 📦 핵심 아키텍처 흐름
>[Pod 로그 via stdout] → [노드 로그 파일] → Fluentd/FluentBit DaemonSet → Elasticsearch (Indexing) → Kibana 시각화

# EFK Stack 설치오류로 실습 중단 !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
```bash

# 🔄 EFK 스택 완전 삭제 및 초기화 절차
# ✅ 1. Helm 릴리스 삭제

helm uninstall elasticsearch -n logging
helm uninstall kibana -n logging
helm uninstall fluent-bit -n logging

# ✅ 2. 네임스페이스 자체 삭제 (완전 초기화 추천)
kubectl get all -n logging 
kubectl delete namespace logging
# ⚠️ stuck 될 경우 Finalizers 수동 제거 필요 (원할 시 방법 안내 가능)

# ✅ 3. 재설치를 위한 초기화
kubectl create namespace logging

# ✅ 4. EFK 스택 재설치 절차 (Helm 기반)
# ✅ Elasticsearch / Kibana / Fluent Bit

# ✅ 5. Helm 저장소 추가 및 업데이트
helm repo add elastic https://helm.elastic.co
helm repo add fluent https://fluent.github.io/helm-charts
helm repo update

# ✅ 6. Elasticsearch 설치 (TLS + 단일 노드 구성)
helm install elasticsearch elastic/elasticsearch -n logging \
  --set replicas=1 \
  --set minimumMasterNodes=1 \
  --set volumeClaimTemplate.resources.requests.storage=5Gi \
  --set resources.requests.cpu=100m \
  --set resources.requests.memory=512Mi \
  --set resources.limits.memory=1Gi \
  --set tls.enabled=true \
  --set tls.selfSigned=true \
  --set tls.createCert=true \
  --set tls.san="{elasticsearch-master,elasticsearch-master.logging,elasticsearch-master.logging.svc,elasticsearch-master.logging.svc.cluster.local}"
# ⏱ 설치 후 몇 분 정도 기다렸다가 다음으로 진행하세요:

kubectl get svc -n logging
kubectl get pods -n logging -l app=elasticsearch-master
kubectl get secret elasticsearch-master-credentials -n logging -o jsonpath='{.data.password}' | base64 -d; echo; 

# ✅ 9.  Kibana 설치
helm install kibana elastic/kibana -n logging \
  --set elasticsearchHosts=https://elasticsearch-master.logging.svc.cluster.local:9200 \
  --set elasticsearchUsername=elastic \
  --set elasticsearchPassword=$(kubectl get secret elasticsearch-master-credentials -n logging -o jsonpath='{.data.password}' | base64 -d) \
  --set elasticsearch.ssl.verificationMode=none \
  --set service.type=NodePort \
  --set server.publicBaseUrl="http://192.168.164.130:32159" \
  --timeout 10m
# ❗️ YOUR_NODE_IP는 클러스터 노드 IP, NODE_PORT는 Kibana가 노출될 포트 (kubectl get svc -n logging)를 기준으로 설정하세요.


🔹 5. Fluent Bit 설치

helm install fluent-bit fluent/fluent-bit -n logging \
  --set backend.type=es \
  --set backend.es.enable=true \
  --set backend.es.host=elasticsearch-master.logging.svc \
  --set backend.es.port=9200 \
  --set backend.es.tls=true \
  --set backend.es.tls_verify=off \
  --set backend.es.auth=true \
  --set backend.es.user=elastic \
  --set backend.es.password=$(kubectl get secret elasticsearch-master-credentials -n logging -o jsonpath='{.data.password}' | base64 -d)


kubectl run -n logging curl-test --image=curlimages/curl --rm -it -- \
  curl -vk -u elastic:$(kubectl get secret elasticsearch-master-credentials -n logging -o jsonpath='{.data.password}' | base64 -d) https://elasticsearch-master:9200
curl -vk -u elastic:$(kubectl get secret elasticsearch-master-credentials -n logging -o jsonpath='{.data.password}' | base64 -d) https://elasticsearch-master:9200

kubectl run -n logging curl-test --image=curlimages/curl -it --restart=Never -- bash


curl -vk -u elastic:iK4bJls17PNHfyzY https://elasticsearch-master:9200


helm install kibana elastic/kibana -n logging \
  --set elasticsearchHosts=https://elasticsearch-master:9200 \
  --set elasticsearchUsername=elastic \
  --set elasticsearchPassword=iK4bJls17PNHfyzY \
  --set elasticsearch.ssl.verificationMode=none \
  --set service.type=NodePort \
  --set server.publicBaseUrl="http://<YOUR_NODE_IP>:<NODE_PORT>" \
  --timeout 10m

helm install kibana elastic/kibana -n logging \
  --set elasticsearchHosts=https://elasticsearch-master.logging.svc.cluster.local:9200 \
  --set elasticsearchUsername=elastic \
  --set elasticsearchPassword=iK4bJls17PNHfyzY \
  --set elasticsearch.ssl.verificationMode=none \
  --set service.type=NodePort \
  --set server.publicBaseUrl="http://192.168.164.130:32159" \
  --timeout 10m










🔹 6. Kibana 접속

kubectl get svc -n logging
출력 예시:


kibana-kibana   NodePort   10.43.x.x   <none>   5601:32159/TCP
웹 브라우저에서 접속:


http://<YOUR_NODE_IP>:32159
🔧 문제 발생 시
Kibana 로그 확인:
kubectl logs -n logging -l app=kibana -f

Elasticsearch 로그 확인:
kubectl logs -n logging -l app=elasticsearch-master -f

```
# EFK Stack 설치오류로 실습 중단 !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!




## 14.2 리소스 모니터링 시스템 구축
* 쿠버네티스  환경에서는 특정서버라는 경계가 모호해 지고 애플리케이션 단위로 모니터링 대상이 세밀해 졌다.
* 모니터링 agent를 설치하여 agent가 metric을 모니터링 시스템이 전달하는 방식(push-based)보다는 모니터링 시스템이
* 수집해야 하는 대상을 찾아(discover) 직접 metric을 수집(pull-based)한다.

### 모니터링 시스템 구축 구성도 및 절차
#### 🎯 목적
* CPU, Memory, Disk, Network 사용량 모니터링
* 시스템 상태 시각화(GUI 대시보드)
* 알람 시스템 설정 (Slack, Email 등)
* Kubernetes 클러스터까지 포함 가능

#### 🔧 대표 구성
```plaintext
[ Node / VM / K8s Cluster ]
        │
[ Metrics Exporters ]
        │
        ▼
[ Prometheus (수집기) ]
        │
        ├──> [ Alertmanager ] ---> Slack/Email/PagerDuty
        │
        ▼
[ Grafana (시각화 대시보드) ]
```

#### ✅ 주요 컴포넌트 설명
| 구성 요소                                  | 설명                                                    |
| -------------------------------------- | ----------------------------------------------------- |
| **Prometheus**                         | 시계열 기반 메트릭 수집기. Exporter에서 메트릭을 Pull 방식으로 가져옴         |
| **Node Exporter**                      | 서버의 CPU, Memory, Disk, Network 정보를 Prometheus 형식으로 제공 |
| **Alertmanager**                       | 메트릭 기반으로 알림 트리거 (Slack, 이메일 등 연동 가능)                  |
| **Grafana**                            | Prometheus의 데이터를 시각화하는 대시보드 도구                        |
| **(옵션) Loki**                          | 로그 수집 및 시각화 (Grafana와 연동)                             |
| **(옵션) Kube-state-metrics / cAdvisor** | Kubernetes 상태 메트릭 수집용                                 |

### 14.2.2 컨테이너 메트릭 정보 수집 원리
#### 📊 컨테이너 메트릭 수집 원리 요약
```text
[ 컨테이너 (Pod) ]
      │
      ▼
[ cgroups / procfs / metrics APIs ]
      │
      ▼
[ cAdvisor / kubelet / Metrics Server ]
      │
      ▼
[ Prometheus ]
      │
      ▼
[ Grafana or Alertmanager ]
```

#### 🔧 주요 구성 요소별 설명
1. cgroups (Control Groups)
* Linux 커널 기능으로, 프로세스 단위로 CPU, 메모리, I/O, 네트워크 사용량을 추적 및 제한 가능
* 컨테이너 런타임이 각 컨테이너를 별도 cgroup으로 관리
* /sys/fs/cgroup/ 경로에서 메트릭 제공
 
2. cAdvisor (Container Advisor)
* Google에서 만든 도구로, 컨테이너의 리소스 사용량(CPU, memory, filesystem, network 등)을 실시간 수집
* Kubelet에 기본 내장되어 있음
* 기본적으로 http://<node-ip>:4194/metrics 또는 /metrics/cadvisor 엔드포인트 제공
 
3. Kubelet
* 각 노드에서 실행되는 Kubernetes 에이전트
* /metrics, /metrics/cadvisor, /metrics/resource 등 여러 메트릭 엔드포인트 제공
* Prometheus가 이를 스크래핑해서 데이터 수집
 
4. Metrics Server
* kubectl top 명령어에 사용하는 리소스 수집 컴포넌트 (하지만 Prometheus와는 별개)
* HPA(Horizontal Pod Autoscaler) 등에서 사용됨
* 간단한 CPU/Memory 정보만 제공
 
5. Prometheus
* 각 노드의 kubelet 또는 cAdvisor로부터 메트릭을 HTTP 기반으로 주기적으로 Pull(스크래핑) 함

#### 📦 수집하는 대표 메트릭
| 메트릭 이름                                   | 설명               |
| ---------------------------------------- | ---------------- |
| `container_cpu_usage_seconds_total`      | 누적 CPU 사용 시간 (초) |
| `container_memory_usage_bytes`           | 사용 중인 메모리 (바이트)  |
| `container_fs_usage_bytes`               | 디스크 사용량          |
| `container_network_receive_bytes_total`  | 네트워크 수신 바이트 수    |
| `container_network_transmit_bytes_total` | 네트워크 송신 바이트 수    |

#### 📈 수집 방식 비교
| 방식                              | 특징                                 |
| ------------------------------- | ---------------------------------- |
| **cAdvisor 직접 수집**              | 컨테이너 단위로 상세 리소스 정보 수집 가능           |
| **Kubelet `/metrics/cadvisor`** | 보안 연결(Kubelet 인증 필요), 상세 메트릭 제공    |
| **Metrics Server**              | 간단한 요약 메트릭 (주로 HPA용)               |
| **Prometheus Exporter**         | custom 컨테이너용 지표 수집용 Exporter 제작 가능 |

#### 🔐 보안 및 인증 관련
* Kubelet 메트릭 엔드포인트는 HTTPS + 인증이 필요함
* Prometheus가 Kubelet 메트릭을 수집하려면 ServiceAccount, Role, RoleBinding 등이 필요

#### ✅ 요약
| 구성 요소          | 역할                              |
| -------------- | ------------------------------- |
| cgroups        | 컨테이너 자원 사용량 추적 (커널 레벨)          |
| cAdvisor       | 컨테이너 메트릭 수집기                    |
| Kubelet        | 각 노드의 메트릭 제공 (cAdvisor 포함)      |
| Prometheus     | 메트릭 수집기 (HTTP pull 방식)          |
| Grafana        | 시각화 도구                          |
| Metrics Server | HPA 및 `kubectl top` 용 요약 메트릭 제공 |

### 14.2.1 Prometheus
* 시계열(time-series) 데이터 기반의 오픈소스 모니터링 및 경보 시스템
* 메트릭 수집, 저장, 쿼리, 시각화, 알림 기능을 모두 제공

#### 🔧 Prometheus 개요
| 항목    | 설명                                              |
| ----- | ----------------------------------------------- |
| 프로젝트  | CNCF(Cloud Native Computing Foundation) 졸업 프로젝트 |
| 라이선스  | Apache 2.0                                      |
| 개발 언어 | Go                                              |
| 특징    | Pull 기반 수집, 시계열 DB 내장, PromQL, Alertmanager 연동  |

#### 🎯 Prometheus의 핵심 기능
1. 메트릭 수집 (Scraping)
   - HTTP로 Exporter 또는 애플리케이션의 /metrics 엔드포인트에서 메트릭 수집
   - Pull 방식이 기본 (Push는 Gateway 필요)
2. 시계열 DB 내장
   - 수집된 메트릭은 Prometheus 내부 스토리지에 저장
   - 외부 저장소와도 연동 가능 (e.g. Thanos, Cortex)
3. PromQL (Prometheus Query Language)
   - 메트릭 쿼리 및 분석용 도메인 언어
   - 예: rate(http_requests_total[5m]) > 100
4. 시각화 : 자체 웹 UI 또는 Grafana에서 시각화 가능
5. 경보 시스템 연동 : Alertmanager와 통합하여 Slack, 이메일, PagerDuty 등으로 알림 전송

#### 📦 Prometheus 구성 요소
```bash
┌────────────┐
│ Exporters  │  → /metrics 제공 (e.g. Node Exporter)
└─────┬──────┘
      ↓
┌──────────────────┐
│   Prometheus     │
│ + TSDB           │
│ + PromQL Engine  │
│ + Scraper        │
└─────┬────────────┘
      ↓
┌─────────────┐
│ Alertmanager│  → 알림 (Slack, Email)
└─────────────┘
      ↓
┌─────────────┐
│ Grafana     │  → 시각화 대시보드
└─────────────┘
```

#### 🚀 기본 설치 (Standalone)
##### 1. Docker로 Prometheus 실행 (테스트용)
```bash
docker run -d \
  -p 9090:9090 \
  -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```

##### 2. prometheus.yml 설정 예
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
```

#### 🌐 웹 UI
* Prometheus 대시보드: http://<host>:9090
* 주요 메뉴:
  - Graph: PromQL로 그래프 조회
  - Targets: 스크래핑 대상 상태
  - Alerts: 현재 알림 상태
  - Status: 설정, 스토리지 등 내부 상태 확인

#### 📈 대표 Exporter
| Exporter           | 설명                       |
| ------------------ | ------------------------ |
| node\_exporter     | 서버 CPU/메모리/디스크/네트워크 등 수집 |
| blackbox\_exporter | 웹사이트, 포트 상태 등 외부 접근성 테스트 |
| mysqld\_exporter   | MySQL 데이터베이스 메트릭         |
| kube-state-metrics | Kubernetes 오브젝트 상태       |
| cadvisor           | 컨테이너 메트릭 수집              |

#### ⚠️ Push 방식은?
* Prometheus는 기본적으로 Pull 방식이지만, Push 방식이 필요한 경우 Pushgateway를 사용합니다.

```bash
[ App ] → [ Pushgateway ] ← Prometheus Pull
# Pushgateway는 짧게 실행되는 배치 작업이나 외부 메트릭 수집에 적합합니다.
```

#### 📚 학습에 유용한 PromQL 예제
| 쿼리                                                   | 설명                      |
| ---------------------------------------------------- | ----------------------- |
| `up`                                                 | 각 타겟의 상태 확인 (1 = alive) |
| `rate(http_requests_total[5m])`                      | 초당 HTTP 요청 수 (5분 평균)    |
| `node_cpu_seconds_total`                             | CPU 사용 시간 총합            |
| `sum by (instance) (node_memory_MemAvailable_bytes)` | 노드별 사용 가능한 메모리          |


#### ✅ Prometheus 요약
| 항목           | 내용                                    |
| ------------ | ------------------------------------- |
| 배포 방식        | Standalone, Kubernetes (Helm), Docker |
| 기본 포트        | `9090`                                |
| Pull vs Push | 기본은 Pull, Push는 Pushgateway 필요        |
| 저장 방식        | 시계열 DB (local 또는 remote)              |
| 쿼리 언어        | PromQL                                |
| 시각화 도구       | 자체 UI, **Grafana 연동 추천**              |
| 알림 시스템       | Alertmanager 연동                       |


### 14.2.2 컨태이너 메트릭 정보 수집 원리

```bash
docker run -d nginx
# 4373b7e095215c23057b1dc4423527239e56a33dbd

docker stats 4373b7e095215c23057b1dc4423527239e56a33dbd
# CONTAINER ID    NAME     CPU %     MEM USAGE / LIMIT     MEM    ...    
# 4af9f73eb06f    dreamy   0.00%     3.227MiB / 7.773GiB   0.04%  ...

docker stop 4373b7e095215c23057b1dc4423527239e56a33dbd
docker rm 4373b7e095215c23057b1dc4423527239e56a33dbd
```

### 14.2.3 Prometheus & Grafana 구축

```bash
helm fetch --untar stable/prometheus-operator --version 8.16.1

vim prometheus-operator/values.yaml
```

```yaml
# 약 495줄 - grafana ingress 설정하기
grafana:
  ...
  ingress:
    enabled: true   # 기존 false
    annotations:
      kubernetes.io/ingress.class: nginx   # 추가
    hosts:
    - grafana.10.0.1.1.sslip.io            # 공인IP 입력
```

```bash
helm install mon ./prometheus-operator
# manifest_sorter.go:192: info: skipping unknown hook: "crd-install"
# manifest_sorter.go:192: info: skipping unknown hook: "crd-install"
# manifest_sorter.go:192: info: skipping unknown hook: "crd-install"
# manifest_sorter.go:192: info: skipping unknown hook: "crd-install"
# manifest_sorter.go:192: info: skipping unknown hook: "crd-install"
# manifest_sorter.go:192: info: skipping unknown hook: "crd-install"
# NAME: mon
# LAST DEPLOYED: Thu Jul 16 08:44:38 2020
# ...

watch kubectl get pod
```

웹 브라우저를 통해 grafana를 접근합니다.

- `username`: admin
- `password`: prom-operator

좌측 상단의 `Home`을 누르면 다양한 대시보드가 생성된 것을 볼 수 있습니다.

### Clean up

```bash
helm delete mon
```
