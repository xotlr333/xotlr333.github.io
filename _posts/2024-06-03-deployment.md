---
layout: post
title: 3. 디플로이먼트와 서비스 사용하기
date: 2024-06-03
categories: [Kubernetes, 4.쿠버네티스 사용하기]
tags: [디플로이먼트, 서비스]
---

![Untitled](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/14a7ae08-6998-43ed-a3b9-3bb21edf7fc0)

# 디플로이먼트 배포 전략

- 디플로이먼트의 배포 전략은 주로 애플리케이션이 변경될 때 사용합니다.
- 배포 방법으로는 **롤링**, **재생성**, **블루/그린**, **카나리**가 있습니다.

## 롤링 업데이트

- **롤링 업데이트**는 **새 버전의 애플리케이션(파드)을 배포할 때 새 버전의 애플리케이션은 하나씩 늘려가고 기존 버전의 애플리케이션은 하나씩 줄여나가는 방식**입니다.
- 새로운 버전에 문제 생길 경우 이전 버전의 파드로 서비스를 대체할 수 있어서 안정적인 배포 방식이지만, 배포가 느리다는 단점이 있습니다.
- 쿠버네티스에서 사용하는 **표준 배포 방식**입니다.

![Untitled 1](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/69949b95-9778-4061-8faa-089864c8adb8)

- 다음과 같은 yaml 파일이 있다고 가정해 보겠습니다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rolling
spec: 
  replicas: 3
  tamplate:
      metadata:
        labels:
          app: rolling
      spec:
        containers:
          - name: rolling
            image: myimage
            ports:
              - containerPort: 8080
```

- 여기서 롤링 업데이트 부분을 추가하면 다음과 같습니다.
- replicas와 template 중간에 추가하면 됩니다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rolling
spec: 
  replicas: 3
	strategy:                    # 여기서부터
			type: RollingUpdate
			rollingUpdate:
				maxSurge: 25%
				maxUnavailable: 25%     # 여기까지
  tamplate:
      metadata:
        labels:
          app: rolling
      spec:
        containers:
          - name: rolling
            image: myimage
            ports:
              - containerPort: 8080
```

- **maxSurge** : 업데이트 중에 만들 수 있는 파드의 최대 개수
- **maxUnavailable** : 업데이트 중에 사용할 수 없는 파드의 개수, 0보다 큰 정수를 지정할 수 있고 실습과 같이 퍼센트로 지정할 수도 있음

## 재생성 업데이트

- **재생성(recreate)** 업데이트는 **모든 이전 버전의 파드를 모두 한 번에 종료하고 새 버전의 파드로 일괄적으로 교체하는 방식**입니다.
- 빠르게 교체할 수 있다는 장점이 있지만, 새로운 버전의 파드에 문제가 발생하면 대처가 늦어질 수 있다는 단점이 있습니다.

![Untitled 2](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/0b818f45-69ce-4a95-9fe4-056f323e8542)

- 재생성 업데이트는 다음과 같이 사용할 수 있습니다.

```yaml
spec:
	replicas: 3
	strategy:             # 여기서부터
			type: Recreate    # 여기까지
	template:
...
```

## 블루/그린 업데이트

- **블루/그린(blue/green)** 업데이트는 **애플리케이션의 이전 버전과 새 버전이 동시에 운영**됩니다
- 서비스 목적으로 접속할 수 있는 것은 새 버전의 파드만 가능하고 이전 버전의 파드는 테스트 목적으로만 접속할 수 있습니다.
- 따라서 새로운 버전의 파드에 문제가 생기면 빠르게 대응할 수 있다는 장점이 있지만, 많은 파드가 필요하므로 많은 자원(CPU, 메모리 등)이 필요하다는 단점이 있습니다.

![Untitled 3](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/de98e6a0-9c60-4e9a-a440-e11c2ebf5826)

- 블루/그린 업데이트를 이용해 애플리케이션을 배포할 때는 version으로 구분해 관리합니다.

```yaml
spec:
	selector:
		app: bluegreen
	version: v1.0.0   # 트래픽이 신규 버전의 애플리케이션으로 변경되는 시점에 버전
...
```

## 카나리 업데이트

- **카나리**는 **두 개의 버전을 모두 배포하지만 새 버전에는 조금씩 트래픽을 증가시켜(예, 처음에는 5% 정도의 트래픽만 흘려보내지만 나중에는 50%의 트래픽을 흘려보냅니다) 새로운 기능들을 테스트**합니다.
- 기능 테스트가 끝나고 **새 버전의 문제가 없다고 판단하면 이전 버전은 모두 종료시키고 새 버전으로만 서비스**합니다.

![Untitled 4](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/ef4062eb-f146-4509-b204-859e00087c91)

- 카나리를 위한 매니페스트는 블루/그린과 유사하므로 블루/그린을 참조하세요.

---

# 디플로이먼트와 서비스 사용하기

- 파드에 위치한 서비스(예, 소셜커머스)를 외부에서 접속하려면 서비스(쿠버네티스의 서비스)를 이용해야 합니다.

![Untitled 5](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/c367f096-921a-4a00-b9a1-59f3a7c96db9)

- 파드가 배포되었다는 의미는 파드를 통해 서비스(예, 소셜커머스)를 하겠다는 것입니다.
- 그러려면 **외부에서 접속할 수 있어야** 하는데, 그 역할을 하는 것이 바로 **서비스**입니다.

## [실습] nginx 이미지를 이용해 디플로이먼트와 서비스 사용하기

### 1. yaml 파일 생성하기

```yaml
vi nginx-deploy.yaml  # niginx-deploy.yaml 파일 생성
```

- 생성된 yaml 파일에 다음 내용을 작성합니다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata: # 디플로이먼트 정보
  name: nginx-deploy # nginx-deploy라는 이름의 디플로이먼트 생성
  labels:
    app: nginx # 디플로이먼트의 레이블
spec:
  replicas: 2 # 2개의 파드 생성
  selector: # 디플로이먼트가 관리할 파드를 선택
    matchLabels:
      app: nginx # 디플로이먼트는 nginx 레이블을 갖는 파드를 선택해 관리
  template: # template에 정의된 내용에 따라 파드를 생성
    metadata:
      labels:
        app: nginx # 생성될 파드의 레이블
    spec: # 컨테이너에 대한 정보
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
```

- 이 매니페스트는 nginx 이미지를 이용해서 nginx-deploy이란 이름의 파드를 생성하겠다는 의미입니다.
- **디플로이먼트 → 파드 → 컨테이너 순서로 상세한 정보들을 정의**합니다.
- 참고로 .spec.tamplate.metadata.label의 설정과 .spec.selector.matchLabels의 설정이 같아야합니다.

<aside>
💡 **레이블이란?**
레이블이란 오브젝트에 키-값 쌍으로 리소스를 식별하고 속성을 지정하는 데 사용하는데, 일종의 카테고리라고 이해하면 쉽습니다.

</aside>

### 2. 파드 생성하기

- 작성한 yaml 파일을 **kubectl apply** 명령어를 통해 실행합니다.

```bash
kubectl apply -f nginx-deploy.yaml   # 디플로이먼트(파드) 생성
```

### 3. 디플로이먼트 확인하기

- 배포된 디플로이먼트 상태(목록)를 확인합니다.

```bash
kubectl get deployments
```

<img width="576" alt="Untitled" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/49314eff-713e-4527-a8d0-a46e01b1abe5">

- 파드도 확인합니다.

```bash
kubectl get pod
```

<img width="633" alt="Untitled 1" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/56bd7f4d-709a-4cdc-84cd-fd5ef7bf4dd3">

### 4. 서비스 yaml 파일 생성하기

- 이제 디플로이먼트로 생성된 파드들이 외부에서 접속할 수 있도록 서비스를 생성합니다.
- 먼저 nginx-svc.yaml 파일을 생성합니다.

```bash
vi nginx-svc.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc # 서비스 이름
  labels:
    app: nginx
spec:
  type: NodePort # 노드포트를 이용해서 서비스를 외부에 노출시킴
  ports:
    - port: 8080
      nodePort: 31472
      targetPort: 80
  selector:
    app: nginx # 서비스를 nginx 레이블을 갖는 파드와 연결
```

### 5. 서비스 생성하기

```yaml
kubectl apply -f nginx-svc.yaml
```

### 6. 서비스 정보 확인하기

```yaml
kubectl get svc
```

<img width="712" alt="Untitled 2" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/02549728-17d7-4bd9-8194-08dddf9edd4b">




---
참고)  
도서-쉽게 시작하는 쿠버네티스