---
title: "PriorityQueue의 기본 정렬 방식과 내림차순 정렬 설정 방법"  
categories:  
  - Java  
tags:  
  - Java  
  - PriorityQueue  
  - Sorting  
  - Comparator  
last_modified_at:   
---

### PriorityQueue의 기본 정렬 방식과 내림차순 정렬 설정 방법

Java의 `PriorityQueue`는 우선순위 큐를 구현하는 데 사용되며, 요소를 특정 순서에 따라 처리하는 데 유용함  
기본적으로 최소 힙(min-heap)을 기반으로 동작하지만, 내림차순(최대 힙, max-heap)으로 정렬하도록 설정할 수 있음  
정렬 방식을 이해하고 커스터마이징하는 방법을 알면 다양한 알고리즘과 데이터 처리에서 효율적으로 활용 가능  

---

#### 📌 용어 설명  
- PriorityQueue: Java의 우선순위 큐로, 내부적으로 힙(heap) 자료구조를 사용  
- 최소 힙(min-heap): 부모 노드가 자식 노드보다 작거나 같은 힙, 가장 작은 요소가 우선순위 높음  
- 최대 힙(max-heap): 부모 노드가 자식 노드보다 크거나 같은 힙, 가장 큰 요소가 우선순위 높음  
- Comparator: 객체 비교 방식을 정의하는 인터페이스, `PriorityQueue` 정렬 순서 커스터마이징에 사용  
- Comparable: 객체 자체의 자연 순서를 정의하는 인터페이스, 기본 정렬에 사용  

---

#### 📌 PriorityQueue의 기본 정렬 방식  

1. 기본 동작:  
   - `PriorityQueue`는 최소 힙을 기반으로 동작, 즉 가장 작은 요소가 큐의 맨 앞에 위치  
   - 요소가 `Comparable` 인터페이스를 구현해야 하며, `compareTo` 메서드에 따라 정렬  
   - 예: `Integer`, `String` 등은 기본적으로 오름차순(작은 값 → 큰 값) 정렬  
   ```java  
   import java.util.PriorityQueue;

   public class PriorityQueueExample {
       public static void main(String[] args) {
           PriorityQueue<Integer> pq = new PriorityQueue<>();
           pq.offer(5);
           pq.offer(2);
           pq.offer(8);
           System.out.println(pq.poll()); // 출력: 2
           System.out.println(pq.poll()); // 출력: 5
           System.out.println(pq.poll()); // 출력: 8
       }
   }
   ```  
   - 결과: 요소가 오름차순으로 poll됨 (2, 5, 8)  

2. 문제 상황:  
   - 기본 오름차순 정렬은 최대값을 먼저 처리해야 하는 경우(예: 최대 힙 필요 시) 적합하지 않음  
   - 커스텀 객체 사용 시 `Comparable` 미구현하면 `ClassCastException` 발생  

> 💡 기본 정렬의 동작 원리  
> `PriorityQueue`는 내부적으로 힙 구조를 유지하며, 요소 추가(`offer`) 또는 제거(`poll`) 시 힙 속성을 보장  
> `Comparable`의 `compareTo`를 통해 요소 비교, 없으면 `Comparator`로 대체 가능  

---

#### 📌 내림차순 정렬 설정 방법  

1. Comparator 사용:  
   - `PriorityQueue` 생성 시 `Comparator`를 제공해 내림차순 정렬 설정  
   ```java  
   import java.util.PriorityQueue;
   import java.util.Comparator;

   public class PriorityQueueDescExample {
       public static void main(String[] args) {
           PriorityQueue<Integer> pq = new PriorityQueue<>(Comparator.reverseOrder());
           pq.offer(5);
           pq.offer(2);
           pq.offer(8);
           System.out.println(pq.poll()); // 출력: 8
           System.out.println(pq.poll()); // 출력: 5
           System.out.println(pq.poll()); // 출력: 2
       }
   }
   ```  
   - `Comparator.reverseOrder()`는 최대 힙을 구현, 가장 큰 요소부터 반환  

2. 커스텀 Comparator 정의:  
   - 특정 기준으로 내림차순 정렬 필요 시 직접 `Comparator` 구현  
   ```java  
   import java.util.PriorityQueue;
   import java.util.Comparator;

   class Student {
       String name;
       int score;

       Student(String name, int score) {
           this.name = name;
           this.score = score;
       }

       @Override
       public String toString() {
           return name + ": " + score;
       }
   }

   public class CustomPriorityQueueExample {
       public static void main(String[] args) {
           PriorityQueue<Student> pq = new PriorityQueue<>(
               (s1, s2) -> Integer.compare(s2.score, s1.score) // 점수 내림차순
           );
           pq.offer(new Student("Alice", 90));
           pq.offer(new Student("Bob", 95));
           pq.offer(new Student("Charlie", 85));
           System.out.println(pq.poll()); // 출력: Bob: 95
           System.out.println(pq.poll()); // 출력: Alice: 90
           System.out.println(pq.poll()); // 출력: Charlie: 85
       }
   }
   ```

3. Collections.reverseOrder() 사용:  
   - `Comparable`을 구현한 객체의 기본 순서를 반대로 설정  
   ```java  
   PriorityQueue<Integer> pq = new PriorityQueue<>(Collections.reverseOrder());
   ```

4. IntelliJ 디버깅 팁:  
   - `PriorityQueue`의 내부 배열 상태를 확인하려면 디버거에서 `pq` 객체의 `queue` 필드 점검  
   - `offer` 또는 `poll` 호출 시 힙 구조 변화 추적 가능  

---

#### 📌 정렬 방식 비교  

| 정렬 방식 | 설명 |  
| --- | --- |  
| 기본(오름차순) | 최소 힙, `Comparable`의 `compareTo` 기반, 작은 값 우선 |  
| 내림차순 | 최대 힙, `Comparator.reverseOrder()` 또는 커스텀 `Comparator`로 큰 값 우선 |  
| 커스텀 정렬 | 특정 기준(예: 객체 필드)으로 정렬, `Comparator` 구현 필요 |  

---

#### 📌 추가 팁  

- 성능 고려: `PriorityQueue`의 추가/제거 연산은 O(log n), 조회는 O(1)  
- Comparable 구현: 커스텀 객체 사용 시 `Comparable` 구현 또는 `Comparator` 제공 필수  
   ```java  
   class Student implements Comparable<Student> {
       String name;
       int score;

       @Override
       public int compareTo(Student other) {
           return Integer.compare(this.score, other.score); // 오름차순
       }
   }
   ```  
- 빈 큐 처리: `poll()` 호출 시 큐가 비어 있으면 `null` 반환, 예외 처리 필요  
   ```java  
   if (!pq.isEmpty()) {
       System.out.println(pq.poll());
   }
   ```

---

#### 📌 주의사항  

* Comparable 미구현: 커스텀 객체에 `Comparable` 또는 `Comparator` 없으면 런타임 에러 발생  
* 불변성: `PriorityQueue`에 추가된 객체의 상태 변경 시 힙 구조 깨질 수 있음, 재정렬 필요  
* 용도 제한: `PriorityQueue`는 순차적 순회 보장 안 함, `poll()`로 순서대로 접근 권장