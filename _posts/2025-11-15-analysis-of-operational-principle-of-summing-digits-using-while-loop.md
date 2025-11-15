---
title: "while문으로 자릿수를 나누어 더하는 과정의 동작 원리 분석"  
categories:  
  - Java  
tags:  
  - Java  
  - Algorithm  
  - While Loop  
  - Digit Sum  
last_modified_at:   
---

### while문으로 자릿수를 나누어 더하는 과정의 동작 원리 분석

Java에서 숫자의 각 자릿수를 나누어 더하는 작업은 알고리즘 문제(예: 백준 2231번 분해합)에서 자주 등장하며, `while` 문을 사용해 간단히 구현 가능  
이 과정은 숫자를 10으로 나누고 나머지를 활용해 자릿수를 추출하는 방식으로 동작  
`while` 문의 동작 원리를 이해하면 자릿수 합 계산의 효율성과 정확성을 높일 수 있음  

---

#### 📌 용어 설명  
- 자릿수 합: 숫자의 각 자릿수를 더한 결과 (예: 123의 자릿수 합 = 1 + 2 + 3 = 6)  
- 나머지 연산(%): 숫자를 10으로 나눈 나머지로 마지막 자릿수 추출 (예: 123 % 10 = 3)  
- 나누기 연산(/): 숫자를 10으로 나누어 자릿수를 한 자리씩 제거 (예: 123 / 10 = 12)  
- while 문: 조건이 참인 동안 반복 실행, 자릿수 추출에 적합  

---

#### 📌 자릿수 합 계산의 동작 원리  

1. 기본 동작:  
   - `while` 문을 사용해 숫자를 10으로 나누며 각 자릿수를 추출하고 합산  
   - `num % 10`으로 현재 숫자의 마지막 자릿수를 구하고, `num / 10`으로 다음 자릿수로 이동  
   ```java  
   public class DigitSumExample {
       public static void main(String[] args) {
           java.util.Scanner scanner = new java.util.Scanner(System.in);
           int num = scanner.nextInt(); // 입력: 123
           int sum = getDigitSum(num);
           System.out.println("자릿수 합: " + sum); // 출력: 6
       }

       static int getDigitSum(int num) {
           int sum = 0;
           while (num > 0) {
               sum += num % 10; // 마지막 자릿수 추가
               num /= 10;       // 다음 자릿수로 이동
           }
           return sum;
       }
   }
   ```

2. 연산 과정:  
   - 입력: `num = 123`  
   - 1단계: `sum = 0`, `num % 10 = 3` → `sum = 3`, `num / 10 = 12`  
   - 2단계: `sum = 3`, `num % 10 = 2` → `sum = 5`, `num / 10 = 1`  
   - 3단계: `sum = 5`, `num % 10 = 1` → `sum = 6`, `num / 10 = 0`  
   - 종료: `num = 0`, 루프 종료 → 결과: 6  

3. 문제 상황:  
   - 음수 입력 시 예상치 못한 결과 발생 가능  
   - 0 입력 시 자릿수 합이 0으로 계산, 별도 처리 필요 없음  
   ```java  
   static int getDigitSum(int num) {
       if (num < 0) num = Math.abs(num); // 음수 처리
       int sum = 0;
       while (num > 0) {
           sum += num % 10;
           num /= 10;
       }
       return sum;
   }
   ```

> 💡 while 문의 동작 원리  
> `while` 문은 `num > 0` 조건을 만족하는 동안 반복하며, `num`을 10으로 나눠 자릿수를 하나씩 제거  
> `num % 10`은 현재 자릿수를 추출, `num /= 10`은 다음 자릿수로 이동해 루프를 효율적으로 처리  

---

#### 📌 동작 과정 분석  

1. 단계별 동작:  
   - 초기: `sum = 0`, `num = 입력값`  
   - 반복:  
     - `num % 10`으로 마지막 자릿수 추출해 `sum`에 추가  
     - `num /= 10`으로 `num`을 갱신, 자릿수 하나 제거  
   - 종료: `num = 0`이 되면 루프 종료, `sum` 반환  

2. 시간 복잡도:  
   - 자릿수 d에 비례, O(log N) (N은 입력 숫자)  
   - 예: N = 123 (3자리) → 3번 반복  

3. 코드 예시 (백준 2231번 연계):  
   ```java  
   import java.util.Scanner;

   public class Baekjoon2231DigitSum {
       public static void main(String[] args) {
           Scanner scanner = new Scanner(System.in);
           int N = scanner.nextInt();
           int result = findGenerator(N);
           System.out.println(result);
       }

       static int findGenerator(int N) {
           int digits = (int) Math.log10(N) + 1;
           int start = Math.max(1, N - (digits * 9));
           for (int i = start; i <= N; i++) {
               if (i + getDigitSum(i) == N) {
                   return i;
               }
           }
           return 0;
       }

       static int getDigitSum(int num) {
           int sum = 0;
           while (num > 0) {
               sum += num % 10;
               num /= 10;
           }
           return sum;
       }
   }
   ```  
   - 입력: 256 → 자릿수 합 계산 후 생성자 245 출력  

4. IntelliJ 디버깅 팁:  
   - `while` 루프 내 `num`과 `sum` 값의 변화를 디버거로 추적  
   - `num % 10`과 `num / 10` 연산 결과 확인  

---

#### 📌 동작 비교  

| 방법 | 설명 |  
| --- | --- |  
| while 문 | 간단하고 직관적, O(log N)으로 효율적 |  
| 문자열 변환 | `String.valueOf(num).chars().sum()` 사용 가능, 메모리 사용량 높음 |  

```java  
static int getDigitSumString(int num) {
    return String.valueOf(num).chars().map(ch -> ch - '0').sum();
}
```

---

#### 📌 추가 팁  

- 음수 처리: `Math.abs()`로 음수 입력 처리  
   ```java  
   if (num < 0) num = Math.abs(num);
   ```  
- 0 처리: `num = 0`은 자릿수 합 0, 별도 처리 불필요  
- Spring Boot 연계: 자릿수 합 계산을 API로 제공  
   ```java  
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RequestParam;
   import org.springframework.web.bind.annotation.RestController;

   @RestController
   public class DigitSumController {
       @GetMapping("/digit-sum")
       public int getDigitSum(@RequestParam int number) {
           int num = Math.abs(number);
           int sum = 0;
           while (num > 0) {
               sum += num % 10;
               num /= 10;
           }
           return sum;
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

* 입력 검증: `Scanner` 사용 시 `nextInt()` 후 버퍼 정리  
   ```java  
   Scanner scanner = new Scanner(System.in);
   int num = scanner.nextInt();
   scanner.nextLine(); // 버퍼 정리
   ```  
* 큰 숫자: `int` 범위 초과 시 `long` 사용  
   ```java  
   static long getDigitSum(long num) {
       long sum = 0;
       while (num > 0) {
           sum += num % 10;
           num /= 10;
       }
       return sum;
   }
   ```  
* 효율성: `while` 문은 문자열 변환 방식보다 메모리 효율적