---
title: "AWS API Gateway 정리 (요청을 받아 뒤로 넘겨주는 입구)"
categories:
  - AWS
tags:
  - AWS
  - API Gateway
last_modified_at:
---

# AWS API Gateway 정리

Lambda를 배우니 "그럼 이 함수를 주소로 어떻게 부르지?" 하는 의문이 생겼음. 그 입구 역할을 하는 게 API Gateway였음


## 1. 문제: Lambda 함수엔 주소가 없다

Lambda 함수는 그냥 코드 덩어리라, 밖에서 `https://...`로 부를 주소가 없음. 누군가 "요청을 받아서 이 함수한테 넘겨주는 입구"가 필요함. 그게 API Gateway임.


## 2. API Gateway = API의 정문

API Gateway는 **밖에서 오는 요청을 받아, 알맞은 뒤쪽(Lambda 등)으로 넘겨주는 입구**임. 건물 정문에서 손님을 받아 해당 부서로 안내하는 안내데스크 같은 거임.

```
[사용자] → https://api.../users 요청
   ↓
[API Gateway] (정문에서 받음)
   ↓
[Lambda 함수] 실행 → 결과
   ↓
[API Gateway] → 사용자한테 응답 전달
```


## 3. 경로별로 다른 함수에 연결

API Gateway에서 "이 주소로 오면 이 함수로 보내라"를 경로별로 정해둠.

```
GET  /users      → getUsers 함수
POST /users      → createUser 함수
GET  /orders     → getOrders 함수
```

이렇게 해두면 사용자는 그냥 주소만 부르면 되고, 뒤에서 어떤 함수가 도는지는 몰라도 됨.


## 4. 입구라서 할 수 있는 일들

요청이 다 이 정문을 거치니까, 여기서 공통 처리를 함.

- **인증 확인**: 토큰 없는 요청은 입구에서 막음.
- **요청 수 제한**: 한 사용자가 너무 많이 부르면 막음(rate limit).
- **HTTPS 처리**: 바깥과는 안전한 HTTPS로 통신.
- **로그**: 누가 뭘 요청했는지 기록.

부서(함수)마다 따로 할 일을 정문에서 한 번에 처리해주는 셈임.


## 5. Lambda와 묶이는 그림

앞 글의 Lambda랑 합치면 흔한 서버리스 구조가 됨.

```
[사용자] → [API Gateway] → [Lambda] → (필요하면 DB)
            정문 역할         실제 일 처리
```

서버를 한 대도 직접 안 켜고도, 주소로 부를 수 있는 API가 만들어짐. 둘 다 쓴 만큼만 과금되고 자동 확장됨.


## 6. 정리

- Lambda 함수엔 부를 주소가 없음 → 입구가 필요함.
- API Gateway가 요청을 받아 알맞은 함수로 넘겨주는 정문 역할.
- 경로별로 함수를 연결하고, 인증·요청제한·HTTPS·로그 같은 공통 처리를 함.
- API Gateway + Lambda로 서버 관리 없는 API를 만들 수 있음.
