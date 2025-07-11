---
title: "Spring Boot 개발 시 자주하는 실수들과 해결법"
categories:
  - Spring Boot
tags:
  - Spring Boot
  - Best Practices
last_modified_at:
---

#### 📌 용어 설명
- DI: 의존성 주입. 객체 간 의존 관계 자동 관리
- JPA: 자바 객체와 데이터베이스 매핑 API
- Hibernate: JPA 구현체. 객체-테이블 매핑 처리
- EntityManager: 엔티티 생명주기 관리
- Transaction: 데이터베이스 작업의 논리적 단위
- EntityGraph: JPA에서 엔티티와 연관된 데이터를 명시적으로 로드하는 방법
- N+1 문제: 연관 엔티티 조회 시 쿼리가 과다 실행되는 성능 문제

---
#### 📌 실수와 해결법이란
Spring Boot 개발에서 자주하는 실수들은  
설정 오류, JPA 오용, 트랜잭션 관리 부실 등  
이로 인해 성능 저하, 데이터 불일치 발생  

JPA와 Hibernate로 데이터 관리  
EntityManager가 엔티티와 데이터베이스 작업 조율  
올바른 사용법으로 문제 예방 가능  

---
#### 📌 실수와 해결법 쉽게 이해하기
Spring Boot를 노트북의 파일 관리 시스템에 비유  
노트북(애플리케이션)은 파일(데이터) 관리  
JPA는 파일 관리 규칙  
Hibernate는 파일 저장/수정 로직(ORM)  
EntityManager는 파일 관리자. 어떤 파일을 어떻게 처리할지 정리  

실수는 파일을 잘못 저장하거나 삭제하는 상황  
예: 파일 덮어쓰기(데이터 불일치) 방지 위해  
EntityManager가 작업 순서 정리, Hibernate가 실행  

---
#### 📌 자주하는 실수와 해결법
H2 데이터베이스 사용 예시로 설명  

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

##### ✅ 실수 1: 트랜잭션 누락
데이터 수정 작업에서 @Transactional 누락  
EntityManager가 작업을 일관되게 관리 못 함  
결과: 데이터 불일치  

**해결법**  
서비스 메서드에 @Transactional 추가  
```java  
@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Transactional
    public User updateUser(Long id, String username) {
        User user = userRepository.findById(id).orElseThrow();
        user.setUsername(username);
        return user;
    }
}
```  

노트북 비유: 파일 저장 시 저장 버튼 누르기  
@Transactional은 저장 확인 과정  

##### ✅ 실수 2: JPA N+1 문제
**N+1 문제란**  
엔티티와 연관된 데이터를 조회할 때  
기본 엔티티 조회 쿼리 1번과 연관 엔티티별 쿼리 N번 실행  
예: 10명의 사용자와 팀 조회 시 1(사용자 목록) + 10(각 팀) = 11번 쿼리  

**원인**  
@ManyToOne에 FetchType.EAGER 사용  
Hibernate가 연관 엔티티를 즉시 로드하며 쿼리 추가 실행  
EntityManager가 각 연관 엔티티를 별도로 요청  

**해결법**  
FetchType.EAGER 대신 LAZY 사용  
LAZY는 연관 엔티티를 필요할 때만 로드  
쿼리 수 감소로 성능 개선  
필요 시 EntityGraph로 명시적 로드  

**EntityGraph란**  
JPA에서 엔티티와 연관 데이터를 한 번에 로드하는 방법  
@NamedEntityGraph로 정의하거나 동적으로 생성  
N+1 문제를 해결하며 쿼리 최적화  

```java  
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "username")
    private String username;

    @ManyToOne(fetch = FetchType.LAZY)
    private Team team;

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    public Team getTeam() { return team; }
    public void setTeam(Team team) { this.team = team; }
}

@Entity
public class Team {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "name")
    private String name;

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}

@NamedEntityGraph(
    name = "User.withTeam",
    attributeNodes = @NamedAttributeNode("team")
)
@Entity
public class User {
    // ... (위와 동일)
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    @EntityGraph(value = "User.withTeam")
    List<User> findAll();
}
```  

노트북 HYPERLINK "file://localhost/ 비유": 폴더 전체 대신 필요한 파일만 열기  
LAZY는 필요 파일만 로드, EntityGraph는 관련 파일 함께 로드  
EAGER는 불필요한 파일까지 열어 성능 저하  

**왜 LAZY로 해결되나**  
EAGER는 연관 데이터를 즉시 로드해 쿼리 수 증가  
LAZY는 요청 시점에 로드해 불필요한 쿼리 방지  
EntityGraph는 필요한 연관 데이터만 한 번에 로드  

##### ✅ 실수 3: ddl-auto 프로덕션 사용
application.yml에서 ddl-auto: create 사용  
테이블 자동 삭제/생성으로 데이터 손실  
결과: 프로덕션 환경에서 치명적 오류  

**해결법**  
프로덕션에서는 ddl-auto: none 또는 validate  
```yml  
spring:
  jpa:
    hibernate:
      ddl-auto: none
```  

노트북 비유: 중요한 파일 자동 삭제 방지  
none은 파일 구조 변경 금지  

##### ✅ 실수 4: 의존성 주입 잘못 사용
@Autowired 직접 사용 또는 필드 주입  
테스트와 유지보수 어려움  

**해결법**  
생성자 주입 사용  
```java  
@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```  

노트북 비유: 파일 관리 도구를 명확히 연결  
생성자 주입은 도구 사용 방식 고정  

##### ✅ 실수 5: 엔티티 직접 노출
컨트롤러에서 엔티티를 직접 반환  
Hibernate 프록시 문제로 JSON 직렬화 오류  

**해결법**  
DTO로 변환 후 반환  
```java  
public class UserDto {
    private Long id;
    private String username;

    public UserDto(User user) {
        this.id = user.getId();
        this.username = user.getUsername();
    }

    // Getters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
}

@RestController
@RequestMapping("/api/users")
public class UserController {
    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/{id}")
    public UserDto getUser(@PathVariable Long id) {
        User user = userService.findUser(id);
        return new UserDto(user);
    }
}
```  

노트북 비유: 파일 원본 대신 복사본 제공  
DTO는 원본 데이터 보호  

---
#### 📌 의존성 설정
build.gradle에 JPA와 H2 의존성  
```groovy  
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    runtimeOnly 'com.h2database:h2'
}
```  

---
#### 📌 실수 예방의 의미
올바른 설정과 사용법으로 안정성 향상  
JPA와 Hibernate로 데이터 관리 간소화  
EntityManager가 작업 일관성 보장  
문제 예방은 개발 효율성 증가  

---
#### ✍ 알아보면서
작은 실수가 큰 오류로 이어질 수 있음  
설정과 테스트의 중요성 깨달음  

---