---
title: "JPA 기초와 엔티티 매핑 완벽 가이드"
categories:
  - Spring Boot
tags:
  - Spring Boot
  - JPA
last_modified_at:
---

#### 📌 용어 설명
- JPA: Java Persistence API. 자바 객체와 데이터베이스 간 매핑 표준
- 엔티티: 데이터베이스 테이블과 매핑되는 자바 클래스
- ORM: 객체-관계 매핑. 객체와 데이터베이스 연결 기술
- Hibernate: JPA를 구현한 ORM 프레임워크. 실제 매핑 작업 처리
- EntityManager: 엔티티의 생명주기를 관리하는 JPA의 핵심 객체
- @Entity: 엔티티 클래스임을 표시
- @Id: 엔티티의 기본 키(PK) 지정

---
#### 📌 JPA란
JPA는 자바 객체와 데이터베이스를 연결하는 표준 API  
SQL을 직접 작성하지 않고 객체 중심으로 데이터 관리  
스프링부트에서는 Spring Data JPA로 쉽게 사용 가능  

Hibernate는 JPA의 구현체로 실제 작업 담당  
EntityManager는 Hibernate와 엔티티 간 조정 역할  

엔티티 매핑은 객체와 테이블의 관계 정의  
이를 통해 데이터 조작 간소화  

---
#### 📌 JPA, Hibernate, EntityManager 쉽게 이해하기
JPA와 엔티티 매핑을 스마트 TV 시스템으로 비유  
TV(데이터베이스)는 콘텐츠(데이터)를 저장  
리모컨(엔티티)은 TV를 조작하는 인터페이스  
JPA는 리모컨의 표준 설계도  
Hibernate는 리모컨의 내부 회로(ORM). 실제 신호 처리  
EntityManager는 리모컨의 버튼 관리자. 어떤 버튼(엔티티 속성)이 어떤 신호(데이터 작업)를 보내는지 조정  

예: 채널 변경(데이터 수정)을 요청  
리모컨(엔티티)의 버튼(속성)을 누르면  
EntityManager가 요청을 정리해 Hibernate에 전달  
Hibernate가 TV(데이터베이스)에 신호를 보내 변경 완료  

마리오네트 인형으로도 비유 가능  
인형(데이터베이스)은 줄(매핑)로 움직임  
인형사(엔티티)가 줄을 조작해 동작 제어  
JPA는 줄 조작의 표준 규칙  
Hibernate는 줄의 실제 연결과 움직임(ORM) 담당  
EntityManager는 인형사의 손. 줄을 어떻게 당길지 결정  

이처럼 JPA는 표준을 제공하고  
Hibernate가 매핑을 실행하며  
EntityManager가 엔티티와 데이터베이스 간 작업을 조율  

---
#### 📌 JPA 설정 예시
스프링부트에서 JPA 설정은 application.yml에서 시작  

```yml  
spring:
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
    username: sa
    password:
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true
```  

H2 데이터베이스 사용 예시  
ddl-auto는 개발 환경에서 테이블 자동 생성  
Hibernate가 내부적으로 매핑 작업 수행  

---
#### 📌 엔티티 매핑 구현
##### ✅ 의존성 추가
build.gradle에 spring-boot-starter-data-jpa 추가  
```groovy  
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
runtimeOnly 'com.h2database:h2'
```  

##### ✅ 엔티티 클래스 작성
User 테이블과 매핑되는 엔티티  
```java  
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "username", nullable = false, length = 50)
    private String username;

    @Column(name = "email", unique = true)
    private String email;

    public User() {}

    public User(String username, String email) {
        this.username = username;
        this.email = email;
    }

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}
```  

@Entity: 클래스와 테이블 매핑  
@Id: 기본 키 지정  
@Column: 컬럼 속성 정의  

TV 비유로 보면  
User는 리모컨, username과 email은 버튼  
EntityManager가 버튼 신호를 Hibernate에 전달  
Hibernate가 데이터베이스에 반영  

##### ✅ 리포지토리 인터페이스
JPA 리포지토리로 CRUD 자동화  
```java  
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
}
```

JpaRepository는 EntityManager와 Hibernate 활용  
기본 CRUD 메서드 자동 제공  

##### ✅ 서비스에서 사용
```java  
@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User saveUser(String username, String email) {
        User user = new User(username, email);
        return userRepository.save(user);
    }
}
```

saveUser는 리모컨 버튼 누르는 행위  
EntityManager가 작업을 조정하고  
Hibernate가 데이터베이스에 저장  

---
#### 📌 주요 엔티티 매핑 어노테이션
- @Table: 테이블 이름 지정
- @Column: 컬럼 속성(길이, null 여부 등) 정의
- @GeneratedValue: 기본 키 값 생성 전략
- @ManyToOne, @OneToMany: 관계 매핑
- @Transient: 매핑 제외 필드  

---
#### 📌 주의사항
1. @Id는 반드시 지정  
2. 엔티티 클래스는 기본 생성자 필수  
3. ddl-auto는 프로덕션에서 create, update 주의  
4. 관계 매핑은 순환 참조 방지  
5. EntityManager의 영속성 컨텍스트 관리 주의  

---
#### 📌 JPA와 엔티티 매핑의 의미
JPA는 표준으로 개발 편의성 제공  
Hibernate는 복잡한 매핑 작업 처리  
EntityManager는 엔티티 생명주기 관리  

올바른 매핑은 유지보수성과 확장성 향상  
스프링부트는 설정 부담 줄여 핵심 로직 집중 가능  

---
#### ✍ 알아보면서
JPA와 Hibernate, EntityManager는 처음엔 복잡했음  
TV와 마리오네트 비유로 역할 분담이 명확해짐  

잘못된 매핑은 데이터 무결성 문제 유발  
EntityManager의 조정 역할이 설계의 핵심  

JPA는 단순한 도구가 아니라  
객체와 데이터베이스의 패러다임 연결  
이를 통해 코드보다 설계에 집중 가능  

---