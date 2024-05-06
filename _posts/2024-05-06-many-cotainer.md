---
layout: post
title: 여러 개의 컨티이너를 연동해 실행해보자
date: 2024-05-06
categories: [Docker]
tags: [컨테이너]
---
# 5. 여러 개의 컨테이너를 연동해 실행해보자

# 워드프레스 구축

- 워드프레스는 워드프레스 컨테이너와 mysql 컨테이너로 구성된다.
- 그저 컨테이너를 두 개 만들기만 해서는 두 컨테이너가 연결되지 않으므로 가상 네트워크를 만들고 이 네트워크에 두 개의 컨테이너를 소속시켜 두 컨테이너를 연결한다.
- 이 가상 네트워크를 만드는 커맨드가 docker network create다.
- 커맨드 뒤로 네트워크 이름을 기재하면 된다.

## 네트워트 생성

```docker
docker network create wordpress000net1
```

## mysql 컨테이너 생성 및 실행

```docker
docker run --name mysql000ex11 -dit --net=wordpressnet1 -e MYSQL_ROOT_PASSWORD=myrootpass -e MYSQL_DATABASE=wordpress00db -e MYSQL_USER=wordpress000kun -e MYSQL_PASSWORD=wkunpass mysql
```

## wordpress 컨테이너 생성 및 실행

```docker
docker run --name wordpress000ex12 -dit --net=wordpress000net1 -p 8085:80 -e WORDPRESS_DB_HOST=mysql000ex11 -e WORDPRESS_DB_NAME=wordpress000db -e WORDPRESS_DB_USER=wordpress000kun -e WORDPRESS_DB_PASSWORD=wkunpass wordpress
```

## wordpress 접근 확인

- [localhost:8085](http://localhost:8085) 접속
- 초기화면 확인

<img width="583" alt="Untitled" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/f1a6f40d-723b-4483-9ae8-a5aa51b60bad">

---
참고)  
도서-그림과 실습으로 배우는 도커&쿠버네티스