---
title: "백엔드 개발자에게 객체지향의 의미와 개념"
categories:
  - CS Basics
tags:
  - OOP
  - Object Oriented
  - Java
  - Spring Boot
  - CS Basics
last_modified_at:
---

### 백엔드 개발자에게 객체지향의 의미와 개념

객체지향 프로그래밍(OOP)은 현실 세계의 사물이나 개념을 프로그램의 객체로 모델링하는 개발 방식

---

#### 📌 용어 설명

- 객체지향(Object-Oriented): 프로그램을 객체들의 모임으로 보고, 객체 간의 상호작용으로 프로그램을 설계하는 방식
- 객체(Object): 데이터(속성)와 동작(메서드)을 하나로 묶은 단위, 현실 세계의 사물이나 개념을 코드로 표현한 것
- 클래스(Class): 객체를 만들기 위한 설계도 또는 틀
- 인스턴스(Instance): 클래스를 기반으로 실제로 생성된 객체
- 절차지향(Procedural): 프로그램을 순차적인 명령어의 나열로 보는 방식

#### 📌 객체지향이란 무엇인가

객체지향의 핵심 의미

객체지향은 단순히 클래스를 만드는 문법이 아니라, 프로그램을 바라보는 관점 자체를 바꾸는 패러다임
현실 세계를 모델링하듯이, 프로그램도 독립적인 객체들이 서로 협력하며 동작하도록 설계 지향

객체지향이 지향하는 것
- 복잡한 문제를 작은 객체 단위로 나누어 해결
- 각 객체가 자신의 역할과 책임을 명확히 가지도록 설계
- 객체 간의 협력을 통해 전체 시스템이 동작하도록 구성
- 변경에 유연하고 확장 가능한 구조 추구

💡 왜 객체지향인가
절차지향에서는 데이터와 함수가 분리되어 있어, 프로그램이 커질수록 어디서 무엇을 수정해야 할지 파악하기 어려움
객체지향은 관련된 데이터와 기능을 하나의 객체로 묶어, 책임을 명확히 하고 변경의 영향 범위를 최소화함

---

#### 📌 실생활 예시로 이해하는 객체지향

카페 주문 시스템으로 이해하기

현실 세계의 카페에는 손님, 바리스타, 커피, 주문서 등이 존재함
각각은 독립적인 역할을 가지고 있고, 서로 협력하여 주문이 완료됨

객체로 표현하면
- 손님 객체: 주문을 요청하는 역할
- 바리스타 객체: 커피를 만드는 역할
- 커피 객체: 커피 정보를 담고 있는 역할
- 주문서 객체: 주문 내역을 기록하는 역할

이 객체들이 협력하는 과정
1. 손님이 바리스타에게 주문 요청
2. 바리스타가 주문서 생성
3. 바리스타가 커피 제조
4. 손님에게 커피 전달

💡 각 객체는 자신의 역할만 책임지고, 다른 객체의 내부 동작은 신경 쓰지 않음
손님은 바리스타가 커피를 어떻게 만드는지 알 필요가 없고, 바리스타는 손님의 개인정보를 알 필요가 없음

---

#### 📌 절차지향 vs 객체지향 코드 비교

절차지향 방식 (데이터와 함수 분리)
```java
// 데이터만 있는 구조
class UserData {
    String name;
    String email;
    int age;
}

// 기능만 있는 함수들
class UserService {
    void sendEmail(UserData user, String message) {
        // 이메일 전송 로직
        System.out.println(user.email + "에게 전송: " + message);
    }
    
    boolean isAdult(UserData user) {
        return user.age >= 19;
    }
}

// 사용
UserData user = new UserData();
user.name = "홍길동";
user.email = "hong@example.com";
user.age = 25;

UserService service = new UserService();
service.sendEmail(user, "안녕하세요");
```

객체지향 방식 (데이터와 기능을 하나로)
```java
// 데이터와 기능이 함께 있는 객체
class User {
    private String name;
    private String email;
    private int age;
    
    public User(String name, String email, int age) {
        this.name = name;
        this.email = email;
        this.age = age;
    }
    
    // 자신의 데이터를 사용하는 기능
    public void sendEmail(String message) {
        System.out.println(this.email + "에게 전송: " + message);
    }
    
    public boolean isAdult() {
        return this.age >= 19;
    }
}

// 사용
User user = new User("홍길동", "hong@example.com", 25);
user.sendEmail("안녕하세요");
```

차이점 분석
- 절차지향: UserData는 단순 데이터 저장소, UserService가 모든 로직 처리
- 객체지향: User 객체가 자신의 데이터와 관련된 기능을 직접 처리

💡 객체지향에서는 User가 자신의 이메일을 어떻게 관리하는지, 나이를 어떻게 검증하는지 외부에서 알 필요가 없음
User 객체에게 "이메일 보내줘"라고 요청만 하면 됨

---

#### 📌 스프링부트에서 객체지향 적용 예시

실제 스프링부트 프로젝트에서의 객체지향
```java
// Order 객체 - 주문의 데이터와 행동을 함께 관리
@Entity
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String productName;
    private int quantity;
    private int price;
    private OrderStatus status;
    
    // 주문 총액 계산은 Order 자신이 처리
    public int getTotalPrice() {
        return this.quantity * this.price;
    }
    
    // 주문 취소 로직도 Order 자신이 처리
    public void cancel() {
        if (this.status == OrderStatus.COMPLETED) {
            throw new IllegalStateException("완료된 주문은 취소할 수 없습니다");
        }
        this.status = OrderStatus.CANCELLED;
    }
    
    // 주문 완료 처리
    public void complete() {
        if (this.status == OrderStatus.CANCELLED) {
            throw new IllegalStateException("취소된 주문은 완료할 수 없습니다");
        }
        this.status = OrderStatus.COMPLETED;
    }
}

// OrderService - Order 객체들을 조율하는 역할
@Service
public class OrderService {
    private final OrderRepository orderRepository;
    
    public void cancelOrder(Long orderId) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new RuntimeException("주문을 찾을 수 없습니다"));
        
        // 주문 취소는 Order 객체가 직접 처리
        order.cancel();
        orderRepository.save(order);
    }
}
```

잘못된 객체지향 예시 (객체를 단순 데이터 저장소로 사용)
```java
// 안티패턴: 데이터만 있고 행동이 없는 객체
@Entity
public class Order {
    private Long id;
    private String productName;
    private int quantity;
    private int price;
    private OrderStatus status;
    
    // getter, setter만 있음
}

// 안티패턴: Service가 모든 로직을 처리
@Service
public class OrderService {
    public void cancelOrder(Long orderId) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new RuntimeException("주문을 찾을 수 없습니다"));
        
        // Service가 Order의 내부 상태를 직접 조작
        if (order.getStatus() == OrderStatus.COMPLETED) {
            throw new IllegalStateException("완료된 주문은 취소할 수 없습니다");
        }
        order.setStatus(OrderStatus.CANCELLED);
        orderRepository.save(order);
    }
}
```

💡 첫 번째 예시는 Order 객체가 자신의 상태를 스스로 관리하고, 두 번째 예시는 Service가 Order의 내부를 직접 조작함
객체지향에서는 객체가 자신의 데이터를 스스로 보호하고 관리하는 것을 지향함

---

#### 📌 객체지향의 4가지 핵심 특징

캡슐화 (Encapsulation)

데이터와 기능을 하나로 묶고, 외부에서 함부로 접근하지 못하도록 보호하는 것
```java
public class BankAccount {
    private int balance;  // private으로 보호
    
    // 잔액 조회는 허용
    public int getBalance() {
        return balance;
    }
    
    // 입금은 유효성 검증 후 허용
    public void deposit(int amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("입금액은 0보다 커야 합니다");
        }
        this.balance += amount;
    }
    
    // 출금도 유효성 검증 후 허용
    public void withdraw(int amount) {
        if (amount > balance) {
            throw new IllegalArgumentException("잔액이 부족합니다");
        }
        this.balance -= amount;
    }
}
```

상속 (Inheritance)

기존 클래스의 특성을 물려받아 재사용하고 확장하는 것
```java
// 기본 동물 클래스
public class Animal {
    protected String name;
    
    public void eat() {
        System.out.println(name + "이(가) 먹이를 먹습니다");
    }
}

// Animal을 상속받은 Dog
public class Dog extends Animal {
    public void bark() {
        System.out.println(name + "이(가) 짖습니다: 멍멍!");
    }
}
```

다형성 (Polymorphism)

같은 인터페이스를 통해 다양한 형태의 객체를 다룰 수 있는 것
```java
// 결제 인터페이스
public interface PaymentMethod {
    void pay(int amount);
}

// 신용카드 결제
public class CreditCard implements PaymentMethod {
    public void pay(int amount) {
        System.out.println("신용카드로 " + amount + "원 결제");
    }
}

// 계좌이체 결제
public class BankTransfer implements PaymentMethod {
    public void pay(int amount) {
        System.out.println("계좌이체로 " + amount + "원 결제");
    }
}

// 사용: 결제 방식이 바뀌어도 코드 변경 없이 동작
public void processPayment(PaymentMethod method, int amount) {
    method.pay(amount);  // 어떤 방식이든 pay() 호출만 하면 됨
}
```

추상화 (Abstraction)

복잡한 내부 구현을 숨기고, 필요한 기능만 외부에 노출하는 것
```java
// 추상 클래스로 공통 기능 정의
public abstract class NotificationSender {
    // 공통 로직
    public void send(String message) {
        validate(message);
        sendMessage(message);
        log(message);
    }
    
    // 구현체마다 다르게 동작할 부분
    protected abstract void sendMessage(String message);
    
    private void validate(String message) {
        if (message == null || message.isEmpty()) {
            throw new IllegalArgumentException("메시지가 비어있습니다");
        }
    }
    
    private void log(String message) {
        System.out.println("알림 전송 완료: " + message);
    }
}

// 이메일 알림
public class EmailSender extends NotificationSender {
    protected void sendMessage(String message) {
        System.out.println("이메일 전송: " + message);
    }
}

// SMS 알림
public class SmsSender extends NotificationSender {
    protected void sendMessage(String message) {
        System.out.println("SMS 전송: " + message);
    }
}
```

---

#### 📌 객체지향을 잘하는 방법

객체에게 묻지 말고 시키기

잘못된 방식 (객체에게 데이터를 물어본 후 외부에서 처리)
```java
if (user.getAge() >= 19) {
    // 성인 관련 처리
}
```

올바른 방식 (객체에게 직접 처리를 시킴)
```java
if (user.isAdult()) {
    // 성인 관련 처리
}
```

객체의 역할과 책임을 명확히
```java
// 주문 객체는 주문 관련 로직만 처리
public class Order {
    public void cancel() { }
    public void complete() { }
    public int getTotalPrice() { }
}

// 결제 객체는 결제 관련 로직만 처리
public class Payment {
    public void process() { }
    public void refund() { }
}

// 배송 객체는 배송 관련 로직만 처리
public class Delivery {
    public void ship() { }
    public void trackStatus() { }
}
```

💡 각 객체가 자신의 역할만 충실히 수행하도록 설계하면, 변경이 발생해도 해당 객체만 수정하면 됨

---

#### 🔄 개념 비교

| 항목 | 절차지향 | 객체지향 |
| --- | --- | --- |
| 관점 | 순차적인 명령 실행 | 객체 간의 협력 |
| 데이터와 기능 | 분리되어 있음 | 하나로 묶여 있음 |
| 변경 영향 | 전체 코드에 영향 | 해당 객체에만 영향 |
| 재사용성 | 함수 단위 재사용 | 객체 단위 재사용 |
| 복잡도 관리 | 규모가 커지면 어려움 | 객체로 나누어 관리 |
| 실제 세계 모델링 | 어려움 | 직관적 |

---

#### 📌 주의사항

- 객체지향은 클래스를 만드는 문법이 아니라 설계 철학임
- getter/setter만 있는 클래스는 객체지향이 아니라 데이터 구조체임
- 모든 것을 객체로 만들 필요는 없고, 상황에 따라 절차적 코드도 적절히 사용해야 함
- 과도한 추상화는 오히려 코드를 복잡하게 만들 수 있음

---

#### 💡 요약

객체지향은 현실 세계를 모델링하듯이 프로그램을 객체들의 협력으로 설계하는 패러다임
각 객체는 자신의 데이터와 기능을 스스로 관리하며, 명확한 역할과 책임을 가짐

스프링부트 개발에서 Entity, Service, Repository는 모두 객체지향의 산물이며, 
각각의 책임을 명확히 나누는 것이 좋은 객체지향 설계

객체지향의 핵심은 복잡한 문제를 작은 객체로 나누고, 객체 간의 협력으로 해결하는 것
