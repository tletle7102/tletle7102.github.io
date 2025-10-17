---
title: "diff 명령어 쉽게 이해하기: 옵션별 비교와 출력 방식"
categories:
  - CS Basics
tags:
  - diff
  - Linux
  - Git
  - CS Basics
last_modified_at: 
---

## diff 명령어에 대하여

---

### 📝 diff란?

- diff는 두 개의 파일 내용을 비교하여 다른 부분을 찾아내는 명령어  
- 소스코드, 설정 파일, 텍스트 문서 등에서 차이를 확인할 때 유용  
- Git에서도 내부적으로 diff 형식을 그대로 활용하기 때문에, 익숙해지면 버전 관리에도 큰 도움이 됨  

---

### 📋 diff 기본 출력 형식

명령어  
```sh  
diff file1.txt file2.txt  
```  

출력 예시  
```plaintext  
3c3  
< cherry  
---  
> orange  
4a5  
> melon  
```  

#### 의미 정리
- `3c3` → file1의 3번째 줄이 file2의 3번째 줄과 다름 (change)  
- `4a5` → file1의 4번째 줄 뒤(after), file2의 5번째 줄에 추가됨 (add)  
- `d` → delete (file1에만 있는 줄)  
- `<` → file1의 내용  
- `>` → file2의 내용  

---

### 📋 diff -u (unified format)

명령어  
```sh  
diff -u file1.txt file2.txt  
```  

출력 예시
```plaintext    
@@ -1,4 +1,5 @@  
 apple  
 banana  
-cherry  
+orange  
 grape  
+melon  
```  

#### 의미 정리
- `-` → file1에만 있던 줄 (삭제됨)  
- `+` → file2에만 있던 줄 (추가됨)  
- `@@ -1,4 +1,5 @@` → 비교 구간(좌: file1, 우: file2)  

✅ Git에서 흔히 보는 diff 출력이 바로 이 unified 형식  

---

### 📋 diff -y (side-by-side 비교)

명령어  
```sh  
diff -y file1.txt file2.txt  
```  

출력 예시
```plaintext    
apple       apple  
banana      banana  
cherry    | orange  
grape       grape  
           > melon  
```  

#### 의미 정리
- `|` → 두 줄이 서로 다름  
- `<` → file1에만 있는 줄  
- `>` → file2에만 있는 줄  

---

### ⚙️ 유용한 옵션들 (side-by-side 확장)

사이드바이사이드 출력(-y)은 직관적이지만, 긴 문자열이 잘리거나 불필요하게 중복되는 줄까지 보여줄 수 있음.  
이를 보완하기 위해 다음 옵션을 함께 사용:  

명령어  
```sh  
diff -y --left-column --suppress-common-lines -W 160 file1.txt file2.txt  
```  

#### 옵션 설명
- --left-column → 동일한 줄은 왼쪽 컬럼만 표시  
- --suppress-common-lines → 동일한 줄은 아예 출력하지 않음  
- -W 160 → 왼쪽 컬럼의 너비를 160자로 설정 (긴 문자열 잘림 방지)  

#### 실행 예시
```plaintext  
cherry      | orange  
            > melon  
```  

---

### 🔄 개념 비교

| 옵션/형식         | 특징                                                                 |
|------------------|----------------------------------------------------------------------|
| 기본 diff        | 최소한의 정보 (c, a, d, <, > 등으로 표현)                           |
| diff -u          | Git과 동일한 출력 형식, +와 -로 직관적으로 차이 확인 가능           |
| diff -y          | 좌우 나란히 비교, 가독성이 좋음                                      |
| -y + 옵션들      | - 중복 줄 숨기기 <br> - 왼쪽 중심 보기 <br> - 너비 조정 등 유용하게 활용 가능 |

---

### 💡 요약

- diff는 파일의 차이를 찾는 기본 도구  
- -u 옵션은 Git에서 흔히 보는 방식  
- -y 옵션은 좌우 비교로 직관적  
- --left-column, --suppress-common-lines, -W를 조합하면 가독성 있는 비교 가능  

처음 diff를 배울 때는 -y와 -u 중심으로 익히고, 
필요할 때 옵션을 추가적으로 조합하는 것이 가장 효율적  