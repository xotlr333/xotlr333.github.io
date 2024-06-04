---
layout: post
title: 4. 레플리카셋 사용하기
date: 2024-06-04
categories: [Kubernetes, 4.쿠버네티스 사용하기]
tags: [레플리카셋]
---

- **레플리카셋**은 **일정한 개수의 동일한 파드가 항상 실행되도록 관리**합니다.

![Untitled](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/6931dfa6-7d03-4383-a48c-62a55e83c2de)

## [실습] 레플리카셋 사용하기

### 1. 매니페스트 생성하기

```bash
vi replicaset.yaml
```

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: 3-replicaset # 레플리카 이름
spec:
  template:
    metadata:
      name: 3-replicaset
      labels:
        app: 3-replicaset
    spec:
      containers:
        - name: 3-replicaset
          image: nginx
          ports:
            - containerPort: 80
  replicas: 3 # 3개의 파드 생성
  selector:
    matchLabels:
      app: 3-replicaset
```

### 2. 레플리카셋 생성하기

```bash
kubectl apply -f replicaset.yaml
```

### 3. 레플리카셋과 파드 상태 확인하기

- 여러 정보를 한 번에 보고 싶다면 다음과 같이 ,를 이용합니다.

```bash
kubectl get replicaset,pods
```

<img width="618" alt="Untitled" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/c31d1e66-6ba2-42ec-baf1-0646ec255f71">

- 3개의 파드가 생성되어있고, 모두 running 상태인 것을 확인할 수 있습니다.

### 4. 파드 하나 삭제하기

- ‘3-replicaset-8sdzx’ 인 파드를 삭제해 보겠습니다.

```bash
kubectl delete pod 3-replicaset-8sdzx
```

### 5. 레플리카셋과 파드 상태 다시 확인하기

```bash
kubectl get replicaset,pods
```

<img width="618" alt="Untitled 1" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/96c4ed7f-be74-4990-9994-a98b95df38ca">

- 파드 하나를 삭제했지만 여전히 3개의 파드가 떠 있는 것을 확인할 수 있습니다.
- 이처럼 레플리카겟은 파드가 삭제되어도 지정된 숫자만큼 파드의 개수를 유지하는 역할을 합니다.

### 6. 파드의 개수 조정하기

- 파드의 개수를 조정하는 것은 kubectl scale 명령어를 이용합니다.

```bash
kubectl scale replicaset/3-replicaset --replicas=5 # --replicas로 파드 개수 조정
```

### 7. 파드 개수 확인하기

<img width="625" alt="Untitled 2" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/f71084bd-e507-4c2f-bc9e-342b11c88dc8">

- 5개의 파드가 확인되었습니다.

### 8. 레플리카셋만 삭제하기

- 파드는 유지하고 레플리카셋만 삭제하기 위해서는 옵션 --cascade=orphan 을 이용합니다.

```bash
kubectl delete -f replicaset.yaml --cascade=orphan
```

<img width="823" alt="Untitled 3" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/c8b4d7c7-0c51-4bbf-98c0-9897ba282a6b">





---
참고)  
도서-쉽게 시작하는 쿠버네티스