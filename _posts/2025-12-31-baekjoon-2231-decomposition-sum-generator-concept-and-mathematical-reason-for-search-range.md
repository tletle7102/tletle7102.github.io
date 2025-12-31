---
title: "백준 2231번 분해합의 ‘생성자’ 개념과 N - (자릿수 × 9) 탐색 범위의 수학적인 이유"  
categories:  
  - Algorithm  
tags:  
  - Baekjoon  
  - Algorithm  
  - Decomposition Sum  
  - Brute Force  
  - Math  
last_modified_at:   
---

### 백준 2231번 분해합의 ‘생성자’ 개념과 N - (자릿수 × 9) 탐색 범위의 수학적인 이유

백준 2231번(분해합)은 주어진 수 N의 생성자를 찾는 문제로, 수학적 이해와 효율적인 탐색 범위 설정이 핵심임  
‘생성자’와 분해합의 개념을 명확히 이해하고, 탐색 범위를 `N - (자릿수 × 9)`로 제한하는 수학적 이유를 알면 문제를 효율적으로 해결할 수 있음  

---

#### 📌 용어 설명  
- 분해합: 자연수 N과 N의 각 자릿수의 합을 더한 값 (예: 245의 분해합은 245 + 2 + 4 + 5 = 256)  
- 생성자: 분해합이 N이 되는 자연수 M (예: 245는 256의 생성자)  
- N - (자릿수 × 9): 생성자를 찾기 위한 탐색 범위의 하한, 수학적으로 계산된 최소값  
- 완전 탐색(Brute Force): 가능한 모든 경우를 확인하는 알고리즘 기법  

---

#### 📌 분해합과 생성자 개념  

1. 분해합의 정의:  
   - 자연수 M의 분해합은 M + (M의 각 자릿수의 합)  
   - 예: M = 245의 분해합 = 245 + 2 + 4 + 5 = 256  
   - N = 256의 생성자는 245  

2. 문제 목표:  
   - 주어진 N에 대해 분해합이 N이 되는 가장 작은 M(생성자)을 찾음  
   - 생성자가 없으면 0 출력  
   ```java  
   public class Baekjoon2231 {
       public static void main(String[] args) {
           java.util.Scanner scanner = new java.util.Scanner(System.in);
           int N = scanner.nextInt();
           int result = findGenerator(N);
           System.out.println(result);
       }

       static int findGenerator(int N) {
           for (int i = 1; i <= N; i++) {
               if (getDecompositionSum(i) == N) {
                   return i;
               }
           }
           return 0;
       }

       static int getDecompositionSum(int num) {
           int sum = num;
           while (num > 0) {
               sum += num % 10;
               num /= 10;
           }
           return sum;
       }
   }
   ```  
   - 입력 N = 256 → 출력: 245 (최소 생성자)  

3. 문제점:  
   - 모든 1부터 N까지 탐색은 비효율적 (최대 N = 1,000,000)  
   - 탐색 범위를 줄여야 시간 초과 방지  

> 💡 생성자 탐색의 핵심  
> 생성자 M은 N보다 작거나 같고, M의 분해합은 M + (M의 자릿수 합) = N이어야 함  
> 자릿수 합은 최대값이 제한되므로, M의 최소값을 수학적으로 계산 가능  

---

#### 📌 N - (자릿수 × 9) 탐색 범위의 수학적 이유  

1. 탐색 범위 최적화:  
   - M이 N의 생성자라면, M + (M의 자릿수 합) = N  
   - M의 자릿수 합은 각 자리가 최대 9일 때 가장 큼  
   - 예: 3자리 수 M(최대 999)의 자릿수 합은 최대 9 × 3 = 27  
   - 따라서, M ≥ N - (자릿수 × 9)  
   - N보다 작은 M 중, N - (자릿수 × 9) 미만은 분해합이 N에 도달할 수 없음  

2. 수학적 계산:  
   - N의 자릿수를 d라고 하면, 최대 자릿수 합 = 9 × d  
   - M = N - (자릿수 합) ≥ N - (9 × d)  
   - 예: N = 256 (3자리), 최대 자릿수 합 = 9 × 3 = 27  
     - 최소 M = 256 - 27 = 229  
     - 따라서 229부터 N까지 탐색하면 충분  

3. 코드 구현:  
   ```java  
   import java.util.Scanner;

   public class Baekjoon2231Optimized {
       public static void main(String[] args) {
           Scanner scanner = new Scanner(System.in);
           int N = scanner.nextInt();
           int result = findGenerator(N);
           System.out.println(result);
       }

       static int findGenerator(int N) {
           int digits = (int) Math.log10(N) + 1; // 자릿수 계산
           int start = Math.max(1, N - (digits * 9)); // 최소 탐색 범위
           for (int i = start; i <= N; i++) {
               if (getDecompositionSum(i) == N) {
                   return i;
               }
           }
           return 0;
       }

       static int getDecompositionSum(int num) {
           int sum = num;
           while (num > 0) {
               sum += num % 10;
               num /= 10;
           }
           return sum;
       }
   }
   ```  
   - 입력 N = 256 → 탐색 범위: 229~256 → 출력: 245  

4. IntelliJ 디버깅 팁:  
   - `getDecompositionSum` 호출 시 자릿수 합 계산 과정 점검  
   - `start` 값 계산 확인: 디버거로 `N - (digits * 9)` 값 검증  

---

#### 📌 탐색 범위 비교  

| 방법 | 설명 |  
| --- | --- |  
| 1부터 N까지 탐색 | 모든 경우 확인, 비효율적, 시간 초과 가능 |  
| N - (자릿수 × 9)부터 탐색 | 수학적 최적화로 탐색 범위 축소, 시간 효율적 |  

---

#### 📌 추가 팁  

- 자릿수 계산: `Math.log10(N) + 1`로 간단히 자릿수 구함  
   ```java  
   int digits = (int) Math.log10(N) + 1;
   ```  
- 입력 처리: `Scanner` 사용 시 `nextInt()` 후 버퍼 정리 필요  
   ```java  
   Scanner scanner = new Scanner(System.in);
   int N = scanner.nextInt();
   scanner.nextLine(); // 버퍼 정리
   ```  
- 큰 입력 대비: N ≤ 1,000,000이므로 `int`로 충분, 더 큰 경우 `long` 고려  

---

#### 📌 주의사항  

* 생성자 없음: N에 생성자가 없는 경우 0 반환, 예외 처리 불필요  
* 입력 범위: 백준 2231번은 1 ≤ N ≤ 1,000,000 보장, 입력 검증 불필요  
* 효율성: 탐색 범위 최적화 없으면 시간 초과 위험, 특히 큰 N에서