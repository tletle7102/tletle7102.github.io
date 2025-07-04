---
title: "스프링부트 컨텍스트와 시큐리티 컨텍스트, 퍼시스턴스 컨텍스트 비교"
categories:
  - Spring Boot
tags:
  - Spring Boot
  - Spring Security
  - JPA
last_modified_at:
---

#### 📌 용어 설명
- ApplicationContext: 스프링의 IoC 컨테이너, 빈 관리와 의존성 주입 처리  
- SecurityContextHolder: 스프링 시큐리티에서 스레드별 보안 정보 저장  
- SecurityContext: 사용자 인증/인가 정보 포함, SecurityContextHolder가 관리  
- PersistenceContext: JPA에서 엔티티 관리, 데이터베이스 동기화  
- 싱글톤 빈: ApplicationContext가 관리하는 단일 인스턴스 객체  
- 상태 없는 빈: 내부 상태(데이터)를 가지지 않는 빈, 스레드 안전  
- 상태 있는 빈: 내부 상태를 가지는 빈, 동시성 문제 주의  

---
#### 📌 ApplicationContext란
스프링 어플리케이션의 핵심 컨테이너  
빈 생성, 의존성 주입, 생명주기 관리  
설정 로딩, 이벤트 처리  

##### 실생활 예시
카페 매장  
직원(빈) 배치, 운영 환경 제공  
매장이 없으면 직원들 일 못 함  

---
#### 📌 SecurityContext란
스프링 시큐리티에서 사용자 인증/인가 정보 관리  
SecurityContextHolder가 스레드별로 SecurityContext 저장  
Authentication 객체로 사용자 정보 제공  

##### 실생활 예시
카페 직원의 배지  
누가 어떤 역할(바리스타, 매니저)인지 확인  
요청마다 배지 확인해 권한 체크  

---
#### 📌 PersistenceContext란
JPA에서 엔티티 객체 관리  
변경 감지, 1차 캐시, 데이터베이스 동기화  
트랜잭션 단위로 엔티티 생명주기 처리  

##### 실생활 예시
카페의 주문 노트  
주문(엔티티) 기록, 완료 시 주방(데이터베이스)에 반영  
중복 주문 방지, 상태 추적  

---
#### 📌 코드 예시

##### ✅ ApplicationContext
```java  
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class ContextExample {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        MyService myService = context.getBean(MyService.class);
        myService.doSomething();
    }
}
```  

##### ✅ SecurityContext
```java  
import org.springframework.security.core.context.SecurityContextHolder;

public class SecurityExample {
    public void printCurrentUser() {
        String username = SecurityContextHolder.getContext().getAuthentication().getName();
        System.out.println("Current user: " + username);
    }
}
```  

##### ✅ PersistenceContext
```java  
import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import org.springframework.stereotype.Service;

@Service
public class UserService {
    @PersistenceContext
    private EntityManager entityManager;

    public void saveUser(String username) {
        User user = new User();
        user.setUsername(username);
        entityManager.persist(user);
    }
}
```  

---
#### 📌 비교

##### ✅ 실생활 비유
- ApplicationContext: 카페 매장, 전체 운영 관리  
- SecurityContext: 직원 배지 관리장부, 역할 확인  
- PersistenceContext: 주문 노트, 주문 처리  

##### ✅ 코드 관점
- ApplicationContext: 빈과 DI 관리, 어플리케이션 전반  
- SecurityContext: 스레드별 인증 정보, 요청 단위  
- PersistenceContext: 엔티티와 데이터베이스 동기화, 트랜잭션 단위  

##### ✅ 관계 정리
| 컨텍스트 | 관리 대상 | 생명주기 | 목적 |
|----------|-----------|----------|------|
| ApplicationContext | 빈 | 어플리케이션 | 전체 환경 제공 |
| SecurityContext | 인증 정보 | 요청/스레드 | 보안 처리 |
| PersistenceContext | 엔티티 | 트랜잭션 | 데이터베이스 동기화 |

---
#### 📌 싱글톤 빈과 스프링
ApplicationContext는 기본적으로 싱글톤 빈 관리  
단일 인스턴스로 메모리 절약, 상태 공유  
상태 없는 빈으로 설계해 스레드 안전성 확보  

##### ✅ 왜 싱글톤?
- 메모리 효율, 일관성 유지  
- 의존성 주입 간소화  

##### ✅ 상태 없는 빈 vs 상태 있는 빈
- 상태 없는 빈: 내부 데이터(필드) 유지 안 함, 스레드 안전  
- 상태 있는 빈: 내부 데이터 유지, 동시성 문제 주의  

###### 상태 없는 빈 예시
```java  
import org.springframework.stereotype.Service;

@Service
public class CalculatorService {
    public int add(int a, int b) {
        return a + b;
    }
}
```  
- 입력값만 처리, 내부 상태 없음  
- 여러 스레드가 동시에 호출해도 안전  

###### 상태 있는 빈 예시
```java  
import org.springframework.stereotype.Service;

@Service
public class CounterService {
    private int count = 0;

    public void increment() {
        count++;
    }
    
    public int getCount() {
        return count;
    }
}
```  
- `count` 필드로 상태 유지  
- 여러 스레드가 동시에 호출 시 `count` 값 충돌 가능  
- 싱글톤 빈으로 사용 시 동시성 문제 해결(예: synchronized) 필요  

##### ✅ 설계 팁
- 상태 없는 빈 선호: 스레드 안전, 유지보수 쉬움  
- 상태 있는 빈은 프로토타입 스코프나 동시성 제어 사용  

##### ✅ 대안
- 프로토타입 스코프: 요청마다 새 인스턴스  
- 리퀘스트 스코프: HTTP 요청 단위  

---
#### 📌 Configuration과 Component
Configuration은 빈 정의 클래스, @Bean으로 빈 등록  
Component는 자동 스캔으로 등록되는 빈  
Configuration은 설계도, Component는 실제 빈  

##### ✅ 코드 예시
```java  
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;

@Configuration
public class AppConfig {
    @Bean
    public MyService myService() {
        return new MyService();
    }
}

@Component
public class MyComponent {
    public void execute() {
        System.out.println("Executing...");
    }
}
```  

---
#### ✍ 알아보면서
처음엔 컨텍스트가 단순히 객체 저장소로 보였음  
각 컨텍스트가 다른 맥락(환경, 보안, 데이터베이스)을 다룬다는 점 깨달음  
싱글톤 빈은 메모리 효율과 DI 편리함 제공  
상태 없는 빈과 상태 있는 빈의 차이 이해, 스레드 안전성 중요성 느낌  
Configuration과 Component는 빈 생성 방식의 차이 파악  
스프링의 구조적 설계가 복잡하지만 체계적임을 알게 됨  
적절한 컨텍스트와 빈 스코프 선택이 개발 효율 좌우  
궁금증 해결하며 스프링의 큰 그림이 보이기 시작  
앞으로 더 깊이 파고들고 싶음  

---