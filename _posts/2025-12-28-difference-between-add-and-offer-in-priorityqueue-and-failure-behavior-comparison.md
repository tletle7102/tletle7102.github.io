---
title: "PriorityQueue의 add()와 offer()의 차이점과 실패 시 동작 원리 비교"  
categories:  
  - Java  
tags:  
  - Java  
  - PriorityQueue  
  - add  
  - offer  
  - Queue  
last_modified_at:   
---

### PriorityQueue의 add()와 offer()의 차이점과 실패 시 동작 원리 비교

Java의 `PriorityQueue`는 우선순위 큐로, 요소를 정렬된 순서로 관리하며 `add()`와 `offer()` 메서드를 통해 요소를 추가함  
두 메서드는 비슷한 역할을 하지만, 실패 시 동작 방식에서 차이가 있음  

---

#### 📌 용어 설명  
- PriorityQueue: 힙 기반 우선순위 큐, 요소를 정렬된 순서로 유지  
- add(): `Queue` 인터페이스의 메서드로, 요소를 큐에 추가하며 실패 시 예외 발생  
- offer(): `Queue` 인터페이스의 메서드로, 요소를 추가하며 실패 시 `false` 반환  
- 힙 구조: `PriorityQueue`의 내부 데이터 구조, 요소 추가/제거 시 자동 정렬  
- 용량 제한: `PriorityQueue`의 최대 크기 제한(기본적으로 무제한, 생성자에서 설정 가능)  

---

#### 📌 add()와 offer()의 차이점  

1. 기본 동작:  
   - `add(E e)`와 `offer(E e)`는 모두 요소를 `PriorityQueue`에 추가하고 힙 구조를 유지  
   - 내부적으로 `offer()`를 호출하여 힙에 삽입, 시간 복잡도는 O(log n)  
   ```java  
   import java.util.PriorityQueue;

   public class PriorityQueueAddOfferExample {
       public static void main(String[] args) {
           PriorityQueue<Integer> pq = new PriorityQueue<>();
           pq.add(5);    // 힙에 5 추가
           pq.offer(2);  // 힙에 2 추가
           System.out.println(pq); // 출력: [2, 5]
       }
   }
   ```

2. 실패 시 동작:  
   - `add()`: 큐가 가득 찬 경우 `IllegalStateException`을 던짐  
   - `offer()`: 큐가 가득 찬 경우 `false`를 반환하며 예외 발생 없음  
   ```java  
   import java.util.PriorityQueue;

   public class FailureExample {
       public static void main(String[] args) {
           // 용량이 2로 제한된 PriorityQueue
           PriorityQueue<Integer> pq = new PriorityQueue<>(2);
           pq.offer(1); // 성공
           pq.offer(2); // 성공
           try {
               pq.add(3); // 실패: IllegalStateException
           } catch (IllegalStateException e) {
               System.out.println("add() 실패: " + e.getMessage());
           }
           boolean result = pq.offer(3); // 실패: false 반환
           System.out.println("offer() 결과: " + result);
       }
   }
   ```  
   출력:  
   ```
   add() 실패: Queue full
   offer() 결과: false
   ```

3. 문제 상황:  
   - `add()` 사용 시 예외 처리를 하지 않으면 프로그램이 비정상 종료될 수 있음  
   - `offer()`는 예외 대신 boolean 값을 반환하므로 실패 처리가 간단함  

> 💡 add()와 offer()의 동작 원리  
> `add()`는 내부적으로 `offer()`를 호출하고, `offer()`가 `false`를 반환하면 `IllegalStateException`을 던짐  
> `offer()`는 실패 시 예외를 발생시키지 않고 결과를 boolean으로 반환해 안전한 처리가 가능  

---

#### 📌 실패 시 동작 비교  

| 메서드 | 성공 시 | 실패 시 |  
| --- | --- | --- |  
| `add()` | 요소 추가, `true` 반환 | `IllegalStateException` 발생 |  
| `offer()` | 요소 추가, `true` 반환 | `false` 반환 |  

---

#### 📌 사용 시나리오와 설정 방법  

1. add() 사용:  
   - 큐 용량 제한이 없거나, 실패가 발생하면 즉시 예외 처리해야 하는 경우 적합  
   - 예외 처리가 필수  
   ```java  
   try {
       PriorityQueue<Integer> pq = new PriorityQueue<>(2);
       pq.add(1);
       pq.add(2);
       pq.add(3); // 예외 발생
   } catch (IllegalStateException e) {
       System.out.println("큐가 가득 찼습니다.");
   }
   ```

2. offer() 사용:  
   - 실패 시 예외 대신 조건문으로 처리하고자 할 때 적합  
   - 코드가 간결하고 안전함  
   ```java  
   PriorityQueue<Integer> pq = new PriorityQueue<>(2);
   if (pq.offer(1) && pq.offer(2) && pq.offer(3)) {
       System.out.println("모두 추가 성공");
   } else {
       System.out.println("추가 실패: 큐가 가득 찼습니다.");
   }
   ```

3. 용량 제한 설정:  
   - `PriorityQueue` 생성 시 초기 용량 지정 가능  
   ```java  
   PriorityQueue<Integer> pq = new PriorityQueue<>(10); // 초기 용량 10
   ```

4. IntelliJ 디버깅 팁:  
   - `add()` 또는 `offer()` 호출 시 내부 `queue` 배열과 `size` 필드를 디버거로 확인  
   - 실패 시 `IllegalStateException`의 스택 트레이스 분석  

---

#### 📌 추가 팁  

- 예외 처리 전략: `add()` 사용 시 `try-catch` 필수, `offer()`는 조건문으로 대체 가능  
- 성능 고려: 두 메서드 모두 O(log n)으로 동작, 성능 차이 없음  
- 용량 관리: 기본적으로 `PriorityQueue`는 동적으로 크기 증가, 하지만 명시적 용량 제한 시 `offer()` 권장  
   ```java  
   PriorityQueue<Integer> pq = new PriorityQueue<>(3);
   boolean success = pq.offer(4);
   if (!success) {
       System.out.println("큐가 가득 찼습니다.");
   }
   ```

- Spring Boot 연계: REST API에서 `PriorityQueue`로 요청 우선순위 처리 시 `offer()` 사용 권장  
   ```java  
   import org.springframework.web.bind.annotation.PostMapping;
   import org.springframework.web.bind.annotation.RequestBody;
   import org.springframework.web.bind.annotation.RestController;
   import java.util.PriorityQueue;

   @RestController
   public class PriorityController {
       private PriorityQueue<Integer> pq = new PriorityQueue<>(3);

       @PostMapping("/add-priority")
       public String addPriority(@RequestBody Integer priority) {
           return pq.offer(priority) ? "추가 성공" : "큐가 가득 찼습니다.";
       }
   }
   ```  
   application.yml:  
   ```yml  
   spring:
     mvc:
       throw-exception-if-no-handler-found: true
   ```

---

#### 📌 주의사항  

* null 요소: `add()`와 `offer()` 모두 `null` 추가 시 `NullPointerException` 발생  
   ```java  
   pq.add(null); // NullPointerException
   ```  
* 용량 초과: 명시적 용량 제한 없으면 자동 확장, 하지만 메모리 사용량 주의  
* 스레드 안전성: `PriorityQueue`는 스레드 안전하지 않음, `PriorityBlockingQueue` 대체 고려