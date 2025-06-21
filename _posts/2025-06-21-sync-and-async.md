---
title: "동기와 비동기의 차이"
categories:
  - CS Basics
tags:
  - CS Basics
last_modified_at:
---

#### 📌 용어 설명
- 동기(Synchronous): 요청 후 결과를 기다림. 해당 작업이 끝나야 다음 작업 진행 가능
- 비동기(Asynchronous): 요청 후 결과를 기다리지 않음. 작업 도중에도 다른 작업 수행 가능
- 블로킹(Blocking): 현재 쓰레드가 결과가 올 때까지 멈춤
- 논블로킹(Non-blocking): 현재 쓰레드가 멈추지 않고 다른 작업 수행 가능
- 쓰레드(Thread): 프로그램 내에서 독립적으로 실행되는 흐름
- 콜백(Callback): 작업 완료 후 실행될 함수를 미리 등록하여, 비동기 작업이 끝났을 때 호출됨
- Future: 자바에서 비동기 결과를 다루는 객체
- CompletableFuture: 자바 8부터 지원하는 비동기 프로그래밍 지원 도구

---
#### 📌 동기란
하나의 작업이 끝날 때까지 다음 작업을 시작하지 않음  
작업 순서가 명확하고 직관적  

##### 실생활 예시  
은행 창구에서 번호표 받고 기다리는 상황  
앞 사람이 창구에서 업무를 보는 동안, 뒤 사람은 기다림  
앞 업무가 끝난 뒤에만 다음 사람이 진행 가능  
**대기 시간 발생**  

---
#### 📌 비동기란
작업을 요청한 뒤, 기다리지 않고 다른 작업을 진행  
작업 완료 시 알림(콜백 등)으로 결과를 처리  

##### 실생활 예시  
패스트푸드점에서 주문 후 대기번호표 받고 자리로 이동  
번호가 호출되면 음식을 받으러 감  
음식 준비 중에도 다른 일을 할 수 있음  
**대기 시간 효율적 사용 가능**  

---
#### 📌 동기와 비동기의 코드 예시  

---
#### 📌 실생활 예시

##### ✅ 동기적 상황
카페에서 커피 주문 후 카운터 앞에서 기다림  
커피를 받아야만 자리에 감  
→ 커피가 나올 때까지 다른 일 못 함

##### ✅ 비동기적 상황
카페에서 커피 주문 후 번호표 받고 자리에 감  
앉아서 노트북 작업 중 알림 울리면 커피 받으러 감  
→ 커피 기다리는 동안 다른 일 가능

---
#### 📌 코드 관점에서 본 동기와 비동기

##### ✅ 자바 동기 예시 (RestTemplate)

```java
import org.springframework.web.client.RestTemplate;

public class SyncExample {
    public static void main(String[] args) {
        RestTemplate restTemplate = new RestTemplate();

        // 동기적 HTTP 요청
        String response = restTemplate.getForObject("https://jsonplaceholder.typicode.com/posts/1", String.class);

        System.out.println("Response: " + response);
        System.out.println("이 출력은 응답을 받은 뒤에 실행됨");
    }
}
```

##### ✅ 자바 비동기 예시 (WebClient)

```java
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Mono;

public class AsyncExample {
    public static void main(String[] args) {
        WebClient webClient = WebClient.create("https://jsonplaceholder.typicode.com");

        // 비동기적 HTTP 요청
        Mono<String> responseMono = webClient.get()
            .uri("/posts/1")
            .retrieve()
            .bodyToMono(String.class);

        responseMono.subscribe(response -> System.out.println("Response: " + response));

        System.out.println("이 출력은 요청 직후에 실행됨");

        try {
            Thread.sleep(2000); // 비동기 결과 기다리기 위해 잠시 대기
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

---
#### 📌 비동기와 논블로킹의 차이

비동기(Asynchronous), 논블로킹(Non-blocking)은 비슷해 보이지만 개념이 다름  
많이 헷갈리기 쉬운 부분

##### ✅ 실생활 예시

카페에서 커피 주문 상황
- 블로킹: 커피가 나올 때까지 계산대 앞에서 기다림
- 논블로킹: 주문 후 번호표 받고 자리에 가서 기다림. 다른 손님도 주문 가능
- 동기: 커피를 반드시 받고 나서야 친구 만나러 감
- 비동기: 커피를 기다리지 않고 친구 먼저 만나러 감. 커피 나오면 호출받음

##### ✅ 코드 관점

- 블로킹: 호출한 쓰레드가 결과가 나올 때까지 대기
- 논블로킹: 호출한 쓰레드는 바로 다른 작업 수행 가능
- 동기: 결과가 나올 때까지 내 코드 흐름이 멈춤
- 비동기: 내 코드 흐름은 계속 진행되고, 결과가 나중에 오면 별도 처리

##### ✅ 관계 정리

|               | 블로킹 | 논블로킹 |
|---------------|--------|----------|
| 동기          | 동기 + 블로킹 → RestTemplate 기본 | 동기 + 논블로킹 → 거의 사용 안 함 |
| 비동기        | 비동기 + 블로킹 → 잘 안 씀 | 비동기 + 논블로킹 → WebClient 등 최신 고성능 방식 |

##### ✅ 왜 중요한가

- 동기 + 블로킹: 구현 쉬움. 서버 리소스 많이 사용
- 비동기 + 논블로킹: 구현 복잡. 서버 효율, 처리량, 반응성 우수  

---
#### 📌 언제 동기, 언제 비동기
- 동기  
  - 순서가 중요한 처리
  - 단순한 흐름으로 처리할 때 유리  
- 비동기  
  - 대기 시간이 긴 작업 (네트워크, 파일 IO 등)
  - UI 응답성을 높이고 싶은 경우
  - 서버에서 요청 처리 성능을 높이고 싶은 경우 (스프링에서 비동기 컨트롤러 활용 등)  

---
#### 📌 비동기의 주요 기술 (자바/스프링 기준)  
- Future, CompletableFuture  
- @Async (스프링 비동기 처리 지원 어노테이션)  
- 비동기 RestTemplate, WebClient (스프링 WebFlux 기반 논블로킹 HTTP 클라이언트)  

---
#### ✍ 알아보면서
처음에는 동기/비동기라는 말이 단순히 빠르냐 느리냐의 차이로만 이해했음  
실제로는 **흐름의 구조**에 대한 개념임을 알게 됨  

초기에는 동기적 코딩이 더 이해하기 쉬웠음  
결과가 순차적으로 오니 흐름 파악이 명확했음

하지만 사용자가 많아질수록 비동기 + 논블로킹 방식의 필요성을 느낌  
비동기 + 논블로킹 방식으로 서버 리소스를 효율적으로 사용하고 빠른 응답 제공 가능  

비동기를 잘 활용하면 서버의 자원 활용 효율이 훨씬 좋아지지만 그에 따라 복잡성도 증가함  
결국 중요한 건 **언제 동기/비동기, 블로킹/논블로킹을 선택할지 고민하는 것**  
적절한 상황에서 올바른 도구 사용이 개발자의 실력으로 연결될 것으로 보임  

---