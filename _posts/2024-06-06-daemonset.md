---
layout: post
title: 5. 데몬셋 사용하기
date: 2024-06-06
categories: [Kubernetes, 4.쿠버네티스 사용하기]
tags: [데몬셋]
---

- **데몬셋(DaemonSet)**은 **모든 노드에 파드를 실행할 때** 사용합니다.
- 데몬셋은 모든 노드에 필요한 모니터링 용도로 많이 사용됩니다.

# [실습] 데몬셋 사용하기

### 1. 데몬셋 yaml 파일 생성하기

```bash
vi daemonsets.yaml
```

- 프로메테우스라는 모니터링 용도의 이미지를 이용해 파드를 배포할 것입니다.

```yaml
apiVersion: apps/v1
kind: DaemonSet # 데몬셋 배포
metadata:
  name: prometheus-daemonset
spec:
  selector:
    matchLabels:
      tier: monitoring # 사용자 정의 레이블로 모니터링 용도로 사용될 것임을 지정
      name: prometheus-exporter
  template:
    metadata:
      labels:
        tier: monitoring # 모니터링 용도
        name: prometheus-exporter
    spec:
      containers:
        - name: prometheus
          image: prom/node-exporter
          ports:
            - containerPort: 80

```

### 2. 데몬셋 실행하기

```yaml
kubectl apply -f daemonsets.yaml
```

### 3. 데몬셋과 파드 확인하기

- 파드의 상세 정보는 두 가지 방법으로 확인할 수 있습니다.
- **kubectl get pod -o wide** : 모든 리소스에 대한 상세 정보를 보여 줍니다.
- **kubectl describe 파드명/디플로이먼트명** : 파드명/디플로이먼트명으로 지정된 오브젝트에 대한 상세 정보를 보여줍니다.

```bash
kubectl describe daemonset/prometheus-daemonset
```

<img width="890" alt="Untitled" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/84acaf2a-56c3-41e0-bdeb-2e5b71ca6c27">

```bash
kubectl get pods -o wide
```

<img width="1132" alt="Untitled 1" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/746dd6c4-d982-4035-b4bc-0c4b498c13b9">

### 4. 데몬셋 삭제하기

```bash
kubectl delete -f daemonsets.yaml
```

- 데몬셋뿐만 아니라 파드도 함께 삭제됩니다.





---
참고)  
도서-쉽게 시작하는 쿠버네티스