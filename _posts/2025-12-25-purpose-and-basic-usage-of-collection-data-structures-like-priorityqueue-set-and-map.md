---
title: "PriorityQueue, Set, Map 같은 컬렉션 자료구조의 목적과 가장 기본적인 사용 예시"  
categories:  
  - Java  
tags:  
  - Java  
  - PriorityQueue  
  - Set  
  - Map  
  - Collections  
last_modified_at:     
---

### PriorityQueue, Set, Map 같은 컬렉션 자료구조의 목적과 가장 기본적인 사용 예시

Java의 컬렉션 프레임워크는 데이터를 효율적으로 저장하고 관리하기 위한 다양한 자료구조를 제공하며, `PriorityQueue`, `Set`, `Map`은 각각 특정 목적에 최적화된 대표적인 클래스임  
`PriorityQueue`는 우선순위 기반 정렬, `Set`은 중복 제거, `Map`은 키-값 쌍 관리를 목적으로 함  

---

#### 📌 용어 설명  
- PriorityQueue: 힙 기반 우선순위 큐, 요소를 정렬된 순서로 처리 (기본: 최소 힙)  
- Set: 중복 요소를 허용하지 않는 컬렉션, 고유 데이터 관리  
- Map: 키-값 쌍을 저장, 키를 통해 값을 빠르게 조회  
- 컬렉션 프레임워크: Java의 데이터 구조와 알고리즘을 제공하는 API (`java.util`)  
- Hash 기반: 해시 테이블을 사용해 빠른 검색/삽입/삭제 제공 (예: `HashSet`, `HashMap`)  

---

#### 📌 각 컬렉션의 목적과 기본 사용 예시  

1. PriorityQueue  
   - 목적: 요소를 우선순위(정렬 기준)에 따라 관리, 가장 우선순위 높은 요소를 빠르게 처리  
   - 특징: 내부적으로 힙 구조 사용, 기본은 최소 힙(작은 값 우선), `Comparator`로 커스터마이징 가능  
   - 사용 예시: 작업 스케줄링, 최단 경로 알고리즘 (예: 다익스트라)  
   ```java  
   import java.util.PriorityQueue;

   public class PriorityQueueExample {
       public static void main(String[] args) {
           PriorityQueue<Integer> pq = new PriorityQueue<>();
           pq.offer(5); // 요소 추가
           pq.offer(2);
           pq.offer(8);
           System.out.println("최소값: " + pq.poll()); // 출력: 2
           System.out.println("다음: " + pq.poll()); // 출력: 5
       }
   }
   ```

2. Set  
   - 목적: 중복 요소 제거, 고유 데이터 집합 관리  
   - 특징: `HashSet`은 해시 테이블 기반으로 O(1) 평균 시간 복잡도, 순서 보장 안 함  
   - 사용 예시: 중복 없는 데이터 저장 (예: 사용자 ID 목록)  
   ```java  
   import java.util.HashSet;
   import java.util.Set;

   public class SetExample {
       public static void main(String[] args) {
           Set<String> set = new HashSet<>();
           set.add("Apple");
           set.add("Banana");
           set.add("Apple"); // 중복, 추가 안 됨
           System.out.println("Set: " + set); // 출력: [Apple, Banana]
       }
   }
   ```

3. Map  
   - 목적: 키-값 쌍으로 데이터 저장, 키를 통해 빠른 조회  
   - 특징: `HashMap`은 해시 테이블 기반, 키는 고유해야 함, 순서 보장 안 함  
   - 사용 예시: 데이터베이스 조회, 설정 정보 저장 (예: 이름-전화번호 매핑)  
   ```java  
   import java.util.HashMap;
   import java.util.Map;

   public class MapExample {
       public static void main(String[] args) {
           Map<String, String> map = new HashMap<>();
           map.put("Alice", "123-4567");
           map.put("Bob", "987-6543");
           map.put("Alice", "111-2222"); // 키 중복, 값 업데이트
           System.out.println("Alice의 번호: " + map.get("Alice")); // 출력: 111-2222
       }
   }
   ```

---

#### 📌 컬렉션 비교  

| 컬렉션 | 목적 | 주요 특징 | 시간 복잡도 |  
| --- | --- | --- | --- |  
| PriorityQueue | 우선순위 기반 정렬 | 최소/최대 힙, `Comparator`로 정렬 조정 | 삽입/삭제 O(log n), 조회 O(1) |  
| Set | 중복 제거 | 고유 요소, 순서 보장 안 함 (`HashSet`) | 추가/조회 O(1) 평균 |  
| Map | 키-값 매핑 | 고유 키, 빠른 조회 | 추가/조회 O(1) 평균 |  

---

#### 📌 효율적인 사용법  

1. PriorityQueue:  
   - 정렬 기준이 필요한 경우 `Comparator` 사용  
   ```java  
   PriorityQueue<Integer> maxHeap = new PriorityQueue<>((a, b) -> b - a);
   maxHeap.offer(5);
   maxHeap.offer(2);
   System.out.println(maxHeap.poll()); // 출력: 5 (최대 힙)
   ```

2. Set:  
   - 중복 제거 후 추가 처리 (예: 리스트로 변환)  
   ```java  
   Set<Integer> set = new HashSet<>();
   set.add(1);
   set.add(1);
   ArrayList<Integer> list = new ArrayList<>(set); // 중복 없는 리스트
   ```

3. Map:  
   - 키 존재 여부 확인 후 처리  
   ```java  
   Map<String, Integer> scores = new HashMap<>();
   scores.putIfAbsent("Alice", 90); // 키 없으면 추가
   scores.computeIfPresent("Alice", (k, v) -> v + 10); // 값 업데이트
   ```

4. IntelliJ 디버깃 팁:  
   - `PriorityQueue`: 내부 `queue` 배열과 힙 구조 확인  
   - `Set`: `HashSet`의 내부 `HashMap` 테이블 점검  
   - `Map`: 키-값 쌍과 해시 테이블 버킷 확인  

---

#### 📌 추가 팁  

- 입력 처리: `Scanner`로 입력 받을 때 버퍼 정리  
   ```java  
   import java.util.Scanner;

   Scanner scanner = new Scanner(System.in);
   int n = scanner.nextInt();
   scanner.nextLine(); // 버퍼 정리
   Set<String> set = new HashSet<>();
   for (int i = 0; i < n; i++) {
       set.add(scanner.nextLine());
   }
   ```  
- Spring Boot 연계: 컬렉션 데이터를 API로 처리  
   ```java  
   import org.springframework.web.bind.annotation.PostMapping;
   import org.springframework.web.bind.annotation.RequestBody;
   import org.springframework.web.bind.annotation.RestController;
   import java.util.HashMap;
   import java.util.Map;

   @RestController
   public class CollectionController {
       @PostMapping("/store-data")
       public Map<String, String> storeData(@RequestBody Map<String, String> input) {
           Map<String, String> map = new HashMap<>();
           input.forEach((k, v) -> map.put(k, v.trim()));
           return map;
       }
   }
   ```  
   application.yml:  
   ```yml  
   spring:
     mvc:
       throw-exception-if-no-handler-found: true
   ```  
- 대안 컬렉션: 순서 유지 필요 시 `LinkedHashSet`, `TreeMap`, 정렬 필요 시 `TreeSet` 고려  

---

#### 📌 주의사항  

* null 처리:  
   - `PriorityQueue`: `null` 추가 시 `NullPointerException`  
   - `HashSet`, `HashMap`: `null` 허용, 처리 주의  
* 커스텀 객체: `Set`과 `Map`에서 `hashCode()`와 `equals()` 재정의 필수  
   ```java  
   class Person {
       String name;
       @Override
       public boolean equals(Object o) { ... }
       @Override
       public int hashCode() { ... }
   }
   ```  
* 성능 고려: `PriorityQueue` 삽입/삭제는 O(log n), `Set`/`Map`은 O(1) 평균