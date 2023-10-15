---
layout: post
title: 팩토리 패턴
date: 2023-10-15
categories: [CS, 디자인패턴]
---

# 팩토리 패턴이란

팩토리 패턴(factory pattern)은 객체를 사용하는 코드에서 객체 생성 부분을 분리해 추상화한 패턴이자 상속 관계에 있는 두 클래스에서 **`상위 클래스가 중요한 뼈대를 결정하고, 하위 클래스에서 객체 생성에 관한 구체적인 내용을 결정하는 패턴`**입니다.

- 상위 클래스와 하위 클래스가 분리되어 느슨한 결합을 가집니다.
- 상위 클래스는 인스턴스 생성 방식에 대해 알 필요가 없기 때문에 더 많은 유연성을 가집니다.
- 코드 리팩토링할 경우, 한 곳만 수정하면 되기 때문에 유지 보수성이 우수합니다.

## 예시

```java
enum CoffeeType {
    LATTE,
    ESPRESSO
}

abstract class Coffee {
    protected String name;

    public String getName() {
        return name;
    }
}

class Latte extends Coffee {
    public Latte() {
        name = "latte";
    }
}

class Espresso extends Coffee {
    public Espresso() {
        name = "Espresso";
    }
}

class CoffeeFactory {
    public static Coffee createCoffee(CoffeeType type) {
        switch (type) {
            case LATTE:
                return new Latte();
            case ESPRESSO:
                return new Espresso();
            default:
                throw new IllegalArgumentException("Invalid coffee type: " + type);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Coffee coffee = CoffeeFactory.createCoffee(CoffeeType.LATTE);
        System.out.println(coffee.getName()); // latte
    }
}

```

카페에서 음료를 주문받는 것을 예시로 했습니다. Coffe 클래스에서는 메뉴 명을 출력하는 것만 있고, 어떤 메뉴인지 정하는 부분은 하위 클래서에서 정의합니다. 이처럼 상위 클래서와 하위 클래서를 나눠서 진행하는 것을 볼 수 있습니다.

## 팩토리 패턴 장점

- 수정에 닫혀 있고 확장에는 열려있는 OCP 원칙을 지킬 수 있습니다.

## 팩토리 패턴 단점

- 많은 클래스를 정의해야하기 때문에 복잡도가 증가합니다.
