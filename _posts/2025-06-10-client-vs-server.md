---
title: "클라이언트와 서버의 차이"
categories:
  - CS Basics
tags:
  - CS Basics
last_modified_at:
---

#### 📌 용어 설명
- 클라이언트(Client): 서비스를 요청하는 주체. 사용자 혹은 사용자의 프로그램
- 서버(Server): 서비스를 제공하는 주체. 요청을 받아서 응답을 반환하는 프로그램 혹은 시스템
- 요청(Request): 클라이언트가 서버에 보내는 데이터나 작업 지시
- 응답(Response): 서버가 클라이언트에게 반환하는 결과 데이터
- 프로세스(Process): 실행 중인 프로그램 단위. 하나의 노트북에서 클라이언트와 서버 프로세스를 동시에 실행 가능

---
#### 📌 클라이언트와 서버의 차이

**클라이언트**와 **서버**는 물리적인 기기 구분이 아니라,  
**역할(role)** 또는 **관계(context)** 로 구분됨.

- 클라이언트는 *요청하는 쪽*  
- 서버는 *응답하는 쪽*

노트북/PC/스마트폰이든 **물리적 장비는 모두 클라이언트 역할도 할 수 있고, 서버 역할도 할 수 있음**.

---
#### 📌 예시로 이해하기

##### 🖥️ 노트북에서 브라우저로 URL 접속할 때
- 사용자가 Chrome 브라우저에 `https://example.com` 입력
- 브라우저(클라이언트)가 서버에 HTTP 요청 보냄
- 서버가 응답(HTML, CSS 등) 반환
- 노트북은 이 경우 **클라이언트 역할**

##### 🖥️ 노트북에서 IntelliJ로 Spring Boot 서버 실행할 때
- 개발자가 IntelliJ에서 `SpringBootApplication` 실행
- 노트북의 특정 포트(`localhost:8080` 등)에서 서버 프로세스가 구동
- 이때 노트북은 **서버 역할**

##### 🖥️ 동일한 노트북에서 브라우저로 `localhost:8080` 접속
- 브라우저는 **클라이언트 역할**
- 브라우저가 노트북 내 실행된 서버 프로세스에 요청
- 동일한 노트북 내에서 **클라이언트와 서버 역할 모두 수행**

---
#### 📌 코드 예시로 살펴보기

##### ✅ 서버 (Spring Boot)

```java
@RestController
public class HelloController {
    
    @GetMapping("/hello")
    public String hello() {
        return "Hello, Client!";
    }
}
```

이 서버는 /hello 요청을 받으면 응답을 반환  
서버 프로세스 실행 시 노트북이 서버 역할  

##### ✅ 클라이언트 (Java HTTP 클라이언트)  

```java
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("http://localhost:8080/hello"))
    .build();

HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
System.out.println(response.body());
```

이 Java 코드가 요청을 보내는 쪽 — 클라이언트 역할  

#### 📌 노트북은 클라이언트일까, 서버일까?  
"노트북은 클라이언트인가 서버인가?" 는 질문 자체가 애매함  
노트북(기기)은 클라이언트도 될 수 있고 서버도 될 수 있음  

- URL 입력 → 클라이언트 역할 수행  
- 서버 프로그램 실행 → 서버 역할 수행  
- 둘을 동시에 수행 가능 (한 프로세스는 서버, 다른 프로세스는 클라이언트)  

결국 '어떤 프로세스가 어떤 역할을 하는가'가 중요  

#### 📌 클라이언트와 서버가 혼동되는 이유
"기기 기준"으로 구분하려고 하기 때문 → 잘못된 접근
"URL 치면 클라이언트", "서버 코드 실행하면 서버"라는 맥락을 헷갈림
역할의 개념(요청 vs 응답) 보다 기기의 역할에 집중