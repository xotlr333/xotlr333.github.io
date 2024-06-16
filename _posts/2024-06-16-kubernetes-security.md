---
layout: post
title: 2. 쿠버네티스 보안
date: 2024-06-16
categories: [Kubernetes, 6. 보안을 고려해 파드를 외부에 노출시키기]
tags: [보안]
---

- 미국 사이버 보안 이프라 보안국(CISA)과 국가안전국(NSA)에서 쿠버네티스 보안 강화 가이드를 발표했습니다.
    - 취약점이 있거나 구성이 잘못된 컨테이너 및 파드를 점검할 수 있어야 한다.
    - 최소한의 권한으로 컨테이너와 파드를 실행해야 한다.
    - 네트워크를 분리해 다른 시스템에 영향을 주지 않도록 해야 한다.
    - 방화벽을 사용해 불필요한 네트워크 연결을 차단해야 한다.
    - 강력한 인증 및 권한 부여를 사용해 사용자 및 관리자 접근을 제한해야 한다.
    - 관리자가 모든 활동을 모니터링하면서 잠재적/악의적 활동에 대응할 수 있도록 주기적으로 로그 감시를 해야 한다.
    - 정기적으로 모든 쿠버네티스 설정을 검토하고 취약점 스캔 및 보안 패치가 적용되어 있는지 확인해야 한다.

## 서비스 어카운트(Service Account)

- 쿠버네티스 보안으로 사용할 수 있는 서비스 어카운트에 대해 알아보기 전에 서비스 어카운트의 핵심 원리인 RBAC에 대해서 먼저 살펴보겠습니다.

### RBAC

- **RBAC(Role Based Access Control)**은 **역할을 기반으로 쿠버네티스에 접속할 수 있는 권한을 관리**합니다.
- 즉, 사용자와 역할 두 가지를 조합해 사용자에게 권한을 부여할 수 있습니다.
- 예를 들어 특정 사용자에 대해 쿠버네티스 클러스터에는 접속할 수 없고 특정 파드에만 접속할 수 있게 권한을 설정할 수 있습니다.

![Untitled](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/72789985-40ca-4081-ad48-15bd3a578150)

- **네임스페이스를 기준**으로 사용자 A는 네임스페이스 A에만 접속할 수 있고, 사용자 B는 네임스페이스 B에만 접속할 수 있는 환경도 구성할 수 있습니다.

![Untitled 1](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/fa8bae39-80b7-4f2f-95a8-9970dba4dbe6)

- 역할은 단순 역할과 클러스터 역할로 두 가지 유형이 있습니다.
    - **단순 역할** : 그 역할이 속한 네임스페이스에만 적용되는 역할입니다.
    - **클러스터 역할** : 클러스터 전체에 적용되는 역할입니다.
- 역할은 다음과 같은 방법으로 구현합니다.

![Untitled 2](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/c3c0a0c3-3399-445e-96ea-c753d0f6f72b)

- **쿠버네티스에서 RBAC 기반의 권한을 관리하는 것**이 **서비스 어카운트**입니다.

### 서비스 어카운트

- **서비스 어카운트**는 **클라이언트가 쿠버네티스 API를 호출할 때 사용**합니다.
- 즉, 사람에게 부여하는 것이 아니라 **쿠버네티스 API에 접근하는 시스템에 부여하는 권한**입니다.

- 서비스 어카운트를 이해하려면 역할과 역할 바인딩이라는 개념을 이해해야 합니다.

| 구분 | 역할 | 역할 바인딩 |
| --- | --- | --- |
| 설명 | 어떤 리소스에 어떤 액션을 수행할 수 있는지를 정의함 | 주체(사용자 혹은 그룹)에 역할을 연결하는 것을 의미함 |

![Untitled 3](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/02fc0243-8f69-4d45-871c-e40a01292330)

## [실습] 서비스 어카운트 생성해보기

- 다음과 같은 과정으로 실습을 진행하겠습니다.

![Untitled 4](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/e6b60c9f-8803-41f3-859b-8cf8c9ccc278)

### 1. 네임스페이스 생성하기

- account라는 이름의 네임스페이스를 생성합니다.

```bash
kubectl create namespace account
```

### 2. 서비스 계정 생성하기

- account 네임스페이스에 api-service-account라는 서비스 계정을 만듭니다.

```bash
kubectl create serviceaccount api-service-account -n account
```

- 생성한 서비스 계정으로 클러스터에 접속하려면 클러스터 역할을 정의해야 합니다.
- **클러스터 역할**이란 **서비스 계정이 클러스터의 어떤 리소스(노드, 파드, 볼륨 등)에 접속할 수 있는지를 정의**합니다.

### 3. 서비스 역할 생성하기

- 먼저 api-cluster-role.yaml라는 파일을 생성합니다.

```bash
vi api-cluster-role.yaml
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: api-cluster-role
  namespace: account # 앞에서 정의한 네임스페이스 지정
rules:
  - apiGroups: # 역할이 사용할 API 그룹들
      - ""
      - apps
      - autoscaling
      - batch
      - extensions
      - policy
      - rbac.authorization.k8s.io
    resources: # 클러스터의 어떤 리소스(예, 파드, 볼륨 등)에 접근가능한지 지정
      - pods
      - componentstatuses
      - configmaps
      - daemonsets
      - deployments
      - events
      - endpoints
      - horizontalpodautoscalers
      - ingress
      - jobs
      - limitranges
      - namespaces
      - nodes
      - pods
      - persistentvolumes
      - persistentvolumeclaims
      - resourcequotas
      - replicasets
      - replicationcontrollers
      - serviceaccounts
      - services
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"] # 리소스에 접속해서 어떤 것들을 수행할 수 있을지에 대한 행위(verbs)를 지정

```

- 클러스터 역할을 생성합니다

```bash
kubectl apply -f api-cluster-role.yaml
```

- 참고로 사용 가능한 리소스를 확인하는 명령어는 다음과 같습니다.

```bash
kubectl api-resources
```

### 4. 역할 바인딩 생성하기

- 먼저 api-cluster-role-binding.yaml 파일을 생성합니다.

```bash
vi api-cluster-role-binding.yaml
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: api-cluster-role-binding
subjects: # 주체는 서비스 어카운트입니다
  - namespace: account
    kind: ServiceAccount
    name: api-service-account
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: api-cluster-role # api-cluster-role을 api-service-account에 매핑
```

- 바인딩을 생성합니다.

```bash
kubectl apply -f api-cluster-role-binding.yaml
```

### 5. 바인딩 테스트하기

- 바인딩이 잘 되었는지 테스트 합니다.

- 다음 명령어는 account 네임스페이스의 api-cluster-account 계정으로 파드의 리스트를 보여 주는지를 검증하는 것입니다.

```bash
kubectl auth can-i get pods --as=system:serviceaccount:account:api-service-account
```

- **kubectl auth can-i** : 현재 사용자가 지정된 작업을 수행할 수 있는지를 알고자 할 때 사용합니다.
- **--as=system:serviceaccount:account:api-service-account** : account 네임스페이스에서 api-service-account 계정으로 파드 목록을 볼 수 있는지 확인합니다.

<img width="1110" alt="Untitled" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/fd5098f9-92ff-4b8c-913a-dc28d2fa6365">

- 이번에는 API 호출을 이용해서 테스트해 보겠습니다.
- 즉, HTTP 호출을 하려면 서비스 계정과 함께 서비스 계정과 연결된 토큰이 있어야 합니다.
- 먼저 api-service-account 계정과 연결된 시크릿 이름(secret name)을 가져옵니다.

```bash
kubectl create token api-service-account -n account
```

<img width="909" alt="Untitled 1" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/f775d674-2b1b-4f24-9d8e-c872ca7e29e2">

- 외부에서 접근하기 위해 클러스터 엔드포인트 정보를 가져와야 합니다.
- 다음 명령어로 클러스터 엔드포인트(IP, DNS) 정보를 가져오지만, 궁극적으로는 포트 정보만 사용할 예정입니다.

```bash
kubectl get endpoints | grep kubernetes
```

<img width="727" alt="Untitled 2" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/5a20df14-abeb-4323-b428-891477d91aa3">

### 6. postman을 통해 확인하기

- 이 부분은 실습이 잘 실행되지 않아서 조금 더 공부를 해야겠습니다.




---
참고)  
도서-쉽게 시작하는 쿠버네티스