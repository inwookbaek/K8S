📘 EFK 스택 설치 가이드 for K3s using Helm

K3s 환경에서 EFK 스택 (Elasticsearch, Fluent Bit, Kibana) Helm 설치 가이드
이 가이드는 K3s 클러스터에 로깅 및 모니터링을 위한 EFK 스택을 Helm을 사용하여 배포하는 방법을 다룹니다.

주의사항:

이 가이드는 K3s 클러스터가 이미 설정되어 있고 kubectl 및 helm 명령어가 작동하는 것을 전제로 합니다.
Elasticsearch는 상당한 리소스를 필요로 합니다. K3s 노드의 리소스 (RAM, CPU)가 충분한지 확인하십시오. 최소 4GB RAM을 권장합니다.
영구 볼륨(Persistent Volume)을 사용하지 않을 경우, Elasticsearch 데이터는 Pod가 재시작되면 손실될 수 있습니다. 여기서는 기본적으로 영구 볼륨을 사용하는 것으로 가정합니다.

1. 필요 리소스 설치 여부 확인 명령
EFK 스택을 설치하기 전에 현재 클러스터에 관련 리소스가 있는지 확인하여 잠재적인 충돌을 방지합니다.

```Bash
# 네임스페이스 확인 (기본적으로 'logging' 네임스페이스를 사용할 예정)
kubectl get ns logging

# 현재 클러스터에 배포된 Helm 릴리스 확인
helm list --all-namespaces

# Elasticsearch 관련 리소스 확인
kubectl get all -n logging -l app.kubernetes.io/name=elasticsearch
kubectl get pvc -n logging -l app.kubernetes.io/name=elasticsearch
kubectl get pv -n logging -l app.kubernetes.io/name=elasticsearch

# Kibana 관련 리소스 확인
kubectl get all -n logging -l app.kubernetes.io/name=kibana

# Fluent Bit 관련 리소스 확인
kubectl get all -n logging -l app.kubernetes.io/name=fluent-bit

# (선택 사항) csi-hostpath-driver 확인 (K3s 기본 스토리지 클래스)
# K3s는 기본적으로 `k3s-storage`라는 hostpath 기반의 CSI 드라이버를 사용합니다.
# 별도의 스토리지 클래스를 구성하지 않았다면 기본 스토리지 클래스가 존재해야 합니다.
kubectl get sc
```

2. EFK 스택과 관련된 모든 리소스 삭제 명령 (설치 여부와 상관없이 삭제)
이 단계는 이전 설치 또는 실패한 설치로 인해 남아있는 모든 리소스를 정리하여 깨끗한 상태에서 시작할 수 있도록 합니다.

```Bash

echo "EFK 스택 관련 리소스 삭제를 시작합니다..."

# Helm 릴리스 삭제 (이름은 기본적으로 'elasticsearch', 'kibana', 'fluent-bit'로 가정)
# 만약 다른 이름으로 설치되었다면 해당 이름으로 수정해야 합니다.
helm uninstall elasticsearch -n logging || echo "Elasticsearch helm release not found."
helm uninstall kibana -n logging || echo "Kibana helm release not found."
helm uninstall fluent-bit -n logging || echo "Fluent Bit helm release not found."

# 네임스페이스 삭제 (모든 관련 리소스를 포함)
# 이 방법은 해당 네임스페이스 내의 모든 리소스를 삭제하므로 주의해서 사용하십시오.
kubectl delete ns logging --ignore-not-found=true

# PV (Persistent Volume) 삭제 - PVC가 삭제되어도 PV는 남아있을 수 있습니다.
# 재사용을 원치 않는 경우 삭제합니다.
# 주의: PV를 삭제하면 해당 볼륨의 데이터가 영구적으로 손실됩니다.
echo "Elasticsearch 관련 PV를 삭제합니다. (주의: 데이터 영구 삭제)"
kubectl get pv -l app.kubernetes.io/name=elasticsearch -o name | xargs -r kubectl delete

echo "모든 EFK 스택 관련 리소스 삭제를 완료했습니다."
echo "잠시 기다려 모든 리소스가 완전히 제거될 시간을 줍니다..."
sleep 30 # 리소스가 완전히 종료될 시간을 줍니다.
3. 생성 전 수행하기 위한 명령 (사전 준비)
EFK 스택을 배포하기 전에 필요한 네임스페이스를 생성하고, Helm 레포지토리를 추가/업데이트합니다.

Bash

echo "EFK 스택 배포를 위한 사전 준비를 시작합니다..."

# 'logging' 네임스페이스 생성
kubectl create namespace logging --dry-run=client -o yaml | kubectl apply -f -
echo "네임스페이스 'logging'이 생성되었거나 이미 존재합니다."

# Helm 레포지토리 추가 및 업데이트
# Elasticsearch와 Kibana는 Elastic Helm Charts를 사용합니다.
helm repo add elastic https://helm.elastic.co
helm repo update

# Fluent Bit은 Fluent Helm Charts를 사용합니다.
helm repo add fluent https://fluent.github.io/helm-charts
helm repo update

echo "Helm 레포지토리 추가 및 업데이트가 완료되었습니다."
4. 생성 명령 (Helm Chart 배포)
이제 EFK 스택의 각 구성 요소를 Helm을 사용하여 배포합니다.

4-1. Elasticsearch 배포

values.yaml 파일을 사용하여 설정하는 것이 좋습니다.

k3s-storage는 K3s의 기본 스토리지 클래스 이름입니다. 사용하는 스토리지 클래스에 따라 변경해야 합니다.

resources는 예시이며, 실제 환경에 맞게 조정해야 합니다.

xpack.security.enabled=false는 개발/테스트 환경에서 인증 없이 Elasticsearch에 접근하기 위함입니다. 프로덕션 환경에서는 반드시 활성화하고 보안을 강화해야 합니다.

Bash

echo "Elasticsearch를 배포합니다..."

# Elasticsearch values.yaml 파일 생성
cat <<EOF > elasticsearch-values.yaml
clusterName: elasticsearch
nodeGroup: hot
nodeSet: hot
replicas: 1 # K3s 단일 노드 환경이므로 1로 설정. 고가용성을 위해 3 이상 권장.
minimumMasterNodes: 1 # replicas와 동일하게 설정
volumeClaimTemplate:
  storageClassName: k3s-storage # K3s 기본 스토리지 클래스 사용
  resources:
    requests:
      storage: 10Gi # 필요한 저장 공간. 최소 10Gi 권장.
resources:
  requests:
    cpu: "500m"
    memory: "2Gi"
  limits:
    cpu: "1"
    memory: "4Gi"
esJavaOpts: "-Xmx1g -Xms1g" # JVM 힙 사이즈 설정 (memory 설정과 일치시키거나 더 작게)
service:
  type: ClusterIP
xpack:
  security:
    enabled: false # 개발/테스트 환경에서는 false로 설정하여 보안 비활성화
EOF

# Helm을 사용하여 Elasticsearch 배포
helm install elasticsearch elastic/elasticsearch \
  --version 8.13.3 \
  --namespace logging \
  -f elasticsearch-values.yaml \
  --wait # Pod가 Ready 상태가 될 때까지 기다림

echo "Elasticsearch 배포 명령이 실행되었습니다."
4-2. Kibana 배포

Kibana는 Elasticsearch와 동일한 네임스페이스에 배포되어야 합니다.

elasticsearchHosts는 Elasticsearch 서비스의 URL을 지정합니다. (기본적으로 http://elasticsearch-master:9200)

ingress는 외부 접근을 위한 Ingress를 설정하는 부분입니다. K3s에서 Ingress 컨트롤러 (예: Traefik, Nginx Ingress Controller)가 작동하고 있다고 가정합니다. 여기서는 Ingress를 활성화합니다.

Bash

echo "Kibana를 배포합니다..."

# Kibana values.yaml 파일 생성
cat <<EOF > kibana-values.yaml
elasticsearchHosts: "http://elasticsearch-master:9200" # Elasticsearch 서비스 URL
service:
  type: ClusterIP # Ingress를 통해 접근하므로 ClusterIP로 충분
resources:
  requests:
    cpu: "200m"
    memory: "500Mi"
  limits:
    cpu: "1"
    memory: "1Gi"
ingress:
  enabled: true
  className: traefik # K3s 기본 Traefik Ingress Controller 사용
  hosts:
    - kibana.k3s.local # Kibana 접속을 위한 도메인 (사용자 환경에 맞게 변경 필요)
  tls: [] # TLS 설정이 필요한 경우 여기에 추가. (예: cert-manager 사용)
EOF

# Helm을 사용하여 Kibana 배포
helm install kibana elastic/kibana \
  --version 8.13.3 \
  --namespace logging \
  -f kibana-values.yaml \
  --wait # Pod가 Ready 상태가 될 때까지 기다림

echo "Kibana 배포 명령이 실행되었습니다."
Kibana Ingress 접근을 위한 /etc/hosts 설정 (로컬 테스트용):

Kibana Ingress가 작동하려면, Ingress의 hosts에 설정한 도메인 (kibana.k3s.local 등)이 K3s 노드의 IP 주소를 가리키도록 설정해야 합니다. 로컬 테스트 환경에서는 /etc/hosts 파일에 추가하는 방법이 있습니다.

Bash

# K3s 노드의 IP 주소를 확인합니다. (kubectl get node -o wide 명령으로 확인 가능)
# 예시: 192.168.1.100 k3s.local
echo "<K3s_Node_IP> kibana.k3s.local" | sudo tee -a /etc/hosts
<K3s_Node_IP> 부분은 실제 K3s 마스터 노드의 IP 주소로 변경해야 합니다.

4-3. Fluent Bit 배포

Fluent Bit은 클러스터의 모든 노드에 DaemonSet으로 배포되어 로그를 수집합니다.

elasticsearch.host는 Elasticsearch 서비스의 이름을, elasticsearch.port는 포트를 지정합니다.

logging 네임스페이스에 로그를 보내도록 설정합니다.

Bash

echo "Fluent Bit을 배포합니다..."

# Fluent Bit values.yaml 파일 생성
cat <<EOF > fluent-bit-values.yaml
backend:
  type: es # Elasticsearch 백엔드 사용
  es:
    host: elasticsearch-master # Elasticsearch 서비스 이름
    port: 9200 # Elasticsearch 포트
    tls: Off # TLS를 사용할 경우 On으로 설정
    logstashFormat: true
    logstashPrefix: fluentbit
    bufferSize: 1MB
tolerations:
  - key: "node-role.kubernetes.io/control-plane"
    operator: "Exists"
    effect: "NoSchedule"
  - key: "node-role.kubernetes.io/master"
    operator: "Exists"
    effect: "NoSchedule"
EOF

# Helm을 사용하여 Fluent Bit 배포
helm install fluent-bit fluent/fluent-bit \
  --version 0.45.0 \
  --namespace logging \
  -f fluent-bit-values.yaml \
  --wait # Pod가 Ready 상태가 될 때까지 기다림

echo "Fluent Bit 배포 명령이 실행되었습니다."
5. 생성 후 확인 명령
모든 구성 요소가 성공적으로 배포되었는지 확인합니다.

Bash

echo "EFK 스택 배포 상태를 확인합니다..."

# 'logging' 네임스페이스의 모든 Pod 상태 확인
kubectl get pods -n logging

# 'logging' 네임스페이스의 모든 Service 상태 확인
kubectl get svc -n logging

# 'logging' 네임스페이스의 모든 Deployment 상태 확인
kubectl get deploy -n logging

# 'logging' 네임스페이스의 DaemonSet 상태 확인 (Fluent Bit)
kubectl get ds -n logging

# 'logging' 네임스페이스의 Persistent Volume Claims (PVC) 상태 확인 (Elasticsearch)
kubectl get pvc -n logging

# Ingress 상태 확인 (Kibana)
kubectl get ing -n logging

echo "--- Elasticsearch Pod 로그 확인 ---"
# Elasticsearch Pod 이름 확인 후 로그 보기
# 예시: kubectl logs elasticsearch-master-0 -n logging
ELASTICSEARCH_POD=$(kubectl get pod -n logging -l app.kubernetes.io/name=elasticsearch,app.kubernetes.io/component=master -o jsonpath='{.items[0].metadata.name}')
if [ -n "$ELASTICSEARCH_POD" ]; then
  echo "Elasticsearch Pod ($ELASTICSEARCH_POD) 로그:"
  kubectl logs "$ELASTICSEARCH_POD" -n logging | tail -n 20
else
  echo "Elasticsearch Pod를 찾을 수 없습니다."
fi

echo "--- Kibana Pod 로그 확인 ---"
# Kibana Pod 이름 확인 후 로그 보기
# 예시: kubectl logs kibana-5xxxxxxx-xxxxx -n logging
KIBANA_POD=$(kubectl get pod -n logging -l app.kubernetes.io/name=kibana -o jsonpath='{.items[0].metadata.name}')
if [ -n "$KIBANA_POD" ]; then
  echo "Kibana Pod ($KIBANA_POD) 로그:"
  kubectl logs "$KIBANA_POD" -n logging | tail -n 20
else
  echo "Kibana Pod를 찾을 수 없습니다."
fi

echo "--- Fluent Bit Pod 로그 확인 ---"
# Fluent Bit Pod 이름 확인 후 로그 보기
# 예시: kubectl logs fluent-bit-xxxxx -n logging
FLUENT_BIT_POD=$(kubectl get pod -n logging -l app.kubernetes.io/name=fluent-bit -o jsonpath='{.items[0].metadata.name}')
if [ -n "$FLUENT_BIT_POD" ]; then
  echo "Fluent Bit Pod ($FLUENT_BIT_POD) 로그:"
  kubectl logs "$FLUENT_BIT_POD" -n logging | tail -n 20
else
  echo "Fluent Bit Pod를 찾을 수 없습니다."
fi

echo "--- Kibana Ingress 접속 확인 ---"
echo "Kibana 접속을 위해 웹 브라우저에서 'http://kibana.k3s.local' (또는 설정한 도메인) 에 접속해 보세요."
echo "정상적으로 Kibana 대시보드가 보인다면 성공입니다."
추가 확인 (Troubleshooting):

kubectl describe pod <pod-name> -n logging 명령을 사용하여 특정 Pod의 자세한 상태, 이벤트, 에러 메시지를 확인합니다.

kubectl top pod -n logging 명령으로 Pod의 리소스 사용량을 확인하여 과도한 리소스 사용으로 인한 문제 여부를 파악할 수 있습니다.

Elasticsearch Pod가 CrashLoopBackOff 상태인 경우, 할당된 리소스 (특히 메모리)가 부족하거나 JVM 힙 설정이 잘못되었을 가능성이 높습니다.

Fluent Bit이 로그를 보내지 않는 경우, Elasticsearch 연결 설정 (elasticsearch.host, elasticsearch.port)을 다시 확인하고 Fluent Bit Pod의 로그를 자세히 살펴보세요.