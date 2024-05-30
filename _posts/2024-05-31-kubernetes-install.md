---
layout: post
title: 3. 쿠버네티스 설치하기
date: 2024-05-27
categories: [Kubernetes, 3.쿠버네티스 설치하기]
tags: [쿠버네티스]
---

- m1 macOS 환경에서 진행하기 위해 책에 있는 방법이 아닌 다른 방법으로 설치를 진행하였습니다.
- 실습은 **미니큐브(minikube)**를 활용하여 진행하겠습니다.

## 1. homebrew 설치하기

- homebrew 패키지 관리자를 이용해 설치할 것입니다.
- homebrew 설치는 아래와 같습니다.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## 2. cask 설치하기

- homebrew을 통해 GUI 기반의 애플리케이션을 설치하기 위해서는 cask가 필요합니다.

```bash
brew install cask
```

## 3. kubernetes cli 설치하기

- kubernetes cli를 이용하기 위해서 설치합니다.

```bash
brew install kubernetes-cli
```

## 4. minikube 설치하기

```bash
 brew install minikube
```

## 5. minikube 시작하기

```bash
minikube start
```

---

참고)

[https://dingdingmin-back-end-developer.tistory.com/entry/Mac-m1-m2-minikube-설치](https://dingdingmin-back-end-developer.tistory.com/entry/Mac-m1-m2-minikube-%EC%84%A4%EC%B9%98)