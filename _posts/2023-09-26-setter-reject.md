---
layout: post
title: Setter 지양
date: 2023-09-26
categories: [객체지향]
tags: [컨벤션]
---

## 지양하는 이유

- Setter 메소드를 사용하면 값을 변경한 의도를 파악하기 힘듦

```java
public Member updateMember(long id) {
    final Member member = findById(id);
    member.setFistName("value");
    member.setLastName("value");
    member.setAge("value");
    return member;
}
```

위의 코드는 Member의 정보를 변경하는 코드로 Setter 메소드들이 나열되어있다. 하지만 위의 코드처럼 Setter를 나열한 것만으로 어떤 의도로 데어터를 변경하는지 명확히 알 수 없다.

- 객체의 일관성을 유지하기 어렵다

자바 빈 규약을 따르는 Setter는 public으로 언제든지 변경할 수 있는 상태가 된다. 위처럼 회원 변경 메소드뿐만아니라 모든 곳에서 회원의 이름을 변경할 수 있는 상태가 되기 때문에 객체의 일관성을 유지하기 어렵다.

## Setter 대신 사용할 수 있는 방법

1. 생성자를 오버로딩
2. Builder 패턴 사용
3. 정적 팩토리 메소드

### 생성자를 오버로딩

```java
public class Member {
    private String name;
    private String age;

    public Member(String name){
    	this.name = name;
    }
    public Member(String name,int age){
    	this.name = name;
    	this.age = age;
    }
```

위의 코드처럼 생성자를 오버로딩하는 방법도 있지만 멤버변수가 많고 다양한 생성자를 가져야 한다면 코드가 길어지고 가독성이 떨어지는 일이 발생한다. 이를 해결하기 위해 Builder 패턴을 사용한다.

### Builder 패턴

빌더패턴을 이용하면 필요한 데이터만 이용하여 클래스를 생성할 수 있다. 그렇기 때문에 다양한 생성자를 만들지 않아도 되고 전체 생성자 하나만 있으면 되서 유지보수 및 가독성이 향상된다. 또한, 객체를 생성할 때 인자 값의 순서가 상관없다는 장점도 있다.

```java
public class Member {
    private String name;
    private String age;

    @Builder
    public Member(String name, int age){
    	this.name = name;
        this.age = age;
    }
```

### 정적 팩토리 메소드

정적 팩토리 메소드를 이용하면 이름을 가지기 때문에 어떤 용도인지 반환될 데이터가 뭔지 추측할 수 있다.

```java
public class Member {
    private String name;
    private String age;

    public static void createMember(String name, int age){
    	this.name = name;
        this.age = age;
    }
    public static void createMemberName(String name){
    	this.name = name;
    }
```

참고)

[https://velog.io/@hope1213/Setter-사용을-왜-지양해야할까](https://velog.io/@hope1213/Setter-%EC%82%AC%EC%9A%A9%EC%9D%84-%EC%99%9C-%EC%A7%80%EC%96%91%ED%95%B4%EC%95%BC%ED%95%A0%EA%B9%8C)
