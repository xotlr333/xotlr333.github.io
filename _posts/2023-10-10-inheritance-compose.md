# 상속 VS 합성

코드 중복을 제거하여 재사용 함으로써 변경, 확장을 용이하게 만드는 기술로 상속과 합성이 있다.

# 상속

상속은 객체 지향 4가지 특징 중 하나로서 클래스 기반의 프로그래밍에서 가장 먼저 배우는 개념이다.

클래스 상속을 통해 자식 클래스는 부모 클래스의 자원을 물려 받게 되며, 부모 클래스와 다른 부분만 추가하거나 재정의함으로써 기존 코드를 쉽게 확장할 수 있다.

그래서 상속 관계는 **`is-a 관계`**라고 표현하기도 한다.

![상속관계](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/33383209-49e9-4e33-a8ef-c5bdc21f0d8e)

하지만, 상속은 엄밀히 말하면 그저 코드 재상용을 위한 기법이 아니다.

일반적인 클래스가 이미 구현이 되어 있는 상태에서 그보다 좀 더 구체적인 클래스를 구현하기 위해 사용되는 기법이며, 그로 인해 상위 클래스의 코드를 하위 클래스가 재사용할 수 있을 뿐이다.

### 코드 재사용으로 상속을 이용할 경우

- 상속을 제대로 활용하기 위해서는 부모클래스의 내부 구현에 대해 상세하게 알아야 하기 때문에 자식 클래스와 부모 클래스 사이의 결합도가 높아진다.
- 상속 관계는 컴파일 타임에서 결정되고 고정되기 때문에 코드를 실행하는 도중에 변경할 수 없다
- 클래스 폭발 문제 : 여러 기능을 조합해야 하는 설계에 상속을 이용하게 된다면 모든 조합별로 클래스를 하나 하나 추가해주어야 한다.
- java8부터는 인터페이스에서 로직 구현이 가능하여 상속의 장점이 약화되고 있다. 그래서 클래스 상속보다는 인터페이스 구현을 이용하라는 풍문이 돌아다닌다.

> **결과적으로 상속은 클래스간의 관계를 한눈에 파악할 수 있고 코드를 재사용할 수 있는 쉽고 간단한 방법일지는 몰라도 좋은 방법이라고 할 수 없다.**

# 합성

합성 기법은 기존 클래스를 상속을 통한 확장하는 대신에, 필드로 클래스의 인스턴스를 참조하게 만드는 설계이다.

서로 관련없는 이질적인 클래스의 관계에서, 한 클래스가 다른 클래스의 기능을 사용하여 구현해야 한다면 합성의 방식을 사용한다고 보면 된다.

```java
class Car {
    Engine engine; // 필드로 Engine 클래스 변수를 갖는다(has)

    Car(Engine engine) {
        this.engine = engine; // 생성자 초기화 할때 클래스 필드의 값을 정하게 됨
    }

    void drive() {
        System.out.printf("%s엔진으로 드라이브~\n", engine.EngineType);
    }

    void breaks() {
        System.out.printf("%s엔진으로 브레이크~\n", engine.EngineType);
    }
}

class Engine {
    String EngineType; // 디젤, 가솔린, 전기

    Engine(String type) {
        EngineType = type;
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Car digelCar = new Car(new Engine("디젤"));
        digelCar.drive(); // 디젤엔진으로 드라이브~

        Car electroCar = new Car(new Engine("전기"));
        electroCar.drive(); // 전기엔진으로 드라이브~
    }
}
```

위의 초기화 코드에서 볼 수 있듯이, 마치 new 생성자에 new 생성자를 받는 형식으로 쓰여진다.

이 방식을 포워딩이라고 하며 필드의 인스턴스를 참조해 사용하는 메소드를 포워딩 메소드라고 부른다.

그래서 클래스 간의 합성 관계를 사용하는데 다른 말로 **`Has-A 관계`**라고 한다.

객체지향에서 다른 클래스를 활용하는 기본적인 방법이 바로 합성을 활용하는 것이다.

합성을 이용할때 꼭 클래스 뿐만 아니라 추상 클래스, 인터페이스로도 가능하다.

# 상속 보다는 합성

정말 개념적으로 연관 관계가 있을 때만 상속을 이용하고 가능하면 지양하는 편이다.

## 상속의 단점

### 1. 결합도가 높아짐

객체지향 프로그래밍에서는 결합도는 낮을수록, 응집도는 높을수록 좋다. 그래서 추상화에 의존함으로써 다른 객체에 대한 결합도는 최소화하고 응집도를 최대화하여 변경 가능성을 최소화 할 수 있다.

하지만 상속을 하게 되면 부모 클래스와 자식 클래스의 관계가 컴파일 시점에 관계가 결정되어 결합도가 당연히 높아질 수 밖에 없다.

컴파일 시점에서 결정되는 관계는 유연성이 상당히 떨어뜨리고, 실행 시점에 객체의 종류를 변경하는 것이 불가능하여 유기적인 다형성 및 객체지향 기술을 사용할 수 없다.

### 2. 불필요한 기능 상속

부모 클래스에 메소드를 추가했을때, 자식 클래스에는 적합하지 않는 메소드가 상속되는 문제가 있다.

### 3. 부모 클래스의 결함이 그대로 넘어옴

만일 상위 클래스에 결함이 있다면, 이를 상속하게 되면 부모 클래스의 결함도 자식 클래스에게 넘어오게 된다.

결국 자식 클래스에서 아무리 구조적으로 잘 설계하였다 하더라도 애초에 부모 클래스에서 결함이 있기 때문에 자식 클래스도 문제가 터진다.

### 4. 부모 클래스와 자식 클래스의 동시 수정 문제

부모 클래스와 자식 클래스 사이의 개념적인 결합으로 인해, 부모 클래스를 변경할 때 자식 클래스도 함께 변경해야 하는 문제가 있다.

```java
class Food {
    final int price;

    Food(int price) {
        this.price = price;
    }
}

class Bread extends Food {
    public Bread(int price) {
        super(price);
    }
}

public class Main {
    public static void main(String[] args) {
        Food bread = new Bread(1000);
    }
}
```

```java
class Food {
    final int price;
    final int count; // 코드 추가

    Food(int price, int count) {
        this.price = price;
        this.count = count; // 코드 추가
    }
}

class Bread extends Food {
    public Bread(int price, int count) {
        super(price, count); // 코드 수정
    }
}

public class Main {
    public static void main(String[] args) {
        Food bread = new Bread(1000, 5); // 코드 수정
    }
}
```

위와 같이 자식 클래스는 물론 클래스 호출 부분까지 전부 수정해야한다.

### 5. 불필요한 인터페이스 상속 문제

자바의 초기 버전에서 상속을 잘못 사용한 대표전인 사례는 java.util.Properies와 java.util.Stack이다.

두 클래스의 공통점은 부모 클래스에서 상속받은 메소드를 사용할 경우 자식 클래스의 규칙이 위반 될 수 있다.

<img width="290" alt="인터페이스 상속" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/056dc9f5-fb6a-425f-a5ba-8e41accfaaf9">

즉, 자식 클래스에게는 부적합한 부모 클래스의 메소드가 상속되기 때문에 자식 클래스 인스턴스의 상태가 불안정해진다.

예를 들어 Stack의 대표적인 동작은 push, pop이지만, 상속산 Vector 클래스의 add 메소드 또한 외부로 노출되게 된다. 그러면서 아래와 같이 개발자가 add 메소드도 스택 클래스에서 사용할 수 있는 메소드인줄 알고 사용했다가 의도치 않은 동작이 실행되면서 오류를 범하게 된다.

```java
Stack<String> stack = new Stack<>();
stack.push("one");
stack.push("two");
stack.push("three");

stack.add(0, "four"); // add 메소드를 호출함으로써 stack의 의미와는 다르게 특정 인덱스의 값이 추가

String str = stack.pop(); // three
System.out.println(str.equals("four")); // false
```

### 6. 클래스 폭발

상속을 남용하여 필요 이상으로 많은 수의 클래스를 추가해야 하는 클래스 폭발 문제가 발생한다.

만약 새로운 요구사항이 생겨 Steak와 Saled 그리고 Pasta로 구성된 세트 메뉴를 추가해야 하는 상황이라고 하자. 그러면 우리는 이를 해결하기 위해 다음과 같이 SteakWithSaladSetAndPasta 클래스를 또 추가해야 한다.

```java
public class SteakWithSaladSetAndPasta extends SteakWithSaladSet {
     ...
}
```

그리고 새로운 메뉴가 또 개발된다면 계속해서 해당 클래스를 추가해야 할 것이고 지나치게 많은 클래스가 생겨야 하는 클래스 폭발 문제가 발생할 것이다.

클래스 폭발 문제는 자식 클래스가 부모 클래스의 구현과 강하게 결합되도록 강요하는 상속의 근본적인 한계 때문에 발생한다. 컴파일 타임에 결정된 자식 클래스와 부모 클래스 사이의 관계는 변경될 수 없기 때문에 자식 클래스와 부모 클래스의 다양한 조합이 필요한 상황에서 유일한 해결 방법은 조합의 수 만큼 새로은 클래스를 추가하는 것 뿐이다.

따라서 여러 기능을 조합해야 하는 설계에 상속을 이용하면 모든 조합 가능한 경우 별로 클래스를 추가해야 한다. 그러므로 이러한 문제를 방지하기 위해서라도 상속보다는 합성을 이용해야한다.

### 6. 단일 상속의 한계

자바에서는 클래스 다중 상속을 허용하지 않는다.

그렇기 때문에 상속이 필요한 해당 클래스가 다른 클래스를 이미 상속 중이라면 문제가 발생할 수 있다. 결국 클래스를 또 나누어 구성해야하는데 결국 클래스 폭발로 이어지게 될 것이다.

결국은 다중 상속 한계점 때문에 인터페이스를 사용하듯이 클래스 상속의 근본적인 문제는 단일 상속 밖에 못한다는 것이다.

## 합성을 사용 해야하는 이유

상속은 자식 클래스 정의에 부모 클래스의 이름을 덧 붙이는 것만으로 코드를 재사용할 수 있으며 쉽게 확장할 수 있다. 그러나 상속을 제대로 활용하기 위해서는 부모 클래스의 내부 구현에 대해 상세히 알아야 하기 때문에 자식과 부모 사이의 **`결합도가 높아질 수 밖에 없다`**.

결과적으로 상속은 코드를 재사용하기 쉬운 방법이긴 하지만 결합도가 높아지는 치명적인 단점이 있다.

또한, 상속 관계는 클래스 사이의 정적인 관계인데 합성 관계는 객체 상이의 동적인 관계이다. 코드 작성 시점에 결정한 상속 관계는 변경이 불가능하지만(컴파일 타임), 합성 관계는 실행 시점에 동적으로 변경할 수 있기 때문이다(런타임).

합성을 이용한 대표적인 디자인 패턴 중에는 **`전략 패턴`**이 있다.

합성은 단순히 메소드 호출을 통해 값을 사용하면 되기 때문에 구현이 어렵지 않다.

그러나 합성에도 단점이 존재하는데, 아무래도 상속과 달리 클래스간의 관계를 파악하는데 있어 시간이 걸린다는 점이다. 한마디로 코드가 복잡해질 수 있다는 점을 떠안고 있다.

### 불필요한 인터페이스 상속 문제 해결

위에서 알아본 Vector 를 상속한 Stack 예제를 다시 보겠다.

Vector를 상속 받아서 사용했기 때문에 문제가 발생하였기 때문에, Stack 클래스 설계서부터 상속을 제거하고 Vector 클래스를 내부 필드로 바꾸어 주면 된다.

```java
public class Stack<E> {
    private Vector<E> elements = new Vector<>(); // 합성

    public E push(E item) {
        elements.addElement(item);
        return item;
    }

    public E pop() {
        if (elements.isEmpty()) {
            throw new EmptyStackException();
        }
        return elements.remove(elements.size() -1);
    }
}
```

이렇게 구성하면, 이제 문제가 되었던 add와 같은 메소드를 사용하는 실수를 원천 봉쇄할 수 있다. 이렇듯 상속을 합성 관계로 변경함으로써 Stack의 규칙이 어겨졌던 것을 막을 수 있다.

### 단일 상속 문제 해결

또한 단일 상속 한계를 어느정도 해결할 수 있다.

클래스 객체 기능이 필요하다면 필요한 만큼 필드에 정의해 두어 사용하면 되기 때문이다.

```java
public class Phone {
    private RatePolicy ratePolicy; // 클래스 합성
    private List<Call> calls = new ArrayList<>(); // 클래스 합성

    public Phone(RatePolicy ratePolicy) {
        this.ratePolicy = ratePolicy;
    }

    public List<Call> getCalls() {
        return Collections.unmodifiableList(calls);
    }
}
```

참고)

[https://inpa.tistory.com/entry/OOP-💠-객체-지향의-상속-문제점과-합성Composition-이해하기](https://inpa.tistory.com/entry/OOP-%F0%9F%92%A0-%EA%B0%9D%EC%B2%B4-%EC%A7%80%ED%96%A5%EC%9D%98-%EC%83%81%EC%86%8D-%EB%AC%B8%EC%A0%9C%EC%A0%90%EA%B3%BC-%ED%95%A9%EC%84%B1Composition-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0)

[https://mangkyu.tistory.com/199](https://mangkyu.tistory.com/199)
