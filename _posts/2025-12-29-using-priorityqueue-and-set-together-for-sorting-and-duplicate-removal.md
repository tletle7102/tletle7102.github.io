---
title: "PriorityQueue와 Set을 함께 활용해 정렬과 중복 제거를 동시에 처리하는 방법"  
categories:  
  - Java  
tags:  
  - Java  
  - PriorityQueue  
  - Set  
  - Sorting  
  - Duplicate Removal  
last_modified_at:   
---

### PriorityQueue와 Set을 함께 활용해 정렬과 중복 제거를 동시에 처리하는 방법

Java에서 `PriorityQueue`는 요소를 정렬된 순서로 관리하고, `Set`은 중복 요소를 제거하는 데 유용함  
이 둘을 조합하면 중복을 제거한 데이터를 정렬된 순서로 처리하는 효율적인 솔루션을 구현할 수 있음  
알고리즘 문제나 데이터 처리에서 중복 제거와 정렬이 동시에 필요한 경우에 특히 효과적   

---

#### 📌 용어 설명  
- PriorityQueue: 힙 기반 우선순위 큐, 요소를 정렬된 순서로 유지 (기본: 최소 힙)  
- Set: 중복 요소를 허용하지 않는 컬렉션, `HashSet`은 빠른 중복 제거 제공  
- 중복 제거: 동일한 요소를 하나로 통합, `Set`의 `hashCode()`와 `equals()` 사용  
- 정렬: 요소를 특정 기준(오름차순/내림차순)으로 순서 정렬, `PriorityQueue`의 `Comparator` 활용  
- HashSet: 해시 테이블 기반 `Set` 구현체, O(1) 평균 시간 복잡도로 중복 제거  

---

#### 📌 PriorityQueue와 Set 조합의 필요성  

1. 문제 상황:  
   - 입력 데이터에 중복이 포함되어 있고, 이를 정렬된 순서로 출력해야 할 때  
   - 예: 숫자 리스트에서 중복을 제거하고 오름차순/내림차순으로 출력  
   - `PriorityQueue` 단독 사용 시 중복 제거 불가, `Set` 단독 사용 시 정렬 불가  

2. 조합 동작:  
   - `Set`으로 중복 제거 후, 결과를 `PriorityQueue`에 넣어 정렬  
   - 반대 순서(정렬 후 중복 제거)도 가능, 상황에 따라 선택  

3. 기본 예시:  
   ```java  
   import java.util.HashSet;
   import java.util.PriorityQueue;
   import java.util.Set;

   public class SetAndPriorityQueueExample {
       public static void main(String[] args) {
           // 입력 데이터 (중복 포함)
           int[] numbers = {5, 2, 8, 2, 5, 1};
           
           // 1. Set으로 중복 제거
           Set<Integer> set = new HashSet<>();
           for (int num : numbers) {
               set.add(num);
           }
           
           // 2. PriorityQueue로 정렬
           PriorityQueue<Integer> pq = new PriorityQueue<>(set);
           System.out.println("오름차순: ");
           while (!pq.isEmpty()) {
               System.out.print(pq.poll() + " "); // 출력: 1 2 5 8
           }
       }
   }
   ```

> 💡 조합의 동작 원리  
> `Set`은 `hashCode()`와 `equals()`로 중복을 제거해 고유 요소만 남김  
> `PriorityQueue`는 `Comparable` 또는 `Comparator`로 요소를 정렬된 순서로 관리  
> 두 컬렉션을 결합하면 중복 없는 정렬된 결과를 효율적으로 얻음  

---

#### 📌 구현 방법  

1. 오름차순 정렬:  
   - `HashSet`으로 중복 제거 후 `PriorityQueue`로 오름차순 정렬  
   ```java  
   import java.util.HashSet;
   import java.util.PriorityQueue;
   import java.util.Set;

   public class AscendingExample {
       public static void main(String[] args) {
           int[] numbers = {5, 2, 8, 2, 5, 1};
           Set<Integer> set = new HashSet<>();
           for (int num : numbers) {
               set.add(num);
           }
           
           PriorityQueue<Integer> pq = new PriorityQueue<>();
           pq.addAll(set);
           System.out.println("오름차순: ");
           while (!pq.isEmpty()) {
               System.out.print(pq.poll() + " "); // 출력: 1 2 5 8
           }
       }
   }
   ```

2. 내림차순 정렬:  
   - `Comparator.reverseOrder()`를 사용해 최대 힙으로 설정  
   ```java  
   import java.util.HashSet;
   import java.util.PriorityQueue;
   import java.util.Set;
   import java.util.Comparator;

   public class DescendingExample {
       public static void main(String[] args) {
           int[] numbers = {5, 2, 8, 2, 5, 1};
           Set<Integer> set = new HashSet<>();
           for (int num : numbers) {
               set.add(num);
           }
           
           PriorityQueue<Integer> pq = new PriorityQueue<>(Comparator.reverseOrder());
           pq.addAll(set);
           System.out.println("내림차순: ");
           while (!pq.isEmpty()) {
               System.out.print(pq.poll() + " "); // 출력: 8 5 2 1
           }
       }
   }
   ```

3. 커스텀 객체 처리:  
   - 커스텀 객체 사용 시 `hashCode()`와 `equals()` 재정의 필요  
   ```java  
   import java.util.HashSet;
   import java.util.PriorityQueue;
   import java.util.Set;

   class Item {
       String name;
       int value;

       Item(String name, int value) {
           this.name = name;
           this.value = value;
       }

       @Override
       public boolean equals(Object o) {
           if (this == o) return true;
           if (o == null || getClass() != o.getClass()) return false;
           Item item = (Item) o;
           return value == item.value && name.equals(item.name);
       }

       @Override
       public int hashCode() {
           return Objects.hash(name, value);
       }

       @Override
       public String toString() {
           return name + ": " + value;
       }
   }

   public class CustomObjectExample {
       public static void main(String[] args) {
           Set<Item> set = new HashSet<>();
           set.add(new Item("Apple", 5));
           set.add(new Item("Banana", 2));
           set.add(new Item("Apple", 5)); // 중복

           PriorityQueue<Item> pq = new PriorityQueue<>((a, b) -> Integer.compare(a.value, b.value));
           pq.addAll(set);
           System.out.println("오름차순 (value 기준): ");
           while (!pq.isEmpty()) {
               System.out.println(pq.poll()); // 출력: Banana: 2, Apple: 5
           }
       }
   }
   ```

4. IntelliJ 디버깅 팁:  
   - `Set`의 내부 `HashMap`과 `PriorityQueue`의 `queue` 배열을 디버거로 확인  
   - `add()` 또는 `offer()` 호출 시 힙 구조 변화 점검  

---

#### 📌 조합의 장점과 한계  

| 항목 | 설명 |  
| --- | --- |  
| 장점 | 중복 제거와 정렬 동시 처리, O(n log n) 시간 복잡도로 효율적 |  
| 한계 | `PriorityQueue`는 순차적 순회 보장 안 함, `poll()`로 접근 필요 |  

---

#### 📌 추가 팁  

- 성능 최적화: `HashSet`의 O(1) 추가와 `PriorityQueue`의 O(log n) 삽입으로 효율적  
- 대안 컬렉션: 정렬된 순서를 유지하려면 `TreeSet` 사용 가능, 단 O(log n) 추가 비용  
   ```java  
   TreeSet<Integer> treeSet = new TreeSet<>();
   treeSet.addAll(set); // 중복 제거 + 정렬
   ```  
- Spring Boot 연계: API에서 중복 제거와 정렬 처리  
   ```java  
   import org.springframework.web.bind.annotation.PostMapping;
   import org.springframework.web.bind.annotation.RequestBody;
   import org.springframework.web.bind.annotation.RestController;
   import java.util.HashSet;
   import java.util.PriorityQueue;
   import java.util.Set;

   @RestController
   public class SortedUniqueController {
       @PostMapping("/sorted-unique")
       public Integer[] getSortedUnique(@RequestBody Integer[] data) {
           Set<Integer> set = new HashSet<>();
           for (int num : data) {
               set.add(num);
           }
           PriorityQueue<Integer> pq = new PriorityQueue<>(set);
           return pq.toArray(new Integer[0]);
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

* null 처리: `HashSet`은 `null` 허용, `PriorityQueue`는 `null` 추가 시 `NullPointerException`  
   ```java  
   Set<Integer> set = new HashSet<>();
   set.add(null); // 허용
   PriorityQueue<Integer> pq = new PriorityQueue<>();
   pq.add(null); // NullPointerException
   ```  
* 커스텀 객체: `Set`과 `PriorityQueue` 모두 `hashCode()`/`equals()` 또는 `Comparable`/`Comparator` 구현 필요  
* 스레드 안전성: 두 컬렉션 모두 스레드 안전하지 않음, 동시성 환경에서는 `Collections.synchronizedSet()` 등 사용