---
layout: post
title: 컨테이너로 이미지 만들기
date: 2024-05-09
categories: [Docker]
tags: [컨테이너, 이미지]
---

- 이미지를 만드는 방법에는 두 가지가 있다.
- 첫번째는 commit 커맨드로 기존 컨테이너를 이미지로 변환하는 방법이고, 두 번째는 Dockerfile 스크립트로 이미지를 만드는 방법이다.

## commit 커맨드로 컨테이너를 이미지로 변환

- 컨테이너만 있으면 명령어 한 번으로 이미지를 만들 수 있어 간편하지만 컨테이너를 먼저 만들어야 한다.
- 기존 컨테이너를 본제하거나 이동해야 할 때 편리하다.

```docker
docker commit 컨테이너_이름 새로운_이미지_이름
```

<img width="708" alt="Untitled" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/38de66dc-9381-4604-ab47-463f4f66477b">

## Dockerfile 스크립트로 이미지 만들기

- Dockerfile 스크립트를 작성하고 이 스크립트를 빌드해 이미지를 만드는 방법이다.
- Dockerfile 스크립트에는 토대가 될 이미지나 실행할 명령어 등을 기재한다.
- 편집은 메모장 같은 텍스트 에디터를 사용한다.

<img width="751" alt="Untitled 1" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/f79d3147-2771-432c-af09-7c29baffd72d">

```docker
// 자주 사용되는 커맨드 예
docker build -t 생성할_이미지_이름 재료_폴더_경로
```

<img width="844" alt="Untitled 2" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/95d72560-4678-41ac-a91c-5db5e06b9139">

- Dockerfile은 첫머리에 오는 FROM 뒤에 이미지 이름을 기재하고, 그 뒤로는 파일 복사 또는 명령어 실행 등 컨테이너를 대상으로 할 일을 기술한다.

<img width="744" alt="Untitled 3" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/9441d485-8c59-4cc8-bdf1-955e15897a2d">

## [실습] commit 커맨드로 컨테이너를 이미지로 변환

- 아파치 컨테이너를 이미지로 변환해보겠다.

1. 아파치 컨테이너 준비

```docker
docker run --name apa000ex22 -d -p 8092:80 httpd
```

1. 컨테이너를 변환한 새로운 이미지 생성
    1. commit 커맨드를 사용해 apa000ex22 컨테이너로부터 새로운 이미지를 만든다.
    2. 이미지의 이름은 ex22_original1로 한다.

    ```docker
    docker commit apa000ex22 ex22_original1
    ```

2. 이미지가 생성됐는지 확인
    1. image ls 커맨드로 이미지가 생성됐는지 확인한다.

## [실습] Dockerfile 스크립트로 이미지 만들기

- httpd 이미지에 새로운 파일을 추가할 것이다.
- 이전에 사용한 apa_folder를 이용할 것이다.

1. 재료 폴더에 재료 준비
    1. 문서 디렉토리 안에 apa_folder 폴더를 준비한다.
    2. apa_folder안에 index.html를 넣어둔다.
2. Dockerfile 스크립트 작성
    1. vi 편집기를 통해 apa_folder 안에 Dockerfile을 생성한다.

    ```docker
    FROM httpd
    COPY index.html /usr/local/apache2/htdocs
    ```

3. 이미지가 생성됐는지 확인


---
참고)  
도서-그림과 실습으로 배우는 도커&쿠버네티스