---
layout: post
title: 1. 파드를 외부에 노출시키기
date: 2024-06-13
categories: [Kubernetes, 6. 보안을 고려해 파드를 외부에 노출시키기]
tags: [외부 노출]
---

- 서비스는 파드가 어떤 IP를 갖건 내부의 다른 파드나 외부의 사용자(혹은 애플리케이션)가 접근할 수 있게 해줍니다.
- 서비스는 클러스터 내부에서 접속하는 것과 외부에서 접속하는 방법에 따라 나뉩니다.

![Untitled](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/bdeb5488-d539-4119-b022-92a632f385ea)

- 노트포트, 로드밸런서, 인그레스는 외부에서 내부에 접근할 때 사용하는 것이고, ExternalName은 내부에서 외부에 접근할 때 사용합니다.

![Untitled 1](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/bf7106ae-df14-4ce6-9e74-b2d871436aaa)

## ClusterIP

- **ClusterIP**는 기본 서비스 타입으로, **클러스터 내 파드에 접근할 수 있는 가상의 IP**입니다.
- ClusterIP는 클러스터 내부에서만 사용할 수 있으며 외부에서는 파드에 접속할 수 없습니다.

### [실습] ClusterIP 타입의 서비스 사용하기

#### 1. ClusterIP yaml 파일 생성하기

```bash
vi ClusterIP.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: clusterip-nginx
spec:
  selector:
    matchLabels:
      run: clusterip-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: clusterip-nginx
    spec:
      containers:
        - name: clusterip-nginx
          image: nginx
          ports:
            - containerPort: 80
```

- 매니페스트에서는 서비스 타입을 별도로 지정하지 않았습니다.
- 이처럼 서비스 유형을 지정하지 않으면 기본으로 ClusterIP를 사용하겠다는 것을 의미합니다.

#### 2. 파드 생성하기

```bash
kubectl apply -f ClusterIP.yaml
```

#### 3. 파드 상태 확인하기

```bash
kubectl get pods -l run=clusterip-nginx -o wide
```

- **-l run=clusterip-nginx** : clusterip-nginx라는 이름 갖는 파드들의 정보를 보여 줍니다.
- **-o wide** : 파드의 상세 정보를 확인할 때 사용하며  --output=wide도 동일한 의미를 갖습니다.

<img width="1187" alt="Untitled" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/0d58b550-679d-4911-9d27-02f53d197a69">

#### 4. 서비스 생성하기

- 서비스 생서하는 방법은 두 가지입니다.
    - 매니페스트를 이용해 생성하는 방법(yaml 파일 이용)
    - kubectl expose 명령어를 사용하는 방법

- 다음 명령어로 다음 생성한 clusterip-nginx 파드를 외부에 노출시킬 서비스를 생성합니다.

```bash
kubectl expose deployment/clusterip-nginx
```

#### 5. 서비스 상태 확인하기

```bash
kubectl get svc clusterip-nginx
```

<img width="688" alt="Untitled 1" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/04575643-ec29-47bc-a830-3e838d515d43">

```bash
# 서비스 상세 정보 확인
kubectl describe svc clusterip-nginx
```

<img width="697" alt="Untitled 2" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/723ba82a-9144-4fdf-a80c-9cf25c0ecabe">

- 생성된 clusterip-nginx 서비스의 CLUSTERIP가 10.98.8.9임을 확인했습니다.

#### 6. 파드의 엔드포인트 확인하기

- **엔드포인트란**?
    - 쿠버네티스에서 서비스와 파드의 통신은 다음과 같이 서비스의 셀렉터와 파드의 레이블을 이용합니다.
    - 만약 파드의 레이블이 서비스의 셀렉터에서 사용한 이름과 같으면 서비스는 해당 파드로 트래픽을 보냅니다.
    - 이것이 가능한 이유는 서비스가 엔드포인트와 파드들을 매핑해 관리하기 때문입니다.
    - 서비스는 엔드포인트에 매핑된 파드들의 IP 정보를 파드에 전달합니다.

- 파드의 엔드포인트(endpoint)를 확인할 때는 kubectl get ep 명령어를 사용합니다.

```bash
kubectl get ep clusterip-nginx
```

<img width="652" alt="Untitled 3" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/2b160f2b-4eea-473e-a2dd-47eb9ed7bb7d">

#### 7. busybox 이미지를 이용한 파드 생성 후 ClusterIP 이용해 콘텐츠 가져오기

```bash
# busybox 파드 생성 후 접속
kubectl run busybox --rm -it --image=busybox /bin/sh

# index.html 콘텐츠 가져오기
wget 10.98.8.9

# index.html 내용 확인
cat index.html

# 파드 빠져나오기
exit
```

<img width="944" alt="Untitled 4" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/e5719b94-b00f-4b20-838a-6c4881e2078b">

## ExternalName

- ExternalName은 클러스터 내부에서 외부의 엔드포인트에 접속하기 위한 서비스입니다.
- 이때 엔드포인트는 클러스터 외부에 위치하는 데이터베이스나 애플리케이션의 API 등을 의미합니다.

![Untitled 2](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/6bc7e215-a1e0-4b29-8406-00d86b31ee1c)

- **external-service**는 **CANME**을 의미합니다.
- **myservice.test.com**은 **외부FQDN**을 의미합니다. 예를 들어, [www.a.com](http://www.a.com) 혹은 [web01.a.com](http://web01.a.com) 같은 형태가 FQDN입니다.
- 즉, **external-service라는 이름으로 요청이 들어온다면 클러스터 외부에 존재하는 myservice.test.com이라는 주소를 반환하는 것**이 ExternalName 서비스입니다.
- 따라서 ExternalName 서비스는 프록시나 특정 이름(혹은 주소)을 다른 이름으로 자동을 변환하는 리다이렉트 용도로 사용합니다.

![Untitled 3](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/99a97841-30b8-4e0b-bcd2-84d24643959a)

## 노드포트

- 노드포트 타입의 서비스는 **외부 사용자(혹은 애플리케이션)가 클러스터 내부에 있는 파드에 접속할 때 사용**하는 서비스 중 하나입니다.
- 예를 들어, 특정 포트(노드포트)로 3001을 개발하면 노드포트 서비스가 이 포트로 들어오는 모든 요청은 해당 업무를 처리할 수 있는 파드로 전달합니다.

![Untitled 4](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/54925979-c63f-4464-943f-126092549b28)

- 어떤 서비스의 어떤 파드에 접속할지는 노트포트 서비스를 생성할 때 정의하는 매니페스트에 정의할 수 있습니다.

![Untitled 5](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/fb7a917e-d1a8-4a6d-8d09-077e1e920b8e)

### [실습] nginx를 이용해 노드포트 사용하기

#### 1. nginx-deployment yaml 파일 생성하기

```bash
vi nginx-deployment.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
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
          image: nginx:latest
          ports:
            - containerPort: 80
```

#### 2. 디플로이먼트 생성하기

```bash
kubectl apply -f nginx-deployment.yaml
```

#### 3. 디플로이먼트 및 파드 상태 확인하기

```bash
# 디플로이먼트 상태 확인
kubectl get deployments
```

<img width="601" alt="Untitled 5" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/683f9a03-048f-4f5a-ab76-f45e8316b7c8">

```bash
# 파드 상태 확인
kubectl get pod
```

<img width="671" alt="Untitled 6" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/e2a1619d-c999-4574-843f-88c0c8cac33a">

#### 4. 파드의 외부 노출을 위한 서비스 생성하기

```bash
# 서비스 yaml 생성
vi nginx-svc.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  type: NodePort # 외부에서 접속할 때 노드포트 서비스 사용
  ports:
    - port: 8080 # 노드포트 서비스에서 사용하는 포트
      targetPort: 80 # 파드(컨테이너)에서 사용하는 포트
      protocol: TCP
      name: http
  selector:
    app: nginx # nginx라는 이름의 레이블을 갖는 파드를 서비스
```

```yaml
# 파드 생성
kubectl apply -f nginx-svc.yaml
```

- 서비스와 디플로이먼트 관계를 정리하면 다음과 같습니다.

![Untitled 6](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/1f2e90eb-09e3-46ba-8bf3-0d39d4a348e8)

#### 5. 외부에서 접속하기

- 먼저 접속하기 위해 서비스 포트를 확인합니다.

```bash
kubectl get svc | grep my-nginx
```

<img width="761" alt="Untitled 7" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/6fa55e68-23ae-4d32-845c-bddf3e600643">

- 현재 minkube를 사용하기 있기에 minikube와 호스트 IP와 바인딩을 해줍니다.

```bash
minikube service clusterip-nginx --url
```

<img width="848" alt="Untitled 8" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/ee78e280-3f13-4adc-acfd-43ae46c73c07">

- 브라우저로 127.0.0.1:63463로 접속합니다.

<img width="583" alt="Untitled 9" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/4221f268-55a1-4a88-88bc-6706323cf75e">

## 로드밸런서

- **로드밸런서(LoadBalancer)**는 **서버에 가해지는 부하를 분산해주는 장치 또는 기술**을 통칭합니다.

- 로드 밸런서는 크게 두 가지 유형으로 분류할 수 있습니다.
    - **L4 로드밸런서** : OSI 7계층에서 **네트워크 계층이나 트랜스포트 계층의 정보를 바탕**으로 로드를 분산합니다. 따라서 **IP 주소나 포트 번호 등을 이용해 트래픽을 분산**합니다.
    - **L7 로드밸런서** : OSI 7계층에서 **애플리케이션 계층을 기반으로 로드를 분산하기 때문에 URL, HTTP 헤더, 쿠키 등과 같은 사용자의 요청을 기준으로 트래픽을 분산**합니다.

### [실습] L4 로드밸런서 사용하기

- L4 로드밸런서는 외부에서 접근할 수 있는 IP를 이용해 로드밸런서를 구성합니다.
- 즉, 외부 IP(공인 IP)를 로드밸런서로 설정하면 클러스터 외부에서 접근할 수 있습니다.

#### 1. httpd 이미지를 이용해 디플로이먼트 생성하기

- 지금까지 yaml 파일을 이용해 디플로이먼트를 생성했지만 다음과 같이 이미지를 이용해서 간단하게 디플로이먼트를 생성할 수도 있습니다.

```bash
kubectl create deployment httpd --image=httpd
```

#### 2. httpd 외부 노출을 위한 서비스 생성하기

- externalIPs에 외부 IP를 입력합니다.

```bash
cat << EOF > httpd-service.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: httpd-service
spec:
  selector:
    app: httpd
  ports:
    - name: http
      port: 80
      targetPort: 80
      protocol: TCP
  externalIPs:
    - 35.89.25.212
EOF # 편집을 끝내는 명령어입니다
```

```yaml
# 서비스 생성
kubectl create -f httpd-service.yaml
```

#### 3. 서비스 상태 확인하기

```yaml
kubectl get svc
```

<img width="757" alt="Untitled 10" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/4222ea2a-df39-4a13-a49a-c6f069ccbdb7">

- 설정한 대로 외부 IP가 생성된 것을 확인할 수 있습니다.

- minikube랑 외부 IP랑 바인딩한 후 접속을 확인합니다.

```bash
# 바인딩
mibikube service httpd-service --url
```

<img width="852" alt="Untitled 11" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/77f1b749-4236-4f6b-9ebf-af846e248662">

```bash
# 바인딩 후 새로운 터미널 열어서 접속 확인
curl -i 127.0.0.1:63615
```

<img width="449" alt="Untitled 12" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/47761693-a893-4a8f-a636-e9e66cec9b2f">

#### 4. 로드밸런서 yaml 파일 생성하기

- 보통 기업에서는 서비스별로 폴더를 만들어서 관리합니다.
- 그래서 이번에는 service 폴더를 생성 후 진행하겠습니다.

```bash
# service 폴더 생성
mkdir service
# service 폴더로 이동
cd service
# 로드밸런서 yaml 파일 생성
vi load-balancer-example.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/name: load-balancer-example
  name: hello-world
spec:
  replicas: 5 # 5개의 파드를 생성하도록 지정
  selector:
    matchLabels:
      app.kubernetes.io/name: load-balancer-example
  template:
    metadata:
      labels:
        app.kubernetes.io/name: load-balancer-example
    spec:
      containers:
        - image: gcr.io/google-samples/node-hello:1.0
          name: hello-world
          ports:
            - containerPort: 8080 # 컨테이너(파드)에서 8080 포트를 사용하도록 지정
```

#### 5. 로드밸런서 디플로이먼트 생성 및 상태 확인하기

```bash
# 디플로이먼트 생성
kubectl apply -f load-balancer-example.yaml

# 상태 확인
kubectl get deployments hello-world
```

<img width="656" alt="Untitled 13" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/731c1af4-a538-42f8-a4cb-2f7f343f99c5">

#### 6. hello-world 디플로이먼트를 외부에 노출시키기 위한 서비스 생성하기

```bash
# 서비스 생성
kubectl expose deployment hello-world --type=LoadBalancer --name=exservice
```

#### 7. 서비스 정보 확인하기

```bash
# 서비스 상태 확인
kubectl get service exservice
```

<img width="642" alt="Untitled 14" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/de1047a7-0bac-44fb-aacc-dd5218674c87">

- 여기서 30941 포트를 사용하는 것을 확인할 수 있습니다.

- 더 자세한 정보를 확인합니다.

```bash
kubectl describe service exservice
```

<img width="748" alt="Untitled 15" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/31d3b6f5-97b6-401a-bf53-825b9cd2fef7">

#### 8. 로드밸런서로 외부 접속 확인하기

- minkube랑 호스트랑 바인딩을 합니다.

```bash
minikube service exservice --url
```

<img width="847" alt="Untitled 16" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/e9c077cd-5f19-4b8c-9f2f-e409e59f700f">

- 127.0.0.1:63785 를 통해 외부 접속을 확인합니다.

```bash
curl http://127.0.0.1:63785
```

<img width="601" alt="Untitled 17" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/dc09b485-46b5-45dd-8ffb-88b7d724e814">

### [실습] L7 로드밸런서 사용하기(인그레이스)

- **인그레스(Ingresss)**는 **클러스터 외부에서 내부로 접근하는 요청(HTTP, HTTPS 요청)을 어떻게 처리할지 정의해 둔 규칙들의 모임**입니다.
- 즉, 인그레스는 어떤 URL 경로로 요청이 왔을 때 어떤 서비스로 연결하라는 ‘규칙’을 정의한 것이 전부입니다.
- 실제로 **인그레스 규칙에 맞게 경로를 찾는 행위를 하는 것**은 **인그레스 컨트롤러**입니다.

#### 1. nginx 인그레스 컨트롤러 설치하기

```bash
# nginx 인그레스 컨트롤러를 위한 파일 생성
vi ingress-nginx-controller.yaml
```

- 인그레스 컨트롤러 yaml 파일

    ```yaml
    apiVersion: v1
    kind: Namespace
    metadata:
      name: ingress-nginx
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
    
    ---
    # Source: ingress-nginx/templates/controller-serviceaccount.yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      labels:
        helm.sh/chart: ingress-nginx-4.0.6
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/version: 1.0.4
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/component: controller
      name: ingress-nginx
      namespace: ingress-nginx
    automountServiceAccountToken: true
    ---
    # Source: ingress-nginx/templates/controller-configmap.yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      labels:
        helm.sh/chart: ingress-nginx-4.0.6
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/version: 1.0.4
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/component: controller
      name: ingress-nginx-controller
      namespace: ingress-nginx
    data:
      allow-snippet-annotations: "true"
      proxy-connect-timeout: "10s"
      proxy-read-timeout: "10s"
      client-max-body-size: "2m"
    ---
    # Source: ingress-nginx/templates/clusterrole.yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      labels:
        helm.sh/chart: ingress-nginx-4.0.6
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/version: 1.0.4
        app.kubernetes.io/managed-by: Helm
      name: ingress-nginx
    rules:
      - apiGroups:
          - ""
        resources:
          - configmaps
          - endpoints
          - nodes
          - pods
          - secrets
        verbs:
          - list
          - watch
      - apiGroups:
          - ""
        resources:
          - nodes
        verbs:
          - get
      - apiGroups:
          - ""
        resources:
          - services
        verbs:
          - get
          - list
          - watch
      - apiGroups:
          - networking.k8s.io
        resources:
          - ingresses
        verbs:
          - get
          - list
          - watch
      - apiGroups:
          - ""
        resources:
          - events
        verbs:
          - create
          - patch
      - apiGroups:
          - networking.k8s.io
        resources:
          - ingresses/status
        verbs:
          - update
      - apiGroups:
          - networking.k8s.io
        resources:
          - ingressclasses
        verbs:
          - get
          - list
          - watch
    ---
    # Source: ingress-nginx/templates/clusterrolebinding.yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      labels:
        helm.sh/chart: ingress-nginx-4.0.6
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/version: 1.0.4
        app.kubernetes.io/managed-by: Helm
      name: ingress-nginx
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: ingress-nginx
    subjects:
      - kind: ServiceAccount
        name: ingress-nginx
        namespace: ingress-nginx
    ---
    # Source: ingress-nginx/templates/controller-role.yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      labels:
        helm.sh/chart: ingress-nginx-4.0.6
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/version: 1.0.4
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/component: controller
      name: ingress-nginx
      namespace: ingress-nginx
    rules:
      - apiGroups:
          - ""
        resources:
          - namespaces
        verbs:
          - get
      - apiGroups:
          - ""
        resources:
          - configmaps
          - pods
          - secrets
          - endpoints
        verbs:
          - get
          - list
          - watch
      - apiGroups:
          - ""
        resources:
          - services
        verbs:
          - get
          - list
          - watch
      - apiGroups:
          - networking.k8s.io
        resources:
          - ingresses
        verbs:
          - get
          - list
          - watch
      - apiGroups:
          - networking.k8s.io
        resources:
          - ingresses/status
        verbs:
          - update
      - apiGroups:
          - networking.k8s.io
        resources:
          - ingressclasses
        verbs:
          - get
          - list
          - watch
      - apiGroups:
          - ""
        resources:
          - configmaps
        resourceNames:
          - ingress-controller-leader
        verbs:
          - get
          - update
      - apiGroups:
          - ""
        resources:
          - configmaps
        verbs:
          - create
      - apiGroups:
          - ""
        resources:
          - events
        verbs:
          - create
          - patch
    ---
    # Source: ingress-nginx/templates/controller-rolebinding.yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      labels:
        helm.sh/chart: ingress-nginx-4.0.6
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/version: 1.0.4
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/component: controller
      name: ingress-nginx
      namespace: ingress-nginx
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: Role
      name: ingress-nginx
    subjects:
      - kind: ServiceAccount
        name: ingress-nginx
        namespace: ingress-nginx
    ---
    # Source: ingress-nginx/templates/controller-service-webhook.yaml
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        helm.sh/chart: ingress-nginx-4.0.6
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/version: 1.0.4
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/component: controller
      name: ingress-nginx-controller-admission
      namespace: ingress-nginx
    spec:
      type: ClusterIP
      ports:
        - name: https-webhook
          port: 443
          targetPort: webhook
          appProtocol: https
      selector:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/component: controller
    ---
    # Source: ingress-nginx/templates/controller-service.yaml
    apiVersion: v1
    kind: Service
    metadata:
      annotations:
      labels:
        helm.sh/chart: ingress-nginx-4.0.6
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/version: 1.0.4
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/component: controller
      name: ingress-nginx-controller
      namespace: ingress-nginx
    spec:
      type: NodePort
      externalTrafficPolicy: Local
      ipFamilyPolicy: SingleStack
      ipFamilies:
        - IPv4
      ports:
        - name: http
          port: 80
          protocol: TCP
          targetPort: http
          appProtocol: http
        - name: https
          port: 443
          protocol: TCP
          targetPort: https
          appProtocol: https
      selector:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/component: controller
    ---
    # Source: ingress-nginx/templates/controller-deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        helm.sh/chart: ingress-nginx-4.0.6
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/version: 1.0.4
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/component: controller
      name: ingress-nginx-controller
      namespace: ingress-nginx
    spec:
      selector:
        matchLabels:
          app.kubernetes.io/name: ingress-nginx
          app.kubernetes.io/instance: ingress-nginx
          app.kubernetes.io/component: controller
      revisionHistoryLimit: 10
      minReadySeconds: 0
      template:
        metadata:
          labels:
            app.kubernetes.io/name: ingress-nginx
            app.kubernetes.io/instance: ingress-nginx
            app.kubernetes.io/component: controller
        spec:
          dnsPolicy: ClusterFirst
          containers:
            - name: controller
              image: k8s.gcr.io/ingress-nginx/controller:v1.0.4@sha256:545cff00370f28363dad31e3b59a94ba377854d3a11f18988f5f9e56841ef9ef
              imagePullPolicy: IfNotPresent
              lifecycle:
                preStop:
                  exec:
                    command:
                      - /wait-shutdown
              args:
                - /nginx-ingress-controller
                - --election-id=ingress-controller-leader
                - --controller-class=k8s.io/ingress-nginx
                - --configmap=$(POD_NAMESPACE)/ingress-nginx-controller
                - --validating-webhook=:8443
                - --validating-webhook-certificate=/usr/local/certificates/cert
                - --validating-webhook-key=/usr/local/certificates/key
              securityContext:
                capabilities:
                  drop:
                    - ALL
                  add:
                    - NET_BIND_SERVICE
                runAsUser: 101
                allowPrivilegeEscalation: true
              env:
                - name: POD_NAME
                  valueFrom:
                    fieldRef:
                      fieldPath: metadata.name
                - name: POD_NAMESPACE
                  valueFrom:
                    fieldRef:
                      fieldPath: metadata.namespace
                - name: LD_PRELOAD
                  value: /usr/local/lib/libmimalloc.so
              livenessProbe:
                failureThreshold: 5
                httpGet:
                  path: /healthz
                  port: 10254
                  scheme: HTTP
                initialDelaySeconds: 10
                periodSeconds: 10
                successThreshold: 1
                timeoutSeconds: 1
              readinessProbe:
                failureThreshold: 3
                httpGet:
                  path: /healthz
                  port: 10254
                  scheme: HTTP
                initialDelaySeconds: 10
                periodSeconds: 10
                successThreshold: 1
                timeoutSeconds: 1
              ports:
                - name: http
                  containerPort: 80
                  protocol: TCP
                - name: https
                  containerPort: 443
                  protocol: TCP
                - name: webhook
                  containerPort: 8443
                  protocol: TCP
              volumeMounts:
                - name: webhook-cert
                  mountPath: /usr/local/certificates/
                  readOnly: true
              resources:
                requests:
                  cpu: 100m
                  memory: 90Mi
          nodeSelector:
            kubernetes.io/os: linux
          serviceAccountName: ingress-nginx
          terminationGracePeriodSeconds: 300
          volumes:
            - name: webhook-cert
              secret:
                secretName: ingress-nginx-admission
    ---
    # Source: ingress-nginx/templates/controller-ingressclass.yaml
    # We don't support namespaced ingressClass yet
    # So a ClusterRole and a ClusterRoleBinding is required
    apiVersion: networking.k8s.io/v1
    kind: IngressClass
    metadata:
      labels:
        helm.sh/chart: ingress-nginx-4.0.6
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/version: 1.0.4
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/component: controller
      name: nginx
      namespace: ingress-nginx
    spec:
      controller: k8s.io/ingress-nginx
    ---
    # Source: ingress-nginx/templates/admission-webhooks/validating-webhook.yaml
    # before changing this value, check the required kubernetes version
    # https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#prerequisites
    apiVersion: admissionregistration.k8s.io/v1
    kind: ValidatingWebhookConfiguration
    metadata:
      labels:
        helm.sh/chart: ingress-nginx-4.0.6
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/version: 1.0.4
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/component: admission-webhook
      name: ingress-nginx-admission
    webhooks:
      - name: validate.nginx.ingress.kubernetes.io
        matchPolicy: Equivalent
        rules:
          - apiGroups:
              - networking.k8s.io
            apiVersions:
              - v1
            operations:
              - CREATE
              - UPDATE
            resources:
              - ingresses
        failurePolicy: Fail
        sideEffects: None
        admissionReviewVersions:
          - v1
        clientConfig:
          service:
            namespace: ingress-nginx
            name: ingress-nginx-controller-admission
            path: /networking/v1/ingresses
    ---
    # Source: ingress-nginx/templates/admission-webhooks/job-patch/serviceaccount.yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: ingress-nginx-admission
      namespace: ingress-nginx
      annotations:
        helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
        helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
      labels:
        helm.sh/chart: ingress-nginx-4.0.6
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/version: 1.0.4
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/component: admission-webhook
    ---
    # Source: ingress-nginx/templates/admission-webhooks/job-patch/clusterrole.yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: ingress-nginx-admission
      annotations:
        helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
        helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
      labels:
        helm.sh/chart: ingress-nginx-4.0.6
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/version: 1.0.4
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/component: admission-webhook
    rules:
      - apiGroups:
          - admissionregistration.k8s.io
        resources:
          - validatingwebhookconfigurations
        verbs:
          - get
          - update
    ---
    # Source: ingress-nginx/templates/admission-webhooks/job-patch/clusterrolebinding.yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: ingress-nginx-admission
      annotations:
        helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
        helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
      labels:
        helm.sh/chart: ingress-nginx-4.0.6
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/version: 1.0.4
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/component: admission-webhook
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: ingress-nginx-admission
    subjects:
      - kind: ServiceAccount
        name: ingress-nginx-admission
        namespace: ingress-nginx
    ---
    # Source: ingress-nginx/templates/admission-webhooks/job-patch/role.yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: ingress-nginx-admission
      namespace: ingress-nginx
      annotations:
        helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
        helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
      labels:
        helm.sh/chart: ingress-nginx-4.0.6
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/version: 1.0.4
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/component: admission-webhook
    rules:
      - apiGroups:
          - ""
        resources:
          - secrets
        verbs:
          - get
          - create
    ---
    # Source: ingress-nginx/templates/admission-webhooks/job-patch/rolebinding.yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: ingress-nginx-admission
      namespace: ingress-nginx
      annotations:
        helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
        helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
      labels:
        helm.sh/chart: ingress-nginx-4.0.6
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/version: 1.0.4
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/component: admission-webhook
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: Role
      name: ingress-nginx-admission
    subjects:
      - kind: ServiceAccount
        name: ingress-nginx-admission
        namespace: ingress-nginx
    ---
    # Source: ingress-nginx/templates/admission-webhooks/job-patch/job-createSecret.yaml
    apiVersion: batch/v1
    kind: Job
    metadata:
      name: ingress-nginx-admission-create
      namespace: ingress-nginx
      annotations:
        helm.sh/hook: pre-install,pre-upgrade
        helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
      labels:
        helm.sh/chart: ingress-nginx-4.0.6
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/version: 1.0.4
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/component: admission-webhook
    spec:
      template:
        metadata:
          name: ingress-nginx-admission-create
          labels:
            helm.sh/chart: ingress-nginx-4.0.6
            app.kubernetes.io/name: ingress-nginx
            app.kubernetes.io/instance: ingress-nginx
            app.kubernetes.io/version: 1.0.4
            app.kubernetes.io/managed-by: Helm
            app.kubernetes.io/component: admission-webhook
        spec:
          containers:
            - name: create
              image: k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1@sha256:64d8c73dca984af206adf9d6d7e46aa550362b1d7a01f3a0a91b20cc67868660
              imagePullPolicy: IfNotPresent
              args:
                - create
                - --host=ingress-nginx-controller-admission,ingress-nginx-controller-admission.$(POD_NAMESPACE).svc
                - --namespace=$(POD_NAMESPACE)
                - --secret-name=ingress-nginx-admission
              env:
                - name: POD_NAMESPACE
                  valueFrom:
                    fieldRef:
                      fieldPath: metadata.namespace
          restartPolicy: OnFailure
          serviceAccountName: ingress-nginx-admission
          nodeSelector:
            kubernetes.io/os: linux
          securityContext:
            runAsNonRoot: true
            runAsUser: 2000
    ---
    # Source: ingress-nginx/templates/admission-webhooks/job-patch/job-patchWebhook.yaml
    apiVersion: batch/v1
    kind: Job
    metadata:
      name: ingress-nginx-admission-patch
      namespace: ingress-nginx
      annotations:
        helm.sh/hook: post-install,post-upgrade
        helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
      labels:
        helm.sh/chart: ingress-nginx-4.0.6
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/version: 1.0.4
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/component: admission-webhook
    spec:
      template:
        metadata:
          name: ingress-nginx-admission-patch
          labels:
            helm.sh/chart: ingress-nginx-4.0.6
            app.kubernetes.io/name: ingress-nginx
            app.kubernetes.io/instance: ingress-nginx
            app.kubernetes.io/version: 1.0.4
            app.kubernetes.io/managed-by: Helm
            app.kubernetes.io/component: admission-webhook
        spec:
          containers:
            - name: patch
              image: k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1@sha256:64d8c73dca984af206adf9d6d7e46aa550362b1d7a01f3a0a91b20cc67868660
              imagePullPolicy: IfNotPresent
              args:
                - patch
                - --webhook-name=ingress-nginx-admission
                - --namespace=$(POD_NAMESPACE)
                - --patch-mutating=false
                - --secret-name=ingress-nginx-admission
                - --patch-failure-policy=Fail
              env:
                - name: POD_NAMESPACE
                  valueFrom:
                    fieldRef:
                      fieldPath: metadata.namespace
          restartPolicy: OnFailure
          serviceAccountName: ingress-nginx-admission
          nodeSelector:
            kubernetes.io/os: linux
          securityContext:
            runAsNonRoot: true
            runAsUser: 2000
    
    ```


```bash
# 컨트롤러 생성
kubectl apply -f ingress-nginx-controller.yaml
```

<img width="848" alt="Untitled 18" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/8da205ad-79cf-4f4c-9432-6476a72e5d22">

- ValidatingWebhookCofiguration 은 파드를 생성할 때 사용자가 정의한 자원이 문제 없는지 검증하기 위해 사용하지만, 활성 상태이면 실습 과정에서 오류가 발생하니 삭제합니다.

```bash
kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission
```

#### 2. 파드 및 서비스 생성하기

- 인그레스를 테스트하기 위한 서비스와 파드를 생성합니다.

```bash
vi cafe.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: coffee # coffee라는 디플로이먼트 생성
spec:
  replicas: 2
  selector:
    matchLabels:
      app: coffee
  template:
    metadata:
      labels:
        app: coffee
    spec:
      containers:
        - name: coffee
          image: nginxdemos/nginx-hello:plain-text
          ports:
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: coffee-svc # coffee-svc라는 서비스 생성
spec:
  ports:
    - port: 80
      targetPort: 8080
      protocol: TCP
      name: http
  selector:
    app: coffee
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tea # tea라는 디플로이먼트 생성
spec:
  replicas: 3
  selector:
    matchLabels:
      app: tea
  template:
    metadata:
      labels:
        app: tea
    spec:
      containers:
        - name: tea
          image: nginxdemos/nginx-hello:plain-text
          ports:
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: tea-svc # tea-svc라는 서비스 생성
  labels:
spec:
  ports:
    - port: 80
      targetPort: 8080
      protocol: TCP
      name: http
  selector:
    app: tea
```

```bash
# 디플로이먼트 및 서비스 생성
kubectl apply -f cafe.yaml
```

<img width="609" alt="Untitled 19" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/e411cc6b-47dc-4626-bc60-db36814d1781">

```bash
# 디플로이먼트, 서비스, 파드 상태 확인
kubectl get deploy,svc,pods
```

<img width="798" alt="Untitled 20" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/e138e3c2-89ef-4a20-ab55-1a9cec2121c1">

#### 3. 인그레스 생성하기

- 이제 요청 경로에 따라 서비스를 연결하기 위한 파일을 생성합니다.

```bash
vi cafe-ingress.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: cafe-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
    - http:
        paths:
          - path: /tea # URL에서 /tea 입력이 들어오면
            pathType: Prefix
            backend:
              service:
                name: tea-svc # tea-svc라는 서비스에 연결
                port:
                  number: 80
          - path: /coffee # URL에서 /coffee 입력이 들어오면
            pathType: Prefix
            backend:
              service:
                name: coffee-svc # coffee-svc라는 서비스에 연결
                port:
                  number: 80
```

```bash
# 인그레스 생성
kubectl apply -f cafe-ingress.yaml
```

```bash
# 인그레스 상태 확인
kubectl get ingress
```

<img width="574" alt="Untitled 21" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/dc2b9fac-7d90-42cc-aacd-924f13848efe">

```bash
# 파드 상태 확인
kubectl get pod -n ingress-nginx
```

<img width="754" alt="Untitled 22" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/7d8f5e4a-81ff-466a-9516-9920ee9c342c">

- ingress-nginx-controller라는 이름이 붙여진 파드가 Running 상태이면 정상입니다.

- 인그레스 컨트롤러의 포트 번호를 확인합니다.

```bash
kubectl get svc -n ingress-nginx
```

<img width="1055" alt="Untitled 23" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/cf9619ef-9083-4818-b854-b4cdf270c5a8">

- 30803 포트 번호를 사용하는 것을 확인할 수 있습니다.

#### 4. 외부 요청으로 인그레스 확인하기

- 엔드포인트 /coffee에 대한 요청은 coffee-svc 서비스에 전달되어 coffee 파드가 응답할 것이며, 엔드포인트 /tea에 대한 요청은 tea-svc 서비스에 전달되어 tea 파드가 응답할 것입니다.

- minikube랑 바인딩합니다.

<img width="865" alt="Untitled 24" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/06ea62dd-abfb-4f93-8671-374938c0631d">

- /tea 로 접속했을 때 응답한 파드가 다른 것을 확인할 수 있습니다.

<img width="609" alt="Untitled 25" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/ca2f67a1-cc8f-41c6-917e-a3314baa25d6">





---
참고)  
도서-쉽게 시작하는 쿠버네티스