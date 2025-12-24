---
title: "재귀(Recursion)의 기본 구조와 ‘기저 조건(base case)’의 의미"  
categories:  
  - CS Basics  
tags:  
  - Java  
  - Recursion  
  - Algorithm  
  - Base Case  
  - CS Basics  
last_modified_at:   
---

### 재귀(Recursion)의 기본 구조와 ‘기저 조건(base case)’의 의미

재귀(recursion)는 함수가 자기 자신을 호출하여 문제를 해결하는 프로그래밍 기법으로, 복잡한 문제를 간단한 하위 문제로 나누어 처리함  
재귀의 핵심은 기저 조건(base case)과 재귀 사례(recursive case)로 구성된 구조로, 기저 조건은 재귀 호출을 종료하는 역할을 함  

---

#### 📌 용어 설명  
- 재귀(recursion): 함수가 자기 자신을 호출하여 문제를 해결하는 방식  
- 기저 조건(base case): 재귀 호출을 멈추는 조건, 결과가 명확한 최소 단위 문제  
- 재귀 사례(recursive case): 문제를 더 작은 하위 문제로 나누어 다시 호출하는 부분  
- 호출 스택(call stack): 재귀 호출 시 함수 호출 정보가 쌓이는 메모리 공간  

---

#### 📌 재귀의 기본 구조  

1. 구성 요소:  
   - 재귀는 기저 조건과 재귀 사례로 구성됨  
   - 기저 조건: 더 이상 분해할 수 없는 상태에서 직접 결과 반환  
   - 재귀 사례: 문제를 작은 단위로 나누어 자기 자신 호출  
   ```java  
   public class RecursionStructureExample {
       public static void main(String[] args) {
           int n = 4;
           int result = sumUpToN(n);
           System.out.println("1부터 " + n + "까지 합: " + result); // 출력: 10
       }

       static int sumUpToN(int n) {
           // 기저 조건
           if (n <= 1) return n;
           // 재귀 사례
           return n + sumUpToN(n - 1);
       }
   }
   ```  
   - 연산 과정:  
     - `sumUpToN(4)` → `4 + sumUpToN(3)`  
     - `sumUpToN(3)` → `3 + sumUpToN(2)`  
     - `sumUpToN(2)` → `2 + sumUpToN(1)`  
     - `sumUpToN(1)` → `1` (기저 조건) → `2 + 1 = 3` → `3 + 3 = 6` → `4 + 6 = 10`  

2. 호출 스택 동작:  
   - 각 재귀 호출은 호출 스택에 쌓이고, 기저 조건에 도달하면 스택을 풀며 결과 조립  
   - 스택 오버헤드로 인해 깊은 재귀는 `StackOverflowError` 발생 가능  

3. 문제 상황:  
   - 기저 조건이 없거나 잘못 정의되면 무한 재귀 발생  
   ```java  
   static int badRecursion(int n) {
       return n + badRecursion(n); // 기저 조건 없음 → StackOverflowError
   }
   ```

> 💡 재귀의 직관적 이해  
> 재귀는 큰 문제를 작은 문제로 나누어 해결하고, 기저 조건에서 시작해 다시 조립하는 과정  
> 비유: 책 더미를 정리하려면 한 권씩(재귀 사례) 쌓다가 더미가 없으면(기저 조건) 멈춤  

---

#### 📌 기저 조건(base case)의 의미  

1. 역할:  
   - 기저 조건은 재귀 호출을 종료하고 결과를 반환하는 기준점  
   - 무한 재귀를 방지하고 호출 스택이 과도하게 쌓이는 문제 해결  
   - 예: 팩토리얼에서 `n == 0` 또는 `n == 1`일 때 `1` 반환  
   ```java  
   static int factorial(int n) {
       if (n == 0 || n == 1) return 1; // 기저 조건
       return n * factorial(n - 1);    // 재귀 사례
   }
   ```

2. 설계 원칙:  
   - 기저 조건은 문제가 더 이상 분해 불가능한 최소 단위에서 정의  
   - 명확하고 간단해야 하며, 모든 입력에 대해 도달 가능해야 함  
   - 예: 피보나치 수열에서 `n <= 1`일 때 `n` 반환  
   ```java  
   static int fibonacci(int n) {
       if (n <= 1) return n; // 기저 조건
       return fibonacci(n - 1) + fibonacci(n - 2); // 재귀 사례
   }
   ```

3. 기저 조건 부재의 위험:  
   - 기저 조건이 없으면 무한 호출로 `StackOverflowError` 발생  
   - 입력 크기가 클수록 스택 오버플로우 위험 증가  

---

#### 📌 기저 조건과 재귀 사례 예시  

1. 합계 계산 예시:  
   ```java  
   public class SumRecursionExample {
       public static void main(String[] args) {
           int n = 5;
           int result = sumUpToN(n);
           System.out.println("1부터 " + n + "까지 합: " + result); // 출력: 15
       }

       static int sumUpToN(int n) {
           if (n <= 1) return n; // 기저 조건
           return n + sumUpToN(n - 1); // 재귀 사례
       }
   }
   ```

2. IntelliJ 디버깅 팁:  
   - 디버거로 호출 스택을 추적, 기저 조건 도달 시 스택 풀림 확인  
   - `sumUpToN(5)` 호출 시 각 `n` 값과 스택 프레임 점검  

---

#### 📌 재귀와 반복문 비교  

| 항목 | 재귀 | 반복문 |  
| --- | --- | --- |  
| 코드 | 간결, 직관적 | 명시적 루프, 길어질 수 있음 |  
| 메모리 | 호출 스택 사용, O(n) | O(1), 메모리 효율적 |  
| 기저 조건 | 필수, 종료 조건 정의 | 루프 조건으로 대체 |  

```java  
// 반복문으로 합계 계산
static int sumUpToNIterative(int n) {
    int sum = 0;
    for (int i = 1; i <= n; i++) {
        sum += i;
    }
    return sum;
}
```

---

#### 📌 추가 팁  

- 기저 조건 설계: 모든 입력에 대해 기저 조건 도달 보장  
   ```java  
   static int factorial(int n) {
       if (n < 0) throw new IllegalArgumentException("음수는 허용되지 않습니다.");
       if (n == 0 || n == 1) return 1;
       return n * factorial(n - 1);
   }
   ```  
- Spring Boot 연계: 재귀를 API 응답 계산에 활용  
   ```java  
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RequestParam;
   import org.springframework.web.bind.annotation.RestController;

   @RestController
   public class RecursionController {
       @GetMapping("/sum")
       public int calculateSum(@RequestParam int n) {
           if (n < 0) throw new IllegalArgumentException("음수는 허용되지 않습니다.");
           return sumUpToN(n);
       }

       private int sumUpToN(int n) {
           if (n <= 1) return n;
           return n + sumUpToN(n - 1);
       }
   }
   ```  
   application.yml:  
   ```yml  
   spring:
     mvc:
       throw-exception-if-no-handler-found: true
   ```  
- 성능 최적화: 중복 계산이 많을 경우 메모이제이션 고려  
   ```java  
   static long fibonacciMemo(int n, long[] memo) {
       if (n <= 1) return n;
       if (memo[n] != 0) return memo[n];
       return memo[n] = fibonacciMemo(n - 1, memo) + fibonacciMemo(n - 2, memo);
   }
   ```

---

#### 📌 주의사항  

* 기저 조건 누락: 무한 재귀로 `StackOverflowError` 발생  
* 입력 검증: 음수, 큰 입력 처리 시 예외 처리 추가  
* 호출 깊이: 깊은 재귀는 스택 메모리 소모, 반복문 대체 고려