---
layout: post
title: 2. 컨테이너 리소스 관리하기
date: 2024-06-23
categories: [Kubernetes, 8. 쿠버네티스의 리소스 관리]
tags: [모니터링, 데이터독, 프로메테우스]
---

## LimitRange 사용하기

- **LimitRange**는 **파드에서 사용할 수 있는 리소스의 사용률을 제한하는 것**을 의미합니다.
- 즉, CPU가 어느 한계(예, 80%)에 다 다르면 더 이상 사용률이 증가하지 못하도록 설정할 수 있습니다.

![Untitled](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/db52887f-b500-44a3-a994-e394e27939cf)

## [실습] LimitRange 사용하기

### 1. LimitRange 생성하기

```bash
vi set-limit-range.yaml
```

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: set-limit-range
spec:
  limits:
    - max:
        cpu: "800m" # 최대값 설정
      min:
        cpu: "200m" # 최소값 설정
      type: Container
```

```bash
# limitrange 생성
kubectl apply -f set-limit-range.yaml
```

```bash
# limitrange 확인
kubectl get limitrange
kubectl describe limitrange set-limit-range
```

<img width="450" alt="Untitled" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/fc7a794b-03ab-4162-afe6-9f08d5507908">

<img width="634" alt="Untitled 1" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/82e0e003-2c03-40a0-bb81-706f634ddbb7">

### 2. CPU 요청량과 최대 사이즈 지정하기

- CPU 요청량(노드에 요청하는 CPU 사이즈)과 최대 사이즈를 지정해봅니다.
- **request** : 적어도 지정된 만큼의 자원은 컨테이너에서 사용할 수 있도록 보장함
- **limits** : 유휴 자원이 있다면 최대 지정된 만큼의 자원까지 컨테이너가 사용할 수 있음

```bash
vi pod-with-cpu-range.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-cpu-range
spec:
  containers:
    - name: pod-with-cpu-range
      image: nginx
      resources:
        limits:
          cpu: "800m" # 최대 사이즈
        requests:
          cpu: "500m" # CPU 요청량
```

```bash
# 파드 생성
kubectl apply -f pod-with-cpu-range.yaml
```

```bash
# 파드 상태 확인
kubectl get pod
kubectl describe pod pod-with-cpu-range.yaml
```

<img width="796" alt="Untitled 2" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/46aa23a6-bc5f-4d54-8348-bbb4d107b85e">

- 하지만 이렇게 자원 사용량에 제한을 두면 서비스 속도에 문제가 발생할 수 있습니다.
- 그래서 서비스에 어떠한 영향도 주지 않으면서 빠른 속도를 보장하기 위해 사용하는 것이 오토스케일링입니다.

## 오토스케일링(autoscaling)

- **오토스케일링**은 **할당된 파드의 자원 사용률이 너무 낮거나 높을 때 리소스의 크기를 조정하는 것**을 의미합니다.

![Untitled 1](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/b038a6a1-8176-472c-8618-9353816d5f79)

### 오토스케일링의 동작

1. 메트릭스 서버에 각 파드에서 사용 중인 자원이 어느 정도인지 질의합니다.
2. 반환값에 따라 몇 개의 파드를 늘려야 하는지 수평적 파드 오토스케일링에서 계산합니다.
3. 계산된 파드의 개수만큼 애플리케이션 파드를 늘립니다.

![Untitled 2](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/afc2f0a1-e5e8-4086-84d7-464b2f6ac9db)

## [실습] 오토스케일링 사용하기

### 1. 메트릭스 서버 설치하기

- 메트릭스 서버는 자동으로 설치되지 않아서 수동으로 설치해야합니다.

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

### 2. 메트릭스 디플로이먼스 설정 변경하기

- 설치가 완료되었다면 메트릭스 디플로이먼트 설정을 변경합니다.
- 변경하지 않으면 메트릭스 서버가 작동하지 않습니다.

```bash
kubectl edit deployment.apps -n kube-system metrics-server
```

<img width="587" alt="Untitled 3" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/3fac3096-ecf7-42a6-bc64-12fdf724182d">

### 3. 디플로이먼트 및 서비스 생성하기

- 오토스케일링을 테스트하기 위한 디플로이먼트와 서비스를 생성합니다.

```bash
vi php-apache.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  selector:
    matchLabels:
      run: php-apache
  replicas: 1
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
        - name: php-apache
          image: k8s.gcr.io/hpa-example
          ports:
            - containerPort: 80
          resources:
            limits:
              cpu: 500m
            requests:
              cpu: 200m
---
apiVersion: v1
kind: Service
metadata:
  name: php-apache
  labels:
    run: php-apache
spec:
  ports:
    - port: 80
  selector:
    run: php-apache
```

```bash
# 디플로인트 및 서비스 생성
kubectl apply -f php-apache.yaml
```

### 4. 오토스케일링 설정하기

- 오토스케일링 테스트를 위해 파드의 개수를 늘리거나 줄이도록 설정합니다.
- 즉, CPU 사용률이 50%를 넘긴다면 파드의 개수를 최대 10개까지 들릴 수 있도록 하고, CPU 사용률이 50% 이하라면 파드의 개수를 줄일 수 있지만 최소 1개는 유지하도록 오토스케일링을 설정합니다.

```bash
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
```

- 어떤 변화도 주지 않았을 때의 CPU 사용률은 0%입니다.

<img width="576" alt="Untitled 4" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/3e55cd2c-2b5f-4031-a922-2be5b3a100c1">

- 각 파드에 대한 자원 사용률과 현재 몇 개의 파드가 실행 중인지도 확인할 수 있습니다.

```bash
kubectl top pods
```

<img width="405" alt="Untitled 5" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/cb5b83ad-60d5-40fa-8a89-fdc51d91b9de">

### 5. 부하 테스트 하기

- 실제로 부하에 따라 파드의 개수가 변하는지 확인하기 위해 강제로 부하를 높여 보겠습니다.

```bash
# 부하 증가
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"

# 부하 확인
kubectl get hpa

# 부하에 따른 파드 개수 확인
kubectl get deployment php-apache
```

<img width="801" alt="Untitled 6" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/26f13568-5299-455e-8419-0ca8ca540c8f">





---
참고)  
도서-쉽게 시작하는 쿠버네티스