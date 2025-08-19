---
title: "JPA Query Methods 활용법"
categories:
  - Spring Boot
tags:
  - Spring Boot
  - JPA
  - Query Methods
last_modified_at:
---

#### 📌 용어 설명
- JPA Query Methods: 메서드 이름으로 SQL 쿼리 자동 생성
- Spring Data JPA: JPA를 쉽게 사용하기 위한 스프링 모듈
- Hibernate: JPA 구현체. 객체-테이블 매핑 처리
- EntityManager: 엔티티 생명주기 관리
- Repository: 데이터 접근 계층. 쿼리 메서드 정의

---
#### 📌 JPA Query Methods란
JPA Query Methods는 메서드 이름을 규칙에 맞게 작성하면  
Spring Data JPA가 자동으로 SQL 쿼리 생성  
복잡한 SQL 작성 없이 데이터 조회/조작 가능  

Hibernate가 쿼리 실행  
EntityManager가 엔티티와 데이터베이스 간 작업 조율  

---
#### 📌 Query Methods 쉽게 이해하기
Query Methods를 노트북의 검색 기능에 비유  
노트북(데이터베이스)은 파일(데이터) 저장  
검색창(Repository)에 키워드(메서드 이름) 입력  
JPA는 검색 규칙  
Hibernate는 파일 찾기 로직(ORM)  
EntityManager는 검색 관리자. 어떤 파일을 어떻게 찾을지 정리  

예: "user"로 시작하는 파일 찾기(Read)  
검색창에 "findByUsernameStartingWith" 입력(메서드 정의)  
EntityManager가 요청 정리해 Hibernate에 전달  
Hibernate가 노트북에서 파일 검색(데이터 조회)  

Query Methods는 검색 키워드처럼 간단히 데이터 접근  

---
#### 📌 Query Methods 설정 예시
스프링부트에서 Query Methods는 Repository 인터페이스에서 시작  
H2 데이터베이스 사용 예시  

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

H2 콘솔로 쿼리 결과 확인 가능  

---
#### 📌 Query Methods 구현
##### ✅ 의존성 추가
build.gradle에 H2와 JPA 의존성 추가  
```groovy  
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'com.h2database:h2'
}
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

@Entity: 노트북에 저장할 파일 구조 정의  
노트북 비유로 보면  
User는 파일, username과 email은 파일 내용  
EntityManager가 검색 요청 정리, Hibernate가 실행  

##### ✅ 리포지토리 인터페이스
Query Methods 정의  
```java  
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    // username으로 시작하는 사용자 조회
    List<User> findByUsernameStartingWith(String prefix);

    // email로 사용자 조회
    Optional<User> findByEmail(String email);

    // username과 email 모두 포함하는 사용자 조회
    List<User> findByUsernameAndEmail(String username, String email);

    // username으로 정렬된 사용자 조회
    List<User> findByOrderByUsernameAsc();
}
```  

메서드 이름이 쿼리 생성  
findByUsernameStartingWith는 "username LIKE 'prefix%'" 생성  
EntityManager가 요청 조정, Hibernate가 쿼리 실행  

##### ✅ 서비스에서 사용
```java  
@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public List<User> findUsersByUsernamePrefix(String prefix) {
        return userRepository.findByUsernameStartingWith(prefix);
    }

    public Optional<User> findUserByEmail(String email) {
        return userRepository.findByEmail(email);
    }

    public List<User> findUsersByUsernameAndEmail(String username, String email) {
        return userRepository.findByUsernameAndEmail(username, email);
    }

    public List<User> findAllUsersSortedByUsername() {
        return userRepository.findByOrderByUsernameAsc();
    }

    public User createUser(String username, String email) {
        User user = new User(username, email);
        return userRepository.save(user);
    }
}
```  

findByUsernameStartingWith는 노트북에서 파일 검색  
EntityManager가 검색 조건 정리, Hibernate가 결과 반환  

##### ✅ 컨트롤러로 테스트
```java  
@RestController
@RequestMapping("/api/users")
public class UserController {
    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @PostMapping
    public User createUser(@RequestParam String username, @RequestParam String email) {
        return userService.createUser(username, email);
    }

    @GetMapping("/prefix")
    public List<User> findUsersByUsernamePrefix(@RequestParam String prefix) {
        return userService.findUsersByUsernamePrefix(prefix);
    }

    @GetMapping("/email")
    public Optional<User> findUserByEmail(@RequestParam String email) {
        return userService.findUserByEmail(email);
    }

    @GetMapping("/search")
    public List<User> findUsersByUsernameAndEmail(@RequestParam String username, @RequestParam String email) {
        return userService.findUsersByUsernameAndEmail(username, email);
    }

    @GetMapping("/sorted")
    public List<User> findAllUsersSortedByUsername() {
        return userService.findAllUsersSortedByUsername();
    }
}
```  

REST API로 Query Methods 테스트  

---
#### 📌 Query Methods의 장점
- SQL 작성 없이 메서드 이름으로 쿼리 생성  
- 코드 가독성과 유지보수성 향상  
- JPA와 Hibernate로 자동화  
- 초보자도 쉽게 데이터 조회 가능  

---
#### 📌 주의사항
1. 메서드 이름은 JPA 규칙 준수  
2. 복잡한 쿼리는 @Query 어노테이션 고려  
3. EntityManager의 영속성 컨텍스트 관리 주의  
4. 쿼리 성능 테스트 필수  
5. show-sql로 생성된 쿼리 확인 권장  

---
#### 📌 Query Methods 활용의 의미
Query Methods는 데이터 조회 간소화  
JPA와 Hibernate로 쿼리 자동화  
EntityManager가 엔티티와 데이터베이스 연결 조율  

초보자도 직관적으로 데이터 접근 가능  
핵심 로직 개발에 집중할 시간 확보  

---
#### ✍ 알아보면서
Query Methods는 처음엔 이름 짓기가 낯설었음    

잘못된 메서드 이름은 쿼리 오류 유발  
규칙 학습과 테스트의 중요성 깨달음  

Query Methods는 데이터 접근의 강력한 도구  
이를 통해 설계와 기능 구현에 더 집중 가능  

---