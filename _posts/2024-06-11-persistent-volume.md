---
layout: post
title: 2. 영구 볼륨과 영구 볼륨 클레임
date: 2024-06-11
categories: [Kubernetes, 5. 스테이트풀을 위한 영구 볼륨]
tags: [영구 볼륨, 영구 볼륨 클레임]
---

- **데이터를 오래 보관하기 위해**서는 외부 볼륨이 필요합니다.

# 영구 볼륨과 영구 볼륨 클레임이란

- **영구 볼륨(Persistent Volume, PV)**은 컨테이너, 파드, 노드의 종료의 무관하게 **데이터를 영구적으로 보관할 수 있는 볼륨**입니다.
- 쿠버네티스에서 영구 볼륨은 시스템 관리자가 외부 저장소에서 볼륨을 생성한 후 쿠버네티스 클러스터에 연결하는 것을 의미합니다.

![Untitled](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/2cf06115-8f38-4d1c-a720-748398b4f770)

- 개발자가 디플로이먼트를 생성할 때 볼륨을 정의하는데, 이때 볼륨 자체를 정의하는 것이 아니라 볼륨을 요청하는 볼륨 클레임을 지정합니다.
- 그 다음 볼륨 클레임에서 볼륨을 할당합니다.
- 이때 **영구 볼륨을 요청하는 볼륨 클레임**을 **영구 볼륨 클레임(Persistent Volume Claim, PVC)**이라고 합니다.

![Untitled 1](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/54e85af5-85f8-45ce-be99-3c10d7fd465d)

- 하지만 엄밀히 말해서 개발자가 요청한 영구 볼륨(영구 볼륨 클레임)은 API 서버에 보내지고 이후 API 서버를 통해서 영구 볼륨이 할당됩니다.

![Untitled 2](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/421be4a7-7da7-432d-af92-3cbb2e85dd07)

# 영구 볼륨과 영구 볼륨 클레임 사용하기

- 파드에서 영구 볼륨 클레임 이름을 지정하며, 해당 이름을 갖는 영구 볼륨 클레임에서는 storageClassName을 이용해 영구 볼륨에 바인딩하겠습니다.

![Untitled 3](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/bf511ac0-d8cf-43d2-9727-d2b01722faf8)

- mysql 파드에 영구 볼륨을 마운트하는 실습을 진행하겠습니다.
- 그런데 외부 저장소가 없으니 노드의 디스크(노드 내의 공간)를 사용합니다.
- 외부 저장소를 사용하는 방법도 이와 유사합니다.

### 1. 영구 볼륨 yaml 파일 생성하기

```bash
vi pv.yaml
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual # 영구 볼륨 클레임의 요청을 해당 영구 볼륨에 바인딩
  capacity:
    storage: 20Gi # 영구 볼륨으로 20Gi 할당
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data" # 외부 저장소가 준비되지 않았기 때문에 노드의 /mnt/data 디렉터리를 볼륨으로 사용

```

### 2. 영구 볼륨 생성하기

```yaml
kubectl apply -f pv.yaml
```

### 3. 영구 볼륨 클레임 yaml 파일 생성하기

```bash
vi pvc.yaml
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
```

### 4. 영구 볼륨 클레임 생성하기

```bash
kubectl apply -f pvc.yaml
```

### 5. mysql 용도 파드 생성하기

```bash
vi pvc-deployment.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql # mysql이라는 디플로이먼트 생성
spec:
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - image: mysql:8.0.29 # mysql 8.0.29 버전의 파드 생성
          name: mysql
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: password # mysql에 접속하기 위한 패스워드
          ports:
            - containerPort: 3306 # mysql에서 사용하는 포트
              name: mysql
          volumeMounts:
            - name: mysql-persistent-storage
              mountPath: var/mysql # /var/mysql 디렉터리로 볼륨 마운트
      volumes:
        - name: mysql-persistent-storage
          persistentVolumeClaim: # 영구 볼륨 할당 요청
            claimName: mysql-pv-claim
```

### 6. 디플로이먼트 생성하기

```bash
kubectl apply -f pvc-deployment.yaml
```

### 7. 외부 접속을 위한 서비스 생성하기

```bash
vi mysql_service.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  clusterIP: None
  ports:
    - port: 3306
  selector:
    app: mysql
```

```yaml
kubectl create -f mysql_service.yaml
```

### 8. 디플로이먼트 상태 확인하기

- 디플로이먼트, 서비스, 영구 볼륨, 영구 볼륨 클레임이 생성되었습니다.
- 이제 정상적으로 생성되었는지 확인하겠습니다.

```bash
kubectl describe deployment mysql
```

<img width="1021" alt="Untitled" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/850762da-ae02-4c10-8d91-2d31c79245db">

- 생성된 디플로이먼트 상태가 정상임을 확인 할 수 있습니다.
- 디플로이먼트에 문제가 있다면 error라는 메시지가 보일 것입니다.
- 또한 mysql이라는 이름의 컨테이너에 /var/mysql라는 디렉토리로 볼륨이 할당됐음을 볼 수 있습니다.

### 9. 파드, 서비스, 영구 볼륨 상태 확인하기

```bash
kubectl get pods

kubectl get service

kubectl get pv
```

<img width="1199" alt="Untitled 1" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/30f2e5d4-ae3d-41a6-b621-b5e6cbcff02c">

- **RECLAIM POLICY** : 영구 볼륨의 라이프사이클을 의미합니다. 영구 볼륨 클레임이 삭제될 때 영구 볼륨과 연결되어 있던 저장소에 저장된 파일을 어떻게 처리할지에 따라 Retain, Delete, Recycle 3가지로 나뉩니다.
    - **Retain 정책** : 영구 볼륨 클레임이 삭제되어도 저장소에 저장되어 있던 파일을 삭제하지 않는 정책입니다.
    - **Delete 정책** : 영구 볼륨 클레임이 삭제되면 영구 볼륨과 연결된 저장소 자체를 삭제합니다.
    - **Recycle 정책** : 영구 볼륨 클레임이 삭제되면 영구 볼륨과 연결된 저장소 데이터는 삭제합니다. 하지만 저장소 볼륨 자체를 삭제하지는 않는다는 점에서 Delete와는 다릅니다.

- 또한 볼륨은 다음 중 하나의 **상태(STATUS)**를 갖습니다.
    - **Available(사용가능)** : 아직 클레임에 바인딩되지 않은 상태입니다.
    - **Bound(바인딩)** : 볼륨이 클레임에 의해 바인딩된 상태입니다.
    - **Released(릴리즈)** : 클레임이 삭제되었지만 클러스터에서 아직 리소스가 반환되지 않은 상태입니다.
    - **Failde(실패)** : 볼륨이 자동 반환에 실패한 상태입니다.

- 볼륨의 대한 자세한 정보는 kubectl describe pv mysql-pv-volume 명령어로 확인할 수 있습니다.

<img width="722" alt="Untitled 2" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/5bdd1f98-1189-4aa0-a1bf-f15e9c3d591e">

### 10. 영구 볼륨 클레임 상태 확인하기

```bash
kubectl get pvc
```

<img width="839" alt="Untitled 3" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/487f0f23-77d7-4f69-ae16-81d1da58b008">

- **ACCESS MODES** : 볼륨에 접근할 수 있는 모드를 나타내며 다음의 값 중에 하나를 갖습니다.
    - **RWO(ReadWriteOnce)** : 하나의 노드에서 해당 볼륨이 읽기-쓰기로 마운트됩니다.
    - **ROX(ReadOnlyMany)** : 볼륨은 많은 노드에서 읽기 전용으로 마운트됩니다.
    - **RWX(ReadWriteMany)** : 볼륨은 많은 노드에서 읽기-쓰기로 마운트됩니다.
    - **RWOP(ReadWriteOnePod)** : 볼륨이 단일 파드에서 읽기-쓰기로 마운트됩니다. 즉, 전체 클러스터에서 단 하나의 파드만 해당 영구 볼륨 클레임을 읽거나 쓸 수 있어야 하는 경우에 사용합니다.

- 영구 볼륨 클레임에 대한 자세한 정보는 kubectl describe pvc mysql-pv-claim 명령어로 확인할 수 있습니다.

<img width="702" alt="Untitled 4" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/6ee122d7-4bc6-4770-b821-6e5768320a52">

### 11. mysql에 접속 후 데이터베이스와 테이블 생성하기

```bash
# mysql 접속
kubectl run -it --rm --image=mysql:latest --restart=Never mysql-client -- mysql -h mysql -ppassword
```

<img width="1266" alt="Untitled 5" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/e9d58847-1782-4a59-b036-a8dfdf33c4d4">

```sql
# 데이터베이스 생성
create database mySQL;

# 테이블 생성
use mySQL;
create table Test
(
	ID INT,
	Name varchar(30) default 'Anonymous',
	ReserveDate DATE,
	RoomNum INT
);

# 테이블 확인
show tables;

# mysql 나가기
quit
```

<img width="514" alt="Untitled 6" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/fdf08ebc-ff91-4664-a5ca-303192d6cc70">






---
참고)  
도서-쉽게 시작하는 쿠버네티스