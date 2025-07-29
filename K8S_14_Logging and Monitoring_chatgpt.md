📘 EFK 스택 설치 가이드 for K3s using Helm

```bash
# 1️⃣ 필요 리소스 설치 여부 확인 명령

# Elasticsearch, Kibana, Fluent Bit 설치 여부 확인
kubectl get all -n logging

# Helm release 목록에서 관련 리소스 확인
helm list -n logging

# Fluent Bit DaemonSet 확인
kubectl get daemonset -n logging

# StorageClass 확인 (Elasticsearch는 PVC를 필요로 함)
kubectl get storageclass

# PersistentVolumeClaim 확인
kubectl get pvc -n logging

# 2️⃣ 관련 리소스 전체 삭제 명령

# EFK 관련 Helm release 삭제
helm uninstall elasticsearch -n logging || true
helm uninstall kibana -n logging || true
helm uninstall fluent-bit -n logging || true

# logging 네임스페이스 삭제
kubectl delete namespace logging --ignore-not-found

# PVC, PV 수동 삭제 (남아 있을 경우)
kubectl delete pvc --all -n logging
kubectl delete pv --all

# CRD가 남아 있다면 수동 삭제
kubectl get crd | grep elastic | awk '{print $1}' | xargs -r kubectl delete crd

# 3️⃣ 생성 전 수행해야 할 명령

# Helm 리포지토리 추가 및 업데이트
helm repo add elastic https://helm.elastic.co
helm repo add fluent https://fluent.github.io/helm-charts
helm repo update

# logging 네임스페이스 생성
kubectl create namespace logging

# storageclass 확인 (default가 있는지)
kubectl get storageclass
# 기본 StorageClass가 없거나 다른 걸 사용하고 싶다면 values.yaml에서 PVC 설정을 수정하세요.

# 4️⃣ 생성 명령 (Helm으로 설치)

# 4.1 Elasticsearch 설치
helm install elasticsearch elastic/elasticsearch -n logging \
  --set replicas=1 \
  --set minimumMasterNodes=1 \
  --set volumeClaimTemplate.resources.requests.storage=5Gi \
  --set resources.requests.cpu=100m \
  --set resources.requests.memory=512Mi \
  --set resources.limits.memory=1Gi \
  --set tls.enabled=true \
  --set tls.selfSigned=true \
  --set tls.san="{elasticsearch-master,elasticsearch-master.logging,elasticsearch-master.logging.svc,elasticsearch-master.logging.svc.cluster.local}" \
  --set tls.createCert=true


# 📝 단일 노드 구성을 위한 설정 (k3s 소규모 환경에 맞게 조정)

# 4.2 Kibana 설치

helm list -n logging

# ① 기존 Hook 관련 리소스 삭제
kubectl delete serviceaccount pre-install-kibana-kibana -n logging --ignore-not-found
kubectl delete configmap kibana-kibana-helm-scripts -n logging --ignore-not-found
kubectl delete role pre-install-kibana-kibana -n logging --ignore-not-found
kubectl delete rolebinding pre-install-kibana-kibana -n logging --ignore-not-found
kubectl delete clusterrole pre-install-kibana-kibana --ignore-not-found
kubectl delete clusterrolebinding pre-install-kibana-kibana --ignore-not-found
kubectl delete job post-delete-kibana-kibana -n logging --force --grace-period=0 --ignore-not-found
kubectl delete job -l "app.kubernetes.io/name=kibana" -n logging --ignore-not-found
helm uninstall kibana -n logging

# elastic 비밀번호확인
kubectl get secret elasticsearch-master-credentials -n logging -o jsonpath="{.data.password}" | base64 -d; echo
# 비밀번호 : 79dcssaXr8wwaiCh

# https 
# install을 upgrade로 작업할 경우 --reuse-values(업그레이드 옵션) 사용
# upgrade 하지 않고 재설치할 경우 kibaba 삭제 : 
# kubectl delete configmap kibana-kibana-helm-scripts -n logging
# helm uninstall kibana -n logging

# ② Kibana 관련 리소스 전체 삭제 후 재설치 (권장)
helm install kibana elastic/kibana -n logging \
  --set elasticsearchHosts=https://elasticsearch-master.logging.svc.cluster.local:9200 \
  --set elasticsearchUsername=elastic \
  --set elasticsearchPassword=sD4U3xzZ6BCVLTbP \
  --set service.type=NodePort \
  --set server.publicBaseUrl="http://192.168.164.130:30896" \
  --timeout 10m

  # 기본 ClusterIP로 설치되므로, kubectl port-forward로 접근해야 함


# 4.3 Fluent Bit 설치

helm install fluent-bit fluent/fluent-bit -n logging \
  --set backend.type=es \
  --set backend.es.host=elasticsearch-master.logging.svc.cluster.local \
  --set backend.es.port=9200 \
  --set backend.es.logstash_prefix=fluentbit \
  --set service.type=ClusterIP

# K3s 노드 로그를 수집해서 Elasticsearch로 전송함

# 5️⃣ 생성 후 확인 명령
# 5.1 Elasticsearch 상태 확인

kubectl get pods -n logging -l app=elasticsearch-master

# port-forward 명령이 실패할 경우 (socat 명령어가 없어서 원인일 경우  scoat 설치)
which socat || echo "socat is missing"
sudo apt-get update
sudo apt-get install socat -y
kubectl port-forward svc/elasticsearch-master 9200 -n logging  # 다른터미널 한개 를 열어서 curl http://localhost:9200 접속확인
kubectl port-forward svc/elasticsearch-master 9200 -n logging > /dev/null 2>&1 & # 백그라운드로 실행

curl http://localhost:9200
# 에러일 경우 curl: (52) Empty reply from server
# Elasticsearch 8.x는 기본적으로 보안(HTTPS + 인증)이 활성화되어 있어서 
# HTTPS 프로토콜로 접속해야 함 (https://localhost:9200) 
# Elasticsearch 비밀번호를 확인
kubectl get secret elasticsearch-master-credentials -n logging -o jsonpath='{.data.password}' | base64 -d; echo # 비밀번호 출력
# 비밀번호 : d0eLP2h3d6ZBLDdm
# curl -u elastic:<위에서얻은비밀번호> -k https://localhost:9200
curl -u elastic:79dcssaXr8wwaiCh -k https://localhost:9200
                

kubectl port-forward svc/elasticsearch-master 9200 -n logging > /dev/null 2>&1 &

curl http://localhost:9200
curl: (52) Empty reply from server

kubectl get secret elasticsearch-master-credentials -n logging -o jsonpath='{.data.password}' | base64 -d; echo
9C8WHJcNGG2FGwlq
kubectl port-forward svc/elasticsearch-master 9200:9200 -n logging
curl -u elastic:9C8WHJcNGG2FGwlq -k https://localhost:9200


# 백그라운드작업 중지
jobs -l
kill <프로세스 ID(PID)>

5.2 Kibana 접속 확인

# 포트 사용 프로세스 확인
sudo lsof -i :5601

kubectl get pods -n logging -l app=kibana
kubectl port-forward svc/kibana-kibana 5601 -n logging
kubectl port-forward svc/kibana-kibana 5601 -n logging > /dev/null 2>&1 & # 백그라운드로 실행


# 외부에서 접속
kubectl patch svc kibana-kibana -n logging -p '{"spec": {"type": "NodePort"}}'
kubectl get svc kibana-kibana -n logging
# NAME            TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
# kibana-kibana   NodePort   10.43.152.39   <none>        5601:30365/TCP   61m
# 브라우저에서 접속: http://192.168.164.130:30896

kubectl delete job pre-install-kibana-kibana -n logging
kubectl delete serviceaccount pre-install-kibana-kibana -n logging --ignore-not-found
kubectl delete configmap kibana-kibana-helm-scripts -n logging --ignore-not-found
kubectl delete role pre-install-kibana-kibana -n logging --ignore-not-found
kubectl delete rolebinding pre-install-kibana-kibana -n logging --ignore-not-found
kubectl delete clusterrole pre-install-kibana-kibana --ignore-not-found
kubectl delete clusterrolebinding pre-install-kibana-kibana --ignore-not-found
kubectl delete job post-delete-kibana-kibana -n logging --force --grace-period=0 --ignore-not-found
kubectl delete job -l "app.kubernetes.io/name=kibana" -n logging --ignore-not-found


helm upgrade --install kibana elastic/kibana \
  --namespace logging \
  --set elasticsearchHosts=https://elasticsearch-master.logging.svc.cluster.local:9200 \
  --set elasticsearchUsername=elastic \
  --set elasticsearchPassword=9C8WHJcNGG2FGwlq \
  --set elasticsearch.ssl.verificationMode=none \
  --set service.type=NodePort \
  --set server.publicBaseUrl="http://192.168.164.130:32159" \
  --timeout 60m


helm upgrade --install elasticsearch elastic/elasticsearch \
  --namespace logging \
  --set replicas=1 \
  --set minimumMasterNodes=1 \
  --set tls.enabled=true \
  --set tls.selfSigned=true \
  --set tls.createCert=true \
  --set tls.san="{elasticsearch-master,elasticsearch-master.logging,elasticsearch-master.logging.svc,elasticsearch-master.logging.svc.cluster.local}" \
  --set volumeClaimTemplate.resources.requests.storage=5Gi \
  --set resources.requests.cpu=100m \
  --set resources.requests.memory=512Mi \
  --set resources.limits.memory=1Gi







kubectl get all -n logging 
helm repo update

helm uninstall kibana -n logging || true
helm uninstall elasticsearch -n logging || true
kubectl delete namespace logging --wait || true

kubectl create namespace logging

helm install elasticsearch elastic/elasticsearch \
  --namespace logging \
  --set replicas=1 \
  --set minimumMasterNodes=1 \
  --set tls.enabled=true \
  --set tls.selfSigned=true \
  --set tls.createCert=true \
  --set tls.san[0]=elasticsearch-master \
  --set tls.san[1]=elasticsearch-master.logging \
  --set tls.san[2]=elasticsearch-master.logging.svc \
  --set tls.san[3]=elasticsearch-master.logging.svc.cluster.local \
  --set volumeClaimTemplate.resources.requests.storage=5Gi \
  --set resources.requests.cpu=100m \
  --set resources.requests.memory=512Mi \
  --set resources.limits.memory=1Gi




kubectl get pod -l app=elasticsearch-master -n logging
# ready 상태가 1/1, status가 running 확인 후 kibana설치 진행
# NAME                     READY   STATUS    RESTARTS   AGE
# elasticsearch-master-0   1/1     Running   0          2m


helm upgrade --install kibana elastic/kibana \
  --namespace logging \
  --set elasticsearchHosts=https://elasticsearch-master.logging.svc.cluster.local:9200 \
  --set elasticsearchUsername=elastic \
  --set elasticsearchPassword=$(kubectl get secret elasticsearch-master-credentials -n logging -o jsonpath='{.data.password}' | base64 -d) \
  --set elasticsearch.ssl.verificationMode=none \
  --set elasticsearch.ssl.certificateAuthorities="" \
  --set service.type=NodePort \
  --set service.nodePort=32159 \
  --set server.publicBaseUrl="http://192.168.164.130:32159" \
  --timeout 10m




kubectl get svc -n logging | grep kibana
kubectl get pod -n logging -l app.kubernetes.io/name=kibana






kubectl logs pod/pre-install-kibana-kibana-wrl79   -n logging









# 5.3 Fluent Bit 로그 확인
kubectl get pods -n logging -l app.kubernetes.io/name=fluent-bit
kubectl logs -l app.kubernetes.io/name=fluent-bit -n logging
→ 로그 수집 여부 확인 (POST /_bulk 로그가 있으면 Elasticsearch로 전달 성공)

📊 추가: Kibana Index 설정
Kibana 접속 후 (localhost:5601)

Management > Stack Management > Index Patterns

Create index pattern 클릭

패턴: fluentbit-* 입력

Time field: @timestamp 선택

생성 완료 후 Discover 메뉴에서 로그 확인 가능

🧹 추가 클린업 (선택)
bash
복사
편집
# 전체 로그 시스템 정리
helm uninstall elasticsearch -n logging
helm uninstall kibana -n logging
helm uninstall fluent-bit -n logging
kubectl delete namespace logging
✨ 참고
Elasticsearch는 메모리/스토리지 자원을 많이 사용하므로 최소 자원 환경(K3s)에선 리소스를 줄여야 함

Kibana는 외부 노출 시 ingress 추가 가능

Fluent Bit는 다양한 파서/필터 설정 가능 (values.yaml 활용)

필요하다면 values.yaml 기반의 커스터마이징 버전도 제공해 드릴 수 있어요.
```