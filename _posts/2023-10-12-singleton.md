---
layout: post
title: 싱글톤 패턴
date: 2023-10-12
categories: [CS, 디자인패턴]
---

# 싱글톤이란

싱글톤 패턴(singleton pattern)은 하나의 클래스에서 오직 하나의 인스턴스만 가지는 패턴입니다. 보통 데이터베이스 연결 모듈에 많이 사용합니다.  
하나의 인스턴스를 만들어 놓고 해당 인스턴스를 다른 모듈들이 공유하며 사용하기 때문에 인스턴스를 생성할 때 드는 비용이 줄어드는 장점이 있습니다. 하지만 의존성이 높아진다는 단점이 있습니다.

## 예시

```java
class Singleton {
    private static class singleInstanceHolder {
        private static final Singleton INSTANCE = new Singleton();
    }
    public static Singleton getInstance() {
        return singleInstanceHolder.INSTANCE;
    }
}

public class HelloWorld{
     public static void main(String []args){
        Singleton a = Singleton.getInstance();
        Singleton b = Singleton.getInstance();
        System.out.println(a.hashCode());
        System.out.println(b.hashCode());
        if (a == b){
         System.out.println(true);
        }
     }
}
/*
705927765
705927765
true
*/

```

# 싱글톤 패턴의 단점

- TDD인 경우 단위 테스트를 진행할 때, 어려움이 있을 수 있습니다. 왜냐하면 단위 테스트는 테스트가 서로 독립적이어야 하며 테스트를 어떤 순서로든 실행할 수 있어야 하기 때문입니다.  
  하지만 싱글톤 패턴은 미리 생성된 하나의 인스턴스를 기반으로 구현하는 패턴이므로 각 테스트마다 '독립적인' 인스턴스를 만들기가 어렵습니다.

## 의존성

싱글톤 패턴은 사용하기가 쉽고 굉장히 실용적이지만 모듈 간의 결함을 강하게 만들 수 있는 단점이 있습니다. 이때 의존성 주입(DI)을 통해 결합을 조금 느슨하게 만들 수 있습니다.  
메인 모듈이 직접 다른 하위 모듈에 대한 의존성을 주기보다는 중간에 의존성 주입자(dependency injector)가 이 부분을 가로채 메인 모듈이 간접적으로 의존성을 주입하는 방식입니다.
이를 통해 메인 모듈은 하위 모듈에 대한 의존성이 떨어집니다. 이를 **`디커플링이 된다`**라고도 합니다.

### 의존성 장점

- 모듈들을 쉽게 교체할 수 있는 구조가 되어 테스팅하기 쉽고 마이그레이션하기도 수월합니다.
- 구현할 때 추상화 레이어를 넣고 이를 기반으로 구현체를 넣어 주기 때문에 애플리케이션 의존성 방향이 일관되고, 애플리케이션을 쉽게 추론할 수 있으며, 모듈 간의 관계들이 조금 더 명확해집니다.

### 의존성 단점

- 모듈들이 분리되므로 클래스 수가 증가되어 복잡성이 증가될 수 있습니다. 그래서 런타임 패널티가 생기기도 합니다.

### 의존성 원칙

- 의존성 원칙은 **`상위 모듈은 하위 모듈에서 어떠한 것도 가져오지 않아야 합니다. 또한, 둘 다 추상화에 의존해야 하며, 이때 추상화는 세부 사항에 의존하지 말아야 합니다.`** 라는 의존성 주입 원칙을 지켜주면서 만들어야 합니다.

참고)  
도서 - 면접을 위한 CS 전공지식 노트
