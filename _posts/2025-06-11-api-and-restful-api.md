---
title: "API와 RESTful API에 대하여"
categories:
  - CS Basics
tags:
  - CS Basics
last_modified_at:
---

#### 📌 용어 설명
- API: Application Programming Interface. 프로그램끼리 상호작용할 수 있게 해주는 인터페이스
- REST: Representational State Transfer. 웹 기반 시스템 설계의 아키텍처 스타일
- RESTful API: REST 원칙을 따르는 API 설계 방식
- HTTP 메서드: 클라이언트와 서버 간의 요청 방법. GET, POST, PUT, DELETE 등
- 엔드포인트: API에서 특정 자원에 접근하기 위한 URL 경로

---
#### 📌 API란
프로그램끼리 데이터를 주고받고 기능을 사용할 수 있게 해주는 통로
직접 내부 로직을 몰라도 외부에서 정해진 규칙만 따르면 기능 사용 가능

예: 지도 API 사용 시 지도 구현 로직을 직접 만들 필요 없이 제공된 API 호출로 지도 기능 사용 가능

---
#### 📌 RESTful API란
REST 아키텍처 스타일을 따르는 API
자원을 URI로 표현하고 HTTP 메서드를 활용해 자원에 대한 행동 정의

REST 원칙을 따를 때 얻는 장점
- 구조적이고 예측 가능한 URL 설계
- HTTP 프로토콜의 기본 기능 활용
- 클라이언트-서버 구조 확립

---
#### 📌 RESTful API 예시
블로그 서비스에서 게시글 자원을 다룬다고 가정

```  
GET /posts → 게시글 목록 조회
GET /posts/1 → 특정 게시글 조회
POST /posts → 새 게시글 생성
PUT /posts/1 → 특정 게시글 수정
DELETE /posts/1 → 특정 게시글 삭제
```  

HTTP 메서드와 URI 설계를 일관되게 사용

---
#### 📌 RESTful API의 특징

##### ✅ Stateless
각 요청은 독립적
서버는 이전 요청 상태를 저장하지 않음

##### ✅ Uniform Interface
일정한 인터페이스 규약
URI는 자원을 표현하고, HTTP 메서드는 행동을 표현

##### ✅ Cacheable
응답은 캐싱 가능
네트워크 효율성 증가

##### ✅ Layered System
중간 서버(프록시, 게이트웨이)를 통한 확장 가능

---
#### 📌 RESTful하지 않은 API 예시

```  
/getPosts
/createPost
```  

- URI에 동사가 들어가는 경우
- HTTP 메서드가 일관되지 않은 경우 → 모든 작업을 POST로 처리

RESTful API는 자원 중심으로 설계하는 것이 핵심

---
#### 📌 API 설계 시 고려할 점
- 일관성
- 명확한 자원 정의
- 적절한 HTTP 메서드 사용
- 상태 코드를 통한 명확한 응답

---
#### ✍ 알아보면서
처음엔 API와 RESTful API 개념이 헷갈렸음
단순한 함수 호출 정도로 생각했지만
REST 원칙을 배우면서 API 설계에 철학이 담겨 있다는 것을 알게 됨

RESTful API를 잘 설계하면 서비스 유지보수와 확장성이 높아짐
좋은 API는 코드보다 더 많은 것을 말해줌
단순히 기능 구현이 아니라, 어떻게 보여주고 어떻게 사용할지를 고민하는 것이 중요

---