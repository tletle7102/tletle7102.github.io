---
title: "스프링 시큐리티에서 권한과 싱글톤 리스트 사용"
categories:
  - Spring Security
tags:
  - Spring Security
  - Spring Boot
  - Authority
last_modified_at:
---

#### 📌 용어 설명
- GrantedAuthority: 스프링 시큐리티에서 권한/역할 나타내는 인터페이스  
- Authentication: 사용자 인증 정보, 권한 리스트 포함  
- Collections.singletonList: 단일 객체를 포함하는 불변 리스트 반환  
- ROLE: 스프링 시큐리티에서 권한의 일반적 접두사 (예: ROLE_USER)  
- 불변 리스트: 수정 불가능한 리스트, 안정성 확보  

---
#### 📌 스프링 시큐리티 권한이란
사용자가 특정 작업 수행 가능 여부 결정  
List<GrantedAuthority>로 권한 목록 관리  
ROLE_USER, ROLE_ADMIN 등으로 역할 정의  

##### 실생활 예시
카페 직원의 역할 배지  
바리스타는 커피 제조 가능, 매니저는 재고 관리 가능  
배지는 직원의 권한 나타냄  

---
#### 📌 Collections.singletonList란
자바의 유틸리티 메서드  
단일 객체를 불변 리스트로 반환  
메모리 효율적, 수정 방지  

##### 실생활 비유
직원에게 역할 하나만 줄 때  
큰 상자(ArrayList)에 배지 담는 대신  
작은 배지 하나(singletonList)만 전달  
공간 절약, 배지 변경 불가  

---
#### 📌 왜 singletonList를 사용하는가
단일 권한 부여 시 간결하고 효율적  
ArrayList보다 메모리 적게 사용  
불변성으로 권한 안정성 보장  

##### 그림으로 이해
ArrayList로 권한 부여  
```
[메모리]
[ArrayList]
   |
   v
[ROLE_USER]  <- 객체 생성, 추가 작업 (메모리 150KB)
```

singletonList로 권한 부여  
```
[메모리]
[singletonList]
   |
   v
[ROLE_USER]  <- 간단 생성 (메모리 100KB)
```

---
#### 📌 코드 예시

##### ✅ singletonList로 권한 설정
```java  
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import java.util.Collections;

public class UserConfig {
    public UserDetails createUser() {
        return new User(
            "user",
            "password",
            Collections.singletonList(new SimpleGrantedAuthority("ROLE_USER"))
        );
    }
}
```  
- 단일 권한 ROLE_USER 부여  
- singletonList로 간결, 메모리 절약  

##### ✅ ArrayList로 권한 설정
```java  
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import java.util.ArrayList;
import java.util.List;

public class UserConfig {
    public UserDetails createUser() {
        List<SimpleGrantedAuthority> authorities = new ArrayList<>();
        authorities.add(new SimpleGrantedAuthority("ROLE_USER"));
        return new User("user", "password", authorities);
    }
}
```  
- 동일 결과, 하지만 ArrayList 생성/관리 비용 증가  

##### ✅ 실제 스프링 시큐리티 설정
```java  
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetailsService;
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
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .requestMatchers("/user/**").hasRole("USER")
                .anyRequest().authenticated()
            )
            .formLogin();
        return http.build();
    }
}
```  
- singletonList로 사용자 권한 설정  
- /user/** 경로는 ROLE_USER만 접근 가능  

---
#### 📌 언제 singletonList를 사용
단일 권한 부여 시 적합  
다중 권한은 ArrayList나 List.of 사용  
불변성 필요한 경우 선호  

##### 실생활 비유
역할 하나(바리스타)면 배지 하나 제공  
여러 역할(바리스타, 캐셔)면 배지 상자 사용  
배지 하나일 때 작은 포장(singletonList) 효율적  

---
#### 📌 주의점
singletonList는 불변, 권한 추가/삭제 불가  
동적 권한 변경 시 ArrayList 사용  
단일 권한이 확실할 때만 singletonList 선택  

---
#### ✍ 알아보면서
스프링 시큐리티 권한 설정 처음엔 복잡했음  
singletonList로 간결함과 효율성 이해  
카페 배지 비유로 메모리 절약 깨달음  
단일 권한 부여 시 singletonList의 장점 느낌  
불변성으로 안정성 보장 알게 됨  
큰 앱에서 리소스 절약의 중요성 체감  
앞으로 권한 설정 시 singletonList 적극 활용 예정  

---