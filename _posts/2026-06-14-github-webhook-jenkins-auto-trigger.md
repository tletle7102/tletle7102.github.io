---
title: "GitHub Webhook과 Jenkins 자동 트리거 (git push 한 번에 배포되는 흐름)"
categories:
  - CI/CD
tags:
  - Jenkins
  - GitHub
  - Webhook
last_modified_at:
---

# GitHub Webhook과 Jenkins 자동 트리거

"git push만 하면 알아서 배포된다"는 게 신기했음. 누가 push한 걸 어떻게 알고 배포가 시작되는지, 그 연결고리인 Webhook을 정리함


## 1. 문제: Jenkins는 push한 걸 모른다

Jenkins는 빌드·배포를 돌려주는 도구임. 근데 Jenkins는 내가 GitHub에 push했는지 스스로 알 방법이 없음.

방법은 두 가지임.

- Jenkins가 "혹시 새 코드 있나?" 하고 GitHub를 계속 물어보기 (= 폴링, 비효율적)
- GitHub가 push 생기면 "야 코드 올라왔어!" 하고 Jenkins한테 알려주기 (= Webhook, 효율적)

두 번째가 Webhook임.


## 2. Webhook이 뭔가

Webhook은 **"무슨 일이 생기면 정해둔 주소로 알림을 쏴주는 기능"** 임.

GitHub에 Webhook을 설정해두면, push가 일어날 때마다 GitHub가 Jenkins 주소로 신호를 보냄.

```
[개발자] git push
   ↓
[GitHub] "어, push 생겼다!" → 미리 등록된 Jenkins 주소로 알림 발사
   ↓
[Jenkins] 알림 받음 → "배포 시작!"
```

초인종 같은 거임. 누가 오면(push) 초인종을 눌러서(webhook) 집주인(Jenkins)한테 알리는 거임. 집주인이 계속 문을 내다볼 필요가 없음.


## 3. 전체 자동화 흐름

```
1. 개발자가 git push
2. GitHub가 Webhook으로 Jenkins에 알림
3. Jenkins가 자동으로 파이프라인 시작
   - 코드 받아오기
   - 빌드
   - 테스트
   - 도커 이미지 만들고 배포
4. 새 버전이 서버에 뜸
```

개발자는 `git push` 한 번만 했을 뿐인데, 그 뒤로는 전부 자동으로 굴러감. 이게 CI/CD가 실제로 돌아가는 모습임.


## 4. 정리

- Jenkins는 스스로 push를 모름.
- GitHub Webhook이 push가 생길 때마다 Jenkins한테 알림을 쏴줌.
- 그 알림을 신호로 Jenkins가 빌드·배포 파이프라인을 자동으로 시작함.

> Webhook = "이벤트 생기면 정해둔 주소로 알려주는 초인종". 이게 git push 한 번으로 배포까지 가는 자동화의 시작점임.
