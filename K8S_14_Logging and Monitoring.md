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



```bash
# fetch stable repository의 elastic-stack
helm fetch --untar stable/elastic-stack --version 2.0.1

vim elastic-stack/values.yaml
```

```yaml
# elastic-stack/values.yaml

# 약 12줄
logstash:
  enabled: false  # 기존 true

# 약 29줄
fluent-bit:
  enabled: true   # 기존 false
```

```bash
# elasticsearch 수정
vim elastic-stack/charts/elasticsearch/values.yaml
```

```yaml
# elastic-stack/charts/elasticsearch/values.yaml
# ...
# 약 110줄
client:
  replicas: 1  # 기존 2
# ...
# 약 171줄
master:
  replicas: 2  # 기존 3
# ...
# 약 225줄
data:
  replicas: 1  # 기존 2
```

```bash
# fluent-bit 수정
vim elastic-stack/charts/fluent-bit/values.yaml
```

```yaml
# elastic-stack/charts/fluent-bit/values.yaml
# ...
# 약 45줄
backend:
  type: es    # 기존 forward
  # ...
  es:
    host: efk-elasticsearch-client   # 기존 elasticsearch -> host 변경

# ...

# 약 226줄
input:
  tail:
    memBufLimit: 5MB
    parser: docker
    path: /var/log/containers/*.log
    ignore_older: ""
  systemd:
    enabled: true   # 기존 false
    filters:
      systemdUnit:
        - docker.service
        - k3.service    # 기존 kubelet.service
        # - node-problem-detector.service  --> 주석처리
```

```bash
# kibana ingress 수정
vim elastic-stack/charts/kibana/values.yaml
```

```yaml
# elastic-stack/charts/kibana/values.yaml
# ...
# 약 79줄 - kibana ingress 설정하기
ingress:
  enabled: true   # 기존 false
  hosts:
  - kibana.10.0.1.1.sslip.io   # 공인IP 입력
  annotations:
    kubernetes.io/ingress.class: nginx
```

```bash
helm install efk ./elastic-stack
# NAME: efk
# LAST DEPLOYED: Sat Jul 11 07:17:06 2020
# NAMESPACE: default
# STATUS: deployed
# REVISION: 1
# NOTES:
# The elasticsearch cluster and associated extras have been installed.
# Kibana can be accessed:
# ...

# 모든 Pod가 다 실행되기까지 wait
watch kubectl get pod,svc
```

index를 생성하는 방법은 다음과 같습니다.

1. `Explore on my own` 클릭
2. 왼쪽 패널 `Discover` 클릭
3. Index pattern에 `kubernetes_cluster-*` 입력 > Next step
4. Time Filter field name에 `@timestamp` 선택 > Create index pattern
5. 다시 `Discover` 패널로 가면 `Pod`들의 로그들을 볼 수 있습니다.

### Clean up

```bash
helm delete efk
```

## 14.2 리소스 모니터링 시스템 구축

### 14.2.2 컨테이너 메트릭 정보 수집 원리

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
