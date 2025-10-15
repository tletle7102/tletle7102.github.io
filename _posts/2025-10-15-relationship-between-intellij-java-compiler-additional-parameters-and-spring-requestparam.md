---
title: "IntelliJ Java Compiler Additional Parameters와 Spring @RequestParam 관계"
categories:
  - Spring Boot
tags:
  - Spring Boot
  - IntelliJ
  - Parameter
  - RequestParam
  - Java Compiler
last_modified_at:
---

### IntelliJ에서 Java Compiler Additional Command Line Parameters 설정 미적용 시 발생 사항  

IntelliJ IDEA의 Settings에 들어가면 여러 메뉴가 보임  
그 중 Build, Execution, Deployment > Compiler > Java Compiler 섹션에서 "Additional command line parameters"를 설정하지 않으면, 기본 컴파일 설정만 적용되어 일반 프로젝트에서는 큰 문제가 없으나 특정 기능 제한이 발생할 수 있음  

---

#### 📌 용어 설명  
- Additional Command Line Parameters: Javac 컴파일러에 추가 옵션을 전달하는 필드 (예: -parameters)  
- @RequestParam: Spring 컨트롤러 메서드에서 쿼리 파라미터를 바인딩하는 어노테이션  
- 리플렉션(Reflection): 런타임에 클래스 정보를 검사하고 사용하는 Java 기능  
#### 📌 Additional Parameters 미설정 시 발생 사항  
1. 기본 컴파일 동작: 대상 JVM 버전, 바이트코드 버전에 따라 컴파일 진행  
대부분의 프로젝트에서 충분  
2. 기능 제한: 메서드 파라미터 이름이 런타임에 "arg0"처럼 익명으로 저장되어 디버깅이나 리플렉션 활용이 불편  
3. 경고 누락: -Xlint:all 같은 옵션 없으면 unchecked/deprecation 경고 미표시, 잠재적 버그 놓칠 수 있음  
4. 호환성 문제: 대형 프로젝트에서 메모리 부족 가능하나 IntelliJ 기본 설정으로 보통 안정적  
5. Spring Boot 영향: AOT 컴파일이나 GraalVM 사용 시 추가 옵션 필요 (예: --enable-preview)  

> 💡 AOT vs JIT  
> AOT란 "Ahead-of-Time"의 약자로, "미리 컴파일"을 의미  
> 쉽게 말하면, 프로그램을 실행하기 전에 미리 모든 코드를 컴파일해서 준비해두는 방식  
> 
> 보통 자바 프로그램은 JIT(Just-In-Time) 컴파일을 사용  
> JIT는 프로그램이 실행되는 도중에 필요한 부분만 컴파일해서 속도를 높이지만, 시작할 때 시간이 좀 더 소요  
> 반대로 AOT는 프로그램을 시작하기 전에 모든 코드를 네이티브 코드(기계어)로 미리 컴파일하므로 프로그램이 빠르게 시작되고, 메모리 적게 사용  

---

#### 📌 @RequestParam 이름 명시와의 관련성

Java 컴파일러에 `-parameters` 옵션을 추가하지 않으면, 컴파일된 클래스 파일에서 메서드 파라미터 이름(예: `String email`)이 런타임에 "arg0"처럼 익명으로 저장됨  
이 때문에 Spring은 @RequestParam에서 name을 생략하면 실제 파라미터 이름을 인식하지 못해, 쿼리 파라미터 바인딩이 실패할 수 있음  

반대로 `-parameters` 옵션을 추가하면 파라미터 이름이 런타임에도 유지되어, @RequestParam(name = "email")처럼 name을 명시하지 않아도 자동으로 파라미터 이름(email)으로 바인딩됨  
이는 Spring의 리플렉션 기반 Argument Resolver가 동작하는 방식 때문  

---

#### 📌 설정 방법  
1. IntelliJ에서 Settings (Ctrl+Alt+S) 열기  
2. Build, Execution, Deployment > Compiler > Java Compiler 이동  
3. Additional command line parameters에 `-parameters` 입력  
4. 적용 후 프로젝트 재빌드  

#### 📌 예시 코드 (Spring Controller)  
```java  
// -parameters 옵션 미적용 시 name 명시 필요  
@GetMapping("/user")  
public String getUser(@RequestParam(name = "email") String email) {  
    return "User email: " + email;  
}  

// -parameters 옵션 적용 시 name 생략 가능  
@GetMapping("/user")  
public String getUser(@RequestParam String email) {  
    return "User email: " + email;  
}  
```  

---

#### 📌 추가 팁  
| **항목** | **설명** |  
| --- | --- |
| `-parameters` | 파라미터 이름 런타임 유지, Spring 리플렉션에 유용 |
| `-Xlint:all` | 모든 경고 활성화, 코드 품질 향상 |