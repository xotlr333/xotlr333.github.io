---
layout: post
title: 컨테이너를 실행해 보자
date: 2024-05-05
categories: [Docker]
tags: [컨테이너]
---
# 4. 컨테이너를 실행해 보자

## 명령어와 대상

- docker 명령어 뒤에 오는 **‘무엇을’** **‘어떻게’**에 해당하는 부분을 **‘커맨드’**라고 한다.
- 커맨드는 다시 상위 커맨드와 하위 커맨드로 구성되며, 상위 커맨드가 ‘무엇을’, 하위 커맨드가 ‘어떻게’에 해당하는 내용을 지정한다.

<img width="480" alt="Untitled" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/1cc4bf9f-484c-438c-a40c-d66ee42d1ab6">

<img width="480" alt="Untitled 1" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/578cd5dc-3a9a-4ff7-92d1-9e21b474d176">

## 옵션과 인자

- 명령어의 기본적인 형태는 docker [커맨드] [대상] 이지만 커맨드에는 **‘옵션’**과**’인자’**라는 추가 정보가 붙는다.

<img width="478" alt="Untitled 2" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/bdcff47e-fe3d-4f5e-ac2a-12a4253fb1cf">

## 대표적인 명령어

<img width="383" alt="Untitled 3" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/bde45771-a57a-4fdc-b6b7-cf079c548acc">

## 컨테이너 조작 관련 커맨드(상위 커맨드 container)

<img width="764" alt="Untitled 4" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/b4e982c6-e782-498e-8b8a-058dfbd420ec">

## 이미지 조작 관련 커맨드(상위 커맨드 image)

<img width="788" alt="Untitled 5" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/31e9a4bc-c00e-43bd-a2fb-f5a9c6cebd9f">

## 볼륨 조작 관련 커맨드(상위 커맨드 volume)

<img width="771" alt="Untitled 6" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/7edcd006-896e-46c3-9a3e-63d1b3264a15">

## 네트워크 조작 관련 커맨드(상위 커맨드 network)

<img width="753" alt="Untitled 7" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/a6a4d47f-e1dc-4e59-9ebb-31ba2380c1ea">

## 그 밖의 상위 커맨드

- 이 밖에도 다음과 같은 상위 커맨드가 있으나, 대부분이 도커 스웜과 관련된 커맨드이다.

<img width="711" alt="Untitled 8" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/58e41e76-999c-4646-ace8-6b9cd982e8a3">

## 단독으로 쓰이는 커맨드

- 상위 커맨드 없이 단독으로 쓰이는 특수한 커맨드가 네 가지 있다.
- 주로 도커 허브의 검색이나 로그인에 사용되는 커맨드이다.

<img width="555" alt="Untitled 9" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/727dca57-12c3-4aad-8563-331d4dab2bad">

## 켄테이너 통신

- 아파치를 예를 들으면 컨테이너는 80포트로 외부 접근을 기다리고 있지만 물리적 컴퓨터와 컨테이너가 연결되어 있지 않아서 접근할 수 없다.

<img width="693" alt="Untitled 10" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/91457391-6aeb-4886-99fe-bb85420847b3">

- 그래서 컨테이너를 실행 중인 물리적 컴퓨터가 외부의 접근을 대신 받아 전달해주도록 한다.

<img width="708" alt="Untitled 11" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/aee7e0fb-a539-4c9b-833b-f9d1230ce39a">

- 좀 더 구체적으로 컨테이너를 실행 중인 컴퓨터(호스트)의 8080번 포트(이 포트는 다른 소프트웨어가 사용하는 포트와 겹치지 않는 한 임의의 숫자를 사용해도 된다)와 컨테이너의 80번 포트를 연결한다.
- 이 설정이 -p 옵션이며, 그 뒤로 ‘호스트의 포트 번호’와 ‘컨테이너의 포트 번호’를 콜론으로 연결한다.

<img width="707" alt="Untitled 12" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/aa25e599-0dc2-47ad-b54d-38051cd8a6e9">

---
참고)  
도서-그림과 실습으로 배우는 도커&쿠버네티스