---
title: "Spring Boot에서 REST API 예외 처리 방법"
categories:
  - Spring Boot
tags:
  - Spring Boot
  - REST API
  - Exception Handling
  - ControllerAdvice
last_modified_at: 
---

### Spring Boot에서 REST API 예외 처리 방법

Spring Boot로 REST API를 개발할 때, 예외 처리는 사용자 경험과 API 안정성을 높이는 데 중요한 역할을 함  
예외를 적절히 처리하지 않으면 클라이언트는 이해하기 어려운 에러 메시지를 받거나, 서버 내부 정보가 노출될 수 있음  
`@ControllerAdvice`와 `@ExceptionHandler`를 활용해 중앙 집중식 예외 처리를 구현하면 코드 가독성과 유지보수성이 향상됨  

---

#### 📌 용어 설명  
- 예외(Exception): 프로그램 실행 중 발생하는 비정상 상황 (예: NullPointerException, IllegalArgumentException)  
- @ControllerAdvice: 전역적으로 예외를 처리하거나 컨트롤러 공통 로직을 정의하는 Spring 어노테이션  
- @ExceptionHandler: 특정 예외를 처리하는 메서드를 정의하는 어노테이션  
- ResponseEntity: HTTP 응답 상태 코드와 본문을 커스터마이징하여 반환하는 Spring 클래스  

---

#### 📌 Spring Boot에서 예외 처리의 필요성  

1. 기본 동작: Spring Boot는 예외 발생 시 기본적으로 HTTP 500 상태 코드와 스택 트레이스를 반환  
   - 클라이언트에게 내부 정보 노출 위험  
   - 사용자 친화적이지 않은 응답  

2. 문제점: 예외마다 개별적으로 처리하면 컨트롤러 코드가 복잡해지고 중복 발생  
   - 유지보수 어려움  
   - 일관성 없는 에러 응답  

3. 해결책: `@ControllerAdvice`로 중앙 집중식 예외 처리 구현  
   - 모든 컨트롤러에 공통적으로 적용  
   - 커스텀 에러 응답 구조 정의 가능  

> 💡 REST API 예외 처리의 핵심  
> REST API는 클라이언트가 이해하기 쉬운 에러 메시지와 적절한 HTTP 상태 코드를 반환해야 함  
> 예를 들어, 잘못된 요청은 400(Bad Request), 리소스 없음은 404(Not Found)로 응답  

---

#### 📌 @ControllerAdvice와 @ExceptionHandler 사용 방법  

1. 의존성 확인  
   Spring Boot Starter Web에 이미 포함된 의존성 사용  
   ```groovy  
   dependencies {
       implementation 'org.springframework.boot:spring-boot-starter-web'
   }
   ```

2. 커스텀 예외 클래스 정의  
   특정 상황에 맞는 예외를 정의해 명확한 에러 처리  
   ```java  
   public class ResourceNotFoundException extends RuntimeException {
       public ResourceNotFoundException(String message) {
           super(message);
       }
   }
   ```

3. @ControllerAdvice 구현  
   전역 예외 처리 클래스 작성  
   ```java  
   import org.springframework.http.HttpStatus;
   import org.springframework.http.ResponseEntity;
   import org.springframework.web.bind.annotation.ControllerAdvice;
   import org.springframework.web.bind.annotation.ExceptionHandler;

   @ControllerAdvice
   public class GlobalExceptionHandler {

       @ExceptionHandler(ResourceNotFoundException.class)
       public ResponseEntity<ErrorResponse> handleNotFoundException(ResourceNotFoundException ex) {
           ErrorResponse error = new ErrorResponse(HttpStatus.NOT_FOUND, ex.getMessage());
           return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
       }

       @ExceptionHandler(Exception.class)
       public ResponseEntity<ErrorResponse> handleGeneralException(Exception ex) {
           ErrorResponse error = new ErrorResponse(HttpStatus.INTERNAL_SERVER_ERROR, "서버 오류가 발생했습니다.");
           return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
       }
   }

   class ErrorResponse {
       private HttpStatus status;
       private String message;

       public ErrorResponse(HttpStatus status, String message) {
           this.status = status;
           this.message = message;
       }

       // Getter, Setter
       public HttpStatus getStatus() { return status; }
       public String getMessage() { return message; }
   }
   ```

4. 컨트롤러에서 예외 발생  
   ```java  
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RestController;

   @RestController
   public class UserController {

       @GetMapping("/users/{id}")
       public String getUser(@PathVariable Long id) {
           if (id <= 0) {
               throw new ResourceNotFoundException("유저를 찾을 수 없습니다: ID=" + id);
           }
           return "User ID: " + id;
       }
   }
   ```

5. application.yml 설정  
   기본 에러 처리 비활성화 및 필요 시 설정 추가  
   ```yml  
   spring:
     mvc:
       throw-exception-if-no-handler-found: true
     web:
       resources:
         add-mappings: false
   ```

6. API 테스트  
   ```shell  
   curl http://localhost:8080/users/0
   ```  
   예상 응답:  
   ```json  
   {
       "status": "NOT_FOUND",
       "message": "유저를 찾을 수 없습니다: ID=0"
   }
   ```

---

#### 📌 추가 팁  

| 항목 | 설명 |  
| --- | --- |
| `ResponseEntity` | HTTP 상태 코드와 응답 본문 커스터마이징 가능 |  
| `ValidationException` | `@Valid`와 함께 사용하여 입력 검증 에러 처리 |  
| `ProblemDetail` | Spring Boot 3.0 이상에서 RFC 7807 표준 에러 응답 지원 |  

---

#### 📌 주의사항  

* 프로덕션 환경: 스택 트레이스 노출 방지를 위해 `@ExceptionHandler`에서 민감 정보 필터링  
* 버전 호환성: Spring Boot 3.0 이상에서는 `ProblemDetail`을 활용해 표준화된 에러 응답 권장  
* 로그 기록: 예외 발생 시 로깅 추가로 디버깅 용이