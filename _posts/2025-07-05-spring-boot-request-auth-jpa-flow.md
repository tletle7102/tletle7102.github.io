---
title: "스프링부트 요청 처리와 인증, JPA 흐름"
categories:
  - Spring Boot
tags:
  - Spring Boot
  - Spring Security
  - JPA
last_modified_at: 2025-06-24
---

#### 📌 용어 설명
- HTTP 요청: 클라이언트가 서버에 보내는 메시지 (GET, POST 등)  
- 인증: 사용자 신원 확인 (로그인 여부)  
- 인가: 사용자 권한 확인 (작업 가능 여부)  
- 컨트롤러: 요청을 받아 처리, 서비스 호출  
- 서비스: 비즈니스 로직 처리  
- JPA 리포지토리: DB와 상호작용, 데이터 관리  

---
#### 📌 스프링부트 요청 흐름이란
클라이언트 요청이 서버에 들어와 처리되는 과정  
인증/인가, 매핑, 서비스, DB 작업, 응답 반환  

##### 실생활 예시
카페에서 손님이 커피 주문  
문지기(시큐리티) 신분 확인, 카운터(컨트롤러) 주문 전달  
바리스타(서비스) 커피 제조, 창고(리포지토리) 재료 관리  
손님에게 커피 제공  

---
#### 📌 단계별 흐름

##### ✅ 1. 클라이언트 요청
프론트엔드에서 HTTP 요청 전송  
예: /user/profile로 사용자 프로필 조회  

```java  
// JavaScript (프론트엔드)
fetch('http://localhost:8080/user/profile', {
    method: 'GET',
    headers: {
        'Authorization': 'Bearer <JWT_TOKEN>'
    }
})
.then(response => response.json())
.then(data => console.log(data));
```  

##### ✅ 2. 인증/인가
스프링 시큐리티가 사용자 확인  
로그인 여부, 권한(ROLE_USER) 체크  

```java  
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import java.util.Collections;

@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public UserDetailsService userDetailsService() {
        User.UserBuilder builder = User.withUsername("user")
            .password("{noop}password")
            .authorities(Collections.singletonList(new SimpleGrantedAuthority("ROLE_USER")));
        return new InMemoryUserDetailsManager(builder.build());
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/user/**").hasRole("USER")
                .anyRequest().authenticated()
            )
            .formLogin();
        return http.build();
    }
}
```  

##### ✅ 3. 요청 매핑
컨트롤러가 URL과 메서드 연결  
서비스 호출  

```java  
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.security.core.context.SecurityContextHolder;

@RestController
@RequestMapping("/user")
public class UserController {
    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/profile")
    public UserProfileResponse getProfile() {
        String username = SecurityContextHolder.getContext().getAuthentication().getName();
        return userService.getUserProfile(username);
    }
}
```  

##### ✅ 4. 서비스 메서드
비즈니스 로직 처리  
리포지토리 호출  

```java  
import org.springframework.stereotype.Service;

@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public UserProfileResponse getUserProfile(String username) {
        UserEntity user = userRepository.findByUsername(username)
            .orElseThrow(() -> new RuntimeException("User not found"));
        return new UserProfileResponse(user.getUsername(), user.getEmail());
    }
}

public record UserProfileResponse(String username, String email) {}
```  

##### ✅ 5. JPA 리포지토리
DB에서 데이터 조회/저장  
엔티티 관리  

```java  
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table(name = "users")
public class UserEntity {
    @Id
    private String username;
    private String email;

    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}

import org.springframework.data.jpa.repository.JpaRepository;
import java.util.Optional;

public interface UserRepository extends JpaRepository<UserEntity, String> {
    Optional<UserEntity> findByUsername(String username);
}
```  

##### ✅ 6. 응답 반환
컨트롤러가 JSON 응답 전송  
클라이언트에 결과 제공  

```
{
    "username": "user",
    "email": "user@example.com"
}
```

---
#### 📌 흐름 그림
```
[클라이언트] ---- GET /user/profile ----> [서버]
                          |
                          v
                [시큐리티] 인증/인가
                          |
                          v
                [컨트롤러] 매핑
                          |
                          v
                [서비스] 로직
                          |
                          v
                [리포지토리] DB
                          |
                          v
                [응답] JSON --> [클라이언트]
```

---
#### 📌 주니어 개발자 팁
시큐리티는 기본 설정만 먼저 익히기  
컨트롤러는 URL 연결과 서비스 호출 집중  
서비스는 로직, 리포지토리는 DB 분리  
JPA는 JpaRepository로 SQL 최소화  
로그 추가로 디버깅 쉬움  

---
#### ✍ 알아보면서
스프링부트 흐름 처음엔 복잡했음  
카페 비유로 각 단계 이해  
시큐리티의 인증/인가 역할 깨달음  
컨트롤러, 서비스, 리포지토리 역할 분리 느낌  
JPA로 DB 작업 간단해진 점 놀라움  
주니어로서 전체 그림 잡히기 시작  
앞으로 단계별 디버깅 연습 예정  

---