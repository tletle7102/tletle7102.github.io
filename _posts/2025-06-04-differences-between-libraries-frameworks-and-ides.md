---
title: "라이브러리, 프레임워크, IDE의 차이"
categories:
  - CS Basics
tags:
  - CS Basics
last_modified_at:
---
#### 📌 용어 설명
- 라이브러리: 특정 기능을 구현한 코드 모음. 가져다 쓰는 도구
- 프레임워크: 전체 구조를 가진 뼈대. 흐름을 제어함
- IDE: 통합 개발 환경. 코드 작성, 실행, 디버깅 지원 소프트웨어

---
#### 📌 개념 정리

- 라이브러리  
필요한 기능을 호출해서 사용하는 코드 집합  
흐름의 제어권은 개발자에게 있음  
예: Lombok, JPA, Jackson, JavaScript의 Axios

- 프레임워크  
전체 애플리케이션 구조를 제공  
흐름의 제어권이 프레임워크에 있음  
개발자는 규칙을 따르며 필요한 부분만 작성  
예: Spring Boot, Spring Framework, JavaScript의 React  

- IDE  
코드 작성부터 디버깅, 실행까지 지원하는 소프트웨어  
개발 생산성을 높이기 위한 도구  
예: IntelliJ IDEA, Eclipse, Visual Studio Code  

---  
#### 📌 예시 비교  

- 라이브러리 예시  
```java  
// Lombok 예시
@Getter
@Setter
@NoArgsConstructor
public class User {
    private String name;
    private int age;
}
```
Lombok은 반복되는 코드 생성을 줄이기 위한 유틸성 라이브러리  
주도권은 개발자가 직접 클래스에 어노테이션을 붙이는 방식

```java
// JPA 예시
User user = entityManager.find(User.class, 1L);
```
JPA는 ORM 기능을 제공하는 라이브러리  
개발자가 직접 메서드를 호출해 객체를 조작  

- 프레임워크 예시  
```java
// Spring Boot Controller 예시
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello, Spring Boot!";
    }
}
```
Spring Boot가 애플리케이션 실행, 라우팅, DI 등 전체 구조를 제어  
개발자는 필요한 비즈니스 로직만 작성  

- IDE 예시  
코드 자동완성, 리팩토링, 빌드/디버그, 단축키, Git 연동, 의존성 관리 등  
IntelliJ IDEA는 Java 개발자에게 최적화된 IDE  
Spring Initializr, 테스트 실행, 설정 추적 등 다양하게 지원

---
#### 📌 요약

| 구분         | 라이브러리                          | 프레임워크                          | IDE                                  |
|--------------|-------------------------------------|-------------------------------------|--------------------------------------|
| 의미         | 기능 모음                           | 전체 구조 제공                      | 통합 개발 도구                      |
| 제어 흐름     | 개발자                              | 프레임워크                          | 해당 없음                           |
| 대표 예시     | Lombok, JPA, Jackson                | Spring Boot, Spring Framework       | IntelliJ, Eclipse, VS Code          |
| 목적         | 재사용 가능한 기능 제공             | 일관된 구조와 규칙 제공             | 개발 생산성 향상                    |

---
#### ✍ 개발 도구를 바라보는 태도

도구는 코드를 잘 짜기 위한 수단이기도 하고  
문제를 더 깊게 이해하기 위한 진입점이기도 함  

Lombok은 반복을 줄이는 길이고,  
JPA는 DB를 객체로 다루는 또 다른 시선이며  
Spring Boot는 전체 구조를 설계할 수 있게 해주는 기반  

어떤 도구든 그 철학과 흐름을 이해하고  
주체적으로 사용하는 개발자  
그게 결국 시스템을 설계할 수 있는 사람

기능보다 구조, 구조보다 태도  
코드보다 중요한 건 도구를 대하는 시선
