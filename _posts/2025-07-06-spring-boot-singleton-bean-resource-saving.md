---
title: "스프링부트 싱글톤 빈과 리소스 절약"
categories:
  - Spring Boot
tags:
  - Spring Boot
  - Singleton
last_modified_at:
---

#### 📌 용어 설명

- 싱글톤 빈: ApplicationContext가 관리하는 단일 인스턴스 객체  
- 리소스: 컴퓨터의 메모리, CPU 등 자원  
- 상태 없는 빈: 내부 데이터(상태)를 가지지 않는 빈, 스레드 안전  
- 상태 있는 빈: 내부 데이터를 가지는 빈, 동시성 문제 주의  
- 프로토타입 스코프: 요청마다 새 인스턴스 생성  

---
#### 📌 싱글톤 빈이란
스프링에서 기본적으로 빈을 싱글톤으로 관리  
ApplicationContext에 단일 인스턴스 저장  
여러 코드에서 동일 인스턴스 공유  

##### 실생활 예시

카페의 커피 머신  
손님마다 새 머신 설치 대신 하나로 모든 주문 처리  
설치 시간, 공간 절약  

---
#### 📌 왜 싱글톤 빈을 사용하는가
단일 인스턴스 공유로 리소스 절약  
메모리, CPU 소모 감소  
의존성 주입 간소화, 일관성 유지  

##### 실생활 비유
카페에 커피 머신 하나만 사용  
손님 A, B, C가 같은 머신으로 커피 주문  
새 머신 설치 시 시간, 돈, 공간 낭비  

##### 그림으로 이해
비-싱글톤 (매번 새 객체)  
```
[메모리]
컨트롤러 A -> [CalculatorService 객체1]  <- 새로 생성 (100KB)
컨트롤러 B -> [CalculatorService 객체2]  <- 새로 생성 (100KB)
컨트롤러 C -> [CalculatorService 객체3]  <- 새로 생성 (100KB)
=> 메모리: 300KB, 생성: 3번
```

싱글톤 (단일 인스턴스)  
```
[메모리]
ApplicationContext
   |
   v
[CalculatorService 객체1]  <- 한 번 생성 (100KB)
   ^         ^         ^
컨트롤러 A  컨트롤러 B  컨트롤러 C
=> 메모리: 100KB, 생성: 1번
```

---
#### 📌 코드 관점

##### ✅ 상태 없는 싱글톤 빈
```java  
import org.springframework.stereotype.Service;

@Service
public class CalculatorService {
    public int add(int a, int b) {
        return a + b;
    }
}
```  
- 내부 상태 없음, 입력값만 처리  
- 여러 스레드 동시 호출 안전  
- 싱글톤으로 메모리 절약  

##### ✅ 상태 있는 빈 (주의)
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
- 싱글톤 시 여러 스레드 충돌 가능  
- 동시성 제어(synchronized)나 프로토타입 스코프 필요  

##### ✅ 프로토타입 스코프 (비-싱글톤)
```java  
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Service;

@Service
@Scope("prototype")
public class CalculatorService {
    public int add(int a, int b) {
        return a + b;
    }
}
```  
- 요청마다 새 객체 생성  
- 메모리, CPU 소모 증가  

---
#### 📌 리소스 절약의 중요성
작은 앱은 리소스 절약 효과 미미  
대규모 앱은 사용자, 빈 많아 싱글톤 필수  
예: 사용자 10,000명, 빈 100개  
- 비-싱글톤: 객체 1,000,000개, 메모리 폭발  
- 싱글톤: 객체 100개, 서버 안정  

##### 실생활 비유
대형 카페, 손님 1,000명  
머신 하나로 주문 처리 vs 머신 1,000개 설치  
싱글톤은 효율적 운영 가능  

---
#### 📌 설계 팁
상태 없는 빈으로 설계 권장  
스레드 안전, 유지보수 쉬움  
상태 있는 빈은 프로토타입 스코프나 동시성 제어 사용  
싱글톤 빈은 공유 자원, 설정 정보에 적합  

---
#### ✍ 알아보면서
싱글톤이 왜 중요한지 처음엔 몰랐음  
카페 머신 비유로 리소스 절약 이해  
상태 없는 빈의 스레드 안전성 깨달음  
앞으로 빈 설계 시 싱글톤과 스코프 고민 예정  

---