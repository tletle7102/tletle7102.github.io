---
title: "Scanner로 입력받을 때 줄바꿈 문자(\n)가 남아있는 이유와 이를 처리하는 방법 정리"  
categories:  
  - Java  
tags:  
  - Java  
  - Scanner  
  - Input Buffer  
  - Line Break  
last_modified_at:   
---

### Scanner로 입력받을 때 줄바꿈 문자(\n)가 남아있는 이유와 이를 처리하는 방법 정리

Java의 `Scanner` 클래스는 콘솔 입력을 처리하는 데 널리 사용되지만, `nextInt()`, `nextDouble()` 같은 메서드와 `nextLine()`을 혼합 사용 시 줄바꿈 문자(`\n`)가 입력 버퍼에 남아 문제를 일으킴  
줄바꿈 문자가 남는 이유와 이를 처리하는 방법을 이해하면 입력 처리 오류를 방지할 수 있음  

---

#### 📌 용어 설명  
- Scanner: Java에서 콘솔, 파일 등의 입력을 읽기 위한 클래스  
- 입력 버퍼: 사용자 입력이 임시로 저장되는 메모리 공간  
- 줄바꿈 문자(\n): Enter 키 입력 시 생성되는 문자, 입력 버퍼에 남음  
- nextInt(): 입력 버퍼에서 다음 정수를 읽고, 줄바꿈 문자는 남김  
- nextLine(): 입력 버퍼에서 한 줄 전체(줄바꿈 문자 포함)를 읽음  

---

#### 📌 줄바꿈 문자가 남아있는 이유  

1. 기본 동작:  
   - `nextInt()`, `nextDouble()` 등은 입력된 특정 데이터(숫자 등)만 읽고, 사용자가 Enter를 눌렀을 때 입력된 줄바꿈 문자(`\n`)는 버퍼에 남김  
   - `nextLine()`은 버퍼에 남아 있는 줄바꿈 문자를 포함해 한 줄을 읽음, 따라서 빈 문자열이나 예기치 않은 입력을 반환할 수 있음  
   ```java  
   import java.util.Scanner;

   public class ScannerIssueExample {
       public static void main(String[] args) {
           Scanner scanner = new Scanner(System.in);
           System.out.print("나이를 입력하세요: ");
           int age = scanner.nextInt(); // 입력: 25[Enter]
           System.out.print("이름을 입력하세요: ");
           String name = scanner.nextLine(); // \n 읽음
           System.out.println("나이: " + age + ", 이름: [" + name + "]");
       }
   }
   ```  
   출력:  
   ```
   나이를 입력하세요: 25
   이름을 입력하세요: 
   나이: 25, 이름: []
   ```  
   - `nextLine()`이 남아 있던 `\n`을 읽어 빈 문자열 반환  

2. 원인 분석:  
   - `nextInt()`는 숫자(예: `25`)만 소비하고, 입력 버퍼에 `\n`이 남음  
   - `nextLine()`은 버퍼의 `\n`을 즉시 읽어 처리, 사용자가 의도한 문자열 입력을 건너뜀  

> 💡 입력 버퍼의 동작 원리  
> `Scanner`는 입력을 버퍼에 저장하고, `nextInt()` 등은 특정 토큰만 읽음  
> 줄바꿈 문자는 토큰 구분자로 처리되지만 소비되지 않아 버퍼에 잔류  
> `nextLine()`은 잔여 `\n`을 포함한 문자열을 읽어 문제 발생  

---

#### 📌 줄바꿈 문자 처리 방법  

1. `nextLine()`으로 버퍼 정리:  
   - `nextInt()` 등 호출 후 `scanner.nextLine()`을 호출해 잔여 `\n` 제거  
   ```java  
   import java.util.Scanner;

   public class ScannerFixedExample {
       public static void main(String[] args) {
           Scanner scanner = new Scanner(System.in);
           System.out.print("나이를 입력하세요: ");
           int age = scanner.nextInt();
           scanner.nextLine(); // 줄바꿈 문자 제거
           System.out.print("이름을 입력하세요: ");
           String name = scanner.nextLine();
           System.out.println("나이: " + age + ", 이름: " + name);
       }
   }
   ```  
   출력:  
   ```
   나이를 입력하세요: 25
   이름을 입력하세요: 홍길동
   나이: 25, 이름: 홍길동
   ```

2. `next()` 사용:  
   - 문자열 입력 시 `next()`를 사용하면 공백 전까지의 토큰만 읽어 `\n` 문제 회피  
   ```java  
   System.out.print("이름을 입력하세요: ");
   String name = scanner.next(); // 공백 전까지 읽음
   scanner.nextLine(); // 잔여 \n 제거
   ```

3. 입력 검증 추가:  
   - `hasNextInt()` 등으로 입력 유효성 확인 후 처리  
   ```java  
   Scanner scanner = new Scanner(System.in);
   System.out.print("나이를 입력하세요: ");
   if (scanner.hasNextInt()) {
       int age = scanner.nextInt();
       scanner.nextLine(); // 버퍼 정리
   } else {
       System.out.println("잘못된 입력입니다.");
       scanner.nextLine(); // 잘못된 입력 제거
   }
   ```

4. IntelliJ 디버깅 팁:  
   - `scanner.nextLine()` 호출 전후로 입력 버퍼 상태 확인  
   - 디버거에서 `scanner` 객체의 내부 상태(`buffer`) 점검  

---

#### 📌 처리 방법 비교  

| 방법 | 설명 |  
| --- | --- |  
| `nextLine()`으로 정리 | 가장 일반적, 모든 잔여 문자 제거 |  
| `next()` 사용 | 공백 전까지 입력 처리, 간단한 문자열 입력에 적합 |  
| 입력 검증 | 잘못된 입력 처리, 안정성 향상 |  

---

#### 📌 추가 팁  

- try-with-resources 사용: `Scanner` 리소스 관리로 메모리 누수 방지  
   ```java  
   try (Scanner scanner = new Scanner(System.in)) {
       int age = scanner.nextInt();
       scanner.nextLine();
       String name = scanner.nextLine();
   }
   ```  
- Spring Boot 연계: API에서 입력 처리 시 `Scanner` 대신 `@RequestBody` 활용  
   ```java  
   import org.springframework.web.bind.annotation.PostMapping;
   import org.springframework.web.bind.annotation.RequestBody;
   import org.springframework.web.bind.annotation.RestController;

   @RestController
   public class InputController {
       @PostMapping("/process-input")
       public String processInput(@RequestBody String input) {
           String trimmed = input.trim(); // 공백 및 \n 제거
           return "처리된 입력: " + trimmed;
       }
   }
   ```  
   application.yml:  
   ```yml  
   spring:
     mvc:
       throw-exception-if-no-handler-found: true
   ```  
- 다중 입력 처리: 반복 입력 시 매번 버퍼 정리 필요  
   ```java  
   while (scanner.hasNextInt()) {
       int num = scanner.nextInt();
       scanner.nextLine();
       String text = scanner.nextLine();
   }
   ```

---

#### 📌 주의사항  

* 입력 순서: `nextLine()`을 먼저 호출하면 `\n` 문제 발생 가능성 낮음  
* 다양한 입력 형식: 숫자 외의 입력(예: 공백, 문자) 시 `hasNext()`로 검증  
* 성능 고려: 빈번한 `nextLine()` 호출은 버퍼 읽기 오버헤드 발생, 필요한 경우만 사용