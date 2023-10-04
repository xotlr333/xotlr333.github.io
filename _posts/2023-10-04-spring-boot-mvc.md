---
layout: post
title: Spring Boot MVC 패턴
date: 2023-10-04
categories: [Spring, 디자인패턴]
tags: [MVC]
---

# MVC 패턴이란

Model-View-Controller의 약자로서 개발을 할 때 3가지 형태로 역할을 나누어 개발하는 방법론이다.

- Model
  - 어플리케이션이 무엇을 할 것인지 정의하는 부분이다. 즉, DB와 연동하여 사용자가 입력한 데이터나 사용자에게 출력할 데이터를 다룬다.
- View
  - 사용자에게 시각적으로 보여주는 부분이다.
- Controller
  - Model이 데이터를 어떻게 처리할지 알려주는 역할을 한다.사용자에 의해 클라이언트가 보낸 데이터가 있으면 모델을 호출하기전에 적절히 가공하고 Model을 호출한다. 그런 다음 모델이 업무 수행을 완료하면 그 결과를 가지고 View에게 전달한다.

## 장점

- 분리되어 각자의 역할에 집중할 수 있다
- 유지보수성, 애플리케이션의 확장성, 유연성 증가, 중복코딩 문제 해결

![MVC](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/4e2a2cc3-3771-4783-a330-7fa62ab538a3)

# Controller란

MVC에서 C에 해당하며 주로 사용자의 요청을 처리한 후 지정된 뷰에 모델 객체를 넘겨주는 역할을 한다.

즉, 사용자의 요청이 진입하는 지점이며 요청에 따라 어떤 처리를 할지 결정을 Service에 넘겨준다. 그 후, Service에서 실질적으로 처리한 내용을 View에게 넘겨준다.

## Contorller에서 사용하는 어노테이션

1. @Controller

전통적인 Spring MVC의 컨트롤러인 @Controller는 주로 View를 반환하기 위해 사용한다. 하지만 @ResponseBody 어노테이션과 같이 사용하면 RestController와 똑같은 기능을 수행할 수 있다.

```java
@Controller
public class Controllerprac {
    @GetMapping("/home") //home으로 Get요청이들어오면
    public String homepage(){
        return "home.html"; //home.html생성
    }
}
```

1. @RestController

RestController는 Controller에서 @ResponseBody 어노테이션이 붙은 효과를 지니게 된다.

즉, 주용도는 JSON/XML 형태로 객체 데이터 반환을 목적으로 한다.

```java
@RestController // JSON으로 데이터를 주고받음을 선언합니다.
public class ProductRestController {

    private final ProductService productService;
    private final ProductRepository productRepository;

    // 등록된 전체 상품 목록 조회
    @GetMapping("/api/products")
    public List<Product> getProducts() {
        return productRepository.findAll();
    }
}
```

# Service란

Service를 이해하기 위해 큰 틀을 보겠다.

1. Client가 Request를 보낸다.
2. Request URL에 알맞은 Controller가 수신을 받는다.
3. Controller는 넘어온 요청을 처리하기 위해 Service를 호출한다.
4. Service는 알맞는 정보를 가공하여 Controller에게 데이터를 넘긴다.
5. Controller는 Service의 결과물을 Client에게 전달한다.

Service가 알맞은 정보를 가공하는 과정을 ‘비즈니스 로직을 수행한다’ 라고 한다.

Service가 비즈니스 로직을 수행하고 데이터베이스에 접근하는 DAO를 이ㅛㅇ해서 결과값을 받아 온다.

# DAO란

단순하게 페이지를 불러오고 DB정보를 한번에 불러오는 간단한 프로젝트의 경우 Service와 DAO는 차이가 거의 없을 수 있다.

DAO는 쉽게 말하면 MySql 서버에 접근하여 SQL문을 실행할 수 있는 객체이다.

# DAO와 JPA

Spring Data JPA는 매우 적은 코드로 DAO를 구현할 수 있도록 해준다. 즉, 인터페이스를 만드는 것만으로도 Entity클래스에 대한 insert, update, delete, select를 실행할 수 있게 해준다. 뿐만 아니라 인터페이스에 메소드를 선언하는 것만으로 라이트한 쿼리를 수행하는 코드를 만드는 것과 동등한 작업을 수행한다.

그러면 JPA를 사용하면 DAO는 직접 구현을 안해도되는건가? 라는 생각을 할 수 있다.

하지만 JPA가 만들 수 있는 코드는 매우 가볍고 쉬운 쿼리를 대체하는 것이라서 복잡도가 높아지면 사용하기가 매우 어렵다.

이때 JPA만으로만 사용한다면 수행능력이 SQL을 직접 사용해서 개발하는 것보다 못한 상황이 벌어질 수도 있다. 그래서 JPA를 깊게 공부를 해서 JPA로 복잡도가 높은 쿼리를 짜거나 아니면 복잡도가 높은 곳은 DAO로 같이 사용한다고 한다.

![네트워크 흐름](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/3affc9a7-425e-40ba-a3b1-7646a8ffa09c)

# Reposity란

Entity에 의해 생성된 DB에 접근하는 메소드들을 사용하기 위한 인터페이스이다. @Entity라는 어노테이션으로 데이터베이스 구조를 만들었다면 여기에 CRUD를 한다. 이것을 어떻게 할 것인지 정의해주는 계층이라고 생각하면 된다.

JPA로 Reposity를 만들어 본다.

```java
public interface ProductRepository extends JpaRepository<Product, Long> {
	...
}
```

ProductRepository라는 인터페이스에 JpaRepository를 상속한다. 저기서 Product는 Entity 클래스 명이고 Long은 그 Entity클래스의 PK의 자료형을 넣어준다.

![save 메소드](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/a36d7a28-c54d-4e0a-a3ff-2fa7a8a42340)

그러면 이 ProductRepository에 JPA가 기본적으로 제공되는 메소드를 사용할 수 있다.

# 순서 정리

![순서 정리](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/dcf4cbd8-9e26-411d-8daa-951c3116664d)

## JPA가 없는 경우

![jpa가 없는 경우](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/475fa77c-c20f-41a6-b4be-ef69e076d667)

## JPA가 있는 경우

![jpa가 있는 경우](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/eadf12b6-158f-4be1-910d-acff081c252d)

참고)

[https://velog.io/@jybin96/Controller-Service-Repository-가-무엇일까](https://velog.io/@jybin96/Controller-Service-Repository-%EA%B0%80-%EB%AC%B4%EC%97%87%EC%9D%BC%EA%B9%8C)
