---
layout: post
title: 옵저버 패턴
date: 2023-10-16
categories: [CS, 디자인패턴]
---

# 옵저버 패턴이란

옵저버 패턴(observer pattern)은 **`주체가 어떤 객체의 상태 변화를 관찰하다가 상태 변화가 있을 때마다 메서드 등을 통해 옵저버 목록에 있는 옵저버들에게 변화를 알려주는`** 디자인 패턴입니다.

<img src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/cc03f7ab-f639-434b-9ac2-b9fba6449baf" width="330" alt="객체와 주체가 따로 있는 옵저버 패턴" />

여기서 주체는 객체의 상태 변화를 보고 있는 관찰자를 말하고, 옵저버는 이 객체의 상태 변화에 따라 전달되는 메서드 등을 기반으로 **`추가 변화 사항`**이 생기는 객체들을 의미합니다.

<img src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/bcc0eade-a3c6-4aea-9520-c19ee290ffc5" width="330"  alt="객체와 주체가 같이 있는 옵저버 패턴" />

앞의 그림처럼 주체와 객체를 따로 두지 않고 상태가 변경되는 객체를 기반으로 구축하기도 합니다.  
옵저버 패턴을 활용한 서비스로는 트위터가 있습니다. 내가 어떤 사람인 주체를 팔로우했다면 주체가 포스팅을 올리게 되면 알림이 팔로워에게 가도록 하는 것이 옵저버 패턴입니다.

또한, 옵저버 패턴은 MVC 패턴에서도 사용됩니다. 예를 들어 주체라고 볼 수 있는 모델에서 변경 사항이 생겨 update() 메서드로 옵저버인 뷰에게 알려주고 이를 기반으로 컨트롤러 등이 작동하는 것입니다.

## 예시

```java
import java.util.ArrayList;
import java.util.List;

interface Subject {
    public void register(Observer obj);
    public void unregister(Observer obj);
    public void notifyObservers();
    public Object getUpdate(Observer obj);
}

interface Observer {
    public void update();
}

class Topic implements Subject {
    private List<Observer> observers;
    private String message;

    public Topic() {
        this.observers = new ArrayList<>();
        this.message = "";
    }

    @Override
    public void register(Observer obj) {
        if (!observers.contains(obj)) observers.add(obj);
    }

    @Override
    public void unregister(Observer obj) {
        observers.remove(obj);
    }

    @Override
    public void notifyObservers() {
        this.observers.forEach(Observer::update);
    }

    @Override
    public Object getUpdate(Observer obj) {
        return this.message;
    }

    public void postMessage(String msg) {
        System.out.println("Message sended to Topic: " + msg);
        this.message = msg;
        notifyObservers();
    }
}

class TopicSubscriber implements Observer {
    private String name;
    private Subject topic;

    public TopicSubscriber(String name, Subject topic) {
        this.name = name;
        this.topic = topic;
    }

    @Override
    public void update() {
        String msg = (String) topic.getUpdate(this);
        System.out.println(name + ":: got message >> " + msg);
    }
}

public class HelloWorld {
    public static void main(String[] args) {
        Topic topic = new Topic();
        Observer a = new TopicSubscriber("a", topic);
        Observer b = new TopicSubscriber("b", topic);
        Observer c = new TopicSubscriber("c", topic);
        topic.register(a);
        topic.register(b);
        topic.register(c);

        topic.postMessage("amumu is op champion!!");
    }
}
/*
Message sended to Topic: amumu is op champion!!
a:: got message >> amumu is op champion!!
b:: got message >> amumu is op champion!!
c:: got message >> amumu is op champion!!
*/
```

여기서 topic이 주체이자 객체가 됩니다. **`Observer a = new TopicSubscriber("a", topic);`**를 통해 옵저버를 선언할 때 해당 이름과 어떤 토픽의 옵저버가 될 것인지를 정했습니다.

## 옵저버 패턴의 장점

- 실시간으로 한 객체의 변경사항을 다른 객체에게 알릴 수 있습니다.
- 느슨한 결합으로 시스템이 유연합니다.

## 옵저버 패턴의 단점

- 옵저버에게 알림이 가는 순서를 보장할 수 없습니다.
- 너무 많이 사용할 경우 상태 관리가 힘들어집니다.

참고)  
도서 - 면접을 위한 CS 전공지식 노트
