---
layout: post
title: 1. 쿠버네티스 모니터링
date: 2024-06-23
categories: [Kubernetes, 8. 쿠버네티스의 리소스 관리]
tags: [모니터링, 데이터독, 프로메테우스]
---

- 쿠버네티스에서 많이 사용하는 모니터링 도구는 데이터독(Datadog)과 프로메테우스(Prometheus)가 있습니다.

![Untitled](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/fb0732d1-aa14-48c2-b01a-28ae474df089)

- 데이터독이 설치도 쉽고 메트릭과 그래프 기능이 더 좋지만 SaaS 형태여서, 기업의 데이터가 외부로 유출되는 것에 민감하다면 데이터독은 좋은 솔루션이 아닙니다.

## 데이터독(Datadog)

### 데이터독이 제공하는 기능

1. **대시보드** : 임계치를 설정해 성능에 관한 지표를 수집하고 수집된 내용을 한눈에 파악할 수 있도록 대시보드에 시작적으로 보여줍니다.
2. **시스템** : 데이터독에 등록된 노드를 포함해 기업에서 관리하는 시스템과 네트워크(트래픽 흐름)에 대한 전반적인 내용을 확인할 수 있습니다.
3. **로그 수집** : 시스템 및 애플리케이션에서 발생하는 로그를 수집합니다.

### 데이터독 구조

![Untitled 1](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/ae7131d0-ef27-428f-b270-5f65a81d75d6)


- 모니터링에 필요한 에이전트에는 두 가지 유형이 있습니다.
    - **클러스터 에이전트** : 쿠버네티스 API 서버와 데이터독 에이전트(모든 워커 노드에 배포) 간의 프록시 역할을 하는 에이전트입니다.
    - **데몬셋 에이전트** : 모든 워커 노드에 배포되는 데몬셋 에이전트입니다.

## [실습] 데이터독 사용하기

### 1. 데이터독 회원가입하기

- [https://www.datadoghq.com](https://www.datadoghq.com/) 에서 회원가입을 진행합니다.

### 2. 회원가입 후 해당 OS에 맞는 설치 방법으로 설치하기

- 저는 맥환경에 맞는 설치 방법을 통해서 설치를 진행했습니다.

<img width="460" alt="Untitled" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/dc061685-a3f4-4a35-9909-ac96cc1f2c08">

### 3. 데이터독 접속하기

- 설치 완료 후 us3.datagodhq.com으로 접속합니다.

<img width="703" alt="Untitled 1" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/98a46947-d5c3-498c-8453-5a7ade04e984">

## 프로메테우스(Prometheus)

### 프로메테우스 특징

1. 그라파나를 통한 시각화를 지원합니다.
2. 많은 시스템을 모니터링할 수 있는 다양한 플러그인을 지원합니다.
3. 쿠버네티스의 모니터링 시스템으로 많이 사용됩니다.

- 프로메테우스는 수집된 메트릭 값을 저장하기 위한 볼륨이 필요하며, 관리자에게 알림을 발송해야 하는 상황에서는 알람 매니저를 이용합니다.

![Untitled 2](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/4d881570-c561-4a70-9473-5a405f3332df)

## [실습] 프로메테우스 사용하기

### 1. 프로메테우스와 관련해 네임스페이스 생성하기

```bash
kubectl create ns monitoring
```

### 2. 클러스터 역할 바인딩 생성하기

```bash
vi prometheus-cluster-role.yaml
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
  namespace: monitoring
rules:
  - apiGroups: [""]
    resources: # 클러스터 리소스 정의
      - nodes
      - nodes/proxy
      - services
      - endpoints
      - pods
    verbs: ["get", "list", "watch"]
  - apiGroups:
      - extensions
    resources:
      - ingresses
    verbs: ["get", "list", "watch"]
  - nonResourceURLs: ["/metrics"]
    verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1 # 클러스터 역할 바인딩
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
  - kind: ServiceAccount
    name: default
    namespace: monitoring
```

```bash
# 역할 바인딩 생성
kubectl apply -f prometheus-cluster-role.yaml
```

### 3. 컨피그맵 생성하기

```bash
vi prometheus-config-map.yaml
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-server-conf
  labels:
    name: prometheus-server-conf
  namespace: monitoring
data:
  prometheus.rules:
    |- # 수집한 지표에 대한 알람 조건(예, CPU 90%)을 정의하고, 그 조건에 맞는다면 알람 매니저에 알람을 보냄
    groups:
    - name: container memory alert
      rules:
      - alert: container memory usage rate is very high( > 55%)
        expr: sum(container_memory_working_set_bytes{pod!="", name=""})/ sum (kube_node_status_allocatable_memory_bytes) * 100 > 55
        for: 1m
        labels:
          severity: fatal
        annotations:
          summary: High Memory Usage on
          identifier: ""
          description: " Memory Usage: "
    - name: container CPU alert
      rules:
      - alert: container CPU usage rate is very high( > 10%)
        expr: sum (rate (container_cpu_usage_seconds_total{pod!=""}[1m])) / sum (machine_cpu_cores) * 100 > 10
        for: 1m
        labels:
          severity: fatal
        annotations:
          summary: High Cpu Usage
  prometheus.yml:
    |- # 프로메테우스를 위한 설정파일로 수집할 지표의 종류와 주기 등을 설정
    global:
      scrape_interval: 5s
      evaluation_interval: 5s
    rule_files:
      - /etc/prometheus/prometheus.rules
    alerting:
      alertmanagers:
      - scheme: http
        static_configs:
        - targets:
          - "alertmanager.monitoring.svc:9093"
    scrape_configs:
      - job_name: 'kubernetes-apiservers'
        kubernetes_sd_configs:
        - role: endpoints
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        relabel_configs:
        - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
          action: keep
          regex: default;kubernetes;https
      - job_name: 'kubernetes-nodes'
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        kubernetes_sd_configs:
        - role: node
        relabel_configs:
        - action: labelmap
          regex: __meta_kubernetes_node_label_(.+)
        - target_label: __address__
          replacement: kubernetes.default.svc:443
        - source_labels: [__meta_kubernetes_node_name]
          regex: (.+)
          target_label: __metrics_path__
          replacement: /api/v1/nodes/${1}/proxy/metrics
      - job_name: 'kubernetes-pods'
        kubernetes_sd_configs:
        - role: pod
        relabel_configs:
        - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
          action: keep
          regex: true
        - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
          action: replace
          target_label: __metrics_path__
          regex: (.+)
        - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
          action: replace
          regex: ([^:]+)(?::\d+)?;(\d+)
          replacement: $1:$2
          target_label: __address__
        - action: labelmap
          regex: __meta_kubernetes_pod_label_(.+)
        - source_labels: [__meta_kubernetes_namespace]
          action: replace
          target_label: kubernetes_namespace
        - source_labels: [__meta_kubernetes_pod_name]
          action: replace
          target_label: kubernetes_pod_name
      - job_name: 'kube-state-metrics'
        static_configs:
          - targets: ['kube-state-metrics.kube-system.svc.cluster.local:8080']
      - job_name: 'kubernetes-cadvisor'
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        kubernetes_sd_configs:
        - role: node
        relabel_configs:
        - action: labelmap
          regex: __meta_kubernetes_node_label_(.+)
        - target_label: __address__
          replacement: kubernetes.default.svc:443
        - source_labels: [__meta_kubernetes_node_name]
          regex: (.+)
          target_label: __metrics_path__
          replacement: /api/v1/nodes/${1}/proxy/metrics/cadvisor
      - job_name: 'kubernetes-service-endpoints'
        kubernetes_sd_configs:
        - role: endpoints
        relabel_configs:
        - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
          action: keep
          regex: true
        - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scheme]
          action: replace
          target_label: __scheme__
          regex: (https?)
        - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
          action: replace
          target_label: __metrics_path__
          regex: (.+)
        - source_labels: [__address__, __meta_kubernetes_service_annotation_prometheus_io_port]
          action: replace
          target_label: __address__
          regex: ([^:]+)(?::\d+)?;(\d+)
          replacement: $1:$2
        - action: labelmap
          regex: __meta_kubernetes_service_label_(.+)
        - source_labels: [__meta_kubernetes_namespace]
          action: replace
          target_label: kubernetes_namespace
        - source_labels: [__meta_kubernetes_service_name]
          action: replace
          target_label: kubernetes_name
```

```bash
# 컨피그맵 생성
kubectl apply -f prometheus-config-map.yaml
```

### 4. 프로메테우스를 위한 파드 생성하기

```bash
vi prometheus-deployment.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus-deployment
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus-server
  template:
    metadata:
      labels:
        app: prometheus-server
    spec:
      containers:
        - name: prometheus
          image: prom/prometheus:latest # 프로메테우스 최신 이미지 사용
          args:
            - "--config.file=/etc/prometheus/prometheus.yml"
            - "--storage.tsdb.path=/prometheus/"
          ports:
            - containerPort: 9090
          volumeMounts: # 프로메테우스는 스테이트풀 애플리케이션이므로 저장소를 연결함
            - name: prometheus-config-volume
              mountPath: /etc/prometheus/
            - name: prometheus-storage-volume
              mountPath: /prometheus/
      volumes:
        - name: prometheus-config-volume
          configMap:
            defaultMode: 420
            name: prometheus-server-conf
        - name: prometheus-storage-volume
          emptyDir: {}
```

```bash
# 프로메테우스 생성
kubectl apply -f prometheus-deployment.yaml
```

### 5. 노드 내보내기 생성하기

- 노트 내보내기(node exporter)는 쿠버네티스 노드에 대한 정보를 수집하는 역할을 합니다.
- 즉, 다양한 메트릭에 대한 정보들을 수집하는 에이전트 역할이라고 생각할 수 있습니다.

```bash
vi prometheus-node-exporter.yaml
```

```yaml
apiVersion: apps/v1
kind: DaemonSet # 데몬셋으로  '노드 내보내기' 생성
metadata:
  name: node-exporter
  namespace: monitoring
  labels:
    k8s-app: node-exporter
spec:
  selector:
    matchLabels:
      k8s-app: node-exporter
  template:
    metadata:
      labels:
        k8s-app: node-exporter
    spec:
      containers:
        - image: prom/node-exporter
          name: node-exporter
          ports:
            - containerPort: 9100
              protocol: TCP
              name: http
---
apiVersion: v1 # '노드 내보내기' 파드를 외부에 접속할 수 있도록 서비스를 생성
kind: Service
metadata:
  labels:
    k8s-app: node-exporter
  name: node-exporter
  namespace: kube-system
spec:
  ports:
    - name: http
      port: 9100
      nodePort: 31672
      protocol: TCP
  type: NodePort
  selector:
    k8s-app: node-exporter
```

```bash
# 노드 내보내기 생성
kubectl apply -f prometheus-node-exporter.yaml
```

### 6. 노드포트 생성하기

- 외부에서 프로메테우스 서비스에 접속할 수 있도록 노드포트를 사용합니다.

```bash
vi prometheus-svc.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: prometheus-service
  namespace: monitoring
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "9090"
spec:
  selector:
    app: prometheus-server
  type: NodePort # 외부에서 프로메테우스 파드에 접근하기 위해 노드포트 사용
  ports:
    - port: 8080
      targetPort: 9090
      nodePort: 30001
```

```bash
kubectl apply -f prometheus-svc.yaml
```

### 7. 프로메테우스 접속하기

- 현재는 마스터 노드 1대, 워커 노드 1대로 구성되어 있습니다.
- 따라서 워커 노드 1개에 데몬셋 파드가 배포되었으며, 프로메테우스 용도의 파드가 1개 실행 중입니다.

```bash
# 파드 확인
kubectl get pods -n monitoring
```

<img width="617" alt="Untitled 2" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/58f002f7-745b-4513-a542-09c01a494548">

- minikube와 노드포트 서비스와 바인딩하여 외부에서 접속할 수 있도록 합니다.

<img width="782" alt="Untitled 3" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/6609cef7-6ae0-42f5-ae4d-fdd4c982dd80">

<img width="671" alt="Untitled 4" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/2dabb6fc-c078-4181-95a7-78085ba22ff4">

# 그라파나(Grafana)

- **그라파나**는 **다양한 유형의 데이터베이스를 단일 대시보드에서 시각적으로 보여주는 툴**입니다.
- 그라파나는 다음 그림과 같이 프로메테우스에서 수집한 메트릭을 이용해 해당 데이터를 대시보드에서 시각적으로 보여줍니다.

![Untitled 3](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/a7ccf1c0-f8ec-4b32-b04d-8feef8ba2938)

## [실습] 그라파나 사용하기

### 1. 그라파나 설치하기

- 그라파나 이미지를 가져와서 디플로이먼트를 생성하고 이를 외부로 노출하기 위한 서비스도 함께 생성합니다.

```bash
vi grafana.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      name: grafana
      labels:
        app: grafana
    spec:
      containers:
        - name: grafana
          image: grafana/grafana:latest # 그라파나 최신 버전의 이미지 사용
          ports:
            - name: grafana
              containerPort: 3000
          env: # 로그인하지 않고도 그라파나 대시보드에 접속할 수 있도록 설정
            - name: GF_SERVER_HTTP_PORT
              value: "3000"
            - name: GF_AUTH_BASIC_ENABLED
              value: "false"
            - name: GF_AUTH_ANONYMOUS_ENABLED
              value: "true"
            - name: GF_AUTH_ANONYMOUS_ORG_ROLE
              value: Admin
            - name: GF_SERVER_ROOT_URL
              value: /
---
apiVersion: v1 # 그라파나를 위한 서비스
kind: Service
metadata:
  name: grafana
  namespace: monitoring
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "3000"
spec:
  selector:
    app: grafana
  type: NodePort
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 30004
```

```bash
# 그라파나 생성
kubectl apply -f grafana.yaml
```

### 2. 프로메테우스와 연동하기

- minikube와 그라파나 서비스와 바인딩합니다.

<img width="796" alt="Untitled 5" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/f9538a60-2057-40e8-9385-f600ae1ac285">

<img width="503" alt="Untitled 6" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/800fe6da-aab1-46d6-a951-1d978f65b11f">

- `Add your first data source` 를 클릭합니다.

<img width="752" alt="Untitled 7" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/ff9ad655-6000-491e-9cc6-53f68164b771">

- `Prometheus` 를 클릭합니다.

<img width="548" alt="Untitled 8" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/e285e0fd-c91a-4b9a-874a-4494b63522cc">

- HTTP URL에 prometheus-service의 IP를 입력하고 포트는 8080을 사용합니다.

<img width="566" alt="Untitled 9" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/aaaef6f8-658d-400e-8eea-1994db1e0eb3">

<img width="641" alt="Untitled 10" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/eaf44fbf-55d1-4513-83df-6d2fb36f8e9a">

- 마지막으로 `Save & test` 을 클릭합니다.

<img width="876" alt="Untitled 11" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/b93a6a94-c6f7-4158-9eb4-831ede61139a">

### 3. 대시보드 생성하기

- [https://grafana.com/grafana/dashboards/](https://grafana.com/grafana/dashboards/) 로 이동하면 다양한 유형의 모니터링 템플릿을 확인할 수 있습니다.
- kubernetes cluster를 검색하여 `Kubernetes Cluster(Prometheus)`를 클릭합니다.

<img width="838" alt="Untitled 12" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/1be281ec-fa06-4fe2-b30d-5fae281e1294">

- 우측에 `Copy ID to Clipboard`를 클릭해 ID를 복사합니다.

<img width="383" alt="Untitled 13" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/ed66360c-65ba-476c-99f6-82659c35c6cf">

- `Dashboards` 메뉴를 클릭한 후 `New` 를 클릭하여 `Import`를 클릭합니다.

<img width="277" alt="Untitled 14" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/952ba0d4-ade7-433b-a82f-723af05fc27e">

<img width="180" alt="Untitled 15" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/23a47e76-6faf-4052-a542-d3922d71c0b6">

- 해당 부분에 복사해둔 ID를 입력한 후 `Load` 를 클릭합니다.

<img width="625" alt="Untitled 16" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/71d06740-a688-4fb5-9991-d3be5385a273">

- prometheus 부분을 선택하여 `prometheus`를 클릭한 후 `import`를 클릭합니다.

<img width="632" alt="Untitled 17" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/19d6e41d-4814-40ad-b736-af00b9c68bc5">

- 모니터링 대시보드가 구성됩니다.

<img width="881" alt="Untitled 18" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/93f6b05f-60ea-4137-9033-2de03cf0d5fb">






---
참고)  
도서-쉽게 시작하는 쿠버네티스