---
title: "Java에서 배열과 ArrayList의 차이점과 언제 어떤 자료구조를 선택해야 하는가"  
categories:  
  - Java  
tags:  
  - Java  
  - Array  
  - ArrayList  
  - Collections  
  - Data Structure  
last_modified_at:   
---

### Java에서 배열과 ArrayList의 차이점과 언제 어떤 자료구조를 선택해야 하는가

Java에서 배열(`array`)과 `ArrayList`는 데이터를 저장하고 관리하는 대표적인 자료구조로, 각각 고정된 크기와 동적 크기라는 특징을 가짐  
이 둘의 차이점을 이해하고 사용 목적에 따라 적절히 선택하면 성능과 코드 유지보수성을 최적화할 수 있음  

---

#### 📌 용어 설명  
- 배열(array): 고정된 크기의 연속된 메모리 공간, 동일한 데이터 타입의 요소 저장  
- ArrayList: 동적 크기를 가진 `List` 인터페이스의 구현체, 내부적으로 배열을 사용  
- 고정 크기: 배열의 크기는 생성 시 결정되며 변경 불가  
- 동적 크기: `ArrayList`는 필요에 따라 크기를 자동으로 확장/축소  
- Collections Framework: Java의 컬렉션 API, `ArrayList`는 이 프레임워크의 일부  

---

#### 📌 배열과 ArrayList의 차이점  

1. 크기 관리:  
   - 배열: 크기가 고정, 생성 시 지정된 크기 변경 불가  
     ```java  
     int[] array = new int[5]; // 크기 5 고정
     array[0] = 1;
     // array[5] = 6; // ArrayIndexOutOfBoundsException
     ```  
   - ArrayList: 크기 동적 조정, 요소 추가/제거 시 자동 확장  
     ```java  
     import java.util.ArrayList;

     ArrayList<Integer> list = new ArrayList<>();
     list.add(1); // 동적 추가
     list.add(2); // 크기 자동 확장
     ```

2. 데이터 타입:  
   - 배열: 기본 타입(`int`, `double` 등)과 참조 타입 모두 지원  
     ```java  
     int[] numbers = {1, 2, 3};
     String[] names = {"Alice", "Bob"};
     ```  
   - ArrayList: 참조 타입만 저장, 기본 타입은 래퍼 클래스(`Integer`, `Double` 등)로 자동 박싱  
     ```java  
     ArrayList<Integer> list = new ArrayList<>();
     list.add(1); // int -> Integer (오토박싱)
     ```

3. 성능:  
   - 배열: 고정 크기로 메모리 접근이 빠름, O(1) 인덱스 접근  
   - ArrayList: 동적 크기 조정으로 추가/삭제 시 O(n) 가능, 내부 배열 복사로 인한 오버헤드  

4. 기능성:  
   - 배열: 기본적인 인덱스 접근 외 기능 제한적  
   - ArrayList: `add()`, `remove()`, `contains()` 등 다양한 메서드 제공  

5. 메모리 사용:  
   - 배열: 고정 크기로 메모리 효율적  
   - ArrayList: 동적 확장 시 추가 메모리 할당, 초기 용량 설정으로 최적화 가능  

---

#### 📌 언제 어떤 자료구조를 선택해야 하는가  

1. 배열을 선택해야 할 때:  
   - 데이터 크기가 고정되어 있고 변경되지 않을 때  
   - 기본 타입 데이터를 다룰 때 (박싱/언박싱 오버헤드 없음)  
   - 최대한의 성능 최적화가 필요한 경우 (예: 대규모 데이터 처리)  
   ```java  
   // 고정된 5개 점수 저장
   int[] scores = new int[5];
   scores[0] = 90;
   scores[1] = 85;
   ```

2. ArrayList를 선택해야 할 때:  
   - 데이터 크기가 가변적이거나 예측 불가능할 때  
   - 추가/삭제 작업이 빈번할 때  
   - `Collections` 프레임워크의 메서드(`sort`, `contains` 등) 활용 필요 시  
   ```java  
   import java.util.ArrayList;

   ArrayList<String> names = new ArrayList<>();
   names.add("Alice");
   names.add("Bob");
   names.remove("Alice"); // 동적 제거
   ```

3. 결합 사용 예시:  
   - `ArrayList`로 동적 처리 후 배열로 변환해 고정 데이터 활용  
   ```java  
   import java.util.ArrayList;

   public class ArrayListToArray {
       public static void main(String[] args) {
           ArrayList<Integer> list = new ArrayList<>();
           list.add(1);
           list.add(2);
           Integer[] array = list.toArray(new Integer[0]);
           System.out.println(java.util.Arrays.toString(array)); // 출력: [1, 2]
       }
   }
   ```

4. IntelliJ 디버깅 팁:  
   - 배열: 디버거로 배열 길이(`length`)와 각 인덱스 값 확인  
   - `ArrayList`: 내부 `elementData` 배열과 `size` 필드 점검  

---

#### 📌 성능 및 사용 비교  

| 항목 | 배열 | ArrayList |  
| --- | --- | --- |  
| 크기 | 고정, 변경 불가 | 동적, 자동 확장 |  
| 성능 | 인덱스 접근 O(1), 메모리 효율적 | 인덱스 접근 O(1), 추가/삭제 O(n) |  
| 데이터 타입 | 기본 타입, 참조 타입 | 참조 타입(래퍼 클래스) |  
| 기능 | 제한적 | 다양한 메서드 제공 |  

---

#### 📌 추가 팁  

- ArrayList 초기 용량 설정: 메모리 효율성을 위해 초기 용량 지정  
   ```java  
   ArrayList<Integer> list = new ArrayList<>(100); // 초기 용량 100
   ```  
- 배열 복사: `Arrays.copyOf()`로 배열 크기 조정 가능, 단 고정 크기 한계 유지  
   ```java  
   int[] array = {1, 2, 3};
   array = java.util.Arrays.copyOf(array, 5); // 크기 확장
   ```  
- Spring Boot 연계: API에서 동적 데이터 처리 시 `ArrayList` 활용  
   ```java  
   import org.springframework.web.bind.annotation.PostMapping;
   import org.springframework.web.bind.annotation.RequestBody;
   import org.springframework.web.bind.annotation.RestController;
   import java.util.ArrayList;

   @RestController
   public class DataController {
       @PostMapping("/process-data")
       public ArrayList<Integer> processData(@RequestBody int[] input) {
           ArrayList<Integer> list = new ArrayList<>();
           for (int num : input) {
               list.add(num);
           }
           return list;
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

* null 처리: `ArrayList`는 `null` 요소 허용, 배열은 타입에 따라 제한  
   ```java  
   ArrayList<String> list = new ArrayList<>();
   list.add(null); // 허용
   String[] array = new String[2];
   array[0] = null; // 허용
   ```  
* 성능 고려: `ArrayList`의 동적 확장은 메모리 복사 비용 발생, 큰 데이터는 초기 용량 설정 권장  
* 기본 타입: `ArrayList` 사용 시 오토박싱/언박싱으로 성능 저하 가능, 배열로 대체 고려