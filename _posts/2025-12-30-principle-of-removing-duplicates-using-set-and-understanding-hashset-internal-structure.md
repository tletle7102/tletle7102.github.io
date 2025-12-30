---
title: "Set을 사용하여 중복을 제거하는 원리와 HashSet의 내부 구조 이해하기"  
categories:  
  - Java  
tags:  
  - Java  
  - Set  
  - HashSet  
  - Collections  
  - Duplicate Removal  
last_modified_at:   
---

### Set을 사용하여 중복을 제거하는 원리와 HashSet의 내부 구조 이해하기

Java의 `Set` 인터페이스는 중복 요소를 허용하지 않는 컬렉션으로, 데이터의 고유성을 보장하는 데 유용함  
`HashSet`은 `Set`의 대표적인 구현체로, 해시 테이블을 기반으로 빠른 중복 제거와 조회를 제공  
중복 제거 원리와 `HashSet`의 내부 구조를 이해하면 효율적인 데이터 처리와 성능 최적화에 도움이 됨  

---

#### 📌 용어 설명  
- Set: Java의 `Collection` 인터페이스를 구현한 컬렉션으로, 중복 요소를 저장하지 않음  
- HashSet: 해시 테이블을 기반으로 한 `Set` 구현체, 빠른 추가/조회/삭제 연산 제공  
- 해시 테이블: 키-값 쌍을 저장하는 자료구조, 해시 함수를 통해 데이터 위치를 계산  
- hashCode(): 객체의 해시 코드를 반환하는 메서드, `HashSet`의 중복 판별에 사용  
- equals(): 두 객체의 동등성을 비교하는 메서드, `HashSet`에서 중복 여부 확인 시 사용  

---

#### 📌 Set을 사용한 중복 제거 원리  

1. 기본 동작:  
   - `Set`은 요소를 추가할 때 `hashCode()`와 `equals()`를 사용해 중복 여부를 판별  
   - 동일한 `hashCode()`를 가진 요소가 있으면 `equals()`로 동등성 확인, 중복 시 추가 거부  
   ```java  
   import java.util.HashSet;
   import java.util.Set;

   public class SetDuplicateExample {
       public static void main(String[] args) {
           Set<String> set = new HashSet<>();
           set.add("apple");
           set.add("banana");
           set.add("apple"); // 중복, 추가되지 않음
           System.out.println(set); // 출력: [apple, banana]
       }
   }
   ```

2. 중복 판별 과정:  
   - 요소 추가 시 `HashSet`은 다음 단계를 거침:  
     1. 추가하려는 요소의 `hashCode()`를 계산  
     2. 해시 코드에 해당하는 해시 테이블의 버킷(bucket) 확인  
     3. 동일한 해시 코드의 기존 요소와 `equals()`로 비교, 중복이면 추가 안 함  
   - 결과: 중복 요소는 자동으로 제외됨  

3. 문제 상황:  
   - 커스텀 객체 사용 시 `hashCode()`와 `equals()`를 재정의하지 않으면 중복 제거 실패  
   ```java  
   class Person {
       String name;
       int age;

       Person(String name, int age) {
           this.name = name;
           this.age = age;
       }
   }

   Set<Person> set = new HashSet<>();
   set.add(new Person("Alice", 25));
   set.add(new Person("Alice", 25)); // 중복으로 간주되지 않음
   ```  
   - `hashCode()`와 `equals()` 미구현으로 동일 객체로 인식되지 않음  

> 💡 중복 제거의 핵심  
> `HashSet`은 `hashCode()`로 빠르게 요소 위치를 찾고, `equals()`로 정확한 동등성을 확인  
> 커스텀 객체 사용 시 두 메서드를 반드시 재정의해야 함  

---

#### 📌 HashSet의 내부 구조  

1. 해시 테이블 기반:  
   - `HashSet`은 내부적으로 `HashMap`을 사용, 요소는 `HashMap`의 키로 저장  
   - 각 요소는 해시 테이블의 버킷에 저장되며, 해시 충돌 시 연결 리스트(또는 Java 8 이상에서 트리)로 관리  

2. 주요 동작:  
   - 추가(add): O(1) 평균 시간 복잡도, 해시 충돌 시 O(log n) 또는 O(n)  
   - 조회(contains): O(1) 평균, 충돌 시 성능 저하  
   - 삭제(remove): O(1) 평균, 충돌 시 성능 저하  

3. 커스텀 객체 사용 예시:  
   ```java  
   class Person {
       String name;
       int age;

       Person(String name, int age) {
           this.name = name;
           this.age = age;
       }

       @Override
       public boolean equals(Object o) {
           if (this == o) return true;
           if (o == null || getClass() != o.getClass()) return false;
           Person person = (Person) o;
           return age == person.age && name.equals(person.name);
       }

       @Override
       public int hashCode() {
           return Objects.hash(name, age);
       }

       @Override
       public String toString() {
           return name + ": " + age;
       }
   }

   public class HashSetCustomExample {
       public static void main(String[] args) {
           Set<Person> set = new HashSet<>();
           set.add(new Person("Alice", 25));
           set.add(new Person("Alice", 25)); // 중복, 추가되지 않음
           System.out.println(set); // 출력: [Alice: 25]
       }
   }
   ```

4. IntelliJ 디버깅 팁:  
   - `HashSet`의 내부 `HashMap` 구조 확인: 디버거로 `table` 필드 점검  
   - 해시 충돌 여부 확인: 동일 버킷에 여러 요소가 연결 리스트로 저장되었는지 확인  

---

#### 📌 HashSet 사용 시 장점과 한계  

| 항목 | 설명 |  
| --- | --- |  
| 장점 | 빠른 중복 제거(O(1) 평균), 간단한 API, 메모리 효율적 |  
| 한계 | 순서 보장 안 함, `hashCode()`와 `equals()` 재정의 필요 |  

---

#### 📌 추가 팁  

- Spring Boot 연계: REST API에서 중복 데이터 처리 시 `HashSet` 활용  
   ```java  
   import org.springframework.web.bind.annotation.PostMapping;
   import org.springframework.web.bind.annotation.RequestBody;
   import org.springframework.web.bind.annotation.RestController;
   import java.util.HashSet;
   import java.util.Set;

   @RestController
   public class UniqueDataController {
       @PostMapping("/unique-data")
       public Set<String> addUniqueData(@RequestBody String[] data) {
           Set<String> uniqueData = new HashSet<>();
           for (String item : data) {
               uniqueData.add(item.trim());
           }
           return uniqueData;
       }
   }
   ```  
   application.yml:  
   ```yml  
   spring:
     mvc:
       throw-exception-if-no-handler-found: true
   ```

- 대안 컬렉션: 순서 보장이 필요하면 `LinkedHashSet`, 정렬 필요 시 `TreeSet` 고려  
- 성능 최적화: `hashCode()` 구현 시 균일한 분포를 위해 `Objects.hash()` 사용 권장  

---

#### 📌 주의사항  

* `hashCode()`와 `equals()` 일관성: 두 메서드가 일관되지 않으면 중복 제거 실패  
* 불변 객체 권장: `HashSet`에 추가된 객체의 상태 변경 시 해시 테이블 구조 깨질 수 있음  
* null 처리: `HashSet`은 `null` 요소 허용, 하지만 `null` 체크 필요  
   ```java  
   Set<String> set = new HashSet<>();
   set.add(null); // 허용