---
layout: post
title: 전략 패턴
date: 2023-10-17
categories: [CS, 디자인패턴]
---

# 전략 패턴이란

전략 패턴(strategy pattern)은 정책 패턴이라고도 하며, 객체의 행위를 바꾸고 싶은 경우 **`직접`** 수정하지 않고 전략이라고 부르는 **`캡슐화한 알고리즘`**을 컨텍스트 안에서 바꿔주면서 상호 교체가 가능하게 만드는 패턴입니다.  
<img src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/5ddd4e9c-6d2c-441f-9c2f-0af95d3c5497" width="330" alt="전략패턴" />

- 컨텍스트란? 상황, 문맥, 맥락을 의미하며, 개발자가 어떠한 작업을 완료하는 데 필요한 모든 관련 정보를 말합니다.

## 예시

```java
import java.text.DecimalFormat;
import java.util.ArrayList;
import java.util.List;

interface PaymentStrategy {
    public void pay(int amount);
}

class KAKAOCardStrategy implements PaymentStrategy {
    private String name;
    private String cardNumber;
    private String cvv;
    private String dateOfExpiry;

    public KAKAOCardStrategy(String nm, String ccNum, String cvv, String expiryDate){
        this.name=nm;
        this.cardNumber=ccNum;
        this.cvv=cvv;
        this.dateOfExpiry=expiryDate;
    }

    @Override
    public void pay(int amount) {
        System.out.println(amount +" paid using KAKAOCard.");
    }
}

class LUNACardStrategy implements PaymentStrategy {
    private String emailId;
    private String password;

    public LUNACardStrategy(String email, String pwd){
        this.emailId=email;
        this.password=pwd;
    }

    @Override
    public void pay(int amount) {
        System.out.println(amount + " paid using LUNACard.");
    }
}

class Item {
    private String name;
    private int price;
    public Item(String name, int cost){
        this.name=name;
        this.price=cost;
    }

    public String getName() {
        return name;
    }

    public int getPrice() {
        return price;
    }
}

class ShoppingCart {
    List<Item> items;

    public ShoppingCart(){
        this.items=new ArrayList<Item>();
    }

    public void addItem(Item item){
        this.items.add(item);
    }

    public void removeItem(Item item){
        this.items.remove(item);
    }

    public int calculateTotal(){
        int sum = 0;
        for(Item item : items){
            sum += item.getPrice();
        }
        return sum;
    }

    public void pay(PaymentStrategy paymentMethod){
        int amount = calculateTotal();
        paymentMethod.pay(amount);
    }
}

public class HelloWorld{
    public static void main(String []args){
        ShoppingCart cart = new ShoppingCart();

        Item A = new Item("kundolA",100);
        Item B = new Item("kundolB",300);

        cart.addItem(A);
        cart.addItem(B);

        // 전략
        // pay by LUNACard
        cart.pay(new LUNACardStrategy("kundol@example.com", "pukubababo"));
        // pay by KAKAOBank
        cart.pay(new KAKAOCardStrategy("Ju hongchul", "123456789", "123", "12/01"));
    }
}
/*
400 paid using LUNACard.
400 paid using KAKAOCard.
*/

```

우리가 어떤 것을 구매할 때 네비어페이, 카카오페이 등 다양한 방법으로 결제하듯이 어떤 아이템을 구매할 때 LUNACard나 KAKAOCard로 결제하는 예제입니다.  
결제 방식의 **`전략`**만 바꿔서 두 가지 방식으로 결제하는 것을 구현했습니다.

## 전략 패턴 장점

- 알고리즘을 쉽게 변경 및 대체할 수 있으므로 유연 및 확장성이 좋습니다.
- 알고리즘을 캡슐화했기 때문에 재사용성이 좋습니다.

## 전략 패턴 단점

- 추가적인 클래스 및 인터페이스가 필요하기 때문에 코드 복잡성이 증가합니다.

참고)  
도서 - 면접을 위한 CS 전공지식 노트
