---
title: "프로세스, 쓰레드, 블로킹과 논블로킹의 원리"
categories:
  - CS Basics
tags:
  - CS Basics
last_modified_at:
---

#### 📌 용어 설명
- 프로세스(Process): 실행 중인 프로그램. OS에서 메모리와 자원을 할당받아 독립적으로 실행
- 쓰레드(Thread): 프로세스 내에서 실행되는 작업 흐름 단위. 프로세스 자원을 공유
- 동기(Synchronous): 요청 후 결과를 기다림. 결과가 나와야 다음 단계 진행
- 비동기(Asynchronous): 요청 후 결과를 기다리지 않음. 결과가 나오지 않아도 다른 작업 수행 가능
- 블로킹(Blocking): 쓰레드(작업 흐름)가 멈춤(대기)
- 논블로킹(Non-blocking): 쓰레드가 멈추지 않고 계속 실행 가능
- 멀티플렉서(Selector, epoll 등): IO 상태를 감시하고 이벤트가 발생하면 알림
- I/O(Input/Output): 외부 세계와 데이터를 주고받는 모든 작업을 의미

---
#### 📌 프로세스와 쓰레드

##### ✅ 프로세스
- 프로그램이 실행되면 운영체제(OS)가 프로세스를 생성
- 프로세스는 고유한 메모리 공간과 자원 할당
- 서로 다른 프로세스는 메모리 공유 불가

##### ✅ 쓰레드
- 프로세스 안에는 최소 1개 이상의 쓰레드 존재 (메인 쓰레드)
- 여러 쓰레드는 프로세스 내에서 메모리(Heap 등)를 공유
- 쓰레드 간 협업으로 병렬 처리 가능

##### ✅ 실생활 예시
- 프로세스 = 카페 매장
- 쓰레드 = 카페 직원들  
→ 매장(프로세스)은 독립적  
→ 직원(쓰레드)은 같은 매장의 커피 재료(메모리)를 공유하며 작업  

---
#### 📌 블로킹과 논블로킹의 원리

##### ✅ 블로킹
- 쓰레드가 IO(파일 읽기, DB 쿼리, 네트워크 통신)를 요청
- 결과가 올 때까지 쓰레드가 멈춤(waiting 상태)
- 많은 요청이 오면 많은 쓰레드가 필요 → 자원 낭비 발생

##### ✅ 논블로킹
- 쓰레드가 IO 요청 시 멈추지 않음
- IO 완료 여부는 OS의 멀티플렉서가 감시
- IO가 끝나면 이벤트 발생 → 쓰레드가 그때 결과 처리

##### ✅ 실생활 예시
카페에서 커피 주문 상황

- 블로킹  
→ 손님이 카운터 앞에서 커피 나오기까지 기다림  
→ 다음 손님은 기다려야 함

- 논블로킹  
→ 손님이 번호표 받고 자리에 감  
→ 카운터는 다른 손님 주문 계속 처리  
→ 커피 나오면 알림 발생 → 손님이 커피 받으러 감

---
#### 📌 멀티플렉서(Selector)의 역할

멀티플렉서 = IO 감시기

> 💡 IO는 왜 멀티플렉서(Selector)에 의해 감시되는가?
> IO는 CPU가 직접 계산하는 것과 달리 외부 장치(디스크, 네트워크, DB 등)를 기다려야 함  
> 그래서 시간이 길고 느림 → 블로킹/논블로킹의 성능 문제가 여기서 발생  
> CPU 연산 = 머릿속으로 더하기 빼기 하는 것(빠름)  
> IO = 누군가에게 전화해서 답을 기다리는 것 (상대방 응답이 필요해서 느림)
> 그래서 대부분의 웹 서버 성능 저하 원인은 CPU가 아니라 IO 작업 때문에 발생

##### ✅ 작동 원리

1️⃣ 쓰레드가 IO 요청을 멀티플렉서(Selector)에 등록  
2️⃣ 쓰레드는 멈추지 않고 다른 작업 수행  
3️⃣ 멀티플렉서는 여러 IO 작업을 동시에 감시  
4️⃣ IO 완료 시 이벤트 발생  
5️⃣ 쓰레드는 해당 IO 결과만 처리 (콜백 등으로)

##### ✅ 카운터라는 표현
- Selector는 **어떤 IO 작업이 끝났는지 카운트/확인** 역할 수행
- "이 작업 끝남", "저 작업 아직 진행 중" → 리스트로 감시
- 반복 루프(Event loop)에서 이를 지속적으로 확인

##### ✅ 기술 예시
- Java NIO → Selector 사용
- Linux → epoll 사용
- macOS → kqueue 사용
- JavaScript(Node.js) → libuv + epoll/kqueue 기반  

---
#### 📌 Selector 내부 동작 원리

##### ✅ IO 요청 처리 흐름 예시

```java
[main thread]
      |
      |--> IO 요청 (ex: 소켓 연결 요청)
      |
[Selector 등록] ----> [IO Channel 1]  (DB 요청)
                   |
                   +-> [IO Channel 2]  (파일 읽기 요청)
                   |
                   +-> [IO Channel 3]  (API 호출 요청)

[main thread]
      |
      |--> 다른 작업 수행
      |
[Event Loop]
      |
      |--> Selector.poll() → IO 완료 이벤트 발생 확인
      |
      |--> 콜백 실행 → 결과 처리
```

##### ✅ 핵심 흐름
- IO 작업은 Selector에 등록  
- 쓰레드는 블로킹 없이 계속 실행  
- Event Loop가 지속적으로 감시  
- IO 완료되면 알림 발생 → 콜백 실행  

---
#### 📌 코드 관점에서 비교

##### ✅ 블로킹 IO (RestTemplate 기본 사용)

```java
import org.springframework.web.client.RestTemplate;

public class BlockingExample {
    public static void main(String[] args) {
        RestTemplate restTemplate = new RestTemplate();

        // 블로킹 IO → 결과가 올 때까지 쓰레드 대기
        String response = restTemplate.getForObject("https://jsonplaceholder.typicode.com/posts/1", String.class);

        System.out.println("Response: " + response);
    }
}
```  

→ 요청 후 결과 받을 때까지 쓰레드 멈춤
→ 결과 받아야 다음 줄 실행

##### ✅ 논블로킹 IO (WebClient 사용)

```java
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Mono;

public class NonBlockingExample {
    public static void main(String[] args) {
        WebClient webClient = WebClient.create("https://jsonplaceholder.typicode.com");

        // 논블로킹 IO → 쓰레드는 멈추지 않고 바로 다음 코드 실행
        Mono<String> responseMono = webClient.get()
            .uri("/posts/1")
            .retrieve()
            .bodyToMono(String.class);

        responseMono.subscribe(response -> System.out.println("Response: " + response));

        System.out.println("이 출력은 요청 직후 바로 실행됨");

        try {
            Thread.sleep(2000); // 결과 출력 기다리기 위한 대기 (예시용)
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```  

→ 요청 후 결과 기다리지 않음
→ 콜백으로 결과 처리
→ 쓰레드는 계속 다른 작업 가능

---
#### 📌 WebClient 같은 논블로킹 IO의 장단점

##### ✅ 장점

- 적은 쓰레드로 다수 요청 처리 가능 → 서버 확장성 우수
- 높은 TPS (Transactions Per Second) 지원
- 자원 사용량(메모리, CPU) 효율적
- 비동기 흐름으로 응답 속도 개선 가능

##### ✅ 단점

- 코드 복잡도 증가 (리액티브 프로그래밍 개념 필요)
- 디버깅 어려움 (콜백 지옥 발생 가능성 있음)
- 기존 동기 API와 혼용 시 주의 필요
- 학습 곡선 존재

##### ✅ 언제 사용하나

| 상황 | 추천 방식 |
|------|----------|
| 트래픽 낮음, 구현 단순화 원함 | RestTemplate(블로킹) |
| 고트래픽 환경, 수천~수만 동시 요청 | WebClient(논블로킹) |
| API Gateway, 마이크로서비스 → 다른 서비스 호출 많음 | WebClient 필수 |

---
#### 📌 관계 정리

|               | 블로킹 | 논블로킹 |
|---------------|--------|----------|
| 동기          | 쓰레드가 멈춤 | 거의 사용하지 않음 |
| 비동기        | 거의 사용하지 않음 | WebClient 등 사용 |

##### ✅ 쓰레드 관점

- 블로킹 IO → 요청마다 새로운 쓰레드 필요 → 리소스 낭비
- 논블로킹 IO → 적은 쓰레드로 다수 요청 처리 가능 → 고성능 서버 구현

##### ✅ 이벤트 루프

- 논블로킹에서 중요한 구조
- 쓰레드가 IO 요청을 등록 → 이벤트 루프가 IO 상태 확인 → IO 완료 시 콜백 실행  

---
#### 📌 결론

- 프로세스 = 프로그램 단위 실행 공간
- 쓰레드 = 프로세스 내 작업 흐름
- 블로킹 IO → 쓰레드 낭비 많음
- 논블로킹 IO → 적은 쓰레드로 효율적 처리 가능
- Selector(epoll/kqueue)는 IO 상태 감시 후 알림 발생 → 이벤트 루프에서 처리
- WebClient 등 논블로킹 기술은 고성능 서버 필수 요소
- 비동기 논블로킹 조합이 고성능 시스템에서 인기

#### ✍ 알아보면서

처음엔 "동기/비동기", "블로킹/논블로킹" 구분 헷갈렸음
하지만 프로세스와 쓰레드, 블로킹과 논블로킹에 대해 정리를 하고 구조를 알고 나니 조금씩 차이에 대해 알 수 있을 거 같음

구조와 원리를 이해하고 나니 "왜 비동기 논블로킹에서 쓰레드가 절약되고 성능이 좋아지는지"에 대해 조금 알게 됨  

결국 중요한 것은  
**상황에 따라 언제 블로킹을 쓰고, 언제 논블로킹을 쓸지 고민하는 것** 임을 깨달음

---
