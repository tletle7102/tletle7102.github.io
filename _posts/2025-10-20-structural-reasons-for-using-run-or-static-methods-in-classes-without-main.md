---
title: "main() 메서드가 없는 클래스에서 run()이나 static 메서드를 사용하는 구조적 이유"
categories:
  - Java
tags:
  - Java
  - Main Method
  - Static Method
  - Run Method
  - OOP
last_modified_at: 
---

### main() 메서드가 없는 클래스에서 run()이나 static 메서드를 사용하는 구조적 이유

Java에서 `main()` 메서드는 프로그램의 진입점(entry point)으로 사용되지만, 모든 클래스가 `main()` 메서드를 가질 필요는 없음  
`main()` 메서드가 없는 클래스에서 `run()`이나 `static` 메서드를 사용하는 것은 코드의 모듈화, 재사용성, 그리고 객체 지향 설계를 강화하기 위한 구조적 선택

---

#### 📌 용어 설명  
- main() 메서드: `public static void main(String[] args)`로 정의된 프로그램의 진입점  
- static 메서드: 클래스 인스턴스 없이 호출 가능한 메서드, 클래스 자체에 속함  
- run() 메서드: 특정 작업을 실행하도록 설계된 메서드, 주로 `Runnable` 인터페이스와 연관  
- 모듈화: 코드를 기능별로 분리하여 관리 및 재사용 용이하게 설계  
- 객체 지향 설계: 캡슐화, 재사용성, 유지보수를 고려한 코드 구조  

---

#### 📌 구조적 이유  

1. 모듈화와 책임 분리:  
   - `main()` 메서드는 프로그램 실행의 시작점 역할, 모든 로직을 포함하면 코드가 복잡해짐  
   - `run()`이나 `static` 메서드는 특정 기능을 독립적으로 처리, 코드 가독성과 유지보수성 향상  
   - 예: `main()`에서 실행 흐름 제어, `run()`에서 특정 작업 수행  
   ```java  
   public class TaskRunner {
       public static void main(String[] args) {
           Task task = new Task();
           task.run(); // 작업 실행
       }
   }

   class Task {
       void run() {
           System.out.println("작업을 실행합니다.");
       }
   }
   ```

2. 재사용성:  
   - `static` 메서드는 인스턴스 생성 없이 호출 가능, 유틸리티 함수에 적합  
   - `run()`은 객체의 특정 동작을 캡슐화, 다른 클래스에서 재사용 가능  
   ```java  
   public class Utility {
       static int add(int a, int b) {
           return a + b;
       }
   }

   public class Main {
       public static void main(String[] args) {
           int sum = Utility.add(5, 3); // static 메서드 호출
           System.out.println("합: " + sum); // 출력: 합: 8
       }
   }
   ```

3. 객체 지향 설계 지원:  
   - `run()` 메서드는 `Runnable` 인터페이스 구현 시 스레드 실행에 사용, 다중 작업 처리 가능  
   - `main()` 없는 클래스는 객체의 책임을 명확히 정의, SRP(단일 책임 원칙) 준수  
   ```java  
   public class Worker implements Runnable {
       @Override
       public void run() {
           System.out.println("스레드 작업 수행");
       }
   }

   public class Main {
       public static void main(String[] args) {
           Thread thread = new Thread(new Worker());
           thread.start(); // run() 호출
       }
   }
   ```

4. 테스트와 유연성:  
   - `main()` 없는 클래스는 독립적으로 테스트 가능, `static`이나 `run()` 메서드로 특정 기능 단위 테스트 용이  
   - 예: JUnit 테스트에서 `main()` 없이 메서드 호출  
   ```java  
   public class Calculator {
       static int multiply(int a, int b) {
           return a * b;
       }
   }
   ```

---

#### 📌 실전 사용 예시  

1. static 메서드 (유틸리티 함수):  
   - 공통 기능 제공, 인스턴스 생성 없이 호출  
   ```java  
   public class MathUtils {
       static int square(int n) {
           return n * n;
       }
   }

   public class Main {
       public static void main(String[] args) {
           int result = MathUtils.square(5);
           System.out.println("5의 제곱: " + result); // 출력: 5의 제곱: 25
       }
   }
   ```

2. run() 메서드 (작업 캡슐화):  
   - 특정 작업을 객체에 위임, 스레드나 작업 단위로 실행  
   ```java  
   public class Processor {
       void run() {
           System.out.println("데이터 처리 중...");
       }
   }

   public class Main {
       public static void main(String[] args) {
           Processor processor = new Processor();
           processor.run();
       }
   }
   ```

3. IntelliJ 디버깅 팁:  
   - `static` 메서드: 클래스 이름으로 호출 추적  
   - `run()` 메서드: 객체 인스턴스와 호출 스택 확인  

---

#### 📌 구조적 이유 비교  

| 항목 | main() 메서드 | run() / static 메서드 |  
| --- | --- | --- |  
| 역할 | 프로그램 진입점 | 특정 기능 수행, 캡슐화 |  
| 호출 | JVM에서 직접 호출 | 다른 클래스/스레드에서 호출 |  
| 재사용성 | 단일 실행 흐름 | 독립적, 재사용 가능 |  
| 모듈화 | 전체 제어 | 특정 작업 분리 |  

---

#### 📌 추가 팁  

- 입력 처리: `main()`에서 `Scanner`로 입력 받아 `run()`이나 `static` 메서드에 전달  
   ```java  
   import java.util.Scanner;

   public class InputProcessor {
       static String processInput(String input) {
           return input.toUpperCase();
       }

       public static void main(String[] args) {
           Scanner scanner = new Scanner(System.in);
           String input = scanner.nextLine();
           System.out.println(processInput(input));
           scanner.close();
       }
   }
   ```  
- Spring Boot 연계: `run()` 또는 `static` 메서드로 서비스 로직 분리  
   ```java  
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RequestParam;
   import org.springframework.web.bind.annotation.RestController;

   @RestController
   public class ServiceController {
       static String formatName(String name) {
           return name.trim().toUpperCase();
       }

       @GetMapping("/format")
       public String format(@RequestParam String name) {
           return formatName(name);
       }
   }
   ```  
   application.yml:  
   ```yml  
   spring:
     mvc:
       throw-exception-if-no-handler-found: true
   ```  
- 테스트 용이성: `main()` 없는 클래스에서 `static` 메서드는 JUnit으로 쉽게 테스트  
   ```java  
   import static org.junit.jupiter.api.Assertions.assertEquals;
   import org.junit.jupiter.api.Test;

   public class MathUtilsTest {
       @Test
       void testSquare() {
           assertEquals(25, MathUtils.square(5));
       }
   }
   ```

---

#### 📌 주의사항  

* static 남용: `static` 메서드 과다 사용 시 객체 지향 설계 저해, 적절히 분리  
* run() 명명: `run()`은 `Runnable` 인터페이스와 연관되므로, 혼동 피하기 위해 명확한 네이밍 고려  
* 입력 검증: `main()`에서 `Scanner` 사용 시 버퍼 정리  
   ```java  
   Scanner scanner = new Scanner(System.in);
   int n = scanner.nextInt();
   scanner.nextLine(); // 버퍼 정리
   ```