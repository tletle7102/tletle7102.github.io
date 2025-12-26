---
title: "힙(Heap) 자료구조를 아스키 아트로 시각화하고 PriorityQueue와의 관계 이해하기"  
categories:  
  - Algorithm  
tags:  
  - Java  
  - Heap  
  - PriorityQueue  
  - Data Structure  
  - Algorithm  
last_modified_at:   
---

### 힙(Heap) 자료구조를 아스키 아트로 시각화하고 PriorityQueue와의 관계 이해하기

힙(Heap)은 특정 순서를 유지하는 완전 이진 트리 기반 자료구조로, 우선순위 큐를 구현하는 데 자주 사용됨  
Java의 `PriorityQueue`는 힙을 내부적으로 활용하여 요소를 정렬된 순서로 관리함  
이 문서에서는 힙의 구조를 아스키 아트로 시각화하고, `PriorityQueue`와 관계를 설명함  

---

#### 📌 용어 설명  
- 힙(Heap): 부모 노드가 자식 노드보다 크거나(최대 힙) 작거나(최소 힙) 같은 완전 이진 트리  
- 최소 힙(min-heap): 부모 노드가 자식 노드보다 작거나 같음, 가장 작은 값이 루트에 위치  
- 최대 힙(max-heap): 부모 노드가 자식 노드보다 크거나 같음, 가장 큰 값이 루트에 위치  
- PriorityQueue: Java의 우선순위 큐, 기본적으로 최소 힙 기반  
- 완전 이진 트리: 모든 레벨이 꽉 찬 이진 트리, 마지막 레벨은 왼쪽부터 채워짐  

---

#### 📌 힙의 구조와 아스키 아트 시각화  

1. 힙의 특징:  
   - 힙은 배열로 구현되며, 완전 이진 트리 형태를 유지  
   - 부모 노드의 인덱스 `i`에 대해:  
     - 왼쪽 자식: `2*i + 1`  
     - 오른쪽 자식: `2*i + 2`  
     - 부모: `(i-1)/2`  
   - 최소 힙 예시 (요소: [3, 5, 7, 9, 12])  

2. 아스키 아트로 최소 힙 시각화:  
   ```
       3
      / \
     5   7
    / \
   9  12
   ```  
   - 배열 표현: `[3, 5, 7, 9, 12]`  
   - 루트(3)는 가장 작은 값, 자식 노드(5, 7)는 부모보다 크거나 같음  

3. 아스키 아트로 최대 힙 시각화:  
   ```
       12
      /  \
     9    7
    / \
   5   3
   ```  
   - 배열 표현: `[12, 9, 7, 5, 3]`  
   - 루트(12)는 가장 큰 값, 자식 노드(9, 7)는 부모보다 작거나 같음  

4. 힙 동작 원리:  
   - 삽입: 새 요소를 배열 끝에 추가 후, 부모와 비교하며 위로 이동 (Heapify-up)  
   - 삭제: 루트 제거 후, 마지막 요소를 루트로 이동 후 아래로 조정 (Heapify-down)  

---

#### 📌 PriorityQueue와의 관계  

1. PriorityQueue의 내부 구현:  
   - Java의 `PriorityQueue`는 최소 힙을 기본으로 사용, 배열로 힙 구조 유지  
   - 요소 추가(`offer`) 또는 제거(`poll`) 시 힙 속성 유지, 시간 복잡도 O(log n)  
   ```java  
   import java.util.PriorityQueue;

   public class PriorityQueueExample {
       public static void main(String[] args) {
           PriorityQueue<Integer> pq = new PriorityQueue<>();
           pq.offer(5);
           pq.offer(3);
           pq.offer(7);
           System.out.println(pq); // 출력: [3, 5, 7] (최소 힙)
           System.out.println("최소값: " + pq.poll()); // 출력: 3
       }
   }
   ```

2. 최대 힙 구현:  
   - `Comparator.reverseOrder()`로 최대 힙 설정  
   ```java  
   PriorityQueue<Integer> maxHeap = new PriorityQueue<>((a, b) -> b - a);
   maxHeap.offer(5);
   maxHeap.offer(3);
   maxHeap.offer(7);
   System.out.println(maxHeap); // 출력: [7, 3, 5] (최대 힙)
   ```

3. 힙과 PriorityQueue의 연관성:  
   - `PriorityQueue`는 힙의 속성을 활용해 가장 우선순위 높은 요소(루트)를 빠르게 제공  
   - 삽입/삭제 시 힙 구조를 유지하며 자동 정렬  
   - 내부 배열은 힙의 완전 이진 트리 구조를 반영  

---

#### 📌 기본 사용 예시  

1. 최소 힙 (PriorityQueue 기본):  
   ```java  
   import java.util.PriorityQueue;

   public class MinHeapExample {
       public static void main(String[] args) {
           PriorityQueue<Integer> pq = new PriorityQueue<>();
           pq.offer(9);
           pq.offer(3);
           pq.offer(12);
           System.out.println("최소 힙 순서: ");
           while (!pq.isEmpty()) {
               System.out.print(pq.poll() + " "); // 출력: 3 9 12
           }
       }
   }
   ```

2. 최대 힙 (PriorityQueue 커스터마이징):  
   ```java  
   import java.util.PriorityQueue;
   import java.util.Comparator;

   public class MaxHeapExample {
       public static void main(String[] args) {
           PriorityQueue<Integer> pq = new PriorityQueue<>(Comparator.reverseOrder());
           pq.offer(9);
           pq.offer(3);
           pq.offer(12);
           System.out.println("최대 힙 순서: ");
           while (!pq.isEmpty()) {
               System.out.print(pq.poll() + " "); // 출력: 12 9 3
           }
       }
   }
   ```

3. IntelliJ 디버깅 팁:  
   - `PriorityQueue`의 내부 `queue` 배열을 디버거로 확인, 힙 구조 점검  
   - `offer` 또는 `poll` 호출 시 배열 변화(Heapify-up/down) 추적  

---

#### 📌 힙과 PriorityQueue 비교  

| 항목 | 힙 | PriorityQueue |  
| --- | --- | --- |  
| 구조 | 완전 이진 트리, 배열로 구현 | 힙 기반, Java 컬렉션 프레임워크 |  
| 목적 | 우선순위 관리 | 우선순위 큐 구현 |  
| 연산 | 삽입/삭제 O(log n), 조회 O(1) | 동일, API로 간편 사용 |  
| 커스터마이징 | 직접 구현 필요 | `Comparator`로 간단 설정 |  

---

#### 📌 추가 팁  

- 커스텀 객체 사용: `PriorityQueue`에 커스텀 객체 추가 시 `Comparable` 또는 `Comparator` 구현  
   ```java  
   class Task implements Comparable<Task> {
       int priority;
       Task(int priority) { this.priority = priority; }
       @Override
       public int compareTo(Task other) { return this.priority - other.priority; }
   }

   PriorityQueue<Task> pq = new PriorityQueue<>();
   pq.offer(new Task(5));
   ```  
- Spring Boot 연계: 힙 기반 우선순위 처리 API  
   ```java  
   import org.springframework.web.bind.annotation.PostMapping;
   import org.springframework.web.bind.annotation.RequestBody;
   import org.springframework.web.bind.annotation.RestController;
   import java.util.PriorityQueue;

   @RestController
   public class PriorityController {
       @PostMapping("/priority-tasks")
       public Integer[] processTasks(@RequestBody Integer[] tasks) {
           PriorityQueue<Integer> pq = new PriorityQueue<>();
           for (int task : tasks) {
               pq.offer(task);
           }
           Integer[] result = new Integer[pq.size()];
           for (int i = 0; i < result.length; i++) {
               result[i] = pq.poll();
           }
           return result;
       }
   }
   ```  
   application.yml:  
   ```yml  
   spring:
     mvc:
       throw-exception-if-no-handler-found: true
   ```  
- 입력 처리: `Scanner`로 입력 받을 때 버퍼 정리  
   ```java  
   import java.util.Scanner;

   Scanner scanner = new Scanner(System.in);
   int n = scanner.nextInt();
   scanner.nextLine(); // 버퍼 정리
   ```  

---

#### 📌 주의사항  

* null 처리: `PriorityQueue`는 `null` 추가 시 `NullPointerException` 발생  
* 힙 속성 유지: 요소 삽입/삭제 후 힙 구조 자동 유지 확인  
* 성능 고려: 큰 데이터셋에서 삽입/삭제 O(log n), 빈번한 작업 시 메모리 주의