---
layout: post
title: 도커 컴포즈 파일을 작성하는 법
date: 2024-05-16
categories: [Docker]
tags: [도커컴포즈파일]
---

## 컴포즈 파일(정의 파일)을 작성하는 방법

- 정의 파일은 YAML 형식을 따른다.
- 이름은 docker-compose.yml 로 사용한다.

### 컴포즈 파일 작성하는 방법

- 맨 앞에는 컴포즈 버전을 적고, 그 뒤로 service와 networks, volumes를 차례로 기재한다.
- service는 컨테이너에 대한 내용이다.

<img width="765" alt="Untitled" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/0a634ad3-faab-4a8c-9019-497165975d4e">

<img width="716" alt="Untitled 1" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/50c1043b-6809-4eeb-8dd3-13c7f1246808">

### 컴포즈 파일의 항목

<img width="313" alt="Untitled 2" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/a5384ae9-9a20-44a9-902c-64141190f211">

<img width="672" alt="Untitled 3" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/b4cd8f5e-3c51-404b-9d8b-19a4aae29bf7">

<img width="784" alt="Untitled 4" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/ae74cf60-3b18-4177-9604-b8d6e4efad9d">

- depends_on은 **다른 서비스에 대한 의존관계**를 나타낸다.
- 예를 들어, penguin 컨테이너의 정의에 ‘depends_on: -namgeuk’라는 내용이 포함됐다면 namgeuk 컨테이너를 생성한 다음에 penguin 컨테이너를 만든다.

<img width="605" alt="Untitled 5" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/1022c339-3c77-42e0-bcbf-9d1ceb96af6b">

## [실습] 컴포즈 파일 작성

- 워드프레스 및 mysql 컨테이너 실행하는 부분을 컴포즈 파일로 작성해 볼 것이다.

1. 문서 디렉토리에 com_folder 폴더를 생성한다.
2. com_folder 폴더 안에 docker-compose.yml 파일을 생성한다.

    ```yaml
    version: "3"
    services:
      mysql000ex11:
        image: mysql:5.7
        networks:
          - wordpress000net1
        volumes:
          - mysql000vol11:/var/lib/mysql
        restart: always
        environment:
          MYSQL_ROOT_PASSWORD: myrootpass
          MYSQL_DATABASE: wordpress000db
          MYSQL_USER: wordpress000kun
          MYSQL_PASSWORD: wkunpass
      wordpress000ex12:
        depends_on:
          - mysql000ex11
        image: wordpress
        networks:
          - wordpress000net1
        volumes:
          - wordpress000vol12:/var/www/html
        ports:
          - 8085:80
        restart: always
        environment:
          WORDPRESS_DB_HOST: mysql000ex11
          WORDPRESS_DB_NAME: wordpress000db
          WORDPRESS_DB_USER: wordpress000kun
          WORDPRESS_DB_PASSWORD: wkunpass
    networks:
      wordpress000net1:
    volumes:
      mysql000vol11:
      wordpress000vol12:
    
    ```



---
참고)  
도서-그림과 실습으로 배우는 도커&쿠버네티스