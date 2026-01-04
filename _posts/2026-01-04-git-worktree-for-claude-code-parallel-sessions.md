---
title: "Git 워크트리로 클로드코드 병렬 세션 돌리기"
categories:
  - Git
tags:
  - Git
  - Worktree
  - Claude Code
  - Parallel
last_modified_at:
---

### Git 워크트리로 클로드코드 병렬 세션 돌리기

Git 워크트리를 활용하면 하나의 스프링부트 프로젝트에서 여러 기능을 서로 다른 디렉토리로 분리해 병렬 개발이 가능함
여기에 클로드코드 세션을 각 워크트리마다 따로 실행하면, 하나의 맥북에서 여러 개의 AI 코딩 비서를 동시에 돌리는 구조를 만들 수 있음

---

#### 📌 용어 설명

- Git 워크트리(Worktree): 하나의 Git 저장소에서 여러 워킹 디렉토리를 동시에 사용하는 기능
- 클로드코드(Claude Code): Anthropic의 AI 기반 코딩 도구, 터미널에서 실행하는 대화형 개발 비서
- 병렬 세션(Parallel Session): 여러 개의 클로드코드를 동시에 실행하여 독립적인 작업을 수행하는 방식
- 워킹 디렉토리(Working Directory): 실제로 파일을 수정하고 작업하는 폴더 공간

#### 📌 Git 워크트리 간단 복습

핵심 개념만 다시 짚기
- 하나의 Git 저장소에서 여러 워킹 디렉토리를 동시에 사용하는 기능이 Git 워크트리임
- 각 워킹 디렉토리는 서로 다른 브랜치를 체크아웃 할 수 있음
- 디렉토리 간에는 코드가 분리되어 있어서, 한쪽에서 실험적인 변경을 해도 다른 쪽에 바로 영향을 주지 않음

기본 명령어 요약

목록 보기
```bash
git worktree list
```

워크트리 추가
```bash
git worktree add -b feature/login ../myapp-login
```

워크트리 제거
```bash
git worktree remove ../myapp-login
```

이 정도만 알아도 클로드코드와 조합해서 쓰기에는 충분함

---

#### 📌 워크트리와 클로드코드 조합 아이디어

왜 이 조합이 좋은지
- 클로드코드는 한 세션에서 여러 파일을 한꺼번에 바꾸기 때문에, 작업 범위를 잘못 주면 코드가 뒤섞이기 쉬움
- 워크트리를 사용해서 기능별로 디렉토리를 나누면, 각 디렉토리에서 돌리는 클로드코드 세션이 서로 간섭하지 않음
- 로그인, 결제, 테스트 같은 작업을 동시에 맡겨놓고 결과만 확인하는 병렬 개발이 가능해짐

기본 아이디어 구조

메인 디렉토리
- ~/projects/todo-app

기능별 워크트리 예시
- ~/projects/todo-app-auth → 로그인, JWT 관련 작업 전용
- ~/projects/todo-app-payment → 결제 기능 전용
- ~/projects/todo-app-tests → 통합 테스트, 리팩터링 전용

각 디렉토리마다 클로드코드를 따로 실행해서, 세션별 역할을 분리함

---

#### 📌 스프링부트 프로젝트 예시 시나리오

예시 프로젝트
- 이름은 todo-app 라고 가정함
- 스택은 JDK 21, Spring Boot, Gradle, application.yml 구조임
- Git 저장소 루트 경로는 ~/projects/todo-app 임

동시에 진행하고 싶은 작업 세 가지

A 작업: 로그인 기능 + JWT 인증 추가
B 작업: 결제 API 기본 골격 만들기
C 작업: 기존 기능 통합 테스트 작성

각 작업을 워크트리 + 클로드코드 세션 하나씩에 매핑하는 구조를 만들 것임

---

#### 📌 단계별 실습 흐름

1단계 워크트리 만들기

메인 디렉토리에서 워크트리를 추가함

```bash
cd ~/projects/todo-app

# 로그인 기능용 워크트리
git worktree add -b feature/auth ../todo-app-auth

# 결제 기능용 워크트리
git worktree add -b feature/payment ../todo-app-payment

# 테스트용 워크트리
git worktree add -b feature/tests ../todo-app-tests
```

이제 디렉토리 구조는 대략 이렇게 됨
- ~/projects/todo-app → main
- ~/projects/todo-app-auth → feature/auth
- ~/projects/todo-app-payment → feature/payment
- ~/projects/todo-app-tests → feature/tests

2단계 각 워크트리에서 클로드코드 세션 열기

맥에서 멀티모니터를 쓴다는 가정으로 UI를 나누면 편함

모니터 1
- Cursor IDE에서 ~/projects/todo-app-auth 열기
- Cursor 내 터미널에서
```bash
cd ~/projects/todo-app-auth
claude
```

모니터 2
- iTerm2 탭1
```bash
cd ~/projects/todo-app-payment
claude
```

- iTerm2 탭2
```bash
cd ~/projects/todo-app-tests
claude
```

결과적으로 세 개의 클로드코드 세션이 동시에 떠 있는 상태가 됨
각 세션은 서로 다른 디렉토리, 서로 다른 브랜치에서 동작함

3단계 세션마다 역할을 못 박아두기

각 세션의 첫 프롬프트에 작업 범위를 명확히 적어두는 것이 중요함

auth 세션 (feature/auth, ~/projects/todo-app-auth)
- 프롬프트 예시
  - 이 세션은 로그인, 회원가입, JWT, 인증 관련 코드만 작업함
  - 보안 필터, AuthController, User 엔티티, 토큰 관련 설정까지만 수정함
  - 결제, 주문, 기타 도메인 코드는 건드리지 말 것

payment 세션 (feature/payment, ~/projects/todo-app-payment)
- 프롬프트 예시
  - 이 세션은 결제 API와 관련된 컨트롤러, 서비스, 리포지토리만 작업함
  - 테스트 코드도 결제 관련만 작성함
  - 인증, 회원가입, 공통 설정 부분은 수정하지 말 것

tests 세션 (feature/tests, ~/projects/todo-app-tests)
- 프롬프트 예시
  - 이 세션은 통합 테스트와 단위 테스트 코드를 작성하는 용도로만 사용함
  - 프로덕션 코드 수정은 가능한 최소로 유지함
  - 기존 기능에 대한 테스트 커버리지 확장 위주로 작업함

💡 이렇게 역할을 세션 시작 단계에서 못 박아두면, AI가 각 세션에서 건드릴 범위를 잘 지키는 데 도움이 됨

4단계 실제 병렬 작업 흐름

실제 작업 예시
- 로그인 세션에서 JWT 기반 로그인 구현을 맡김
- 동시에 결제 세션에서 결제 API 뼈대와 DTO, 컨트롤러를 생성하도록 요청함
- 테스트 세션에서는 기존 Todo API에 대한 통합 테스트와 간단한 리팩터링 테스트를 생성하도록 요청함

이 세 가지가 동시에 돌아가기 때문에
- 한 세션이 생각 중일 때 다른 세션에서 결과를 확인하거나, 다음 작업을 시킬 수 있음
- 브랜치를 바꾸거나 스태시 하지 않아도, 그때그때 디렉토리와 터미널만 바꿔가며 작업할 수 있음

---

#### 📌 병렬 작업 운영 팁

폴더 이름 규칙 정하기
- todo-app-auth, todo-app-payment, todo-app-tests 처럼 한눈에 역할이 보이게 이름을 짓는 것이 좋음

세션 라벨링
- 각 터미널 탭 이름, iTerm2 프로파일 이름, 커서 프로젝트 이름에 역할을 적어두면 헷갈리지 않음

작은 단위로 커밋
- 워크트리마다 자주 커밋을 해두면, 나중에 main 브랜치로 머지할 때 충돌이 줄어듦

---

#### 📌 머지와 정리 흐름

각 워크트리에서 작업이 끝났다고 가정

1단계 각 워크트리에서 커밋

```bash
cd ~/projects/todo-app-auth
git add .
git commit -m "feat: add jwt auth"

cd ~/projects/todo-app-payment
git add .
git commit -m "feat: add payment api"

cd ~/projects/todo-app-tests
git add .
git commit -m "test: add integration tests"
```

2단계 메인 저장소에서 머지

```bash
cd ~/projects/todo-app
git checkout main
git merge feature/auth
git merge feature/payment
git merge feature/tests
```

3단계 워크트리 제거

```bash
git worktree list
git worktree remove ../todo-app-auth
git worktree remove ../todo-app-payment
git worktree remove ../todo-app-tests
```

---

#### 📌 주의사항

- 같은 기능을 서로 다른 워크트리에서 동시에 수정하면 머지 충돌이 크게 날 수 있음
- 워크트리 제거 전에 각 브랜치의 변경사항이 커밋되었는지 꼭 확인해야 함
- 클로드코드 세션이 어떤 디렉토리에서 실행 중인지 항상 인지하고 있어야 함

---

#### 🔄 개념 비교

| 항목 | 설명 |
| --- | --- |
| 단일 세션 작업 | 브랜치 전환과 스태시 반복, 작업 맥락 손실 |
| 워크트리 + 병렬 세션 | 디렉토리별 독립 작업, 동시 진행 가능 |
| 세션 역할 분리 | 각 세션에 명확한 작업 범위 지정으로 코드 충돌 방지 |
| 머지 전략 | 작은 단위 커밋으로 충돌 최소화 |

---

#### 💡 요약

Git 워크트리는 원래도 유용한 기능이지만, 클로드코드와 조합하면 병렬 개발 도구로서 진가가 드러남
여러 기능을 동시에 개발하고 싶을 때, 워크트리로 디렉토리를 나누고 각 디렉토리에서 클로드코드 세션을 돌리는 패턴을 한 번 적용해보면 좋음
처음에는 두 개 정도의 워크트리와 세션으로 시작해서, 익숙해지면 점차 늘려가는 방식이 부담이 덜함
