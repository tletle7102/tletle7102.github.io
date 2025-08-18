---
title: "MySQL 연동하고 데이터 CRUD 구현하기"
categories:
  - Spring Boot
tags:
  - Spring Boot
  - MySQL
  - JPA
last_modified_at:
---

#### 📌 용어 설명
- MySQL: 관계형 데이터베이스. 데이터 영구 저장
- CRUD: Create, Read, Update, Delete. 데이터 기본 작업
- JPA: 자바 객체와 데이터베이스 매핑 API
- Hibernate: JPA 구현체. 객체-테이블 매핑 처리
- EntityManager: 엔티티 생명주기 관리
- JDBC: 자바와 데이터베이스 연결 표준

---
#### 📌 MySQL 연동과 CRUD란
MySQL은 데이터를 영구 저장하는 데이터베이스  
스프링부트와 연동해 데이터 관리 가능  
CRUD는 데이터 생성, 조회, 수정, 삭제 작업  

JPA와 Hibernate로 객체 기반 CRUD 구현  
EntityManager가 엔티티와 MySQL 간 작업 조율  

---
#### 📌 MySQL 연동 쉽게 이해하기
MySQL을 노트북의 외장 하드에 비유  
외장 하드(MySQL)는 데이터를 영구 저장  
노트북(애플리케이션)은 하드에 데이터 기록/조회  
JPA는 하드 사용 규칙  
Hibernate는 데이터 저장/수정 로직(ORM)  
EntityManager는 파일 관리자. 어떤 데이터를 어디에 저장할지 정리  

예: 하드에 사용자 정보 저장(Create)  
사용자가 정보 입력(엔티티 생성)하면  
EntityManager가 데이터를 정리해 Hibernate에 전달  
Hibernate가 하드에 저장(MySQL에 삽입)  

MySQL은 안정적인 저장소 역할  
JPA, Hibernate, EntityManager가 협력해 CRUD 구현  

---
#### 📌 MySQL 설정 예시
스프링부트에서 MySQL 설정은 application.yml에서 시작  

```yml  
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/testdb
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: your-password
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true
```  

MySQL 서버 실행 후 testdb 생성  
url, username, password는 환경에 맞게 설정  

---
#### 📌 MySQL과 CRUD 구현
##### ✅ 의존성 추가
build.gradle에 MySQL과 JPA 의존성 추가  
```groovy  
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'mysql:mysql-connector-java'
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

@Entity: 외장 하드에 저장할 데이터 구조 정의  
외장 하드 비유로 보면  
User는 하드의 파일, username과 email은 파일 내용  
EntityManager가 내용을 정리, Hibernate가 저장  

##### ✅ 리포지토리 인터페이스
MySQL과 CRUD 작업 자동화  
```java  
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
}
```  

JpaRepository는 EntityManager와 Hibernate 사용  
외장 하드에 데이터를 기록/조회하는 메서드 제공  

##### ✅ 서비스에서 CRUD 구현
```java  
@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    // Create
    public User createUser(String username, String email) {
        User user = new User(username, email);
        return userRepository.save(user);
    }

    // Read
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }

    public Optional<User> getUserById(Long id) {
        return userRepository.findById(id);
    }

    // Update
    public User updateUser(Long id, String username, String email) {
        Optional<User> userOptional = userRepository.findById(id);
        if (userOptional.isPresent()) {
            User user = userOptional.get();
            user.setUsername(username);
            user.setEmail(email);
            return userRepository.save(user);
        }
        return null;
    }

    // Delete
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
}
```  

createUser는 하드에 파일 생성  
getAllUsers는 파일 목록 조회  
updateUser는 파일 내용 수정  
deleteUser는 파일 삭제  
EntityManager가 작업 조정, Hibernate가 MySQL에 반영  

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

    @GetMapping
    public List<User> getAllUsers() {
        return userService.getAllUsers();
    }

    @GetMapping("/{id}")
    public Optional<User> getUserById(@PathVariable Long id) {
        return userService.getUserById(id);
    }

    @PutMapping("/{id}")
    public User updateUser(@PathVariable Long id, @RequestParam String username, @RequestParam String email) {
        return userService.updateUser(id, username, email);
    }

    @DeleteMapping("/{id}")
    public void deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
    }
}
```  

REST API로 CRUD 기능 테스트  

---
#### 📌 MySQL의 장점
- 데이터 영구 저장으로 프로덕션 적합  
- JPA와 Hibernate로 쉬운 연동  
- 대량 데이터 처리 가능  
- 커뮤니티와 자료 풍부  

---
#### 📌 주의사항
1. MySQL 서버 설치와 DB 생성 필수  
2. JDBC URL, 사용자 인증 정보 정확히 설정  
3. ddl-auto는 프로덕션에서 create, update 주의  
4. EntityManager의 영속성 컨텍스트 관리 주의  
5. 대량 데이터 작업 시 성능 최적화 필요  

---
#### 📌 MySQL 연동과 CRUD의 의미
MySQL은 안정적인 데이터 저장소  
JPA와 Hibernate로 CRUD 간소화  
EntityManager가 엔티티와 MySQL 연결 조율  

---
#### ✍ 알아보면서
MySQL 연동은 처음엔 설정이 복잡했음  
JPA, Hibernate, EntityManager의 역할 명확해짐  

잘못된 설정은 데이터 불일치 유발  
정확한 환경 구성과 테스트 중요성 깨달음  

MySQL과 JPA 조합을 통해 설계와 기능 구현에 더 집중 가능  

---