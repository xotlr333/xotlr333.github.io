---
layout: post
title: 볼륨과 마운트
date: 2024-05-08
categories: [Docker]
tags: [볼륨, 마운트]
---

## 볼륨과 마운트

- 볼륨 : 스토리지의 한 영역을 분할한 것.
- 마운트 : ‘연결하다’라는 의미 그대로 대상을 연결해 운영체제 또는 소프트웨어의 관리하에 두는 일을 말한다.
- 데이터 퍼시스턴시(data persistency) : 처음부터 컨테이너 외부에 둔 데이터에 접근해 사용하는 것.

- 도커에서 스토리지의 마운트는 볼륨 마운트와 바인드 마운트 두 가지 종류가 있다.

### 볼륨 마운트

- 볼륨 마운트는 도커 엔진이 관리하는 영역 내에 만들어진 볼륨을 컨테이너에 디스크 형태로 마운트한다.
- 볼륨에 비해 직접 조작하기 어려우므로 **‘임시 목적의 사용’** 이나 **‘자주 쓰지는 않지만 지우면 안 되는 파일’**을 두는 목적으로 많이 사용한다.

### 바인드 마운트

- 바인드 마운트는 도커가 설치된 컴퓨터의 문서 폴더 또는 바탕화면 폴더 등 도커 엔진에서 관리하지 않는 영역의 기존 디렉토리를 컨테이너에 마운트하는 방식이다.
- 디렉토리가 아닌 파일 단위로도 마운트가 가능하다.
- 폴더 속에 파일을 직접 두거나 열어볼 수 있기 때문에 **자주 사용하는 파일**을 두는 데 사용한다.

### 볼륨 마운트와 바인드 마운트 차이점

<img width="679" alt="Untitled" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/23ea26ec-ac52-444b-93d6-0e420cd43c97">

- **파일을 직접 편집해야 할 일이 많다면** 바인드 마운트를 사용하고, **그렇지 않다면** 볼륨 마운트를 사용하는 것을 권장한다.

### 볼륨관련 커맨드

- docker volume 커맨드 볼륨_이름

<img width="803" alt="Untitled 1" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/9673ad5e-ef97-4ab2-b777-a2351a3ff8fc">

- 스토리지를 마운트하는 커맨드
    - -v 옵션 뒤에 ‘스토리지 실제 경로’ 또는 ‘볼륨 이름’, ‘컨테이너 마운트 경로’ 순서대로 기재한다.
    - docker run (생략) -v 스토리지_실제_경로:컨테이너_마운트_경로 (생략)

## [실습] 바인드 마운트해보기

- index.html 파일을 배치하고 웹 브라우저에서 접근했을 때 아파치의 초기 화면이 index.html 파일의 내용으로 바뀌는지 확인해볼 것이다.

1. 마운트 원본 폴더 생성
    1. 문서 디렉토리에 apa_folder라는 폴더를 생성한다.
2. run 커맨드로 아파치 컨테이너 실행

```docker
docker run --name apa000ex20 -d -p 8090:80 -v /Users/gongtaesig/Documents/apa_folder/:/usr/local/apache2/htdocs httpd
```

1. 웹 브라우저 초기 화면 확인
    1. 빈 폴더이므로, ‘Index of/’라는 메시지가 출력된다.

   <img width="441" alt="Untitled 2" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/f6618760-0b67-43e6-97e0-51b126e0e94f">

2. 마운트된 폴더에 index.html 파일 배치
    1. index.html 파일을 apa_folder 폴더로 옮긴다.
3. 초기화면 다시 확인

   <img width="434" alt="Untitled 3" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/ed25c189-2121-4276-881d-e0c8dbf131fc">


## [실습] 볼륨 마운트해보기

1. 볼륨 생성

    ```docker
    docker volume create apa000vol1
    ```

2. run 커맨드로 아파치 컨테이너 실행

    ```docker
    docker run --name apa000ex21 -d -p 8091:80 -v apa000vol1:/usr/local/apache2/htdocs httpd
    ```

3. volume inspect 커맨드로 볼륨의 상세 정보 확인
    1. volume inspect 커맨드를 사용해 볼륨의 상세 정보를 확인한다.

       <img width="524" alt="Untitled 4" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/97d039ef-d3aa-4df8-ad87-c60af4ad18f1">

    2. container inspect 커맨드로 볼륨이 컨테이너에 마운트됐는지도 확인한다.

        ```docker
        docker container inspect apa000ex21
        ```

       <img width="462" alt="Untitled 5" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/cc0c9813-812e-4df2-a06e-d7ec77d40c00">


---
참고)  
도서-그림과 실습으로 배우는 도커&쿠버네티스