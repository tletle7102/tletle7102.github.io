---
title: "H2 데이터베이스로 빠른 개발 환경 구축하기"
categories:
  - Spring Boot
tags:
  - Spring Boot
  - H2 Database
last_modified_at:
---

#### 📌 용어 설명
- H2: 경량 인메모리 데이터베이스. 개발과 테스트에 최적화
- 인메모리: 데이터를 메모리에 저장. 속도 빠름
- JDBC: 자바와 데이터베이스 연결 표준
- JPA: 자바 객체와 데이터베이스 매핑 API
- Hibernate: JPA 구현체. 객체-테이블 매핑 처리
- EntityManager: 엔티티 생명주기 관리

---
#### 📌 H2 데이터베이스란
H2는 가볍고 빠른 데이터베이스  
개발 환경에서 빠르게 테스트 가능  
인메모리 모드로 메모리에 데이터 저장  
설치 간단, 스프링부트와 통합 쉬움  

JPA와 Hibernate로 객체 기반 데이터 관리  
EntityManager가 엔티티와 H2 간 작업 조율  

---
#### 📌 H2 데이터베이스 쉽게 이해하기
H2를 노트북의 메모장으로 비유  
메모장(H2)은 데이터를 빠르게 기록하고 수정  
노트북(애플리케이션)은 메모장을 쉽게 사용  
JPA는 메모장 사용 규칙  
Hibernate는 메모장의 저장/수정 로직(ORM)  
EntityManager는 메모장 관리자. 어떤 데이터를 어떻게 기록할지 결정  

예: 메모장에 이름 기록(데이터 저장)  
사용자가 이름 입력(엔티티 생성)하면  
EntityManager가 입력을 정리해 Hibernate에 전달  
Hibernate가 메모장에 기록(H2에 저장)  

H2는 가볍고 빠른 메모장 역할  
JPA, Hibernate, EntityManager가 협력해 데이터 관리  

---
#### 📌 H2 설정 예시
스프링부트에서 H2 설정은 application.yml에서 시작  

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
  h2:
    console:
      enabled: true
      path: /h2-console
```

jdbc:h2:mem:testdb는 인메모리 모드  
h2-console은 웹 UI로 데이터 확인 가능  

---
#### 📌 H2 개발 환경 구현
##### ✅ 의존성 추가
build.gradle에 H2와 JPA 의존성 추가  
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

@Entity: 메모장에 기록할 데이터 구조 정의  
메모장 비유로 보면  
User는 메모장의 한 페이지, username과 email은 항목  
EntityManager가 항목을 정리, Hibernate가 기록  

##### ✅ 리포지토리 인터페이스
H2와 CRUD 작업 자동화  
```java  
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
}
```  

JpaRepository는 EntityManager와 Hibernate 사용  
메모장에 데이터를 기록/조회하는 메서드 제공  

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

saveUser는 메모장에 항목 기록  
EntityManager가 작업 조정, Hibernate가 H2에 저장  

##### ✅ H2 콘솔 확인
애플리케이션 실행 후  
http://localhost:8080/h2-console 접속  
JDBC URL: jdbc:h2:mem:testdb  
테이블과 데이터 확인 가능  

---
#### 📌 H2의 장점
- 설치 불필요. 의존성 추가로 즉시 사용  
- 인메모리 모드로 빠른 테스트  
- H2 콘솔로 데이터 시각화  
- JPA와 Hibernate 통합 쉬움  

---
#### 📌 주의사항
1. 인메모리 모드는 애플리케이션 종료 시 데이터 삭제  
2. 프로덕션 환경에는 부적합. 테스트용으로 사용  
3. ddl-auto 설정은 개발 환경에 맞게 조정  
4. H2 콘솔은 보안상 프로덕션에서 비활성화  
5. EntityManager의 영속성 컨텍스트 관리 주의  

---
#### 📌 H2로 개발 환경 구축의 의미
H2는 빠른 개발과 테스트 지원  
JPA와 Hibernate로 데이터 관리 간소화  
EntityManager가 엔티티와 H2 간 연결 조율  
핵심 로직 개발에 집중할 시간 확보  

---
#### ✍ 알아보면서
H2는 처음엔 단순한 데이터베이스처럼 보였음  
JPA, Hibernate, EntityManager의 협력 이해  

잘못된 설정은 테스트 결과 왜곡  
정확한 환경 구성의 중요성 깨달음  

H2는 개발 속도를 높이는 도구  
이를 통해 설계와 로직에 더 집중 가능  

---