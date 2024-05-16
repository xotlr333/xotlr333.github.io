---
layout: post
title: 도커 컴포즈 실행
date: 2024-05-17
categories: [Docker]
tags: [도커컴포즈파일]
---
## 도커 컴포즈 커맨드

- 도커 컴포즈는 docker-compose 명령을 사용한다.

### 컨테이너와 주변 환경을 생성하는 docker-compose up 커맨드

- 컴포즈 파일의 내용을 따라 컨테이너와 볼륨, 네트워크를 생성하고 실행한다.

```yaml
// 자주 사용하는 커맨드 예
docker-compose -f 정의_파일_경로 up 옵션
```
<img width="410" alt="Untitled" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/d84dd46f-0764-4fb2-bc49-93810252491d">

### 컨테이너와 네트워크를 삭제하는 docker-compose down 커맨드

- 컴포즈 파일의 내용을 따라 컨테이너와 네트워크를 종료 및 삭제한다.
- 볼륨과 이미지는 삭제하지 않는다.

```yaml
// 자주 사용하는 커맨드 예
docker-compose -f 컴포즈_파일_경로 down 옵션
```

<img width="681" alt="Untitled 1" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/7a986124-e018-4f93-98ea-213f2c632b69">

### 컨테이너를 종료하는 docker-compose stop 커맨드

- 컴포즈 파일의 내용에 따라 컨테이너를 종료한다.

```yaml
// 자주 사용하는 예
docker-compose -f 컴포즈_파일_경로 stop 옵션
```

### 주요 커맨드 목록

<img width="562" alt="Untitled 2" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/1a2f6b1d-57f2-4824-957c-7f962af3f4d9">

<img width="540" alt="Untitled 3" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/734b9eb9-bd9b-4893-8548-ab7945a7b9f5">

## [실습] 도커 컴포즈 실행

- 작성한 컴포즈를 실행해 볼 것이다.

1. 컴포즈 파일을 적절한 위치에 배치
    1. com_folder  폴더에 docker-compose.yml를 둔다.
2. 컴포즈 파일의 내용을 실행
    1. docker-compose up 명령어를 실행하면 컴포즈 파일의 정의대로 컨테이너 및 주변 환경이 생성된다.

    ```docker
    docker-compose -f /Users/gongtaesig/Documents/com_folder/docker-compose.yml up -d
    ```

3. 웹 브라우저로 초기화면 확인

    <img width="476" alt="Untitled 4" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/ae804134-ad16-4635-a191-cf8cac7c5a9d">

4. 컨테이너와 네트워크를 종료 및 삭제
    1. 확인이 끝나면 docker-compose down 명령어를 사용해 컨테이너와 네트워크를 종료 및 삭제한다.

    ```docker
    docker-compose -f /Users/gongtaesig/Documents/com_folder/docker-compose.yml down
    ```

5. 마무리
    1. down 커맨드를 사용해도 이미지와 볼륨은 삭제되지 않는다.
    2. 직접 삭제해야한다.


### 실습 중 오류
    - 오류 메시지

    <img width="860" alt="Untitled 5" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/10bf57df-9767-4fde-946e-8ed10ee351b3">

    - 해결 방안
        - 플랫폼을 지정한다.

        ```yaml
        version: "3"
        services:
          mysql000ex11:
        		platform: linux/x86_64 // 플랫폼 지정
            image: mysql:5.7
            networks:
              - wordpress000net1
            volumes:
              - mysql000vol!:/var/lib/mysql
            restart: always
        ...
        ```


---
참고)  
도서-그림과 실습으로 배우는 도커&쿠버네티스